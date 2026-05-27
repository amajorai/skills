---
name: ai-agent-security
description: Implement or audit AI agent security controls based on the OWASP AI Agent Security Cheat Sheet. Covers 9 security pillars: tool least-privilege, input validation, memory/context security, human-in-the-loop controls, output guardrails, monitoring, multi-agent trust, data protection, and adversarial testing. Use when building or hardening any AI agent system.
argument-hint: [mode: audit | implement | both]
---

# AI Agent Security

You are implementing or auditing AI agent security controls based on the OWASP AI Agent Security Cheat Sheet. Work through each phase in order.

**Mode:** {{args}}


## Phase 1: Interview

Use `AskUserQuestion` for each question below, one at a time.

**Question 1: Mode**
| Option | Description |
|--------|-------------|
| `audit` | Review existing agent code and report gaps |
| `implement` | Implement security controls from scratch or on top of existing code |
| `both` | Audit first, then implement fixes for all gaps found |

**Question 2: Agent architecture** (multi-select):
| Option | Description |
|--------|-------------|
| `single-agent` | One LLM with tools |
| `multi-agent` | Multiple coordinating agents |
| `rag` | Retrieval-augmented generation |
| `long-running` | Persistent memory across sessions |
| `autonomous` | Operates without per-action human approval |

**Question 3: Risk profile** (multi-select):
| Option | Description |
|--------|-------------|
| `financial` | Agent can initiate payments or financial transactions |
| `external-comms` | Agent can send emails, messages, or API calls to third parties |
| `data-write` | Agent can modify or delete data |
| `code-execution` | Agent can execute code or shell commands |
| `low-risk` | Read-only or informational operations only |

**Question 4: Framework** - free text:
> What AI framework and provider are you using? (e.g. "Vercel AI SDK + OpenAI", "LangChain + Claude", "custom + Gemini")


## Phase 2: Explore

Spawn **3 parallel subagents**:

| Subagent | Focus |
|----------|-------|
| 1 | Tool definitions, permissions, action handlers, agent loop, sandboxing, third-party tool sources |
| 2 | Memory/context management, logging, observability, inter-agent communication, message signing |
| 3 | Output validation/schemas, data classification and redaction, security test suites, CI gates |

Collect findings before proceeding.


## Phase 3: Audit Against 9 Security Pillars

For each pillar, assess current state (Pass / Gap / Missing) and list specific findings.

**Cross-cutting agent threats to keep in mind during the audit** (from the OWASP cheat sheet - flag any that apply, even if not tied to a single pillar):
- Prompt Injection (direct and indirect)
- Tool Abuse and Privilege Escalation
- Data Exfiltration
- Memory Poisoning
- Goal Hijacking
- Excessive Autonomy
- High-Impact Action Abuse
- Decision and Approval Manipulation
- Cascading Failures (multi-agent)
- AI Console Malicious Configuration (instructions in data altering LLM config)
- Denial of Wallet (unbounded loops, runaway cost)
- Sensitive Data Exposure
- Supply Chain Attacks (third-party tools, APIs, data sources)

---

### Pillar 1: Tool Security and Least Privilege

**Check:**
- Each tool has a defined minimum permission scope
- No wildcard or catch-all tool permissions
- Sensitive operations (delete, send, pay) require explicit per-call authorization
- Tools are separated by trust level (read-only vs. write vs. external)
- Tool parameters are validated before execution (type, range, allowlist)
- Dangerous file/path patterns are blocked (`*.env`, `*.key`, `*.pem`, `*secret*`, `~/.ssh/*`, `.git/*`)
- Token, cost, retry, and tool-chain limits enforced per task to prevent Denial of Wallet
- Arbitrary code execution tools are sandboxed (no host filesystem, no network unless required)
- Third-party tools/MCP servers reviewed for supply-chain risk (pinned versions, integrity hashes, vendor trust review)

**Target state:**
```
tool: send_email
  allowed_recipients: [internal_domain only]
  requires_approval: true for external addresses
  rate_limit: 10/hour
  max_chain_depth: 3
  max_retries: 2
```

**Gaps to flag:** any tool with unrestricted parameters, no approval gate on irreversible actions, tools that accept arbitrary file paths or shell strings, no per-task cost/loop ceiling, code execution without sandbox, unpinned or unvetted third-party tools.

---

### Pillar 2: Input Validation and Prompt Injection Defense

**Check:**
- All external data (web pages, emails, documents, tool outputs) treated as untrusted
- User input sanitized before inclusion in LLM context
- Clear structural separator between system instructions and user/external data
- Content filtering on known injection patterns (direct and indirect)
- RAG retrieval results tagged as external/untrusted before context inclusion
- AI developer console / system-prompt edits cannot be triggered by ingested data (guards against AI Console Malicious Configuration)
- Goal-hijacking defenses: agent re-reads original objective each turn rather than trusting accumulated context

