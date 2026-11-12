# Original Douay-Rheims Bible: JSON and USFM Dataset

The complete text of the Original Douay-Rheims Bible (Old Testament 1609, New
Testament 1582) in structured JSON, with footnotes, cross-references,
annotations, and reference documents.

This is the unmodified historic translation, which predates the 18th-century
Challoner revision, rendered faithfully from the original printed volumes.

**License:** [CC0 1.0 Universal](LICENSE). Public domain. Use freely, no
attribution required.

---

## Contents

| Path | Description |
|------|-------------|
| `bible/tagged/` | Bible text with all original markup (`<sc>`, `<i>`, `<na>`, `<mn>`, `<cr>`): 76 books. **Canonical.** |
| `bible/raw/` | Bible text as plain prose, with all markup and markers stripped |
| `annotations/` | Chapter-level annotation sidecars (commentary, extended notes), 1,521 annotations |
| `usfm/` | USFM 3 files, one per book, with footnotes and cross-references |
| `usfm-study/` | The same, plus the annotations as `\ef` study notes |
| *(release assets)* | Reading editions as EPUB, OSIS, DOCX and HTML, verse-only and study, in three layouts. See below |
| *(release assets)* | Verse-only edition: scripture text alone, with no notes, markers, cross-references or summaries. JSON, USFM, Markdown, plain text, CSV and SQLite. See below |
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
| `<sc>Word</sc>` | Small caps for proper nouns and significant passages |
| `<i>word</i>` | Italic for words supplied by the translators, absent from the Latin |

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
reusing the marker token the source printed (`\f 1`, `\f a`, or `\f -` where
the note had no token of its own).

Cross-references are embedded inline using `\x - \xt {text}\x*`. The reference
text uses original ODR abbreviations (e.g. `Io. 8, 12.` = John 8:12).

---

## Reference documents

Files in `reference/ot/` and `reference/nt/` contain the front matter from
the original printed volumes: prefaces to the reader, tables of contents,
glossaries, historical tables, doctrinal essays, and other apparatus.

Most files have a `paragraphs` array of `{ "text": "..." }` objects. Some have
`subsections`, `entries`, or `articles` arrays. See `SCHEMA.md` for the full
structure of each document type.

---

## Books included

76 books: the 73 of the Catholic canon, including the deuterocanonical books
(Tobias, Judith, 1 & 2 Machabees, Wisdom, Ecclesiasticus, Baruch), plus the
three the ODR prints in its appendix: 3 Esdras, 4 Esdras, and the Prayer of
Manasses.

The ODR follows Vulgate numbering, so its Esdras books are not the ones a modern
reader expects: 1 and 2 Esdras are Ezra and Nehemias (`EZR`, `NEH`), and 3 and 4
Esdras are the apocryphal books (`1ES`, `2ES`). `SCHEMA.md` has the full table
and the reasoning.

---

## Related

