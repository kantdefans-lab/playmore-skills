---
name: cnki-download
description: Resolve an openable full-text link from a CNKI paper page (PDF preferred). Use when user wants a direct link to open, not local downloading.
argument-hint: "[paper URL or blank if already on detail page]"
---

# CNKI Open Link Resolver

## Prerequisites

This skill is link-first. Do not click download buttons.

## Arguments

`$ARGUMENTS` is optionally a paper detail URL. If blank, uses current page.

## Steps

### 1. Navigate (if URL provided)

If URL is provided, use `navigate_page` to open it directly.

Do not click result links from tables to avoid extra tab-management overhead.

### 2. Extract title + openable link (single async evaluate_script)

```javascript
async () => {
  await new Promise((resolve, reject) => {
    let n = 0;
    const check = () => {
      if (document.querySelector('.brief h1')) resolve();
      else if (++n > 30) reject(new Error('timeout'));
      else setTimeout(check, 500);
    };
    check();
  });

  const captcha = document.querySelector('#tcaptcha_transform_dy');
  if (captcha && captcha.getBoundingClientRect().top >= 0) {
    return {
      error: 'captcha',
      message: 'CNKI is showing slider captcha. Please solve it in Chrome and retry.'
    };
  }

  const title = document.querySelector('.brief h1')
    ?.innerText
    ?.trim()
    ?.replace(/\s*网络首发\s*$/, '') || '';

  const notLogged = document.querySelector('.downloadlink.icon-notlogged')
    || document.querySelector('[class*="notlogged"]');
  if (notLogged) {
    return {
      error: 'not_logged_in',
      title,
      message: 'CNKI login is required before a valid full-text link can be returned.'
    };
  }

  const pdfLink = document.querySelector('#pdfDown') || document.querySelector('.btn-dlpdf a');
  const cajLink = document.querySelector('#cajDown') || document.querySelector('.btn-dlcaj a');

  const candidates = [
    { url: pdfLink?.href || '', link_kind: 'pdf' },
    { url: cajLink?.href || '', link_kind: 'caj' }
  ].filter(x => !!x.url && /^https?:\/\//i.test(x.url));

  if (candidates.length === 0) {
    return {
      error: 'no_open_link',
      title,
      message: 'No openable PDF/CAJ link found on this page.'
    };
  }

  const best = candidates[0];
  return {
    title,
    open_url: best.url,
    source: 'cnki',
    link_kind: best.link_kind
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

- `captcha`: ask user to solve captcha manually
- `not_logged_in`: ask user to log in first
- `no_open_link`: state no valid full-text link is available on this page

## Tool calls

1-2 (`navigate_page` if URL + `evaluate_script`)

## Verified selectors

| Element | Selector | Notes |
|---------|----------|-------|
| PDF link | `#pdfDown` | preferred |
| CAJ link | `#cajDown` | fallback |
| Not logged in | `.downloadlink.icon-notlogged` | |
| Title | `.brief h1` | strip trailing `网络首发` |

## Captcha detection

Check `#tcaptcha_transform_dy` element's `getBoundingClientRect().top >= 0`.
Only active when `top >= 0` (visible).