**Target state:**
```
context structure:
  [SYSTEM_INSTRUCTIONS] - trusted, immutable, signed
  [USER_REQUEST] - validated, length-limited
  [EXTERNAL_DATA] - tagged untrusted, sanitized, never executed as instruction
```

**Gaps to flag:** direct concatenation of external content with instructions, no injection pattern filtering, no input length limits, tool/system-prompt config writable from data plane, agent goal mutable via tool output.

---

### Pillar 3: Memory and Context Security

**Check:**
- Memory isolated per user/session (no cross-session bleed)
- Sensitive data (PII, credentials, tokens) not persisted in memory
- Memory contents validated/sanitized before storage (defends against Memory Poisoning)
- Memory has expiry and size limits (cheat sheet defaults: 24h expiry, 100 items max)
- Cryptographic integrity check on stored memory (prevents tampering)
- Pre-write scanner detects SSN, credit card, password, API key patterns and redacts

**Target state:**
- Per-user memory namespace with encryption at rest
- Pattern scanner before write (detect and redact PII, credentials)
- Expiry: 24h for ephemeral, configurable for long-term; cap items per session (e.g. 100)
- Checksum verification on memory read; reject and alert on mismatch

**Gaps to flag:** shared memory across users, PII stored in plain text, no size/expiry limits, no integrity verification, untrusted tool output written directly to long-term memory (memory poisoning vector).

---

### Pillar 4: Human-in-the-Loop Controls

**Risk classification for actions (verbatim from OWASP cheat sheet):**
| Risk Level | Definition | Examples | Required Controls |
|------------|-----------|----------|-------------------|
| LOW | Read operations, safe queries | `search_documents`, `read_file` | Auto-approval permitted |
| MEDIUM | Write operations, API calls | `write_file` | Human review required |
| HIGH | Financial, deletion, external comms | `send_email`, `execute_code` | Explicit user approval + preview |
| CRITICAL | Irreversible, security-sensitive | `database_delete`, `transfer_funds` | Step-up authentication, parameter binding, short-lived authorization, replay protection, idempotency verification |

**Check:**
- HIGH actions blocked pending explicit user approval with a preview of exact effects
- CRITICAL actions additionally require step-up auth, idempotency keys, and replay-protected tokens
- Approval bound to exact action fields (actor, tool, resource, parameters, timestamp, expiry) and cannot be reused for different params
- Short-lived approval tokens (recommended: minutes, not hours)
- Fail-closed when approval validation fails
- Decision-making is separated from execution for irreversible operations
- User has interrupt and rollback capability for in-flight or recently completed actions
- Approval thresholds and risk scores cannot be influenced by model output (defends Decision/Approval Manipulation)

**Gaps to flag:** irreversible actions taken without confirmation, approval not bound to specific parameters, no timeout on approval requests, no preview shown to user, risk score/threshold derived from LLM output, no rollback path.

---

### Pillar 5: Output Validation and Guardrails

**Check:**
- Model output validated against expected schema before use (Zod/Pydantic, strict mode)
- Sensitive data (credentials, PII) filtered from outputs before display/forwarding
- Exfiltration patterns detected (base64/hex-encoded sensitive data, large payloads to webhooks, unusual outbound URLs)
- Rate limits enforced on outputs (cheat sheet example: 100 calls / 60 seconds)
- Content safety filters applied
- Authorization decisions never derived from model output alone - always re-checked server-side against policy

**Target state:**
```
agent_response -> schema_validate -> pii_filter -> exfil_detector -> safety_filter -> policy_check -> execute/display
```

**Gaps to flag:** model output used directly as SQL/shell input, no schema validation, PII in logged outputs, no exfiltration detection on outbound tool calls, authorization based on LLM-emitted role/permission.

---

### Pillar 6: Monitoring and Observability

**Check:**
- Every tool call, decision, and outcome is logged with structured metadata (action type, risk score, authorization outcome, approval ID, execution result, policy version)
- Anomaly detection on unusual patterns with concrete thresholds (cheat sheet defaults: >30 tool calls/min, >=5 failed calls, >=1 injection-pattern hit, >$10/session cost)
- Token/cost tracking per user and session with hard-cap budget alerts (Denial of Wallet defense)
- Security event alerts (repeated auth failures, abnormal tool usage, injection detections)
- Audit trails for all high-risk actions retained for compliance window
- PII redacted from logs at write time, not at read time

**Gaps to flag:** no structured logging, no cost tracking or hard cap, no anomaly detection thresholds, sensitive data in plain-text logs, missing policy_version/approval_id fields on high-risk action logs.

