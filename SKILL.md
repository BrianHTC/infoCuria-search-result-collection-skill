---
name: infocuria-search-collector
description: "Collect, validate, and organize InfoCuria search results as PDFs by case. Use when downloading InfoCuria/CURIA search results, saving Judgments, Opinions, Applications, Requests for a preliminary ruling, Abstracts, or other case documents, checking missing or contaminated files, or renaming case folders."
argument-hint: "InfoCuria search page, CSV export, or case-result collection to download and verify"
user-invocable: true
---

# InfoCuria Search Collector

Reliably collect the documents represented by an InfoCuria search result, save them as readable English PDFs, and organize them by case without losing files during renaming.

## Non-negotiable safety rules

- Treat the CSV export as the authoritative manifest. Do not infer completeness from visible result cards or folder counts alone.
- Never download by repeatedly clicking virtualized result cards. Obtain document-specific metadata (`case`, `procedure`, `logicDocId`, document type, date, and format) first.
- Never rename the only copy of a case folder in place. Build a complete destination tree in a sibling staging directory, validate it, then promote it with a reversible two-phase move.
- Never delete or merge a source folder until the destination manifest, file hashes, document counts, and PDF text checks all match.
- A `%PDF` header is not sufficient validation. Extract text and reject landing pages, placeholders, server-error pages, unrelated case content, and very-low-content PDFs.
- Use Windows-safe folder names: replace `/` in case numbers and dates with `-`, and sanitize other forbidden characters (`< > : " / \ | ? *`).

## Workflow

### 1. Capture the search manifest

1. Use the shared InfoCuria browser page and confirm the query, language (`EN`), date range, sort, and procedure filters.
2. Export the result CSV and preserve it unchanged.
3. Parse every row, including duplicate document dates. The manifest key is:
   `case number + document type + document date + document title/ECLI`.
4. Record the expected total row count and distinct case count before downloading.
5. Create a machine-readable manifest containing the CSV row, expected case metadata, and a status field. Do not use folder names as identifiers.

### 2. Resolve document-specific sources

Use InfoCuria metadata/search responses to resolve each manifest row to its exact document:

- `affId`, for example `C/0135/23/00000000RP/01`
- `procedure`, for example `C/0135/23/00000000RP/01/P/01`
- `logicDocId` and its numeric ID
- document type code, date, language, and format

Prefer the document-specific PDF route observed in the application:

`https://infocuria.curia.europa.eu/document/{jurisdiction}-{number4}-{year}-{year2}-00000000RP-01-P-01-{logicDocNumericId}-1-EN.pdf`

For HTML documents, retrieve the matching InfoCuria HTML source and print that source to PDF. Do not substitute an adjacent document with the same case or date.

### 3. Download into an immutable staging tree

Create a unique staging directory beside the final output, for example:

`InfoCuria.staging.<timestamp>\`

Write one PDF per manifest row using a deterministic temporary filename containing a stable row ID. Keep the original source URL, HTTP status, byte count, SHA-256, and extracted text length in the manifest.

If a request returns an HTTP error, an HTML error page, or a PDF with insufficient text, mark that row failed and retry using the browser/application route. Do not create a success-shaped placeholder.

### 4. Validate every document before organization

For each staged PDF:

- Open it with a PDF parser.
- Confirm it has at least one page and meaningful extracted text.
- Confirm the text contains the expected case number in a normalized form.
- Confirm the expected document type or title markers where available.
- For judgments/opinions/applications, check the ECLI or title when the CSV provides one.
- Detect cross-case contamination by comparing extracted case identifiers with the manifest row.
- Preserve duplicate-date documents as separate rows; never deduplicate by date alone.

Stop the workflow if any row is missing, duplicated, low-content, unreadable, or assigned to the wrong case. Produce a failure report listing the row ID and reason.

### 5. Build the final tree without destructive renames

1. Derive the final folder name from CSV metadata:
   `C-182-24 - SACD and Others. 05-03-2024 - 18-12-2025`
2. Keep the slash-free Windows form only for the filesystem name; preserve the original case number in the manifest and PDF filename metadata.
3. Create all final folders under a second sibling tree, for example:
   `InfoCuria.final.<timestamp>\`.
4. Copy or move validated staged PDFs into their final folders with deterministic numbered names ordered by document date, then stable manifest row ID.
5. Write a `manifest.json` and `checksums.sha256` at the staging/final root.
6. Validate the final tree against the CSV before promotion:
   - exact expected row count
   - exact distinct case count
   - one final PDF per manifest row
   - no extra PDFs
   - no empty case folders
   - all hashes and extracted-text checks match

### 6. Promote atomically and verify

Do not rename individual live case folders while the collection is incomplete. Instead:

1. Rename the existing output directory to a timestamped backup name, only after the complete final tree is ready.
2. Rename the validated final tree to the requested output name.
3. Re-run the full manifest audit against the promoted path.
4. Keep the backup until the audit passes and the user has a recoverable copy.
5. If promotion or audit fails, restore the backup; never merge partially renamed folders into the new tree.

## Recovery procedure

If a previous run was interrupted:

1. Stop immediately; do not merge folders based on case-name prefixes.
2. Re-read the CSV and reconstruct the expected manifest.
3. Hash and inspect every existing PDF; assign it to a manifest row only when content and metadata agree.
4. Treat ambiguous, duplicate, or contaminated PDFs as invalid and regenerate them from document-specific sources.
5. Rebuild a clean sibling tree from the manifest, then use the promotion procedure above.

## Completion report

Report the CSV row count, case count, staged/downloaded/validated counts, failed rows, final folder count, final PDF count, and whether the backup was retained. Include the output folder path and any rows requiring user action. Do not claim completion from a file-count check alone.