- [Original Douay-Rheims Bible](https://odr.app): web reader built on this data

## Verse-only edition

The scripture text with everything else removed: no footnotes, annotations,
cross-references, marginal markers, chapter summaries, introductions or end
matter. It is generated from the same data as the rest of the bundle and
published as release assets rather than committed, so the repository stays
small.

| Release asset | Contents |
|---------------|----------|
| `original-douay-rheims-verse-only-json.zip` | `<book>.json` per book, `{ book, code, title, chapters: [{ chapter, verses: [{ verse, text }] }] }`, plus `original-douay-rheims.json` with all 76 books |
| `original-douay-rheims-verse-only-usfm.zip` | USFM 3, one file per book, with `\c`, `\p`, `\v`. Italics (`\it`) and small caps (`\sc`) are kept |
| `original-douay-rheims-verse-only-markdown.zip` | One `.md` per book, bold verse numbers, italics kept |
| `original-douay-rheims-verse-only-text.zip` | One `.txt` per book, wrapped at 76 columns with hanging verse numbers |
| `original-douay-rheims-verse-only.md` / `.txt` | The whole Bible in one file |
| `original-douay-rheims-verse-only.csv` | `book,code,chapter,verse,part,text`, one row per verse |
| `original-douay-rheims-verse-only.sqlite` | Tables `books` and `verses`, plus an FTS5 index `verses_fts` for full-text search |

`part` is empty unless a chapter repeats a verse number, which the current text
does not.

## Reading editions (EPUB, OSIS, DOCX, HTML)

The text as e-books, Word documents, Bible-software files and web pages, published as release assets. Every file is a combination of three choices, and every combination is built:

| Choice | Values |
|--------|--------|
| Edition | `verse-only` (scripture text alone) or `study` (with the printed apparatus) |
| Layout | `verse` (one paragraph per verse), `paragraph` (the site's paragraphs) or `flowing` (one paragraph per chapter) |
| Format | `epub`, `osis`, `docx` or `html` |

Each combination has two assets:

- **Whole Bible**: one file, `original-douay-rheims-<edition>-<layout>.<ext>`, for example `original-douay-rheims-verse-only-verse.html`.
- **Book by book**: one zip of 76 files, one per book, `original-douay-rheims-<edition>-<layout>.<ext>.by-book.zip`, for example `original-douay-rheims-study-paragraph.epub.by-book.zip`. The zip holds only single-book files and does not repeat the whole Bible.

That is 24 whole-Bible files and 24 book-by-book zips. Only EPUB files carry the 1582 Rheims title page as a cover image. DOCX, HTML and OSIS have no images.

### Editions

The verse-only edition has no notes, markers, cross-references, summaries or introductions. The study edition carries:

- Book introductions, chapter summaries (the "arguments"), articles and end matter.
- Marginal notes and cross-references as footnotes (OSIS: `<note>` and `<note type="crossReference">`).
- The extended annotations, after the verse they belong to.
- In the whole-Bible files, the transcribed title pages, prefaces, censures, tables, glossary, index and errata, as front and back matter of each testament.

Twenty-three notes have no marker in the text. The USFM files leave them out; the study files keep each one as a footnote at the end of its verse, summary or paragraph. The Sancta Clara Press cross-references that have no marker in the text are left out, as in the USFM files. OSIS notes cannot nest, so an annotation's own sub-notes are flattened into a superscript number and trailing text, the same as in the `usfm-study` files.

### Layouts

`paragraph` uses the paragraph breaks from the reading site, which come from the CPDV edition's USFM and are applied to this text by verse number. They are not the breaks of the 1582 and 1609 printings. Psalms, the Canticle of Canticles, 3 Esdras, 4 Esdras and the Prayer of Manasses have no paragraph data, so in the `paragraph` layout each of their chapters is one paragraph. The `verse` layout covers those books verse by verse. In OSIS, `flowing` puts the verses directly in the chapter and the other two layouts wrap them in `<p>`.

### Formats

| Format | What it gives you |
|--------|-------------------|
| `epub` | EPUB 3 with a navigation document by testament, book and chapter. Real footnotes in the study edition |
| `osis` | OSIS 2.1.1 XML with an `osisID` on every book, chapter and verse (`Gen.1.1`), valid against `osisCore.2.1.1.xsd`. Italics are `<hi type="italic">` and small caps are `<hi type="small-caps">`. Study annotations are `<note type="study">`, summaries are `<div type="summary">` and introductions are `<div type="introduction">` |
| `docx` | Word documents with heading styles for testament, book and chapter, Word footnotes in the study edition, and a contents page of links at the start: every book, with a link to each of its chapters. Each book also opens with a row of chapter links |
| `html` | One self-contained page per file with no images, a collapsible contents sidebar, a row of chapter links under each book title, and light and dark themes. It works offline and without scripts, though the sidebar collapse needs them |

Releases v3.1 to v3.4 hold earlier builds under their older names (`original-douay-rheims-verse-only-epub.zip` and so on), and v3.4 still has the cover image in its DOCX and HTML files. Those DOCX files have a Word table of contents field that stays empty until the reader updates it. The v3.4 DOCX files replace it with the contents page of links.
