# Schema Reference

## bible/{raw,plain}/{book}.json

Top-level fields:

| Field | Type | Description |
|-------|------|-------------|
| `book` | string | Book slug (e.g. `genesis`, `matthew`) |
| `book_title` | string | Full title as printed in the original |
| `short_title` | string | Short form of the title |
| `hebrew_title` | string | Hebrew/Greek title where applicable |
| `intros` | array | Chapter introduction objects (see below) |
| `chapters` | array | Chapter objects (see below) |

### Chapter object

| Field | Type | Description |
|-------|------|-------------|
| `chapter` | number | Chapter number |
| `verses` | array | Verse objects (see below) |

### Verse object

| Field | Type | Description |
|-------|------|-------------|
| `verse` | number | Verse number |
| `text` | string | Verse text |
| `notes` | array? | Inline footnotes attached to this verse (optional) |

### Note object (inline)

| Field | Type | Description |
|-------|------|-------------|
| `marker` | number \| string | Footnote marker symbol |
| `text` | string | Footnote text |

### Markup tags (`raw` only)

| Tag | Preserved in `clean`? | Meaning |
|-----|-----------------------|---------|
| `<sc>Word</sc>` | content only | Small caps |
| `<i>word</i>` | content only | Supplied word (absent from Latin source) |
| `<cr>†</cr>` | removed | Cross-reference marker |
| `<na>*</na>` | removed | Footnote anchor |
| `<mn>[1]</mn>` | removed | Marginal note number |

---

## annotations/{book}/{chapter}.json

Files are named with zero-padded chapter numbers (e.g. `001.json`, `012.json`).

| Field | Type | Description |
|-------|------|-------------|
| `book` | string | Book slug |
| `chapter` | number | Chapter number |
| `annotations` | array | Annotation objects (see below) |

### Annotation object

| Field | Type | Description |
|-------|------|-------------|
| `verse` | number | Verse this annotation is anchored to |
| `title` | string? | Short title for the annotation |
| `text` | string? | Main annotation body text |
| `notes` | array? | Sub-notes with markers (see below) |

### Annotation sub-note object

| Field | Type | Description |
|-------|------|-------------|
| `marker` | string \| number | Reference marker (e.g. `a`, `b`, `1`) |
| `text` | string | Sub-note text |

---

## reference/{ot,nt}/{document}.json

Each reference document has a `section`, `title`, and one or more content arrays
depending on the document type.

### Documents with `paragraphs`

Most documents (prefaces, approbations, glossaries, etc.) use this shape:

| Field | Type | Description |
|-------|------|-------------|
| `section` | string | Internal identifier |
| `title` | string | Document title |
| `subtitle` | string? | Optional subtitle |
| `paragraphs` | array | `{ "text": "..." }` objects |

### Documents with `subsections`

`nt/scripture-authority.json` uses numbered subsections, each with its own
`heading` and `paragraphs` array.

### Documents with `entries`

Glossaries and word-explication files (`ot/glossary.json`,
`nt/explication-words.json`) use `entries` arrays of
`{ "term": "...", "definition": "..." }` objects.

### Documents with `articles`

`nt/apostles-creed.json` uses an `articles` array of creed articles.

### Table documents

`nt/table-corruptions.json`, `nt/table-paul.json`, `nt/table-peter.json`,
`nt/evangelical-history.json` etc. use document-specific shapes;
see the individual files.

---

## Reference document index

### Old Testament (`reference/ot/`)

| File | Description |
|------|-------------|
| `title-page.json` | Title page of the 1609 Old Testament |
| `approbatio.json` | Approbation by the censors |
| `censura.json` | Censure of the three English theologians |
| `preface.json` | Preface to the reader |
| `privilege.json` | Royal privilege |
| `epistles-table.json` | Table of epistles from the Old Testament |
| `glossary.json` | Glossary of principal subjects |
| `historical-table-age-1.json` — `historical-table-age-6.json` | Historical tables of the six ages of the world |

### New Testament (`reference/nt/`)

| File | Description |
|------|-------------|
| `title-page.json` | Title page of the 1582 New Testament |
| `censure.json` | Censure and approbation |
| `preface.json` | Preface to the reader |
| `scripture-authority.json` | On the authority of Holy Scripture |
| `apostles-creed.json` | The Apostles' Creed with commentary |
| `evangelical-history.json` | Sum and order of the Evangelical history |
| `explication-words.json` | Explication of certain words in the translation |
| `table-catholic-truths.json` | Table of Catholic truths |
| `table-corruptions.json` | Table of corruptions in Protestant translations |
| `table-epistles-gospels.json` | Table of epistles and gospels for the liturgy |
| `table-paul.json` | Table of St Paul |
| `table-peter.json` | Table of St Peter |
