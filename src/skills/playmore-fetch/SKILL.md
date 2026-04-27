---
name: playmore-fetch
description: Fetch academic papers from Google Scholar, CNKI, OpenAlex, and Semantic Scholar. Deduplicates by DOI/title. English sources first, then Chinese. Called by playmore.
argument-hint: "[search keywords]"
---

# Playmore Fetch

Search all four sources for $ARGUMENTS. English sources first, then Chinese.

## Step 0 — Expand keywords using researcher profile

If `~/Desktop/playmore/profile.md` exists, read the keyword weight table.

Take the top 5 keywords by weight (weight ≥ 3) and merge them with $ARGUMENTS:
- If a profile keyword is semantically related to $ARGUMENTS, add it to the search query
- If a profile keyword is unrelated, skip it (don't pollute the search)

Example: user searches "平台经济"，profile has "数字经济(8), 劳动力市场(7), 零工经济(5)"
→ Expanded query: "平台经济 数字经济 零工经济" (劳动力市场 is too broad, skip)

Use the expanded query for OpenAlex and Semantic Scholar. Use original $ARGUMENTS for Google Scholar and CNKI (to avoid over-broad results from browser-based search).

---

## Step 1 — OpenAlex API (English)

```
GET https://api.openalex.org/works?search={EXPANDED_KEYWORDS}&sort=relevance_score:desc&per-page=20&filter=language:en
```

Extract per result:
- `title`, `doi`, `publication_year`, `primary_location.source.display_name` (journal)
- `primary_location.source.is_oa`, `primary_location.landing_page_url`, `open_access.oa_url`
- `cited_by_count`, `authorships[].author.display_name`
- `abstract_inverted_index` → reconstruct abstract
- `primary_location.source.type`
- Journal tier: check `primary_location.source.host_organization_name`

Source label: `OpenAlex`

## Step 2 — Semantic Scholar API (English)

```
GET https://api.semanticscholar.org/graph/v1/paper/search?query={EXPANDED_KEYWORDS}&limit=20&fields=title,authors,year,abstract,externalIds,journal,citationCount,openAccessPdf,publicationVenue
```

Extract per result:
- `title`, `year`, `abstract`, `citationCount`
- `authors[].name`
- `journal.name` or `publicationVenue.name`
- `externalIds.DOI`, `externalIds.ArXiv`
- `openAccessPdf.url` (PDF link if available)

Source label: `Semantic Scholar`

## Step 3 — Google Scholar (English)

Invoke `/gs-search $ARGUMENTS` and collect results.

For each result extract:
- title, authors, journalYear, citedBy, fullTextUrl, dataCid, snippet

Source label: `Google Scholar`

## Step 4 — CNKI (Chinese)

Invoke `/cnki-search $ARGUMENTS` (translate keywords to Chinese if needed) and collect results.

For each result extract:
- title, authors, journal, year, source link, download link

Source label: `CNKI`

## Step 5 — Deduplicate

Merge all results into one list. Deduplicate by:
1. Exact DOI match
2. Normalized title similarity (>90% match = same paper)

Keep the entry with the most complete metadata. Note all source platforms in a `sources` array.

## Step 6 — Output

Return a unified JSON list. Each item:

```json
{
  "title": "",
  "authors": [],
  "year": "",
  "journal": "",
  "abstract": "",
  "keywords": [],
  "doi": "",
  "arxiv_id": "",
  "pdf_url": "",
  "landing_url": "",
  "cited_by": 0,
  "sources": ["OpenAlex", "Google Scholar"],
  "raw_snippet": ""
}
```

Pass this list to `/playmore-score`.
