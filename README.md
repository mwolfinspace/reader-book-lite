# Reader Book Lite

A streamlined ebook reader plugin for SiYuan Notes, forked from [SiReader](https://github.com/mika-si/siyuan-sireader).

Supports PDF, EPUB, MOBI, and TXT formats with annotations, bookmarks, dictionary, translation, TTS, and reading themes.

## Features

- All SiReader features — unobstructed (no license/activation gates)
- English-only UI (no Chinese strings)
- Default translation target: Vietnamese (Tiếng Việt) — switchable per session
- Community-maintained fork

## Installation

1. Download the latest release from [Releases](https://github.com/mwolfinspace/reader-book-lite/releases)
2. In SiYuan, go to `Settings → Plugins → Import plugin` and select the downloaded `.zip`
3. Enable the plugin

## Usage

Open any supported ebook file (PDF, EPUB, MOBI, TXT) from SiYuan's file tree. The reader toolbar provides:

- **Highlight/underline/outline** text annotations
- **Dictionary** lookup (long-press or select text)
- **Translation** — default target is Vietnamese, changeable in the translate panel
- **Bookmarks** and **navigation** via table of contents
- **TTS** read-aloud
- **Reading themes** (day/night/sepia)

## Changes from SiReader

| Change | Detail |
|--------|--------|
| License gating | Removed — all features free |
| Activation/upgrade prompts | Removed |
| Membership settings section | Hidden (binding preserved) |
| UI language | English-only (~637 string replacements) |
| Default translation | Vietnamese (`vi`) instead of Chinese (`zh-CN`) |
| Plugin name/version | `reader-book-lite` v1.0.0 |

## Offline Dictionaries

Reader Book Lite supports **StarDict** offline dictionaries. You can download and import your own dictionaries to look up words without internet access.

### Compatible Dictionary Sources

These sources provide StarDict `.ifo`/`.idx`/`.dict.dz` files that work with the plugin:

| Source | Languages | Notes |
|--------|-----------|-------|
| [tudien](https://github.com/redphx/tudien) | English ↔ Vietnamese | Modern dictionaries, HTML definitions |
| [OVDP stardict-dict](https://github.com/lpdevs/stardict-dict) | Vietnamese ↔ English, Vietnamese ↔ Chinese, English ↔ Chinese | Plain-text format, large word counts |
| [StarDict archive](https://web.archive.org/web/20231001000000/http://abloz.com/huzheng/stardict-dic/) | Many language pairs | classic StarDict collection |
| [FreeDict](https://freedict.org/) | Many language pairs | Open-source dictionary collection |
| [Book Reader Dict](https://github.com/BoboTiG/ebook-reader-dict) | Wiktionary-based, many languages | XML format (convert to StarDict) |

> For Vietnamese users, the recommended dictionaries are:
> - **tudien-en-vi** (EN→VI, 235K words) — best quality
> - **stardict_en_vi** (EN→VI, 387K words) — larger word count
> - **stardict_vi_en** (VI→EN, 42K words) — reverse lookup

### How to Import

1. **Download** a StarDict dictionary. It should contain 3 files with the same base name:
   - `your-dict.ifo` — metadata (required)
   - `your-dict.idx` — word index (required)
   - `your-dict.dict.dz` — compressed definition data (required)

   > Some dictionaries ship as `.dict` (uncompressed). The plugin does **not** accept `.dict` files directly. If your dictionary has only `.dict`, you need to convert it first (see Troubleshooting below).

2. **Open** SiYuan → Reader Book Lite reader → select any text → click the Dictionary button → click the **settings gear** in the dictionary panel, or go to **Plugin settings → Reader Book Lite → Dictionary**.

3. Click **Add Dictionary**.

4. In the file picker, **select all 3 files at once** (hold Ctrl or Shift and click each file). You must select `.ifo`, `.idx`, and `.dict.dz` simultaneously.

5. The dictionary is now imported and ready to use immediately — no restart needed. It will appear in the dictionary tab bar when you next look up a word.

### Dictionary Order

Dictionaries are queried in the order they appear in the list. The first dictionary that returns a result wins. You can reorder them by clicking **Sort** in the dictionary settings panel.

### Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| "No complete dictionary group found" | You didn't select all 3 files, or the filenames don't match | Select `.ifo`, `.idx`, and `.dict.dz` **at the same time** — all must share the same base name |
| "Subfield ID should be RA" | The `.dict.dz` is plain gzip, not proper DictZip format | Use a dictionary that ships with `.dict.dz` (not `.dict`), or convert using the script below |
| "File not found" | File path in config is incorrect | Use the **Add Dictionary** UI to import — do not place files manually |
| Dictionary not showing in lookup | Dict is disabled or no results found | Check that the dictionary is enabled (toggle switch) and the word exists in that dictionary |

### Converting .dict to .dict.dz (Advanced)

If your dictionary ships as `.dict` (uncompressed), you need to encode it as a proper **DictZip** file (not plain gzip). Use this Node.js script:

```javascript
const fs = require('fs');
const zlib = require('zlib');

function makeDictZip(inputPath, outputPath) {
    const raw = fs.readFileSync(inputPath);
    const CHLEN = 8192;
    const chunks = [];
    for (let offset = 0; offset < raw.length; offset += CHLEN) {
        const chunk = raw.subarray(offset, Math.min(offset + CHLEN, raw.length));
        chunks.push(zlib.deflateRawSync(chunk, { level: 9 }));
    }
    const subfieldDataLen = 6 + 2 * chunks.length;
    const xlen = 4 + subfieldDataLen;
    const extra = Buffer.alloc(xlen);
    let off = 0;
    extra[off++] = 0x52; extra[off++] = 0x41; // "RA"
    extra.writeUInt16LE(subfieldDataLen, off); off += 2;
    extra.writeUInt16LE(1, off); off += 2;     // version
    extra.writeUInt16LE(CHLEN, off); off += 2; // chunk length
    extra.writeUInt16LE(chunks.length, off); off += 2; // chunk count
    for (const c of chunks) extra.writeUInt16LE(c.length, off); off += 2;

    const name = outputPath.split(/[\\/]/).pop().replace(/\.dz$/, '') || 'dict';
    const fname = Buffer.from(name, 'utf-8');
    const header = Buffer.alloc(12 + xlen + fname.length + 1);
    off = 0;
    header[off++] = 0x1F; header[off++] = 0x8B; header[off++] = 0x08;
    header[off++] = 0x0C; // FEXTRA | FNAME
    off += 4; // mtime = 0
    header[off++] = 0; header[off++] = 0; // xfl, os
    header.writeUInt16LE(xlen, off); off += 2;
    extra.copy(header, off); off += extra.length;
    fname.copy(header, off); off += fname.length;
    header[off] = 0;
    fs.writeFileSync(outputPath, Buffer.concat([header, ...chunks]));
}

makeDictZip('input.dict', 'output.dict.dz');
```

Run: `node convert.js`

## Credits

Original plugin: [SiReader](https://github.com/mika-si/siyuan-sireader) by mika-si.
