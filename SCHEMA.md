# Schema

The Original Douay-Rheims Bible (1582/1610), exported as structured data.

Everything here derives from one corpus. The JSON trees are canonical and
lossless; the USFM trees are a projection of that corpus into a format that
cannot express all of it. Where the two disagree, the JSON is right.

## Output tree

```
manifest.json                    schema version, generated date, source commit, counts
SCHEMA.md                        this file
LICENSE                          CC0 1.0
bible/tagged/<book>.json         CANONICAL — inline markup preserved
bible/raw/<book>.json            derived — markup stripped, apparatus kept structured
usfm/<NN-BBB>.usfm               derived — \f and \x only
usfm-study/<NN-BBB>.usfm         derived — the same, plus annotations as \ef
annotations/<book>/<NNN>.json    1,677 annotations in 397 files
reference/{ot,nt}/*.json         26 documents of prefatory matter
index/lemmas/<book>.json         derived — catchword spans into the tagged text
```

## Counts

| Quantity | Value |
|---|---|
| Books | 76 |
| Chapters | 1,361 |
| Verses in the corpus | 37,180 |
| Verse entries in the JSON trees | 37,131 |
| `\v` lines emitted in USFM | 37,130 |
| Annotations | 1,677 (in 397 files) |
| Annotation sub-notes | 3,609 |
| Reference documents | 26 |

The three differ for two reasons. 49 fragments the corpus numbers as verse 0
are not scripture and are folded into the summary they complete (see *Verse 0*),
which accounts for the JSON trees holding 37,131 entries. The USFM drops one
more because the duplicate verse in 3 Esdras 2 is emitted as a segment rather
than a second `\v` line. `manifest.json` reports the corpus figure as `verses`
and the USFM figure as `usfmVerses`.

## Book file shape

```
{ book, book_title, short_title, hebrew_title, intros[], chapters[], endMatters[] }

intros[]      { title, text, notes[] }              notes[]: { marker, text }
endMatters[]  { title, text, notes[] }              notes[]: { marker, text }
chapters[]    { chapter, verses[], summary, summary_notes[], summary_continuation, articles[] }
  summary_notes[]        { marker, text }
  summary_continuation   { text, notes[], cross_refs[] }   — 49 chapters, see Verse 0
  articles[]       { title, text, notes[] }         notes[]: { marker, text }
verses[]      { verse, text, has_annotation, cross_refs[], notes[] }
  cross_refs[]  { text }
  notes[]       { label, text }
```

**Two note vocabularies coexist.** Verse notes key on `label` (a string). Every
other notes array keys on `marker` (a number, or the string `"◦"`). Do not
assume one.

`endMatters` appears on four books: `2-machabees`, `acts`, `job`, `psalms`. It
is shaped like `intros[]` and carries its own markup and notes.

`book_title` / `short_title` / `hebrew_title` are absent on the three appendix
books (`3-esdras`, `4-esdras`, `prayer-of-manasses`), which carry `version_abbr`
and `date` instead.

`has_annotation` is a derived boolean living inline on the verse. It is an
inconsistency kept for compatibility with earlier releases.

### annotations/{book}/{chapter}.json

```
{ chapter, annotations[ { verse, part, title, text, notes[] } ] }
notes[]  { marker, text }
```

Annotations attach at verse granularity through an exact `verse` field. `part`
distinguishes several annotations on one verse. `title` is the catchword: the
phrase from the verse the annotation comments on.

## Inline markup

`bible/tagged/` and `annotations/` preserve the source markup. The complete
vocabulary is nine tags. None carries attributes.

| Tag | Meaning | Resolves against |
|---|---|---|
| `<i>` | italic | — |
| `<na>` | note marker | verse `notes[]` by `label`; `summary_notes[]` by `marker` |
| `<mn>` | marginal note marker | that block's `notes[]` by `marker` |
| `<cr>` | cross-reference marker | verse `cross_refs[]` |
| `<sc>` | small caps | — |
| `<alt>` | the span a marginal variant applies to | the adjacent marker |
| `<br>` | paragraph break — **void, never closed** | — |
| `<col-left>`, `<col-right>` | two-column layout (once, in the Romans 9 annotation) | — |

