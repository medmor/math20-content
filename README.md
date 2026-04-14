# math20-content

Moroccan math curriculum content CDN. Each lesson is a self-contained HTML file organized by grade, track, and semester.

## Structure

```
chapters/
  tc-s/              Tronc Commun Sciences
  tc-l/              Tronc Commun Lettres
  1-bac/             1ère Baccalauréat
    sef/             Sciences Expérimentales Françaises (BIOF)
    seg/             Sciences Économiques et Gestion
    sh-l/            Sciences Humaines - Lettres
    sm/              Sciences Mathématiques
  2-bac/             2ème Baccalauréat
    eco/             Économie
    pc-svt/          Physique-Chimie / SVT
    sm-f/            Sciences Mathématiques Françaises (BIOF)
images/              Lesson images (diagrams, figures)
```

Each track folder follows the pattern:

```
chapters/<level>/<track>/mathematiques/semestre-<N>/<NN>-<slug>.html
```

Example: `chapters/1-bac/sef/mathematiques/semestre-1/01-notions-de-logique.html`

## HTML format

Each lesson file is plain HTML (no doctype, no head/body tags — just fragment HTML):

```html
<!-- Chapter title -->
<h1>Chapter title</h1>

<h2>I. Section title</h2>
<h3>1-1. Subsection</h3>
<p>Content with <strong>bold terms</strong> and Unicode math symbols: ∀, ∃, ∈, ⇒, ⇔, ¬, ∧, ∨, ≡</p>
<ul>
  <li>List items</li>
</ul>
<table border="1" cellpadding="5" cellspacing="0">
  <tr><th>P</th><th>Q</th></tr>
  <tr><td>V</td><td>F</td></tr>
</table>
```

### Conventions

- Files start with `<!-- Title -->` comment then `<h1>Title</h1>`
- Sections use Roman numerals: `<h2>I.`, `<h2>II.`, etc.
- Subsections use numbered dashes: `<h3>1-1.`, `<h3>2-3.`, etc.
- Math notation uses Unicode symbols, not LaTeX
- Tables use `border="1" cellpadding="5" cellspacing="0"`
- Exercise sections are numbered: `<h3>Exercice 1</h3>`, `<h3>Exercice 2</h3>`, etc.
- Proof endings use `□`

## How to create a chapter

### Prerequisites

- `ollama` with `qwen3-vl:235b-instruct-cloud` model
- `gs` (Ghostscript) for PDF-to-image conversion
- `curl` for downloading PDFs

### Step 1: Find the source on AlloSchool

Navigate the AlloSchool hierarchy for BIOF (French international) courses:

1. Go to https://www.alloschool.com/category/high-school
2. Select the level (Tronc Commun → Sciences, 1ère Bac → Sciences Expérimentales, etc.)
3. Look for the **(BIOF)** French version of the math course
4. If no BIOF version exists (e.g., TC-L), use the Arabic version — the math content is the same

AlloSchool URLs follow this pattern:
- Course page: `https://www.alloschool.com/course/<course-slug>`
- Chapter page: `https://www.alloschool.com/section/<id>`
- PDF link: `https://www.alloschool.com/element/<id>/pdf`

The **premium** PDFs (listed under "Contenu Premium" sections) work without auth and contain the full lesson with exercises.

### Step 2: Download the PDF

```bash
mkdir -p /tmp/math20-pdfs
curl -L -o /tmp/math20-pdfs/<slug>.pdf "https://www.alloschool.com/element/<id>/pdf"
```

Verify it's a real PDF (sometimes alloschool returns HTML for auth-required PDFs):

```bash
file /tmp/math20-pdfs/<slug>.pdf
# Should say: PDF document, version X.Y, N page(s)
```

If the file is HTML instead of PDF, try the premium doc element ID from the course page.

### Step 3: Convert PDF pages to images

```bash
mkdir -p /tmp/math20-pdfs/pages
gs -dNOPAUSE -dBATCH -sDEVICE=png16m -r200 -sOutputFile=/tmp/math20-pdfs/pages/page_%d.png /tmp/math20-pdfs/<slug>.pdf
```

Verify the pages:

```bash
ls -la /tmp/math20-pdfs/pages/
```

### Step 4: Extract content using Ollama vision model

Process each page. The first page is usually a table of contents — skip it. For content pages, run:

```bash
ollama run qwen3-vl:235b-instruct-cloud "Read this scanned PDF page carefully. It is a French math lesson page from AlloSchool. Extract the EXACT text, including ALL mathematical formulas, definitions, theorems, examples, and proofs. Do NOT paraphrase — copy the text exactly. Output in clean HTML: h2/h3/h4 for headers, p for text, strong for bold, ul/ol/li for lists, table for tables. Use Unicode math symbols (∀, ∃, ∈, ⇒, ⇔, ¬, ∧, ∨, ≡). Output only raw HTML, no code fences or commentary." /tmp/math20-pdfs/pages/page_N.png
```

**Important**: Pass image files as arguments (not stdin). The model cannot read images piped via `<` stdin redirection — it must receive the file path as an argument.

Process all content pages and collect the output.

### Step 5: Assemble and write the HTML file

Combine the extracted content into a single HTML file following these rules:

1. Start with `<!-- Title -->` and `<h1>Title</h1>`
2. Match the section numbering from the original PDF (Roman numerals for main sections, dash-separated for subsections)
3. Remove duplicate headers/introductions that appear on each page
4. Ensure consistent HTML formatting across all pages
5. Use Unicode math symbols consistently
6. Include all examples, proofs, and exercises from the original
7. Do NOT add any HTML comments (except the title comment on line 1)

Write the file to:

```
chapters/<level>/<track>/mathematiques/semestre-<N>/<NN>-<slug>.html
```

Where `<NN>` is the chapter number (zero-padded: 01, 02, ...) matching the curriculum order.

### Step 6: Clean up temporary files

```bash
rm -rf /tmp/math20-pdfs
```

### Step 7: Verify

Check the file has real content (not just a stub):

```bash
wc -l chapters/<level>/<track>/mathematiques/semestre-<N>/<NN>-<slug>.html
```

A completed chapter should have 50+ lines. A stub has only 2 lines (`<!-- Title -->` + `<h1>Title</h1>`).

## Progress tracking

| Level | Track | Sem 1 | Sem 2 | Status |
|-------|-------|--------|--------|--------|
| TC-S  | Math  | 10/10  | 5/5   | ✅ Complete |
| TC-L  | Math  | 0/6    | 0/5   | ❌ All stubs |
| 1-BAC | SEF   | 1/6    | 0/6   | 🔄 In progress |
| 1-BAC | SEG   | 0/5    | 0/5   | ❌ All stubs |
| 1-BAC | SH-L  | 0/5    | 0/4   | ❌ All stubs |
| 1-BAC | SM    | 0/7    | 0/10  | ❌ All stubs |
| 2-BAC | ECO   | 0/5    | 0/4   | ❌ All stubs |
| 2-BAC | PC-SVT| 0/9    | 0/4   | ❌ All stubs |
| 2-BAC | SM-F  | 0/9    | 0/5   | ❌ All stubs |

## AlloSchool course URLs (BIOF French editions)

- TC-S: `https://www.alloschool.com/course/mathematiques-tronc-commun-sciences-biof`
- 1-BAC SEF: `https://www.alloschool.com/course/mathematiques-1er-bac-sciences-experimentales-biof`
- 2-BAC SM-F: `https://www.alloschool.com/course/mathematiques-2bac-sciences-mathematiques-biof`