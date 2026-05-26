---
name: youtube-to-skill
description: Convert any YouTube video into a reusable Claude Code skill. Paste a YouTube URL and this skill fetches the transcript, extracts the actionable content, and writes a ready-to-use .claude/skills/<name>.md file. Best for tutorial videos, how-to guides, workflow demos, or instructional content.
argument-hint: <YouTube URL>
---

# YouTube to Skill

You are converting a YouTube video into a reusable Claude Code skill file. Work through each step in order.

**YouTube URL:** {{args}}


## Step 1: Extract Video ID

Parse the video ID from the URL. Handle all common YouTube URL formats:
- `https://www.youtube.com/watch?v=VIDEO_ID`
- `https://youtu.be/VIDEO_ID`
- `https://www.youtube.com/shorts/VIDEO_ID`
- `https://youtube.com/live/VIDEO_ID`
- `https://m.youtube.com/watch?v=VIDEO_ID`

Extract `VIDEO_ID` and confirm it looks valid (11 alphanumeric characters).


## Step 2: Fetch Video Metadata and Transcript

Use your web fetch tool to retrieve the pages below (replace `VIDEO_ID` with the ID from Step 1). Gather both metadata and transcript content:

**Source A: YouTube page (metadata: title + description + channel):**
Fetch `https://www.youtube.com/watch?v=VIDEO_ID`

Extract from the HTML:
- Video title (`<title>` tag or `"title":"..."` in the JSON data)
- Description (from `"shortDescription":"..."` or the `<meta name="description">` tag)
- Channel name
- Duration

**Source B: Transcript (primary content source):**
Fetch `https://youtubetranscript.com/?server_vid2=VIDEO_ID`

If Source B returns a transcript, use it as the primary content source. If it fails or returns no content, try Source C.

**Source C: Fallback YouTube transcript API:**
If Source B fails, try fetching:
`https://www.youtube.com/api/timedtext?v=VIDEO_ID&lang=en&fmt=json3`

Parse the `events[].segs[].utf8` fields to reconstruct the full transcript text.

If both Source B and Source C fail, rely on Source A's title and description only, and note that the transcript was unavailable.


## Step 3: Understand the Content

Read through everything collected. Determine:

1. **What is this video teaching or demonstrating?**
2. **Who is the intended audience?** (beginner, intermediate, expert; developer, designer, etc.)
3. **What is the main workflow or technique?**
4. **What are the key steps, commands, or instructions the viewer should follow?**
5. **What tools, technologies, or concepts does it reference?**
6. **What outcomes does following the video produce?**

If the transcript is available, extract all concrete steps, commands, code snippets, and decision points verbatim. Do not paraphrase away specifics.


## Step 4: Design the Skill

Before writing, decide:

**Skill name:** Short, kebab-case, describes what the skill *does* (not the video title). Example: `deploy-to-fly`, `setup-drizzle-orm`, `build-realtime-chat`.

**Skill type:** Which pattern fits best?
- **Tutorial**: Linear steps A→B→C. User follows the video's workflow.
- **Reference**: Commands, config, or flags from the video organized for quick lookup.
- **Workflow**: Multi-phase process with decision points and branching.
- **Setup**: One-time environment or tool configuration.

**Does it need `{{args}}`?** Only if the skill is parameterized (e.g., project name, target URL). If the skill is self-contained, omit `{{args}}`.


## Step 5: Write the Skill File

Write the skill to `.claude/skills/<name>.md` using this structure. The frontmatter at the top must be enclosed in `---` delimiters (omit the `argument-hint` line entirely if the skill takes no arguments):

```markdown
---
name: <kebab-case-name>
description: <One sentence: what it does and when to use it. Include key tech names so the trigger is precise.>
argument-hint: <optional: what args the user should pass>
---

# <name>: <Human-Readable Title>

<One paragraph: what this skill does, what it produces, and why it exists. Attribute the source: "Based on [Video Title] by [Channel Name].">

**Source:** [Video Title]({{args}}) by [Channel Name]


<Skill body: structured content extracted from the video>
```

### Body content rules:

- **Preserve specificity.** Keep exact commands, flags, config values, and code snippets from the video. Don't generalize them away.
- **Steps should be executable.** Each step should be something Claude can actually do or instruct the user to do.
- **Use phases for complex workflows.** `## Phase 1: X`, `## Phase 2: Y`
- **Use code blocks for every command or code snippet.** Even single-line commands.
- **Include decision points.** If the video says "if you're using PostgreSQL, do X; if SQLite, do Y": keep the branch.
- **Strip filler.** Don't include the presenter's personal anecdotes, channel intros/outros, or ad reads.
- **If transcript was unavailable**, note it at the top: `> Note: Transcript was unavailable. This skill is based on the video title and description only. Verify steps before running.`


## Step 6: Confirm and Save

After writing the file, report back:

1. The skill name and file path
2. A one-line description of what the skill does
3. How to invoke it (`/<name>` or `/<name> <args>`)
4. Any caveats: e.g., transcript unavailable, video was too abstract to extract concrete steps, commands couldn't be verified

If the video content was too abstract, conceptual-only, or completely inaccessible, tell the user and suggest they paste the transcript manually for better results.