Every tag but `<br>` is balanced.

**`<na>` and `<mn>` are not interchangeable.** No verse contains `<mn>`; no
intro, article, or annotation contains `<na>`. `<na>` is the verse-and-summary
marker; `<mn>` is the marker used in prose apparatus.

**The markup does not recurse.** No marker tag ever appears inside a note's own
text; note bodies contain `<i>` and nothing else. The apparatus is exactly two
levels deep, so a tokenizer needs no recursion.

### Marker tokens

Markers are not all `[n]`. Three forms occur:

| Form | Example | Binds to |
|---|---|---|
| bracketed number | `<na>[1]</na>` | `label` / `marker` `"1"` |
| parenthesised letter | `<na>(a)</na>` | `label` / `marker` `"a"` |
| ring | `<mn>◦</mn>` | positional (see below) |

One tag may carry several tokens: `<na>(c)[1]</na>` is two markers at one
position, binding to notes `"c"` and `"1"`. This occurs 33 times. Parse tag
content as a *sequence* of tokens, never as a single label.

Index arithmetic of the form `<cr>[n]</cr> → cross_refs[n-1]` is wrong for the
lettered form. Do not use it.

### Resolution rule

> Within one text, the **k-th occurrence** of token `t` binds to the **k-th
> entry** of that block's notes array whose `label` / `marker` equals `t`. A `◦`
> token matching no entry binds instead to the next not-yet-consumed note in
> array order.

The `◦` fallback is needed because one array can hold many notes all marked
`"◦"`. This rule binds 13,606 of the corpus's 13,608 markers. The two failures
are source defects, listed at the end.

### `<alt>`

`<alt>` marks the printed span a marginal variant applies to; the note supplies
the alternative reading:

```
are you not <na>[1]</na> <alt>men</alt>?     notes: [{ label: "1", text: "<i>carnal</i>" }]
```

New Testament only: 106 spans across 21 books. It binds to the **nearest
adjacent marker on either side**, and that marker may be a `<cr>`. Of the 106,
101 follow an `<na>`, 4 follow a `<cr>`, and 1 precedes its `<na>`. A rule that
only looks backwards for `<na>` misses five cases.

## bible/raw/

The same structure with all markup stripped from every text field. Notes and
cross-references are kept as structured data, so nothing is lost except the
*position* of each marker within its text. Use `bible/tagged/` if you need
marker placement.

## index/lemmas/

Keyed by `"chapter:verse"`, each value an array of tuples:

```json
{ "1:1": [[0, 67, 2], [0, 29, 1]] }
```

`[start, length, part]` — a character offset and length into the verse text,
**markup included**, plus the `part` of the annotation whose catchword the span
locates.

Two constraints:

- **The offsets are valid only against `bible/tagged/`.** They index the raw
  text with tags in place. Applying them to `bible/raw/` gives the wrong span.
- **The spans come from fuzzy matching.** Annotation catchwords quote the verse
  loosely, so the resolver works down five tiers from exact match to partial.
  The tier is reported at build time but is *not* recorded in this file, so a
  consumer cannot tell an exact span from a partial one. Treat every span as a
  best-effort anchor for highlighting, not an authoritative citation.

Spans are sorted by start offset, outermost first where two begin together, so
a containing highlight opens before the one nested inside it.

This index is kept out of the verse objects because it is derived, it is coupled
to markup byte offsets (any tag edit silently invalidates it), and inlining a
fuzzy result would present it as fact.

## reference/{ot,nt}/{document}.json

Each reference document has a `section`, a `title`, and one or more content
arrays depending on its type.

### Documents with `paragraphs`

Most documents (prefaces, approbations, and the like):

| Field | Type | Description |
|---|---|---|
| `section` | string | Internal identifier |
| `title` | string | Document title |
| `subtitle` | string? | Optional subtitle |
| `paragraphs` | array | `{ "text": "..." }` objects |

