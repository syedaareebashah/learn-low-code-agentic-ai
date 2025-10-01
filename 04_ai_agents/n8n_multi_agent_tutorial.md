# n8n Multi‑Agent Tutorial: Multi‑Agent Newsletter Automation (Beginner‑Friendly)

[The Tutorial is based on this Video: Watch Me Build a Multi-Agent Newsletter System in n8n (step-by-step)](https://www.youtube.com/watch?v=pxzo2lXhWJE)

> Build a 3‑agent workflow in n8n that researches, drafts, and formats a newsletter, then creates a Gmail draft for human review. We’ll follow the video’s flow **and** fill in the missing pieces (prompts, schemas, expressions, error‑handling, hosting, and security). Yes, like adding fries to a perfectly good burger 🍟.

---

## What You’ll Build (at a glance)
A weekly workflow that:
1) **Researches** recent news for a niche → 2) **Plans** the edition title + topics (Agent 1) → 3) **Deep‑researches** each topic → 4) **Writes** three sections (Agent 2) → 5) **Aggregates** sections → 6) **Edits & styles** into responsive HTML (Agent 3) → 7) **Creates a Gmail draft**. It’s linear, like a conga line—only with fewer awkward hips. 💃

```
Schedule Trigger
  → Tavily Search (initial research)
    → Agent 1: Planning (title + 3 topics, structured output)
      → Split Out (topics → 3 items)
        → Tavily Search (per topic, include raw content)
          → Agent 2: Section Writer (writes 3 sections)
            → Aggregate (3 → 1)
              → Agent 3: Editor (HTML + subject, structured output)
                → Gmail: Create Draft (HTML)
```

---

## Prerequisites (the stuff the video zooms past)
- **n8n**: Cloud or self‑host.
  - **Enable Community Nodes**: *Settings → Community Nodes → Allow installation from community*. No community, no party 🎉.
- **Credentials** you’ll need:
  - **Tavily** API key (free tier available). Used for web search. Like a librarian who doesn’t shush you.
  - **OpenRouter** API key (or your preferred model provider). Used by all AI agents.
  - **Gmail** (OAuth) to create a **Draft**. It’s like sending an email… but with commitment issues.
- **Optional**: A test inbox, e.g., a secondary Gmail account.

**Self‑hosting tip (Docker)**
```bash
docker run -it --rm \
  -p 5678:5678 \
  -e N8N_ENCRYPTION_KEY="change_me" \
  -e N8N_SECURE_COOKIE=true \
  -e N8N_PORT=5678 \
  -e WEBHOOK_URL="https://your-domain" \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n:latest
```
Set credentials inside n8n. Avoid hard‑coding keys—secrets hate attention. 🤫

---

## Step‑by‑Step Build

### 1) Schedule Trigger (weekly)
- **Node**: *Schedule Trigger*
- **Every**: 1 week; **On**: Sunday; **Time**: 00:00 (adjust to your timezone)
- Leave **Active** off while building. Like removing the car keys before opening the hood. 🧰

### 2) Initial Research (Tavily)
- **Node**: *Tavily → Search*
- **Query**: your niche (e.g., `AI adoption for small businesses`)
- **Options**:
  - *topic*: `news`
  - *time_range*: `week`
  - *max_results*: `3`
  - *include_raw_content*: `false` (we’ll enable raw later)
- **Rename**: `Initial Research`
- **Tip**: *Pin Data* after a good run to speed up testing. Your CPU will send a thank‑you note. 💌

### 3) Agent 1 — Planning Agent (Title + Topics)
- **Node**: *AI Agent*
- **Chat Model**: *OpenRouter → gpt‑5‑mini* (or your favorite; lower cost is fine here)
- **User Message**: Provide the initial research articles in a clean, readable list. Use a small **Code** node before the agent to format the Tavily array:

**Node**: *Code* (JavaScript) → `Format Initial Research`
```js
// Input: 1 item with Tavily results under items[0].json.results (fallbacks included)
const res = (items[0].json.results || items[0].json.data || items.map(i => i.json)).slice(0, 10);
function line(r, i){
  const published = r.publishedDate || r.published_time || r.date || "";
  const snippet = r.content || r.snippet || r.summary || "";
  return `[${i+1}] ${r.title || "Untitled"}\nURL: ${r.url}\nPublished: ${published}\nSnippet: ${snippet}`;
}
const formatted = res.map(line).join("\n\n");
return [{ json: { formatted, count: res.length } }];
```
- **Agent → User Message** (Expression):
```
Here are recent articles to consider:\n\n{{$node["Format Initial Research"].json["formatted"]}}
```
- **Agent → System Message** (paste):
```
You are an expert newsletter planner. You receive 3–5 short article digests (title, URL, published date, snippet) from the past week.
Task: propose a catchy edition title (≤ 80 chars) and exactly 3 concise topics (each 3–5 words) that reflect distinct angles for our audience of small‑business leaders.
Constraints: unique topics; no duplicates; no clickbait; be informative.
Output only via the required schema (no extra text).
```
- **Require specific output format**: *ON* → **Structured Output Parser** → *Define using JSON Schema*:
```json
{
  "type": "object",
  "additionalProperties": false,
  "properties": {
    "title": { "type": "string", "minLength": 10 },
    "topics": {
      "type": "array",
      "minItems": 3,
      "maxItems": 3,
      "items": { "type": "string", "minLength": 3, "maxLength": 50 }
    }
  },
  "required": ["title", "topics"]
}
```
- **Rename**: `Planning Agent`
- *Pin Data* after you like the output. Like bookmarking a great pizza place. 🍕

