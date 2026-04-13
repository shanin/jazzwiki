---
name: jazz-video-wiki
description: Process a jazz tutorial transcript (TSV file) into an Obsidian wiki note, create all entity pages, then add wiki links. Use whenever a new video has been scraped and needs to be added to the wiki. Triggers on phrases like "add video [ID] to the wiki", "process transcript for [ID]", "generate wiki note for [video]", or "new transcript ready".
---

# Jazz Video Wiki — Full Pipeline: Transcript → Wiki Note + Entity Pages + Links

This skill processes one video end-to-end:
1. Writes the video note
2. Creates any missing entity pages (concepts, songs, personas)
3. Adds `[[wiki links]]` into the video note prose — only for entities that now have pages

**Never add a `[[link]]` to a page that does not exist. Entity pages must be created first.**

---

## Paths

- **Transcripts:** `scraping/openstudio/tts/{VIDEO_ID}.tsv`
- **Metadata:** `scraping/openstudio/metadata.json` — real video titles from yt-dlp
- **Wiki videos:** `jazzwiki/videos/{VIDEO_ID}.md`
- **Wiki concepts:** `jazzwiki/concepts/{Name}.md`
- **Wiki songs:** `jazzwiki/songs/{Title}.md`
- **Wiki personas:** `jazzwiki/personas/{Name}.md`

All paths relative to the jazz-tutorials workspace root.

---

## Step 1 — Get the video title

Look up the VIDEO_ID in `scraping/openstudio/metadata.json`. Use the `title` field → `title_source: yt-dlp`. If not found, infer from first 30 TSV rows → `title_source: inferred`.

---

## Step 2 — Read the COMPLETE transcript

Read **every single row** of the TSV. Columns: `start_ms`, `end_ms`, `text`.

- Duration = `end_ms` of the **last row**
- Do not stop reading early. The last sections of a video often contain the most advanced content.

---

## Step 3 — Identify section boundaries

Scan the full TSV and identify 8–14 natural topic boundaries. For each, record the **exact `start_ms`** of the TSV row where that topic first appears. Search TSV text for key phrases — do not estimate.

**Timestamp formula:**
- `total_seconds = start_ms ÷ 1000` (integer)
- Display: `MM:SS`
- YouTube link: `https://www.youtube.com/watch?v={VIDEO_ID}&t={total_seconds}s`

**Every timestamp must come from an actual TSV row's `start_ms`. No exceptions.**

---

## Step 4 — Extract entities

While reading, collect:
- **Concepts** — musical ideas, techniques, frameworks introduced or named
- **Tunes** — songs and standards referenced by name
- **Personas** — musicians and educators mentioned by name

For each entity, also note:
- The **first TSV row** where it is meaningfully mentioned → record `start_ms`
- A brief description of how it appears in this video (for the Appearances entry)

---

## Step 5 — Write the video note (without wiki links yet)

Use this structure. Do not add `[[links]]` in the transcript yet — that happens in Step 7 after entity pages exist.

```markdown
---
title: "TITLE"
video_id: VIDEO_ID
youtube_url: https://www.youtube.com/watch?v=VIDEO_ID
title_source: yt-dlp | inferred
channel: Open Studio
presenter: Adam Maness
duration: "MM:SS"
tags: [tag1, tag2]
type: guided-practice | lesson | analysis | interview
---

# TITLE

**Presenter:** Adam Maness | **Channel:** Open Studio
**Duration:** MM:SS | **YouTube:** [▶ Watch](https://www.youtube.com/watch?v=VIDEO_ID)

## Overview
2–3 sentences: what is taught, for whom, and why it matters.

## Tunes & Standards Referenced
- *Tune Name* (composer if mentioned) — how it's used
(No [[links]] here yet — added in Step 7)

## Musicians & Influences Referenced
- Person Name — context
(No [[links]] here yet — added in Step 7)

## Concepts Introduced
- [[Concept Name]] — one-line description
(Links here are fine — concept names are canonical identifiers)

## Practice Notes
Tempo, key, repetitions, what to listen for — specific instructor guidance only.

## Transcript

### [MM:SS] Section Title — [▶](https://www.youtube.com/watch?v=VIDEO_ID&t=Xs)

[MM:SS](https://www.youtube.com/watch?v=VIDEO_ID&t=Xs) Textbook-style prose in third person.
Keep close to the instructor's phrasing. Remove filler, repetition, YouTube engagement,
and purely conversational asides. Focus on what is being taught, not what is being played.
No note names, no fingering analysis.

Multiple paragraphs per section are fine. Each paragraph starts with its timestamp link.
```

