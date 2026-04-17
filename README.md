# Original Douay-Rheims Bible — JSON Dataset

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
| `bible/raw/` | Bible text with original markup tags (73 books) |
| `bible/clean/` | Bible text with all tags stripped — plain text only |
| `annotations/` | Chapter-level annotation sidecars (commentary, notes) |
| `reference/ot/` | Old Testament front matter (preface, tables, glossary, etc.) |
| `reference/nt/` | New Testament front matter (preface, tables, glossary, etc.) |

---

## Bible JSON format

Each file in `bible/raw/` and `bible/clean/` represents one book.

```json
{
  "book": "genesis",
  "testament": "ot",
  "chapters": [
    {
      "chapter": 1,
      "verses": [
        {
          "verse": 1,
          "text": "In the beginning God created heaven and earth.",
          "notes": [
            { "marker": 1, "text": "Note text here." }
          ]
        }
      ]
    }
  ]
}
```

### Markup tags (raw version only)

| Tag | Meaning |
|-----|---------|
| `<sc>Word</sc>` | Small caps — proper nouns and significant passages |
| `<i>word</i>` | Italic — words supplied by translators, absent from the Latin |
| `<cr>†</cr>` | Cross-reference marker (symbol only, no content) |
| `<na>*</na>` | Footnote anchor marker (symbol only, no content) |
| `<mn>[1]</mn>` | Marginal note number |

In `bible/clean/`, all tags are stripped. `<cr>`, `<na>`, and `<mn>` are removed
entirely (including their marker content). `<sc>` and `<i>` content is preserved
as plain text.

---

## Annotations format

Files in `annotations/{book}/{chapter}.json` contain verse-level commentary and
extended notes.

```json
{
  "book": "genesis",
  "chapter": 1,
  "annotations": [
    {
      "verse": 1,
      "title": "Annotation title",
      "text": "Commentary text.",
      "notes": [
        { "marker": "a", "text": "Sub-note text." }
      ]
    }
  ]
}
```

Not all books or chapters have annotation files. Coverage is heaviest in the
New Testament and major Old Testament books.

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

The 73 books of the Catholic canon, including the deuterocanonical books
(Tobit, Judith, 1 & 2 Maccabees, Wisdom, Sirach, Baruch).

---

## Related

- [Original Douay-Rheims Bible](https://odr.app) — web reader built on this data