### 4) Split Topics → Items
- **Node**: *Item Lists → Split Out Items*
- **Source**: `{{$node["Planning Agent"].json["topics"]}}`
- This yields 3 items (one per topic). Like turning a pie into slices—equally delicious. 🥧

### 5) Deeper Research per Topic (Tavily)
- **Node**: *Tavily → Search*
- **Query**: `{{$json}}` (the topic string)
- **Options**:
  - *time_range*: `month`
  - *max_results*: `5`
  - *include_raw_content*: `true` (important for drafting)
- **Rename**: `Topic Research`
- *Pin Data* after a healthy run. It’s your workflow’s multivitamin. 💊

### 6) Agent 2 — Section Writer (3 passes)
- **Node**: *AI Agent*
- **Chat Model**: same as Agent 1 (cost‑efficient)
- **Pre‑format research**: Add a *Code* node `Format Topic Research` before the agent:
```js
// Input: 1 item per topic with Tavily results at items[0].json.results
const res = items[0].json.results || [];
function line(r, i){
  const raw = r.raw_content || r.content || r.snippet || "";
  return `• ${r.title || "Untitled"} (URL: ${r.url})\n${raw.slice(0, 1200)}...`;
}
const formatted = res.slice(0, 6).map(line).join("\n\n");
return [{ json: { topic: items[0].json.query || items[0].json.topic || $json, formatted } }];
```
- **Agent → User Message** (Expression):
```
Topic: {{$node["Format Topic Research"].json["topic"]}}

Use these sources to write one standalone section:

{{$node["Format Topic Research"].json["formatted"]}}
```
- **Agent → System Message**:
```
You are a professional newsletter section writer. Write ONE self-contained section (350–500 words) for small-business leaders.
Requirements:
- Start with an H2 heading matching the Topic (e.g., “## <topic>”).
- Synthesize facts from the provided sources (don’t invent).
- Inline-cite claims with superscript markers like [1], [2].
- After the prose, add a short “Sources” list: [n] Title — Domain (linked).
- Tone: clear, expert, engaging. No overall intro or conclusion; just this section.
- Output plain Markdown (the editor will convert to HTML).
```
- **Rename**: `Section Writer`
- Run on each topic (it will execute 3 times). Like a hat trick, but for keyboards. 🎩

### 7) Aggregate Sections (3 → 1)
- **Node**: *Aggregate*
- **Combine**: field `output` (or the agent’s text field) into an array
- **Result**: one item with `sections: [sec1, sec2, sec3]`
- **Rename**: `Sections`
- Your sections are now roommates. Hopefully tidy ones. 🧹

### 8) Agent 3 — Editor (HTML + Subject via structured output)
- **Node**: *AI Agent*
- **Chat Model**: *OpenRouter → gpt‑5* (use a stronger model for style/HTML)
- **User Message** (Expression; join sections with blank lines):
```
Title candidate: {{$node["Planning Agent"].json["title"]}}

Sections:
{{$node["Sections"].json["output"].join("\n\n\n")}}
```
- **System Message** (paste):
```
You are the newsletter editor and layout stylist.
Input: one title candidate and 3 Markdown sections with [n] footnotes.
Produce:
1) subject — an email subject (≤ 80 chars, no emojis),
2) content — VALID, responsive HTML for the newsletter.
HTML Requirements:
- Include: header with title + date, short intro paragraph, the 3 sections (convert Markdown to HTML), a “Key Sources” section that deduplicates and numbers links, and a short friendly sign-off.
- Typography: use <h1>/<h2>, <p>, <ul>/<ol>, <a>. Inline CSS for readability (max-width container, readable font size/line-height).
- Links: ensure anchors use rel="noopener noreferrer" and target="_blank".
- Accessibility: semantic headings; alt text if any images appear (don’t add images unless supplied).
- No external CSS/JS. No tracking pixels.
- Output ONLY via the required schema.
```
- **Require specific output format**: *ON* → **Structured Output Parser** → JSON Schema:
```json
{
  "type": "object",
  "additionalProperties": false,
  "properties": {
    "subject": { "type": "string", "maxLength": 80 },
    "content": { "type": "string", "description": "Complete HTML body" }
  },
  "required": ["subject", "content"]
}
```
- **Rename**: `Editor`
- If output seems slow, pin it while developing. Patience is a feature, not a bug. 🐢

