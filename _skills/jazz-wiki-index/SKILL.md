---
name: jazz-wiki-index
description: Regenerate the jazz wiki home page (Home.md) from all existing video notes. Use whenever new videos have been added or existing notes updated. Triggers on phrases like "regenerate the index", "update the home page", "rebuild the wiki index", or after a batch of new video notes are created.
---

# Jazz Wiki Index — Regenerate Home Page

This skill reads all existing video wiki notes and writes a fresh `jazzwiki/Home.md`.

## Step 1 — Collect all video notes

Read every `.md` file in `jazzwiki/videos/`. From each, extract:
- `title` (frontmatter)
- `video_id` (frontmatter)
- `duration` (frontmatter)
- `type` (frontmatter)
- `tags` (frontmatter)
- First sentence of the `## Overview` section

## Step 2 — Group by topic

Group videos into thematic clusters based on their tags and content. Typical clusters for this collection:
- **Chord Voicings** — voicings, comping, rootless, rooted, drop-two, spread, block chords
- **Harmony & Substitution** — ii-V-I, tritone substitution, secondary dominants, turnarounds
- **Scales & Modes** — pentatonic, scales, modes, bebop scale, sixth diminished
- **Improvisation** — triad pairs, enclosures, transcription, ear training, scale running
- **Technique & Practice** — guided practice sessions, physical technique, practice methodology
- **Analysis** — analyses of specific tunes or recordings

A video can appear in more than one group if appropriate.

## Step 3 — Write Home.md

Use this structure:

```markdown
---
updated: YYYY-MM-DD
video_count: N
---

# Jazz Tutorial Wiki

A knowledge base of jazz tutorial transcripts from Open Studio / You'll Hear It,
presented by Adam Maness. Each note contains a lesson plan, timestamped deep-links
into the video, and a textbook-style transcript rewrite.

> [!NOTE]
> Video titles marked *inferred* were derived from transcript content —
> external title lookup was unavailable at processing time.

## Videos by Topic

### Chord Voicings
| Video | Duration | Type |
|-------|----------|------|
| [[videos/VIDEO_ID\|Title]] | MM:SS | lesson |

### Harmony & Substitution
...

(one table per cluster)

## All Videos (A–Z by title)
| Title | Duration | Tags |
|-------|----------|------|
| [[videos/VIDEO_ID\|Title]] | MM:SS | tag1, tag2 |

## Concepts Index
Links to all `[[concept]]` pages that exist in `jazzwiki/concepts/`.
If the concepts folder is empty, omit this section.

## About
- **Source channel:** Open Studio / You'll Hear It (openstudiojazz.com)
- **Presenter:** Adam Maness
- **Transcripts:** Auto-generated via whisper-style TTS, stored as TSV files
- **Last updated:** YYYY-MM-DD
```

## Notes

- Use Obsidian wiki-link syntax: `[[videos/VIDEO_ID|Display Title]]`
- The `updated` date should be today's date
- Sort within each topic group by title alphabetically
- Keep the home page concise — it's a navigation hub, not a content page
