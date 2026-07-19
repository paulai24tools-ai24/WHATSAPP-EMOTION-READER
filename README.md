# Core Sample

**Turn your chat archive into a story you can feel.**

Drop in an export from WhatsApp, Instagram, Snapchat, or TikTok. Two AI agents read every line — one maps the emotional terrain of the conversation over time, the other digs out ideas, problem statements, and business concepts buried inside it. What comes back is a "core sample": a chronological, layered read of the whole archive.

This is a **single self-contained HTML file** — no build step, no server, no dependencies to install. Open it in a browser or host it on GitHub Pages and it works.

---

## Live demo

Once pushed, enable **GitHub Pages** (Settings → Pages → Deploy from branch → `main` / root) and the app will be live at:
`https://<your-username>.github.io/<repo-name>/`

Because the file is named `index.html`, GitHub Pages serves it automatically at the repo root — no config needed.

---

## What it does

1. **Parse** — auto-detects the export format (WhatsApp `.txt`, Instagram `.json`, Snapchat `.json`/`.html`, TikTok `.json`, or a zip containing any of these) and normalizes every message to `{sender, timestamp, text}`.
2. **Chapter it** — groups messages into chronological chapters (by month, or into even chunks for very long histories).
3. **Emotion Agent** — reads each chapter and scores it against a fixed 17-emotion palette (joy, longing, tension, nostalgia, anxiety, pride, etc.), writing a short narrative for what was actually happening.
4. **Idea Agent** — separately scans each chapter for anything idea-shaped: a business model, a problem statement, a project concept — and only reports what's genuinely there.
5. **Render** — a vertical "core sample" strip (colored bands = chapters, sized by message volume, colored by dominant emotion) next to structured chapter cards ("Act I," "Act II"...), with an ideas gallery below.

## Privacy / data handling

- Everything runs client-side in the browser. A zip's media files (photos/videos) are never read — only `.txt/.json/.html` entries inside it are extracted, so multi-gigabyte exports process quickly.
- Message text is sent to the Claude API (`api.anthropic.com`) per chapter, for analysis only. Nothing is stored server-side by this app; there's no backend.
- If you fork this for public/shared use, put your own usage notice here — right now it assumes a personal, single-user context.

## Model

Uses `claude-sonnet-4-6` for both agents — enough nuance to read emotional tone (not just keyword-match sentiment) while staying fast enough to run one call per chapter without a long wait.

## Known limitations

- Instagram/TikTok/Snapchat export JSON structures vary across app versions; the parser uses a best-effort heuristic scan and may miss some layouts. WhatsApp's format is the most reliably handled.
- Very large conversations (10k+ messages) mean more chapters, which means more sequential agent calls — processing time scales accordingly.
- No persistence: refresh the page and you start over. That's intentional for privacy, but worth knowing.

---

## Repo structure

```
/
├── index.html   ← the entire app (HTML + CSS + JS, single file)
└── README.md    ← this file
```

---

## Master prompt (for extending or rebuilding this with an AI assistant)

This is the full spec used to build this app. Paste it into Claude (or any capable coding assistant) if you want to regenerate, fork, or meaningfully extend this project — it captures the product intent and the design system, not just the code.