### 9) Gmail — Create Draft
- **Node**: *Gmail → Create Draft*
- **To**: your testing inbox
- **Subject**: `{{$node["Editor"].json["subject"]}}`
- **Message**: `{{$node["Editor"].json["content"]}}`
- **Email Type**: `HTML`
- **Rename**: `Create Draft`
Draft will appear in Gmail, ready for human eyes (and hopefully applause). 👀

---

## Production Hardening (filling the gaps)

### A) Error Handling & Fallbacks (because the internet sometimes naps)
- **IF (Function)**: After each Tavily call, check `results.length`.
  - If `0`, branch to a fallback provider (e.g., SERP API, Perplexity, Bing Web Search) via another node; merge back.
- **Error Trigger**: Add an *Error Trigger* workflow that logs to Google Sheets and pings Slack/Email.
- **Wait & Retry**: Wrap flaky nodes with *Wait* → *Move On* → *IF retried < N* → *Run again*. Like Groundhog Day, but with APIs. ⏳

### B) Cost Control & Guardrails
- Use a cheaper model for Agents 1 & 2; keep the stronger model only for the Editor.
- Lower temperature (e.g., 0.3–0.5) for consistency; set max tokens.
- Limit Tavily results to 3–5; keep raw content only for the topic research step.
- Schedule during off‑hours to avoid rate caps; staggering is your friend.

### C) Logging & Human‑in‑the‑Loop (HITL)
- **Google Sheets** (or DB): Log `run_id`, `title`, `topics[]`, section headings, token counts, and status.
- **Approval**: Keep “Create Draft” only (no auto‑send). For approval, add a simple **Webhook** that, when called, flips a Google Sheets cell from `PENDING` → `APPROVED` and triggers a separate “Send Mail” workflow. Humans keep the crown 👑.

### D) Prompt & Schema Management
- Store long prompts in a single *Code* or *Set* node and reference via expressions.
- Version prompts (v1, v2) with a sheet or env var (`PROMPT_VERSION`).
- Unit test prompts by feeding fixed inputs (use pinned data). Test like you mean it. 🧪

### E) Security & Secrets
- Use **Credentials** for API keys. Do NOT paste keys into nodes.
- Rotate keys; least‑privilege OAuth scopes for Gmail.
- If self‑hosting, set `N8N_ENCRYPTION_KEY` and use HTTPS. Secret agents love encryption. 🕵️‍♀️

---

## Troubleshooting Quickies
- **Agent output in one blob?** Ensure Structured Output Parser is ON where you need separate fields.
- **Weird `[object Object]` in messages?** Pre‑format arrays using a Code node and pass a string.
- **HTML shows `<p>` tags as text?** Set Gmail node **Email Type** to **HTML**.
- **Timeouts?** Reduce Tavily results, shorten raw excerpts, choose faster models, or pin while iterating.

---

## Optional Extensions
- Swap Tavily with Perplexity/SerpAPI/Bing and **merge** results for diversity.
- Add a *Policy/Lint Agent* that checks for claims without sources before the Editor. It’s your newsroom’s grumpy copy desk. 📝
- Post the HTML to your CMS or Mailchimp; attach a plaintext version too.

---

## Copy‑Paste Assets (ready to use)

### JSON Schema — Planning Agent
```json
{
  "type": "object",
  "additionalProperties": false,
  "properties": {
    "title": { "type": "string", "minLength": 10 },
    "topics": {
      "type": "array",
      "minItems": 3,
      "maxItems": 3,
      "items": { "type": "string", "minLength": 3, "maxLength": 50 }
    }
  },
  "required": ["title", "topics"]
}
```

### JSON Schema — Editor Agent
```json
{
  "type": "object",
  "additionalProperties": false,
  "properties": {
    "subject": { "type": "string", "maxLength": 80 },
    "content": { "type": "string" }
  },
  "required": ["subject", "content"]
}
```

### System Prompts (Planning / Section / Editor)
**Planning**
```
You are an expert newsletter planner... (see Agent 1 above)
```
**Section Writer**
```
You are a professional newsletter section writer... (see Agent 2 above)
```
**Editor**
```
You are the newsletter editor and layout stylist... (see Agent 3 above)
```

### Expressions / Joins
- Join sections for the Editor:
```
{{$node["Sections"].json["output"].join("\n\n\n")}}
```
- Pull title from Planning:
```
{{$node["Planning Agent"].json["title"]}}
```

### Code Node — Formatters
See **Format Initial Research** and **Format Topic Research** in the steps above; paste as‑is. They turn raw arrays into neat strings the agents can digest (like cutting steak into bites 🥩).

---

## Next Steps
1) Duplicate the workflow for other niches.
2) Add approval & send flow.
3) Log runs + feedback to evolve prompts.

Congrats—you just built a practical multi‑agent system in n8n. High‑five your future self for the hours you’ll save ✋.