### Other shapes

| Shape | Files |
|---|---|
| `subsections`, each with `heading` and `paragraphs` | `nt/scripture-authority.json` |
| `entries` of `{ term, definition }` | `ot/glossary.json`, `nt/explication-words.json` |
| `articles` | `nt/apostles-creed.json` |
| document-specific tables | `nt/table-corruptions.json`, `nt/table-paul.json`, `nt/table-peter.json`, `nt/evangelical-history.json` |

### Document index

Old Testament (`reference/ot/`):

| File | Description |
|---|---|
| `title-page.json` | Title page of the 1609 Old Testament |
| `approbatio.json` | Approbation by the censors |
| `censura.json` | Censure of the three English theologians |
| `preface.json` | Preface to the reader |
| `privilege.json` | Royal privilege |
| `epistles-table.json` | Table of epistles from the Old Testament |
| `glossary.json` | Glossary of principal subjects |
| `historical-table-age-1.json` … `historical-table-age-6.json` | Historical tables of the six ages of the world (age 3 has a second part, `-age-3b`) |

New Testament (`reference/nt/`):

| File | Description |
|---|---|
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

## USFM

### File naming

`<NN>-<CODE>.usfm`, where **`NN` is the book's position in the ODR canon
(01–76), not its Paratext number.** Paratext numbers the deuterocanon from 68
upward, which would sort Tobias, Judith, Wisdom, Ecclesiasticus, Baruch, and the
Machabees after Revelation. That is not the order the ODR prints. The Paratext
number is recorded in `manifest.json` for consumers who want it.

Under this scheme the three appendix books land at 47–49, between Malachie and
Matthew, which is where the ODR puts them.

### Why two trees

`usfm/` is text plus `\f` and `\x`. `usfm-study/` adds the 1,677 annotations as
`\ef` study notes. Folding the annotations into the plain files would force
existing consumers to strip 3,609 note blocks to get back what they had, and
roughly triples the file size. Both trees come from one render call differing by
a flag, so they cannot diverge.

### Mapping

| Source | USFM |
|---|---|
| `book_title` | `\h`, `\toc1-3`, `\mt1` |
| `intros[]` | `\is` + `\ip` |
| `endMatters[]` | `\is` + `\ip`, after the last chapter |
| `chapters[].summary` | `\cd` |
| `summary_notes[]` | `\f - \ft …\f*` on the `\cd` |
| `chapters[].articles` | `\is` heading + `\ip` |
| `<sc>…</sc>` | `\sc …\sc*`, or `\+sc …\+sc*` when nested |
| `<i>…</i>` | `\it …\it*`, or `\+it …\+it*` when nested |
| `<br>`, and `\n\n` in prose | end the `\ip`, open a new one |
| `<col-left>`, `<col-right>` | flattened to sequential text |
| `<cr>` + its `cross_refs` entry | `\x - \xt …\x*` inline at the marker |
| verse `notes[]` | `\f <label> \fr c.v \ft …\f*`, reusing the original token |
| `<alt>` | `\fq` inside the bound footnote, or `\xq` where it anchors to a `<cr>` |
| annotations (`usfm-study/`) | `\ef - \fr c.v \fq <title> \ft <text>\ef*` |

**Character-marker nesting.** A note is a character environment, so any
character marker opened inside one takes the `\+` prefix on both its opening and
closing form. The same applies to a marker opened inside another marker. Body
text takes the bare form.

```usfm
are you not \f 1 \fr 3.4 \fq men \ft \+it carnal\+it*\f* ?
```

`\eft` does not exist. USFM 3.0 deliberately avoided minting parallel content
markers for study notes: `\ef` reuses `\fr`, `\fq`, `\fk`, and `\ft`.

### Annotation sub-notes are flattened

**USFM notes cannot nest.** The specification permits nested *character* markers
inside a note via `\+`, but there is no legal way to place an `\f` or `\ef`
inside another.

