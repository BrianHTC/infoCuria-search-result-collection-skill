# InfoCuria Search Collector

This personal VS Code Copilot skill automates collection of InfoCuria search-result documents as validated English PDFs organized by case.

## Use it when

- Saving an InfoCuria/CURIA search result as PDFs
- Collecting Judgments, Opinions, Applications, Requests for a preliminary ruling, Abstracts, or related documents
- Checking a collection for missing, duplicated, contaminated, or placeholder files
- Reorganizing or renaming case folders

Invoke it with `/infocuria-search-collector`, or describe an InfoCuria collection task in chat.

## Required inputs

- The open InfoCuria search page, including its language and filters
- The CSV downloaded from InfoCuria's **Download results** toolbar menu
- An output directory, or permission to use the default `Downloads\InfoCuria`

Before starting the collection, click **Download results** above the results and choose the CSV option. Use that downloaded file as the authoritative manifest; do not copy visible result rows or save the browser page as a substitute.

The CSV is the authoritative manifest. The skill preserves duplicate-date rows and does not treat visible virtualized result cards as a reliable document identity.

## Safety guarantees

The workflow:

1. Resolves each CSV row to document-specific InfoCuria metadata and source URLs.
2. Downloads into a timestamped staging directory rather than directly into the final tree.
3. Validates PDF readability, extracted text, case identity, document type, and expected row mapping.
4. Builds and audits a complete destination tree before changing the live output.
5. Promotes the destination using a timestamped backup and reversible directory-level moves.
6. Re-audits the promoted tree and retains the backup until verification succeeds.

It stops with a failure report when a row is missing, duplicated, low-content, unreadable, or associated with the wrong case. It does not merge partially renamed folders or delete the only copy of a file.

## Naming

Windows-safe folder names use hyphens instead of slashes:

```text
C-182-24 - SACD and Others. 05-03-2024 - 18-12-2025
```

The original case number and CSV values remain available in the generated manifest.

## Completion criteria

A collection is complete only when the final audit confirms:

- Exact CSV row count
- Exact distinct case count
- One PDF for every manifest row
- No extra or empty case folders
- Valid PDF text and matching case identity for every document
- Checksums and staged/final mappings agree

See [`SKILL.md`](./SKILL.md) for the full operational procedure and recovery steps.
