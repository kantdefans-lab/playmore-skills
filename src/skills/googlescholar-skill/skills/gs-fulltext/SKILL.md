---
name: gs-fulltext
description: Resolve an openable full-text link for a Google Scholar result. Return title + direct open URL only.
argument-hint: "[data-cid or result number from previous search]"
---

# Google Scholar Open Link Resolver

Resolve one openable full-text link for a target Scholar result.

## Arguments

`$ARGUMENTS` can be:

- A `data-cid` from a previous search result
- A result number (for the current page context)

## Behavior

- Prefer the right-column direct full-text link (`.gs_ggs a` / `.gs_or_ggsm a`)
- Fallback to publisher link in title (`.gs_rt a`)
- Do not return DOI-only links
- Do not return Sci-Hub links

## Steps

### 1. Ensure result page context

If not already on a relevant Scholar result page, run `gs-search` first.

### 2. Extract title + openable URL (evaluate_script)

```javascript
async () => {
  const cid = "DATA_CID_HERE";

  const item = document.querySelector(`.gs_r.gs_or.gs_scl[data-cid="${cid}"]`);
  if (!item) {
    return { error: 'not_found', message: 'Paper not found on current page. Search again first.' };
  }

  const titleEl = item.querySelector('.gs_rt a');
  const title = titleEl?.textContent?.trim()
    || item.querySelector('.gs_rt')?.textContent?.trim()
    || '';

  const rightLink = item.querySelector('.gs_ggs a') || item.querySelector('.gs_or_ggsm a');
  const fullTextUrl = rightLink?.href || '';
  const fullTextType = rightLink?.querySelector('span.gs_ctg2')?.textContent?.trim() || '';

  const publisherUrl = titleEl?.href || '';

  const isHttp = (u) => /^https?:\/\//i.test(u || '');
  const isDoi = (u) => /(^https?:\/\/)?(dx\.)?doi\.org\//i.test(u || '');

  let openUrl = '';
  let linkKind = '';

  if (isHttp(fullTextUrl) && !isDoi(fullTextUrl)) {
    openUrl = fullTextUrl;
    linkKind = (fullTextType || '').toLowerCase().includes('pdf') ? 'pdf' : 'html';
  } else if (isHttp(publisherUrl) && !isDoi(publisherUrl)) {
    openUrl = publisherUrl;
    linkKind = 'detail';
  }

  if (!openUrl) {
    return {
      error: 'no_open_link',
      title,
      message: 'No non-DOI openable full-text link found for this result.'
    };
  }

  return {
    title,
    open_url: openUrl,
    source: 'scholar',
    link_kind: linkKind
  };
}
```

### 3. Report

If success:

```
1) {title}
   链接: {open_url}
```

If error:

- `not_found`: paper not in current page context
- `no_open_link`: no valid non-DOI full-text link available

## Tool calls

1-2 (`evaluate_script` only, or `navigate_page` + `evaluate_script`)

## Notes

- `data-cid` remains the canonical selector key
- This skill is link-first and intentionally does not output DOI/Sci-Hub