The corpus has two levels of apparatus: 3,609 sub-notes inside 1,677 annotation
texts. Rather than misrepresent this, the export flattens it explicitly. Each
`<mn>` marker becomes a Unicode superscript at its original character position,
and the sub-notes follow as a trailing `\fq` / `\ft` run inside the same `\ef`:

```usfm
\ef - \fr 1.1: \fq In the beginning. \ft Holy Moyses telleth what was done¹
… he could not believe the Gospel² … \fq ¹ \ft S. Aug. l. 11. de Gen. ad lit.
c. 4. \fq ² \ft Contra Epist. Fund. c. 5.\ef*
```

The superscript comes from the note's ordinal within its annotation, not from
the marker token, because the token may be `[1]`, `(a)`, or `◦`, and `◦` carries
no number to render.

Position, text, and association survive. Structural containment does not: a
parser sees one flat note. **`annotations/` is the canonical form.**

### Known deviations

The 1582/1610 apparatus is richer than USFM 3 can express. Four constructs are
emitted knowingly, because every alternative alters the text. Counts are against
`usfm/`.

| Construct | Count | Why it stands |
|---|---|---|
| `\f` inside an introduction `\ip` | 684, in 63 books | The introduction content model admits no note, and USFM 3 offers no sanctioned way to footnote an introduction (`\iex` shares the model). The printed source genuinely footnotes its introductions. Inlining would alter the text; dropping would lose it. |
| `\f` inside a chapter article `\ip` | 589, in 10 books | Same construct, same reasoning, in the articles between chapters. |
| `\f` inside `\cd` | 220 | A parser makes the note a sibling of the `cd` paragraph rather than its content, so a consumer may render the chapter description without its notes. The alternative is dropping the association entirely. |
| A `\sc` or `\it` span opened in body text and closed after `\f*` / `\x*` | 13 lines | The source italicises a phrase a note interrupts. Tolerated by the grammar, but a character environment straddling a note boundary is not something every renderer reconstructs. |

Two further points. `\cd` is a single line, so a chapter summary's notes trail
the description rather than sitting at their markers: marker *position* within a
summary is the one thing the USFM does not carry, and the JSON keeps it. And
prose paragraph breaks are lost inside notes, where USFM has no legal paragraph
marker; they survive in intro, article, and end-matter prose as real `\ip`
paragraphs.

## Book codes

### The composite-book rule

Four books in the ODR are ones USFM also publishes in split form. One rule
settles all of them:

> **Ship the composite the source actually has.** Where the corpus presents one
> book, emit one file under the code denoting that composite. Never split a book
> into pieces USFM offers for translations that print them separately, because
> that invents a structure the ODR does not have.

| Book | ODR chapters | What the extra chapters are | Code | Rejected |
|---|---|---|---|---|
| `esther` | 16 | 11–16 are the Greek additions | `ESG` | `EST` (Hebrew Esther) |
| `daniel` | 14 | 13 is Susanna, 14 is Bel | `DAG` | `DAN` (Hebrew Daniel) |
| `baruch` | 6 | 6 is the Letter of Jeremiah | `BAR` | `BAR` + `LJE` split |
| `4-esdras` | 16 | 1–2 preface, 15–16 conclusion | `2ES` | `EZA` + `5EZ` + `6EZ` split |

### The Esdras family

The ODR follows Vulgate numbering, so its four Esdras books are not the four a
modern reader expects. The USFM registry names the Vulgate equivalences
directly, which decides every case:

- `1ES` — "The 9 chapter book of Greek Ezra in the LXX … called '3 Esdras' in the Vulgate"
- `2ES` — "The 16 chapter book of Latin Esdras … called '4 Esdras' in the Vulgate"
- `EZR` — "for Hebrew Ezra, sometimes called 1 Ezra or 1 Esdras"
- `NEH` — "called 2 Esdras in the Vulgate"

**`EZA` / `5EZ` / `6EZ` are rejected.** They are not an alternative spelling of
the same book. `EZA` is the 12-chapter Ezra Apocalypse alone, `5EZ` the
2-chapter Latin preface, `6EZ` the 2-chapter Latin conclusion — three codes for
translations that publish the pieces separately. The corpus does not:
`4-esdras` is one 16-chapter book whose chapter 1 opens the preface, chapter 3
the apocalypse, and chapter 15 the conclusion. That composite is what `2ES`
denotes.

