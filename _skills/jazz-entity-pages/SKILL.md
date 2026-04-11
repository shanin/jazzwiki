---
name: jazz-entity-pages
description: Create or update concept, song, and persona pages in the jazz wiki from one or more video notes. Use whenever a new video note has been added and its entities (concepts, tunes, musicians) need wiki pages. Triggers on phrases like "update entity pages", "create concept pages for [video]", "add the concepts from the new videos", or "sync the entity pages".
---

# Jazz Entity Pages — Create & Update Concepts, Songs, Personas

This skill reads video notes from `jazzwiki/videos/` and maintains three entity folders:
- `jazzwiki/concepts/` — musical concepts and techniques
- `jazzwiki/songs/` — tunes and standards
- `jazzwiki/personas/` — musicians and educators

## Important: summaries improve over time

Concept (and song) summaries written from a single video are inherently limited — they reflect only one instructor's framing at one moment. As more videos are processed, descriptions should be revised to be more complete, accurate, and context-independent.

**Do not treat existing summaries as authoritative.** If a new video offers a clearer, broader, or corrective framing of a concept, update the summary to reflect that.

---

## When to run

After one or more new video notes have been added (via the `jazz-video-wiki` skill or manually), run this skill to ensure every entity mentioned in those notes has a page, and every existing entity page has an updated Appearances section.

---

## Step 1 — Collect entities from target video notes

For each target video note, extract:
1. **Concepts** — everything in the `## Concepts Introduced` section (the `[[wiki link]]` names)
2. **Songs** — everything in `## Tunes & Standards Referenced`
3. **Personas** — everything in `## Musicians & Influences Referenced`

Also note the video's `video_id` from the frontmatter.

---

## Step 2 — Find first-mention timestamps from the TSV

For each extracted entity, search `scraping/openstudio/tts/{VIDEO_ID}.tsv` for the **first row** where the entity is meaningfully mentioned. Record the `start_ms` of that row.

YouTube deep-link: `https://www.youtube.com/watch?v={VIDEO_ID}&t={start_ms // 1000}s`

If the entity structures the whole video, use the timestamp where it's first explicitly named.

---

## Step 3 — For each entity, create or update its page

### If the page does not exist — create it

Use the templates below. Set `sources: [VIDEO_ID]` in frontmatter.

### If the page already exists — update it

Do two things:

**A. Revise the summary if warranted.**
Read the existing summary. Read how the concept is framed in the new video. Ask:
- Does the new video present this concept more completely or accurately?
- Does it correct a partial or video-specific framing?
- Does it broaden the definition beyond a single use case?

If yes to any of these: rewrite the summary to incorporate the improved understanding. Add the new `VIDEO_ID` to the `sources` list in frontmatter.

If the existing summary is already accurate and the new video just uses the concept in a similar way: leave the summary as-is, just add the new source and new Appearances bullet.

**B. Append a new bullet to `## Appearances`.**

---

## Templates

### Concept page

```markdown
---
type: concept
tags: [tag1, tag2]
sources: [VIDEO_ID]
---

# Concept Name

A clear, concise explanation of what this concept is — what it does, why it matters, and how it relates to jazz broadly. Write for someone who knows jazz but may not know this specific term. 2–5 sentences. Do not analyse specific notes or fingerings.

*Based on N video(s). Description may be refined as more sources are added.*

## Appearances

- **[[../videos/VIDEO_ID|Video Title]]** [▶ MM:SS](https://www.youtube.com/watch?v=VIDEO_ID&t=Xs) — how this concept was introduced and used in that lesson.
```

### Song page

```markdown
---
type: song
composer: Composer Name
tags: [jazz-standard]
sources: [VIDEO_ID]
---

# Song Title

Brief description: composer, era, what makes the tune harmonically or structurally notable. 2–4 sentences.

*Based on N video(s). Description may be refined as more sources are added.*

## Appearances

- **[[../videos/VIDEO_ID|Video Title]]** [▶ MM:SS](https://www.youtube.com/watch?v=VIDEO_ID&t=Xs) — how it's used in the lesson.
```

### Persona page

```markdown
---
type: persona
role: pianist | saxophonist | guitarist | drummer | etc.
era: swing | bebop | hard-bop | post-bop | fusion | contemporary
tags: [tag1, tag2]
sources: [VIDEO_ID]
---

# Person Name

3–5 factual sentences: instrument, era, contribution to jazz. Do not invent details.

## Appearances

- **[[../videos/VIDEO_ID|Video Title]]** [▶ MM:SS](https://www.youtube.com/watch?v=VIDEO_ID&t=Xs) — context in which they are mentioned.
```

Note: persona summaries are biographical facts that don't need the "may be refined" caveat — they're either right or wrong, not context-dependent.

---

## Appearances entry format (required)

```
- **[[../videos/VIDEO_ID|Video Title]]** [▶ MM:SS](https://www.youtube.com/watch?v=VIDEO_ID&t=Xs) — one sentence describing how this entity appears in the video
```

- `[▶ MM:SS]` points to the first meaningful mention in the TSV — never invented.
- Description: one sentence, specific to this video's framing.

---

## Rules

- **Do not analyse specific notes or fingerings.**
- **Relative paths** for video links: `../videos/VIDEO_ID` (one level up from entity folder).
- **Only write what you can source.** For summaries, general knowledge is fine, but flag uncertainty rather than invent.
- **One file per entity** — multiple videos → one page with multiple Appearances bullets.
- **Update `sources`** in frontmatter whenever a page is revised or appended.
- Report back: pages created, pages updated (summary revised vs. bullet-only), pages skipped.
