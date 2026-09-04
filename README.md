# Original Douay-Rheims Bible — JSON and USFM Dataset

The complete text of the Original Douay-Rheims Bible (Old Testament 1609, New
Testament 1582) in structured JSON, with footnotes, cross-references,
annotations, and reference documents.

This is the unmodified historic translation — not the 18th-century Challoner
revision — rendered faithfully from the original printed volumes.

**License:** [CC0 1.0 Universal](LICENSE) — public domain. Use freely, no
attribution required.

---

## Contents

| Path | Description |
|------|-------------|
| `bible/tagged/` | Bible text with all original markup (`<sc>`, `<i>`, `<na>`, `<mn>`, `<cr>`) — 76 books. **Canonical.** |
| `bible/raw/` | Bible text as plain prose — all markup and markers stripped |
| `annotations/` | Chapter-level annotation sidecars (commentary, extended notes) — 1,677 annotations |
| `usfm/` | USFM 3 files — one per book, with footnotes and cross-references |
| `usfm-study/` | The same, plus the annotations as `\ef` study notes |
| `index/lemmas/` | Where each annotation's catchword sits in the verse it annotates |
| `reference/ot/` | Old Testament front matter (preface, tables, glossary, etc.) |
| `reference/nt/` | New Testament front matter (preface, tables, glossary, etc.) |
| `manifest.json` | Schema version, source commit, and counts for every part of the bundle |

The JSON trees are canonical and lossless. The USFM trees are a projection into
a format that cannot express the whole 1582/1610 apparatus; `SCHEMA.md` lists
every deviation with its count.

---

## Bible JSON format

Each file in `bible/tagged/` and `bible/raw/` represents one book.

```json
{
  "book": "genesis",
  "book_title": "The Book of Genesis",
  "short_title": "Genesis",
  "chapters": [
    {
      "chapter": 1,
      "verses": [
        {
          "verse": 6,
          "text": "God also said: Be a firmament made amidst the waters.",
          "notes": [
            { "label": "a", "text": "The firmament is all the space from the earth..." }
          ]
        }
      ]
    }
  ]
}
```

### Markup tags (`bible/tagged/` only)

| Tag | Meaning |
|-----|---------|
| `<sc>Word</sc>` | Small caps — proper nouns and significant passages |
| `<i>word</i>` | Italic — words supplied by the translators, absent from the Latin |

`bible/raw/` strips all tags. `bible/tagged/` preserves everything including
footnote anchors (`<na>`), marginal note numbers (`<mn>`), and cross-reference
markers (`<cr>`). Note content is always in the verse's `notes[]` array regardless
of version.

**No verse is numbered 0.** Where a chapter summary overran its field in the
printed source, the continuation joins the `summary` it completes and is also
kept, with its own notes, in `summary_continuation`. See `SCHEMA.md`.

---

## Annotations format

Files in `annotations/{book}/{chapter}.json` contain verse-level commentary and
extended notes. Chapters without annotations have no file.

```json
{
  "chapter": 1,
  "annotations": [
    {
      "verse": 1,
      "part": 1,
      "title": "In the beginning.",
      "text": "Commentary text.",
      "notes": [
        { "marker": "a", "text": "Sub-note text." }
      ]
    }
  ]
}
```

`part` distinguishes several annotations on the same verse. `title` is the
catchword: the phrase from the verse the annotation comments on.

---

## USFM format

Files in `usfm/` and `usfm-study/` follow the Unified Standard Format Markers
(USFM 3) spec. They are named `<NN>-<CODE>.usfm`, where `NN` is the book's
position in the ODR canon (01–76) rather than its Paratext number, so the files
sort in the order the ODR prints them. The Paratext number is in
`manifest.json`.

Typographic markup is converted to USFM character styles:

| ODR tag | USFM marker |
|---------|-------------|
| `<sc>Word</sc>` | `\sc Word\sc*` |
| `<i>word</i>` | `\it word\it*` |

A character marker opened inside a note takes the `\+` prefix on both forms
(`\+it word\+it*`), as does one opened inside another marker.

Footnotes are embedded inline using `\f <label> \fr {ch}.{v} \ft {text}\f*`,
reusing the marker token the source printed — `\f 1`, `\f a`, or `\f -` where
the note had no token of its own.

Cross-references are embedded inline using `\x - \xt {text}\x*`. The reference
text uses original ODR abbreviations (e.g. `Io. 8, 12.` = John 8:12).

---

## Reference documents

Files in `reference/ot/` and `reference/nt/` contain the front matter from
the original printed volumes: prefaces to the reader, tables of contents,
glossaries, historical tables, doctrinal essays, and other apparatus.

Most files have a `paragraphs` array of `{ "text": "..." }` objects. Some have
`subsections`, `entries`, or `articles` arrays — see `SCHEMA.md` for the full
structure of each document type.

---

## Books included

76 books: the 73 of the Catholic canon, including the deuterocanonical books
(Tobias, Judith, 1 & 2 Machabees, Wisdom, Ecclesiasticus, Baruch), plus the
three the ODR prints in its appendix — 3 Esdras, 4 Esdras, and the Prayer of
Manasses.

The ODR follows Vulgate numbering, so its Esdras books are not the ones a modern
reader expects: 1 and 2 Esdras are Ezra and Nehemias (`EZR`, `NEH`), and 3 and 4
Esdras are the apocryphal books (`1ES`, `2ES`). `SCHEMA.md` has the full table
and the reasoning.

---

## Related

- [Original Douay-Rheims Bible](https://odr.app) — web reader built on this data