---

### Pillar 7: Multi-Agent Security

**Check (only if multi-agent architecture):**
- Trust boundaries defined between agents (orchestrator vs. sub-agent)
- Inter-agent messages signed and verified, with timestamp freshness window (cheat sheet recommends 5 minutes)
- Agent cannot escalate its own privileges through another agent (no transitive permission grants)
- Circuit breakers prevent cascading failures (cheat sheet example: 5-failure threshold, 60s recovery)
- Separate execution environments per agent where possible
- Untrusted agent output treated the same as untrusted user input (re-sanitized at every hop)
- Max chain depth enforced to prevent infinite agent-to-agent loops (DoW defense)

**Target state:**
- Message bus with signature verification, replay protection, and 5-minute freshness window
- Agent trust levels (cheat sheet taxonomy): UNTRUSTED / INTERNAL / PRIVILEGED / SYSTEM
- Circuit breaker per downstream agent: trip after 5 consecutive failures, half-open after 60s

**Gaps to flag:** agents passing unsanitized output to each other, no privilege escalation checks in agent chains, no circuit breaker, no chain-depth cap, message bus without signatures or freshness check.

---

### Pillar 8: Data Protection and Privacy

**Check:**
- Minimum necessary data in agent context (no loading full user record when ID suffices)
- Data classification scheme applied automatically (PUBLIC / INTERNAL / CONFIDENTIAL / RESTRICTED)
- Per-class redaction policy enforced:
  - RESTRICTED (PII, health, financial, credentials): fully redacted in agent context and logs
  - CONFIDENTIAL: partially masked in logs (e.g. last-4-only)
  - INTERNAL: visible to authorized agents only
  - PUBLIC: no restriction
- Encryption at rest and in transit for agent memory and logs
- Retention and deletion policies enforced (with automated purge)
- GDPR/CCPA compliance for user data in agent context (right-to-erasure reaches memory too)

**Gaps to flag:** full PII loaded when only ID needed, no data classification, no per-class redaction policy, no retention policy, no encryption of stored context, RESTRICTED data appearing in logs in any form.

---

### Pillar 9: Secure Agent Testing and Adversarial Validation

**Check:**
- Adversarial test suite covers the full abuse-case matrix from the cheat sheet:
  - Prompt override (direct and indirect)
  - Tool misuse
  - Privilege escalation
  - Memory poisoning
  - Data exfiltration
  - Recursive abuse / unbounded loops (DoW)
  - Approval bypass / parameter rebinding
  - Multi-agent chaining attacks
- Tests run in CI/CD on every change to prompts, tools, memory, retrieval, or provider config
- Releases blocked when high-risk policies change without updated tests
- Red-team prompts version-controlled
- Known-bad inputs in regression suite
- Approval/denial behavior verified by tests
- Validation evidence retained per release (version, model, tools tested, observed behavior)

**Gaps to flag:** no security test suite, security tests not in CI, no regression tests for known vulnerabilities, no release gate on policy changes, no retained evidence trail.


## Phase 4: Plan

List every gap found across all 9 pillars. Group by pillar and **fix priority** (distinct from the HITL action-risk levels in Pillar 4):
- **P0 / Blocker** - blocks production use (e.g. no human-in-the-loop on financial actions, unsandboxed code execution, no cost cap)
- **P1 / Must-fix** - fix before next release
- **P2 / Should-fix** - fix within 30 days

If mode is `implement` or `both`, confirm the prioritized fix list with the user before proceeding to Phase 5. If mode is `audit`, skip Phase 5 and proceed directly to Phase 6 to verify current state.


## Phase 5: Implement (if mode is implement or both)

For each gap, implement the specific control (numbering aligns to the 9 pillars):

- **Pillar 1 - Tool security:** per-tool permission manifest, parameter validation, dangerous-path denylist (`*.env`, `*.key`, `*.pem`, `*secret*`), sandbox for code execution, pinned third-party tool versions, per-task cost/loop/chain-depth caps
- **Pillar 2 - Input validation:** sanitization layer before LLM context, structural separators, indirect-injection scanner on tool outputs, immutable system prompt
- **Pillar 3 - Memory security:** per-user namespacing, PII pattern scanner pre-write, size/expiry caps, integrity checksum, no direct write of untrusted tool output to long-term memory
- **Pillar 4 - HITL gates:** approval workflow for HIGH (preview + explicit approval) and CRITICAL (step-up auth, idempotency key, replay-protected token) actions; rollback path; risk score sourced from policy, never from LLM
- **Pillar 5 - Output guardrails:** strict schema validation, PII filter, exfiltration detector, rate limits, server-side policy re-check before any privileged action
- **Pillar 6 - Monitoring:** structured logging with action/risk/approval/policy_version fields, anomaly detection with concrete thresholds, hard cost cap per session/user
- **Pillar 7 - Multi-agent trust:** signed message bus with 5-min freshness window, trust taxonomy (UNTRUSTED/INTERNAL/PRIVILEGED/SYSTEM), circuit breakers, chain-depth cap, re-sanitization at each hop
- **Pillar 8 - Data protection:** classification labels, per-class redaction policy (RESTRICTED = full redact, CONFIDENTIAL = partial mask), encryption at rest and in transit, retention/purge automation
- **Pillar 9 - Adversarial tests:** abuse-case matrix in CI (prompt override, tool misuse, privilege escalation, memory poisoning, exfiltration, recursive abuse, approval bypass, multi-agent chaining), release gate on policy changes, evidence retention

