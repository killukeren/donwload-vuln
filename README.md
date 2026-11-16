# Zen Browser — download filename origin spoof

PoC for a UI misrepresentation bug in Zen Browser 1.24.2b.

The download list renders the page-controlled filename as the entry title. The title uses
`text-overflow: ellipsis`, which hides the tail of the string. Unicode lookalikes for `:`
and `/` survive Firefox's filename sanitization, so a page can make a downloaded `.apk`
appear to come from `google.com`.

## Payload

```js
const name = "https\u2236\u2215\u2215google.com" + "\u2800".repeat(40) + ".apk";
```

- `U+2236` RATIO — looks like `:`
- `U+2215` DIVISION SLASH — looks like `/`
- `U+2800` BRAILLE PATTERN BLANK — invisible filler, pushes `.apk` past the ellipsis

## Result

| | |
|---|---|
| Saved filename | `https∶∕∕google.com` + 40×U+2800 + `.apk` |
| Visible title | `https∶∕∕google.com` + blanks |
| Hidden | `.apk` |

Firefox 160 is not affected by this variant: its download item uses `crop="center"`, so the
extension stays visible.

## Affected code

- `src/zen/library/ZenLibraryWidget.sys.mjs` — `#fileName()`, title assignment
- `src/zen/common/styles/zen-library-widget.css` — `text-overflow: ellipsis` on
  `.zen-library-download-list-title`

## Live demo

https://killukeren.github.io/donwload-vuln/