### Full table

| NN | slug | USFM | Paratext | Chapters |
|---|---|---|---|---|
| 01 | genesis | GEN | 01 | 50 |
| 02 | exodus | EXO | 02 | 40 |
| 03 | leviticus | LEV | 03 | 27 |
| 04 | numbers | NUM | 04 | 36 |
| 05 | deuteronomy | DEU | 05 | 34 |
| 06 | josue | JOS | 06 | 24 |
| 07 | judges | JDG | 07 | 21 |
| 08 | ruth | RUT | 08 | 4 |
| 09 | 1-kings | 1SA | 09 | 31 |
| 10 | 2-kings | 2SA | 10 | 24 |
| 11 | 3-kings | 1KI | 11 | 22 |
| 12 | 4-kings | 2KI | 12 | 25 |
| 13 | 1-paralipomenon | 1CH | 13 | 29 |
| 14 | 2-paralipomenon | 2CH | 14 | 36 |
| 15 | 1-esdras | EZR | 15 | 10 |
| 16 | 2-esdras | NEH | 16 | 13 |
| 17 | tobias | TOB | 68 | 15 |
| 18 | judith | JDT | 69 | 16 |
| 19 | esther | ESG | 70 | 16 |
| 20 | 1-machabees | 1MA | 78 | 16 |
| 21 | 2-machabees | 2MA | 79 | 15 |
| 22 | job | JOB | 18 | 42 |
| 23 | psalms | PSA | 19 | 150 |
| 24 | proverbs | PRO | 20 | 31 |
| 25 | ecclesiastes | ECC | 21 | 12 |
| 26 | canticle-of-canticles | SNG | 22 | 8 |
| 27 | wisdom | WIS | 71 | 19 |
| 28 | ecclesiasticus | SIR | 72 | 51 |
| 29 | isaie | ISA | 23 | 66 |
| 30 | jeremie | JER | 24 | 52 |
| 31 | lamentations | LAM | 25 | 5 |
| 32 | baruch | BAR | 73 | 6 |
| 33 | ezechiel | EZK | 26 | 48 |
| 34 | daniel | DAG | 27 | 14 |
| 35 | osee | HOS | 28 | 14 |
| 36 | joel | JOL | 29 | 3 |
| 37 | amos | AMO | 30 | 9 |
| 38 | abdias | OBA | 31 | 1 |
| 39 | jonas | JON | 32 | 4 |
| 40 | micheas | MIC | 33 | 7 |
| 41 | nahum | NAM | 34 | 3 |
| 42 | habacuc | HAB | 35 | 3 |
| 43 | sophonias | ZEP | 36 | 3 |
| 44 | aggeus | HAG | 37 | 2 |
| 45 | zacharias | ZEC | 38 | 14 |
| 46 | malachie | MAL | 39 | 4 |
| 47 | prayer-of-manasses | MAN | 84 | 1 |
| 48 | 3-esdras | 1ES | 82 | 9 |
| 49 | 4-esdras | 2ES | 83 | 16 |
| 50 | matthew | MAT | 41 | 28 |
| 51 | mark | MRK | 42 | 16 |
| 52 | luke | LUK | 43 | 24 |
| 53 | john | JHN | 44 | 21 |
| 54 | acts | ACT | 45 | 28 |
| 55 | romans | ROM | 46 | 16 |
| 56 | 1-corinthians | 1CO | 47 | 16 |
| 57 | 2-corinthians | 2CO | 48 | 13 |
| 58 | galatians | GAL | 49 | 6 |
| 59 | ephesians | EPH | 50 | 6 |
| 60 | philippians | PHP | 51 | 4 |
| 61 | colossians | COL | 52 | 4 |
| 62 | 1-thessalonians | 1TH | 53 | 5 |
| 63 | 2-thessalonians | 2TH | 54 | 3 |
| 64 | 1-timothy | 1TI | 55 | 6 |
| 65 | 2-timothy | 2TI | 56 | 4 |
| 66 | titus | TIT | 57 | 3 |
| 67 | philemon | PHM | 58 | 1 |
| 68 | hebrews | HEB | 59 | 13 |
| 69 | james | JAS | 60 | 5 |
| 70 | 1-peter | 1PE | 61 | 5 |
| 71 | 2-peter | 2PE | 62 | 3 |
| 72 | 1-john | 1JN | 63 | 5 |
| 73 | 2-john | 2JN | 64 | 1 |
| 74 | 3-john | 3JN | 65 | 1 |
| 75 | jude | JUD | 66 | 1 |
| 76 | apocalypse | REV | 67 | 22 |

