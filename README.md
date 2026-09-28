# QR Code Generator

> A single-file, dependency-free HTML document that generates QR codes entirely in the browser.

![HTML](https://img.shields.io/badge/HTML-single--file-E34F26?logo=html5&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-vanilla-F7DF1E?logo=javascript&logoColor=black)
![Dependencies](https://img.shields.io/badge/dependencies-none-brightgreen)
![Runs](https://img.shields.io/badge/runs-100%25%20offline-blueviolet)
![QR Spec](https://img.shields.io/badge/ISO%2FIEC-18004-informational)
![Versions](https://img.shields.io/badge/QR%20versions-1%E2%80%9340-blue)
![EC Levels](https://img.shields.io/badge/EC%20levels-L%20%7C%20M%20%7C%20Q%20%7C%20H-orange)
![Output](https://img.shields.io/badge/output-PNG%20%7C%20SVG-success)

---

## Overview

Drop the file into any browser. Type something, pick a data type, choose a background, hit **Small PNG**, **Large PNG**, or **SVG**. Nothing is uploaded, no build step, no CDN, no library.

The encoder implements ISO/IEC 18004 from scratch: Galois-field Reed–Solomon, BCH format/version codes, all eight mask patterns with penalty scoring, and the full version 1–40 table.

---

## Features

| | |
|---|---|
| **Single file** | One `.html` file. Open it, or host it anywhere. |
| **Offline** | No network calls of any kind. |
| **Three input types** | Numeric, URL, Alphanumeric — each with its own validation. |
| **Whitespace normalization** | Optional cleanup of pasted text (Unicode spaces, tabs, zero-width characters, stray edges). |
| **Live preview** | Updates as you type; shows version, module size, mask, encoding mode. |
| **Two raster sizes** | 256 × 256 px icon, or 1024 × 1024 px image. |
| **Vector SVG** | Resolution-independent, editable as text, tiny file. |
| **White or transparent** | Solid white background, or fully transparent for compositing. |
| **Auto version** | Picks the smallest QR version that fits your data. |
| **Auto mode** | Chooses numeric / alphanumeric / byte encoding per your input. |
| **Auto mask** | Tries all 8 masks, keeps the lowest-penalty one. |

---

## Usage

1. Open the HTML file in a browser.
2. Pick a **data type**: `Numeric`, `URL`, or `Alphanumeric`.
3. Type your content. The preview and status message update live.
4. Leave **Normalize whitespace** on unless you specifically need exact input preservation (see below).
5. Adjust the **error correction level** if you need to (see below).
6. Choose **White** or **Transparent** background.
7. Click **Small PNG**, **Large PNG**, or **SVG**.

The character counter under the textarea shows `used / max` for the current data type and EC level. If you exceed the maximum, the counter turns red and the download buttons disable. When normalization changes your input, a note appears under the counter listing what was adjusted; it fades after a few seconds.

---

## Input Types

Each type applies its own validation rules before encoding. The validation is about *what you intended*, not about what QR can technically hold — the encoder will happily encode anything the validation lets through.

### Numeric
**Allowed:** digits `0-9`, space, hyphen `-`, underscore `_`.

**Why the extra characters?** They're excluded from the compact numeric encoding, but people type phone numbers, order IDs, and part numbers with separators. The separators are preserved in the QR content exactly as typed — they are not stripped.

**Encoding note:** If the input is *purely* digits, the compact numeric mode is used (3 digits per 10 bits). If it contains any separator, the encoder falls back to alphanumeric or byte mode, which is denser. A 20-digit numeric string and a 20-character string with one hyphen can need different QR versions. A warning appears when this happens.

### URL
**Allowed:** any string with a scheme (`https://…`, `mailto:…`, `ftp://…`) or a bare domain (`example.com`, `www.example.com/path?q=1`).

**No spaces.** A URL with whitespace is rejected — this is almost always a typo.

**Encoding note:** URLs are encoded as byte mode (UTF-8). Case matters and is preserved. `Example.com` and `example.com` are different strings and produce different QR codes. You do not need the `https://` prefix for the QR to work in most scanners, but including it is more reliable across older apps.

### Alphanumeric
**Allowed:** any printable ASCII (`0x20`–`0x7E`), including letters, digits, punctuation, and spaces.

**Encoding note:** This is a misnomer inherited from the QR spec. The *alphanumeric encoding mode* only covers 45 specific characters (`0-9 A-Z space $ % * + - . / :`) and is case-sensitive — lowercase letters force byte mode. So typing `HELLO WORLD` uses the compact 45-character mode, while `Hello World` falls back to byte mode and produces a denser, larger QR for the same visible text.

**Tip:** If your content is case-insensitive — a short code, a coupon ID, a licence plate — typing it in uppercase produces a measurably smaller QR code.

---

## Whitespace Normalization

Pasting text from chats, docs, and web pages is unreliable. Smart quotes, non-breaking spaces, tabs, and invisible zero-width characters are all common and all silently change what gets encoded — producing a QR code that scans correctly but contains the wrong string. Normalization fixes this before encoding.

**What it does, in order:**

| Rule | Effect |
|---|---|
| Unicode space → regular space | `U+00A0`, `U+2000–U+200A`, `U+202F`, `U+205F`, `U+3000` become `U+0020` |
| Tab → space | `\t` becomes a single space |
| Zero-width removal | `U+200B–U+200D`, `U+2060`, `U+FEFF` are deleted |
| Line-ending normalization | `\r\n` and `\r` become `\n` |
| Trim (on blur only) | Leading and trailing whitespace is removed |

**When it runs:** Character-level rules run on every keystroke, so a stray non-breaking space from a paste is corrected immediately. Trimming runs on blur, so typing a trailing space isn't aggressively deleted mid-edit. All rules are skipped if the checkbox is off.

**Cursor behaviour:** All rules except zero-width removal are 1:1 character substitutions, so the caret doesn't move. Zero-width removal shortens the string; the caret is shifted to account for the characters removed before it, so mid-string editing is not disrupted.

**International input:** Normalization is suspended during IME composition, so typing CJK or other composition-based scripts is not disturbed.

**When to turn it off:** If the exact byte sequence matters — e.g. you're encoding a cryptographic token, a base64 payload, or content with intentional leading/trailing whitespace — disable the checkbox. The default is on because for typical use it prevents a whole class of silent failures.

---

## Output Formats

### Small PNG (256 × 256)
An icon-sized raster. Useful for app UI, favicons, avatars, and small on-screen embeds. At this size, high-version QR codes (30+) may have 1-pixel modules and become unreliable for camera scanning — the preview metadata line shows the version, so you can check.

### Large PNG (1024 × 1024)
The general-purpose raster. Comfortable for print at a few centimetres, and for on-screen use at any reasonable size. Same content and quiet zone as the small PNG, just more pixels.

### SVG
A vector file, resolution-independent, and typically a few kilobytes. The `viewBox` is measured in module units (each module is exactly 1 unit, plus the quiet zone), so the file scales to any size without loss and prints crisply at any output resolution.

Horizontal runs of dark modules are merged into single path segments before writing, which keeps the file compact for structured patterns — finder squares, timing patterns, and alignment patterns all collapse well.

The SVG declares a default `width="512" height="512"`, but you can override either attribute (or CSS `width`/`height`) to render at any size. Use it in HTML with an `<img>` tag, inline it for CSS-stylable colour control, or open it in Illustrator / Inkscape / Figma to edit.

**Which to pick:**
- On-screen icon, fixed small size → Small PNG
- On-screen display, moderate size → Large PNG
- Print, posters, scaling, further editing → SVG
- Compositing onto a coloured background → any format with Transparent enabled

---

## Background Options

### White
A solid `#ffffff` rectangle behind the QR. The safest choice — most scanners assume a light background and dark modules, and this is what every QR image on the web uses.

### Transparent
No background fill. The modules (black squares) are drawn on an empty canvas or SVG. Useful for:
- Placing the QR on a coloured card, poster, or UI where a white box would look wrong
- Overlaying on photography or illustrations
- Compositing into a larger layout programmatically

**Caveat:** Scanners need contrast between the modules and whatever is behind them. A transparent QR placed on a dark background may not scan at all. If you're using transparency for stylistic reasons, test the final composited result with a real scanner before committing it.

The preview shows a checkerboard pattern behind transparent output, so you can see exactly which pixels are empty.

---

## Encoding Nuances

These are the details that actually change the output. Worth reading once.

### Encoding mode is chosen automatically, and it matters

QR has four encoding modes. The encoder picks the most compact one that fits your data:

| Mode | Covers | Bits per character |
|---|---|---|
| Numeric | `0-9` only | ~3.33 (10 bits per 3 digits) |
| Alphanumeric | 45 chars: `0-9`, `A-Z`, `space`, `$ % * + - . / :` | ~5.5 (11 bits per 2 chars) |
| Byte | Any bytes (UTF-8 here) | 8 |
| Kanji | Shift-JIS double-byte | ~13 |

A 40-digit number fits in QR version 1 (21×21 modules). The same length in lowercase letters needs a much larger version. **Same visible length, very different QR size.**

### The Numeric input type does not guarantee numeric encoding

The type selection is a *validator*, not a mode lock. If you pick Numeric and type `123-456`, the validator accepts it, but the encoder falls back to alphanumeric mode because of the hyphen. You'll see a yellow warning in the status area. If QR size matters, keep it digits-only.

### Byte mode counts UTF-8 bytes, not characters

If you paste non-ASCII text (accented letters, emoji, CJK), byte mode encodes UTF-8. `é` is 2 bytes. An emoji is often 4 bytes. The character counter shows characters, but the limit is checked against bytes — so an "80 character" string of emoji can be over the limit.

### Version is chosen as the smallest that fits — there is no override

The QR version (1–40, meaning 21×21 up to 177×177 modules) is derived from your data length and EC level. You cannot force version 10. This keeps the code simple and always produces a scannable result.

### The quiet zone is baked in

Every generated QR — PNG or SVG — includes the spec-mandated 4-module quiet zone on all four sides. If you drop the PNG onto a dark background without padding, the code may not scan — add a white margin or use a light background. This applies to transparent output too.

### PNGs are exactly the stated pixel size

The PNG buttons produce square files at exactly 256 × 256 or 1024 × 1024 pixels. The QR is centred, and the module size is the largest whole-pixel integer that fits. Very large QRs (version 30+) at 256 px will have 1-pixel modules and may be unreliable for camera scanning — use the large download or SVG for those.

### The SVG is measured in modules, not pixels

The `viewBox` is `0 0 N N` where `N = modules + 8` (the quiet zone). Rendering the SVG at 200 px or 2000 px produces identical geometry with no quality loss. If you edit the file by hand, changing `width` and `height` on the root element does not affect the geometry — only the `viewBox` does.

---

## Error Correction Level

QR codes carry redundant data so they can survive damage. The EC level controls the trade-off between **resilience** and **capacity**.

| Level | Recovers up to | Relative capacity | Best for |
|---|---|---|---|
| **L** | ~7% | 100% (baseline) | Clean digital display, short content, max data density |
| **M** | ~15% | ~85% | General use — the default. Good balance. |
| **Q** | ~25% | ~70% | Print, stickers, industrial labels |
| **H** | ~30% | ~55% | Harsh environments, logo overlays, partial occlusion |

**Why it matters for you:** Higher EC levels mean *less room for your data*, which means a *larger QR version* for the same content. A 100-character string fits in version 5 at L, but needs version 7 at H. On screen this is invisible; at 256 px download size it can mean the difference between 8-pixel modules and 4-pixel modules.

**Practical defaults:**
- Screen-only, in-app deep links → **L** or **M**
- Printed flyers, business cards → **M**
- Stickers, packaging, anything that might get scratched → **Q**
- Centre has a logo, or the code may be partially obscured → **H**

All four levels respect the correct capacity for every version 1–40.

---

## How It Works

Short version of the pipeline:

1. **Normalize** (optional) — clean Unicode spaces, tabs, zero-width characters, line endings, and edges.
2. **Detect mode** from the input string (numeric / alnum / byte).
3. **Pick version** — smallest v in 1–40 whose data capacity holds the header, mode/count indicator, and payload.
4. **Build bitstream** — mode indicator (4 bits), character count (10–16 bits depending on mode and version), payload, terminator, byte-align padding, alternating pad bytes `0xEC 0x11`.
5. **Split into RS blocks** per the version's block structure and compute Reed–Solomon EC codewords over GF(256) with primitive polynomial `0x11D`.
6. **Interleave** data codewords across blocks, then EC codewords across blocks.
7. **Place modules** — finder patterns, separators, alignment patterns, timing patterns, dark module, format-info reservation, version-info reservation (v ≥ 7), then data in the standard zig-zag.
8. **Try all 8 masks**, write format and version bits for each, score with the four penalty rules, keep the lowest.
9. **Render** to canvas (PNG) or to an SVG string, with the chosen background and the quiet zone.

---

## Browser Support

Any browser with `canvas`, `Blob`, and `URL.createObjectURL` — Chrome, Firefox, Safari, and Edge from roughly 2015 onward. No polyfills needed. IME composition support is feature-detected via the `isComposing` flag on input events, which is universal in modern browsers.

---

## Limitations

- **No Kanji mode.** Non-ASCII characters go through byte mode (UTF-8), which works but is less compact than true Shift-JIS Kanji mode.
- **No structured append.** Very long content must fit in a single version-40 symbol.
- **No custom size input.** PNG output is fixed at 256 px and 1024 px. For any other size, use the SVG.
- **No logo / centre image overlay.** Add it downstream if you need it — and use EC level Q or H when you do.
- **No multi-colour output.** Modules are always black; the background is white or transparent.
- **Normalization is not configurable per-rule.** It's all-or-nothing via the checkbox. If you need, say, tabs preserved but zero-width characters stripped, edit the `normalizeWhitespace` function.

---