```
Build "Core Sample" — a single self-contained HTML file (HTML+CSS+JS, no build
step, works when opened directly or hosted on GitHub Pages) that turns a
personal chat export into an emotional, chaptered story.

PROBLEM IT SOLVES
A person opens WhatsApp/Instagram/Snapchat/TikTok and has years of messages
they'll never read end to end. This tool extracts the shape of the
conversation instead: an emotional timeline plus any ideas that were ever
floated inside it.

FUNCTIONAL REQUIREMENTS
1. Accept a .zip, .txt, .json, or .html file via drag-and-drop or file picker.
2. If it's a zip, extract only .txt/.json/.html entries — never touch media
   (images/video), so multi-gigabyte exports still process in seconds.
3. Auto-detect the export format (best effort, no single format needs to be
   perfect):
   - WhatsApp .txt: lines like "[3/14/23, 9:15:32 PM] Name: message" or
     "3/14/23, 9:15 PM - Name: message"
   - Instagram .json: { participants: [...], messages: [{sender_name,
     timestamp_ms, content}] }
   - TikTok / Snapchat .json: structure varies by export version — implement
     a generic recursive scanner that looks for arrays of objects containing
     sender-like keys (from/sender/author/name), text-like keys
     (text/content/message/body), and time-like keys (time/date/timestamp)
   - .html exports or anything unrecognized: strip tags, fall back to
     treating non-trivial lines as messages
4. Normalize everything to {sender, timestamp (Date or null), text}, merge
   messages from multiple files in the zip, sort chronologically.
5. Group messages into chapters: one chapter per calendar month if the date
   range is <=14 months and most messages have timestamps; otherwise split
   into ~8-10 equal chunks by message count.
6. Run two independent AI agents per chapter, calling the Claude API
   (model: claude-sonnet-4-6, max_tokens: 1000, no API key needed if running
   inside an environment that injects it — otherwise the user supplies their
   own key):

   EMOTION AGENT — system prompt instructs it to choose only from a fixed
   17-emotion taxonomy (joy, love, excitement, gratitude, nostalgia, sadness,
   anxiety, anger, frustration, hope, pride, relief, longing, confusion,
   tension, humor, curiosity), and to return strict JSON:
   { "title": short chapter title, "narrative": 2-4 sentence description of
   what was happening and how it felt, grounded in the actual excerpt,
   "emotions": [{"name":..., "score":0-1}, 2 to 5 entries sorted desc] }

   IDEA AGENT — system prompt instructs it to scan the same chapter text for
   anything idea-shaped (business model, problem statement, project concept,
   creative concept), returning strict JSON:
   { "ideas": [{"title":..., "description":..., "category": one of
   "Business Model"/"Problem Statement"/"Project Idea"/"Creative Concept"}] }
   Must return an empty array rather than inventing ideas if none exist.

7. Render results as:
   - A vertical "core sample" strip: one colored band per chapter, height
     proportional to message count, color = dominant emotion's color from the
     fixed palette, clickable to scroll to that chapter's card.
   - Chapter cards labeled "Act I / Act II / ..." with title, date range,
     message count, the narrative, and emotion tags (color dot + name +
     score %).
   - An "ideas" gallery section below: cards grouped/labeled by category.
   - A progress indicator during processing showing which chapter each agent
     is currently reading.
8. No backend, no persistence, no localStorage — everything lives in memory
   for the session. State resets on reload by design (privacy).

DESIGN SYSTEM (do not default to generic AI-design patterns — no cream +
terracotta serif hero, no near-black + single neon accent, no broadsheet
hairline newspaper layout)

Concept: geological / archaeological. The conversation is sediment; the tool
extracts a core sample and shows its strata. This metaphor should show up in
copy ("dig", "layers", "core sample", "Act"), not just visuals.

Colors:
  --ink #10131A          deep charcoal-navy background
  --surface #191E28       card surface
  --surface-raised #212736
  --paper #EDE6D6         warm off-white for primary text on dark
  --paper-dim #B9B2A2      secondary/muted text
  --amber #E8A33D         primary accent (extraction glow / CTA)
  --rust #C1553C          secondary accent
  --line rgba(237,230,214,0.10)   hairline dividers
  Plus a 17-color emotion palette (one hex per emotion, used functionally in
  strata/tags, not decoratively) — e.g. joy=#F2B705, love=#E85D75,
  anxiety=#C77DFF, tension=#B23A48, curiosity=#5AAAD9, etc.

Type: Fraunces (serif, display — headlines, chapter titles) + Inter (body
copy, UI text) + IBM Plex Mono (utility — timestamps, chapter numbers,
eyebrows, data labels). Three distinct roles, used consistently.

Layout: hero is a thesis — one clear line of copy plus a small illustrative
"core tube" graphic showing dummy strata, next to a drop zone. Results page
is a sticky two-column layout: a slim vertical core-sample strip on the left
(the signature element — real data drives its bands, not decoration), full
chapter cards on the right, ideas gallery in a grid below.

Signature element: the clickable vertical core-sample strip. Its bands must
be driven by real data (message volume + dominant emotion per chapter), not
just visual flourish — this is what should make the page memorable.

Motion: minimal — hover brightness on strata, a spinner + progress bar while
processing, no scroll-triggered animation flourishes.

NON-GOALS
- Do not attempt to process media files (images/video/audio) from a zip.
- Do not claim perfect parsing for every possible export version — degrade
  gracefully with a generic line-based fallback instead of failing.
- Do not persist or transmit data anywhere except the analysis API calls.
```

---

## Extending this

Ideas for a v2, if you want to keep iterating:
- Let the user pick which emotions matter to them (a custom subset of the 17).
- A "relationship over time" trend line separate from per-chapter chapters.
- Export the finished story as a PDF or shareable image.
- Support multi-person group chats with per-person emotional breakdowns.