## Known source defects

These are defects in the corpus, which the export only reads. It neither repairs
nor silently drops them: each is pinned by an inventory in the build, and
anything outside those inventories is a fatal error. A new unbound marker means
the corpus changed and the export stops.

### Markers that cannot bind

| Ref | Problem |
|---|---|
| 1 Timothy 2:6 | marker `[1]` printed twice against a single note |
| Ecclesiasticus 14:10 | a `†` marker, and the verse has no `notes` array at all |

### Notes never referenced

26 notes are never referenced from their text, among them `1-john 4:21`,
`john 21:25`, `matthew intro[1]`, `romans intro[1]`, and `acts ann 8:38`.

Together with the two above, these 28 cases are the complete known set. 13,632
notes less the 26 unreferenced leaves 13,606; 13,608 markers less the 2 that
cannot bind leaves the same 13,606. Every note and every marker is accounted for.

### The duplicate verse in 3 Esdras 2

`3-esdras` chapter 2 numbers two entries `1`: two readings of one verse,
differing in capitalisation, the second trailing malformed cross-reference
residue. It is the only repeated verse number in the corpus.

Which reading is canonical is an editorial judgment about scripture, so the
export makes no choice: the first keeps `\v 1`, the second becomes `\v 1b`.
**The segment letter records a corpus defect. It is not a claim that the printed
text divides this verse.**

### Verse 0

49 chapters across 27 books hold a verse numbered 0. It is never scripture: it
is the tail of a chapter summary that ran past its field, finishing the
summary's sentence.

```
numbers 25  summary: "…Phinees his zeal in stabbing to death two fornicators is commended"
            verse 0: "by God, and rewarded."
```

USFM verse numbers start at 1, so emitting these as `\v 0` would be invalid and
would claim the editor's summary is text of the Bible. Each fragment is appended
to the `\cd` it continues, and the chapter's verse loop skips it.

12 of the 49 sit in a chapter with no `summary` at all, so the fragment becomes
the whole `\cd`. 10 carry notes, and the Tobias preface carries five
cross-references; those travel with the words in the trailing form `\cd` already
gives summary notes.

### How the JSON trees carry them

The fragment's words join the `summary` they complete, so `summary` reads as
the whole sentence and the 12 orphan chapters have one at last. The fragment
also keeps its own entry:

```json
"summary": "Phinees his zeal … is commended by God, and rewarded.",
"summary_continuation": { "text": "by God, and rewarded.", "notes": [ … ] }
```

`verses[]` therefore contains only scripture, and **no verse is numbered 0 in
this bundle.**

`summary_continuation` exists for two reasons. Ten fragments carry a note and
the Tobias preface carries five cross-references, so merging the words alone
would silently drop 15 pieces of apparatus. And keeping the fragment separate
preserves the seam, so a consumer can still tell which words overran the field.

The apparatus is deliberately *not* converted into `summary_notes`. Verse notes
key on `label` and summary notes on `marker`; moving one vocabulary into the
other risks binding a fragment's note to an unrelated summary note carrying the
same token. The continuation keeps its arrays in their original keying, so read
`summary_continuation.notes` with the verse rules, not the summary rules.

## License

CC0 1.0 Universal. Public domain. Copy, share, adapt, and redistribute for any
purpose, with no restrictions and no attribution required.
