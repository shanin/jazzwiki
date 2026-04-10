---
name: jazz-entity-pages
description: Create or update concept, song, and persona pages in the jazz wiki from one or more video notes. Use whenever a new video note has been added and its entities (concepts, tunes, musicians) need wiki pages. Triggers on phrases like "update entity pages", "create concept pages for [video]", "add the concepts from the new videos", or "sync the entity pages".
---

# Jazz Entity Pages — Create & Update Concepts, Songs, Personas

This skill reads video notes from `jazzwiki/videos/` and maintains three entity folders:
- `jazzwiki/concepts/` — musical concepts and techniques
- `jazzwiki/songs/` — tunes and standards
- `jazzwiki/personas/` — musicians and educators

## When to run

After one or more new video notes have been added (via the `jazz-video-wiki` skill or manually), run this skill to ensure every entity mentioned in those notes has a page, and every existing entity page has an updated Appearances section.

## Step 1 — Collect entities from target video notes

For each target video note, extract:
1. **Concepts** — everything in the `## Concepts Introduced` section (the `[[wiki link]]` names)
2. **Songs** — everything in `## Tunes & Standards Referenced`
3. **Personas** — everything in `## Musicians & Influences Referenced`

Also note the video's `video_id` from the frontmatter.

## Step 2 — Find first-mention timestamps from the TSV

For each extracted entity, search the TSV file at `scraping/openstudio/tts/{VIDEO_ID}.tsv` for the **first row** where the entity is meaningfully mentioned. Record the `start_ms` of that row.

Convert to YouTube deep-link: `https://www.youtube.com/watch?v={VIDEO_ID}&t={start_ms // 1000}s`

This timestamp link will appear in the entity's Appearances entry. It should point to the moment in the video where the entity is first introduced or named.

If the entity is present throughout (e.g. a concept that structures the whole video), use the timestamp where it's first explicitly named or introduced.

## Step 3 — For each entity, create or update its page

### If the page does not exist — create it

**Concepts** → `jazzwiki/concepts/{Concept Name}.md`
**Songs** → `jazzwiki/songs/{Song Title}.md`
**Personas** → `jazzwiki/personas/{Person Name}.md`

Use the templates below.

### If the page already exists — append to Appearances

Add a new bullet to the `## Appearances` section. Do not rewrite the summary or existing appearances. Only add what's new.

---

## Templates

### Concept page

```markdown
---
type: concept
tags: [tag1, tag2]
---

# Concept Name

A clear, concise explanation of what this concept is — what it does, why it matters, and how it relates to jazz broadly. Write for someone who knows jazz but may not know this specific term. 2–5 sentences. Do not analyse specific notes or fingerings.

## Appearances

- **[[../videos/VIDEO_ID|Video Title]]** [▶ MM:SS](https://www.youtube.com/watch?v=VIDEO_ID&t=Xs) — how this concept was introduced and used in that lesson. Reference the specific exercise, context, or framing from the video note.
```

### Song page

```markdown
---
type: song
composer: Composer Name
tags: [jazz-standard, bebop]
---

# Song Title

Brief description: composer, approximate date, notable recordings, what makes the tune harmonically or structurally distinctive. 2–4 sentences.

## Appearances

- **[[../videos/VIDEO_ID|Video Title]]** [▶ MM:SS](https://www.youtube.com/watch?v=VIDEO_ID&t=Xs) — how the tune is used in the lesson (vehicle for a specific concept, analysed, performed, etc.)
```

### Persona page

```markdown
---
type: persona
role: pianist | saxophonist | guitarist | drummer | etc.
era: swing | bebop | hard-bop | post-bop | fusion | contemporary
tags: [tag1, tag2]
---

# Person Name

Brief biographical description: instrument, era, what they are known for musically, why they matter in the jazz tradition. 3–5 sentences. Factual — do not invent details you are uncertain about.

## Appearances

- **[[../videos/VIDEO_ID|Video Title]]** [▶ MM:SS](https://www.youtube.com/watch?v=VIDEO_ID&t=Xs) — context in which they are mentioned (technique cited, inspiration, recording referenced, etc.)
```

---

## Appearances entry format (critical)

Every Appearances bullet must follow this exact format:

```
- **[[../videos/VIDEO_ID|Video Title]]** [▶ MM:SS](https://www.youtube.com/watch?v=VIDEO_ID&t=Xs) — description of context
```

- The `[▶ MM:SS]` link points to the first moment in the video where this entity is meaningfully mentioned.
- The timestamp must come from the TSV (Step 2 above) — never invented.
- The description should be a single sentence summarising how the entity is used in that video.

---

## Rules

- **Do not analyse specific notes or fingerings** in any entity page.
- **Relative paths** for video links must be `../videos/VIDEO_ID` (one level up from the entity folder).
- **Only write what you can source** from the video notes. For the summary sections, you may draw on general knowledge, but flag uncertainty rather than invent.
- **Persona summaries** should be factual and neutral — not hagiographic.
- **One file per entity** — if two videos reference the same concept, there is still only one concept page; add a second bullet to Appearances.
- After creating/updating pages, report back: how many pages were created, how many updated, and list any entities that were skipped (e.g. because they were too vague to warrant a standalone page).