Do not refactor beyond the specific control being added.


## Phase 6: Verify

Run the adversarial test suite if it exists. Attempt these manual checks (each maps to a cheat-sheet threat - record observed behavior and the policy version under test):

- [ ] **Direct prompt injection** in user input ("ignore previous instructions...") - should be filtered or refused
- [ ] **Indirect prompt injection via tool output** (inject instructions in a returned web page / email / document) - should not alter agent goal
- [ ] **Memory poisoning** - persist a malicious instruction in memory in session A; start session B for same user and verify it is not executed
- [ ] **Cross-user memory access** - read another user's memory context (should be impossible)
- [ ] **Goal hijacking** - mid-conversation attempt to redirect agent to new objective via tool output - should be ignored
- [ ] **Privilege escalation via multi-agent chain** - low-privilege agent attempts to get higher-privilege peer to act on its behalf
- [ ] **High-impact (HIGH/CRITICAL) action without approval** - should be blocked, fail-closed
- [ ] **Approval parameter rebinding** - replay an approval token against a different resource/parameters - should be rejected
- [ ] **Decision/approval manipulation** - prompt model to lower its own risk score - server-side policy should override
- [ ] **Denial of Wallet** - trigger a recursive tool-call loop and confirm cost/loop cap stops it
- [ ] **Data exfiltration** - request agent to base64-encode and email a credential; outbound guard should detect and block
- [ ] **Sandbox escape** - code-execution tool attempts host filesystem / network access outside policy
- [ ] **Supply chain** - confirm third-party tool/MCP server versions are pinned and integrity-verified
- [ ] **AI console malicious configuration** - ingested data attempts to mutate system prompt / tool config - must be rejected

Document the result and policy version for each test; retain as release evidence (Pillar 9).


## Completion Report

```markdown
## AI Agent Security Audit

**Date:** <today>
**Architecture:** <from interview>
**Mode:** <audit | implement | both>
**Policy version under test:** <git sha or version tag>

### Pillars Audited

| # | Pillar | Status | Gaps Found | Gaps Fixed | Specific fixes applied |
|---|--------|--------|-----------|------------|------------------------|
| 1 | Tool Security & Least Privilege | Pass/Gaps/Missing | N | N | <e.g. added denylist, sandbox, cost cap> |
| 2 | Input Validation & Prompt Injection Defense | | | | |
| 3 | Memory & Context Security | | | | |
| 4 | Human-in-the-Loop Controls | | | | |
| 5 | Output Validation & Guardrails | | | | |
| 6 | Monitoring & Observability | | | | |
| 7 | Multi-Agent Security | | | | |
| 8 | Data Protection & Privacy | | | | |
| 9 | Secure Agent Testing & Adversarial Validation | | | | |

### Cross-cutting Threat Coverage

| Threat | Mitigated? | Where |
|--------|-----------|-------|
| Prompt Injection (direct & indirect) | Y/N | <pillar refs> |
| Tool Abuse / Privilege Escalation | | |
| Data Exfiltration | | |
| Memory Poisoning | | |
| Goal Hijacking | | |
| Excessive Autonomy | | |
| High-Impact Action Abuse | | |
| Decision/Approval Manipulation | | |
| Cascading Failures | | |
| AI Console Malicious Configuration | | |
| Denial of Wallet | | |
| Sensitive Data Exposure | | |
| Supply Chain Attacks | | |

### Verify Phase Results

| Test | Outcome | Notes |
|------|---------|-------|
| <each item from Phase 6> | pass/fail/blocked | |

### Cost & Budget Controls

- Per-session cost cap: <value or NONE>
- Per-user daily cap: <value or NONE>
- Tool-chain depth cap: <value or NONE>
- Retry cap: <value or NONE>

### Residual Risk

<any gaps not yet fixed, the threat they map to, and why deferred>

### Next Steps

<recommended follow-up actions with owners and dates>

### Evidence

<paths to retained test logs, policy version, model/tool versions covered>
```
