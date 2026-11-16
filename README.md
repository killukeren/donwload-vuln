# Poc Download Spoofing

Zen Browser 1.24.2b — the download list shows a page-controlled source URL and hides the
real extension.

## Payload

```js
const a = document.createElement("a");
a.href = "data:,https://chrome.google.com";
a.download = "chrome-pro.exe";
a.click();
```

The row title is `chrome-pro.exe`. Hovering it replaces the status line with
`data:,https://chrome.google.com`, so the source reads as a trusted Chrome URL.

## Why it works

- `ZenLibraryDownloadsSection.mjs` renders the source URL with `#formatUrl()`, which only
  strips `https://` and `www.` — a `data:` URL is shown verbatim.
- `zen-library.css` hides `.zen-library-download-url` and shows it on `:hover`, replacing
  the status line.
- The title uses `text-overflow: ellipsis`, so a long filename loses its tail (the real
  extension).

Firefox 160 does not do this: its download item uses `crop="center"` and shows `Open File`
on hover.

## Live demo

https://killukeren.github.io/donwload-vuln/