---

## Step 6 — Create or update entity pages

**Important: concept summaries improve over time.** A description written from one video reflects only one framing. If a new video offers a clearer, broader, or corrective view of a concept, update the summary — don't just append.

For each entity collected in Step 4, check whether its page already exists.

**If it does not exist → create it now.** Use the templates below. Set `sources: [VIDEO_ID]` in frontmatter. Add the line *"Based on N video(s). Description may be refined as more sources are added."* below the summary for concept and song pages (not persona pages).

**If it already exists → do two things:**
1. Read the existing summary. If this video frames the concept more completely or accurately, revise the summary and add this VIDEO_ID to the `sources` list.
2. Append a new bullet to `## Appearances`. Do not remove existing bullets.

### Appearances entry format

```
- **[[../videos/VIDEO_ID|Video Title]]** [▶ MM:SS](https://www.youtube.com/watch?v=VIDEO_ID&t=Xs) — one sentence describing how this entity appears in the video
```

The `[▶ MM:SS]` timestamp comes from the entity's first-mention `start_ms` recorded in Step 4.

### Concept page template

```markdown
---
type: concept
tags: [tag1, tag2]
sources: [VIDEO_ID]
---

# Concept Name

2–5 sentence explanation: what this concept is, why it matters, how it relates to jazz.
No note names or fingerings.

*Based on 1 video. Description may be refined as more sources are added.*

## Appearances

- **[[../videos/VIDEO_ID|Video Title]]** [▶ MM:SS](https://www.youtube.com/watch?v=VIDEO_ID&t=Xs) — context in this video
```

### Song page template

```markdown
---
type: song
composer: Name
tags: [jazz-standard]
sources: [VIDEO_ID]
---

# Song Title

2–4 sentence description: composer, era, what makes it harmonically notable.

*Based on 1 video. Description may be refined as more sources are added.*

## Appearances

- **[[../videos/VIDEO_ID|Video Title]]** [▶ MM:SS](https://www.youtube.com/watch?v=VIDEO_ID&t=Xs) — how it's used
```

### Persona page template

```markdown
---
type: persona
role: pianist | saxophonist | etc.
era: bebop | post-bop | etc.
tags: [tag1, tag2]
sources: [VIDEO_ID]
---

# Person Name

3–5 factual sentences: instrument, era, contribution to jazz. No invented details.

## Appearances

- **[[../videos/VIDEO_ID|Video Title]]** [▶ MM:SS](https://www.youtube.com/watch?v=VIDEO_ID&t=Xs) — context
```

---

## Step 7 — Add [[wiki links]] to the video note

Now that all entity pages exist, go back through the saved video note and add `[[links]]` in the prose:

- In `## Tunes & Standards Referenced`: wrap each tune name → `*[[Tune Name]]*`
- In `## Musicians & Influences Referenced`: wrap each name → `[[Person Name]]`
- In `## Transcript` prose: on the **first mention** of any entity in each section paragraph, wrap it

Rules:
- **Only link to pages that exist.** If no page was created for something, do not link it.
- Match the exact filename (case-sensitive, spaces as-is).
- In `## Concepts Introduced`, links are already there from Step 5 — verify they match actual filenames.
- Do not link the same entity more than once per paragraph.
- Do not link generic terms that are not entity page titles.

---

## What NOT to do

- **Never add a `[[link]]` without first confirming the target page exists.**
- Never invent or estimate timestamps — every `[MM:SS]` must come from a TSV row's `start_ms`.
- Never transcribe specific note names, chord fingerings, or interval sequences.
- Never invent biographical or musical facts.
- **Never use backslashes in filenames.** Entity page filenames must use regular spaces (e.g. `Moo chord.md`, not `Moo\ chord.md`). When writing files via Bash, quote paths properly rather than escaping spaces with backslashes.

---

## Quality checks before finishing

1. **Duration** in frontmatter matches last TSV row's `end_ms`.
2. Every `[MM:SS]` timestamp traces to a real TSV row.
3. Every `[[link]]` in the note has a corresponding file in the wiki.
4. Transcript spans from near 0:00 to near the video's end with no gaps > 5 minutes.
5. Entity pages all have the correct `[▶ MM:SS]` deep-link in their Appearances entry.
