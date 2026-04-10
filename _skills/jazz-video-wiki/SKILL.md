---
name: jazz-video-wiki
description: Process a jazz tutorial transcript (TSV file) into an Obsidian wiki note. Use this skill whenever a new video has been scraped and you need to generate or regenerate a wiki note for it. Triggers on phrases like "add video [ID] to the wiki", "process transcript for [ID]", "generate wiki note for [video]", or "new transcript ready".
---

# Jazz Video Wiki — Transcript → Wiki Note

This skill turns a single transcript TSV file into a structured Obsidian wiki note for the jazz tutorial knowledge base.

## Paths

- **Transcripts:** `scraping/openstudio/tts/{VIDEO_ID}.tsv`
- **Metadata:** `scraping/openstudio/metadata.json` — contains real video titles from yt-dlp
- **Wiki output:** `jazzwiki/videos/{VIDEO_ID}.md`
- **Base YouTube URL:** `https://www.youtube.com/watch?v={VIDEO_ID}`

Both paths are relative to the jazz-tutorials workspace root.

## Step 1 — Get the video title

Look up the VIDEO_ID in `scraping/openstudio/metadata.json`. If found, use the `title` field and set `title_source: yt-dlp`. If not found, infer the title from the first 30 lines of the transcript and set `title_source: inferred`.

## Step 2 — Read the COMPLETE transcript

**Read every single row of the TSV.** Columns: `start_ms`, `end_ms`, `text`.

- Duration = the `end_ms` value of the **last row** in the file
- Do not stop reading early. Do not skim. The last sections of long videos are where the most important advanced content lives.

## Step 3 — Identify section boundaries from the TSV

Before writing anything, scan the full TSV and identify 8–14 natural section boundaries (topic shifts, level introductions, exercise changes, etc.).

**For each section boundary, record the EXACT `start_ms` of the TSV row where it first appears.**

Do this by searching the TSV text for key phrases. For example:
- If Level 2 starts when a row says "What's level two", the section's `start_ms` is that row's value.
- If an exercise starts when a row says "Let's try this", the section's `start_ms` is that row's value.

**NEVER estimate or interpolate a timestamp.** If you cannot find the exact row, use the nearest row whose text clearly marks the transition.

## Step 4 — Timestamp formula

For any `start_ms` value from the TSV:
- Total seconds = `start_ms ÷ 1000` (integer division, discard remainder)
- Display: `MM:SS` — e.g. 838000ms → 838s → 13:58
- YouTube deep-link: `https://www.youtube.com/watch?v={VIDEO_ID}&t={total_seconds}s`

**Every timestamp in the output MUST be derived from an actual TSV row's `start_ms`. Zero exceptions.**

## Step 5 — Write the wiki note

Use this exact structure:

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
- *[[Tune Name]]* (composer if mentioned) — how it's used in the lesson

## Musicians & Influences Referenced
- [[Person Name]] — context in which they're mentioned

## Concepts Introduced
- [[Concept Name]] — one-line description of how it's framed here
(All concepts use Obsidian [[double-bracket]] wiki links)

## Practice Notes
Specific instructions the instructor gives: tempo, key, repetitions, what to listen for, etc.

## Transcript

### [MM:SS] Section Title — [▶](https://www.youtube.com/watch?v=VIDEO_ID&t=Xs)
[MM:SS](https://www.youtube.com/watch?v=VIDEO_ID&t=Xs) Textbook-style prose describing
what happens in this section. Third person. Keep close to the instructor's original phrasing
but remove filler, repetitions, YouTube engagement requests, and conversational asides.
Do not describe what is being played — write about what is being explained or taught.

Multiple timestamp paragraphs can appear under one section heading when the topic
continues across several minutes. Each paragraph starts with its own timestamp link.

### [MM:SS] Next Section — [▶](https://www.youtube.com/watch?v=VIDEO_ID&t=Xs)
...
```

The `## Transcript` section replaces both the old `## Lesson Plan` and `## Transcript` sections.
Section headings provide structure; timestamped prose paragraphs provide detail.
Aim for 8–14 sections covering the full video.

## Step 6 — Add inline wiki links throughout the note

After drafting the full note, go back through the text and add `[[double-bracket]]` links wherever a concept, tune, or musician is mentioned by name:

- **Concepts** — whenever a concept from `## Concepts Introduced` is named in the transcript prose, wrap it: e.g. "broken groups" → `[[Broken Groups]]`
- **Tunes** — whenever a tune name appears in prose, wrap it: e.g. "Lady Bird" → `[[Lady Bird]]`
- **Personas** — whenever a musician's name appears in prose, wrap it: e.g. "Herbie Hancock" → `[[Herbie Hancock]]`

Rules:
- Link on **first mention** in each section; subsequent mentions in the same paragraph can remain plain text.
- The link text must match the entity page filename exactly (case-sensitive).
- Do not link generic terms that are not entity page titles (e.g. "jazz" or "scales" are not links unless there is a specific page for them).
- In `## Tunes & Standards Referenced` and `## Musicians & Influences Referenced`, every name should already be linked. Check this.

## Step 7 — Record first-mention timestamps for entity pages

For each concept, tune, and persona identified in the note, find the **first TSV row** where that entity is meaningfully mentioned. Record the `start_ms` of that row. This will be used by the entity-pages skill to add YouTube deep-links to Appearances entries.

Output a quick summary at the end (not written to the file):
```
Entity first-mention timestamps (VIDEO_ID):
- [[Pentatonic Scales]]: 154000ms → 2:34 → &t=154s
- [[Herbie Hancock]]: 569000ms → 9:29 → &t=569s
...
```

## What NOT to do

- **Do not invent or guess timestamps.** Every `[MM:SS]` and `&t=Xs` value must come from an actual TSV row's `start_ms`. If you're writing a timestamp and you haven't looked up the corresponding TSV row, stop and look it up.
- Do not transcribe or analyse specific notes, chord fingerings, or note names. Focus on what is being taught and why, not what specific pitches are being played.
- Do not include purely musical demonstration passages in the transcript.
- Do not invent content. If something is unclear in the transcript, write around it or omit it.

## Mandatory quality checks before saving the file

1. **Duration:** frontmatter `duration` matches `end_ms` of the final TSV row.
2. **Timestamp accuracy:** For each section heading and each inline `[MM:SS]` timestamp, confirm you can point to the specific TSV row whose `start_ms` equals that time. If you cannot, fix it before saving.
3. **Coverage:** Transcript section headings span from near 0:00 to near the video's end. No large gaps (> 5 minutes undocumented).
4. **`title_source`:** accurately reflects whether the title came from metadata.json or was inferred.
5. **Wiki links:** Every concept, tune, and persona from the structured sections also appears linked (at least once) in the transcript prose where relevant.
