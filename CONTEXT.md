# Reader Book Lite — Full Session Context

> This file contains the complete modification history for AI assistants to resume work. Last updated: 2026-06-14.

## Project Goal

Fork `siyuan-sireader` into `reader-book-lite` with:
- License gating removed (all features free, no purchase/activation/upgrade prompts)
- English-only UI (no visible Chinese strings)
- Default translation target: Vietnamese (`vi`) instead of Chinese (`zh-CN`)
- No Membership section in settings (replaced with hidden div preserving template ref binding)
- Plugin name: `reader-book-lite`, version 1.0.0

## Repository Structure

```
D:\Logging\data\plugins\reader-book-lite\
├── CONTEXT.md          ← This file — full session context for AI catch-up
├── README.md           ← User-facing documentation
├── i18n/               ← Localization files (all English)
├── sql.js              ← SQL utilities (unmodified)
├── icon.png            ← Plugin icon
├── index.css           ← Plugin styles (unmodified)
├── index.js            ← Main bundle — all patches applied
├── index.js.bak        ← Backup of current working state
├── plugin.json         ← Plugin configuration
├── preview.png         ← Preview image
```

## Source Repository

Clone at `D:\Logging\data\plugins\siyuan-sireader-source\` (original `siyuan-sireader`).

A non-minified build (`dist/index.js`, ~10.6 MB) with `vite.config.ts` `minify: false` preserves original function/class/variable names for reference mapping.

## All Modifications Applied

### 1. Core Files
- **plugin.json**: name → `reader-book-lite`, version → `1.0.0`, author → `community`, all i18n descriptions English, URL → `github.com/mwolfinspace/reader-book-lite`
- **i18n/\*.json** and **i18n/i18n/\*.json**: All entries set to English
- **README_zh_CN.md**: Deleted (UI is English-only)

### 2. License Gating Bypass (`index.js`)

All applied to minified class `j1`:

| Patch | Original | Patched |
|-------|----------|---------|
| `can()` bypass | `static can(f,c)` in class `j1` | `return!0` — returns `true` unconditionally |
| Activation popup | `_openLicenseContent` | `()=>{}` — no-op |
| Upgrade message | `showUpgrade` | `L=>{}` — no-op |

These bypass all 16 feature-tier checks in `$oe`. The `can()` function is `static can(f,c)` in class `j1` — changing it to `return!0` effectively unlocks everything.

### 3. Membership Section Hidden (`index.js`)

- **Original**: 3034-byte `O("ul",{ref_key:"licenseRef"...})` — rendered the full Membership settings panel with license activation UI
- **Patched**: `O("div",{ref_key:"licenseRef",ref:B,style:{display:"none"}})` — hidden `<div>` that preserves the `B` ref binding
- **Why**: Removing the element entirely caused `Cannot read properties of null (reading 'el')` Vue runtime errors because the template `ref:B` was null. The hidden div keeps the ref valid.

### 4. Default Translation Language

**Function-level** (5 translation functions in `index.js`):
- `e4e` (Google Translate): default `f` parameter → `"vi"`
- `i4e` (Azure Translate): default `f` parameter → `"vi"`
- `r4e` (Yandex Translate): default `f` parameter → `"vi"`
- `n4e` (AI Free Translate): default `f` parameter → `"vi"`
- `o4e` (AI Siyuan Translate): default `f` parameter → `"vi"`

**Component-level** (Translate component):
- `b=i0("zh-CN")` → `b=i0(window.__sireader_tgt||"vi")`
- Language options array: `["vi","Tiếng Việt"]` moved to first position (default selection)

**Engine names** (English):
- `"AI Translate (Free)"` instead of Chinese name
- `"AI Translate (Siyuan)"` instead of Chinese name
- AI Siyuan prompt in English

### 5. Target Language Persistence Fix

**Root cause**: Translate component in `MarkPanel.vue` uses `v-if` (`<Translate v-if="state.panel === 'translate'" />`) — component is destroyed/recreated each time panel opens, resetting `b` ref to initial value.

**Fix** (applied to `index.js`):
- Init: `b=i0("vi")` → `b=i0(window.__sireader_tgt||"vi")`
- On change: `onClick:te=>{b.value=H,Q.value=!1,L()}` → `onClick:te=>{window.__sireader_tgt=H,b.value=H,Q.value=!1,L()}`

Uses `window.__sireader_tgt` global to persist across component mounts for the session duration. Language resets to `"vi"` default on page reload.

### 6. English-Only UI (~637 string replacements)

All Chinese UI strings replaced with English equivalents across the minified bundle. Includes:

| Component | Strings changed |
|-----------|----------------|
| Bookshelf | `"最近阅读"→"Recent"`, `"我的书架"→"Bookshelf"`, `"上传书籍"→"Upload Books"`, `"阅读进度"→"Reading Progress"`, `"序言"→"Foreword"`, `"全部"→"All"`, etc. |
| MarkPanel | `"标注"→"Mark"`, `"复制"→"Copy"`, `"词典"→"Dictionary"`, `"翻译"→"Translate"`, `"朗读"→"Read Aloud"`, `"发送到"→"Send To"`, `"笔记"→"Note"`, `"导入"→"Import"`, `"删除"→"Delete"`, `"无标题"→"Untitled"`, `"无内容"→"No Content"`, `"打开块"→"Open Block"`, etc. |
| Reader | `"加载更多..."→"Load More..."`, `"没有更多"→"No More"`, etc. |
| ReaderToc | `"目录"→"Table of Contents"` |
| Stats | `"阅读统计"→"Reading Stats"`, `"今日"→"Today"`, `"本周"→"This Week"`, `"本月"→"This Month"`, `"总阅读"→"Total Reading"`, `"分钟"→"min"`, `"已完成"→"Completed"`, `"阅读中"→"Reading"`, `"想看"→"Want to Read"` |
| TTSMini | `"倍速"→"Speed"`, `"语速"→"Speed"`, `"停止"→"Stop"`, `"播放"→"Play"`, `"暂停"→"Pause"` |
| OnlineReader | `"搜索"→"Search"`, `"搜索书籍..."→"Search books..."`, `"加载中"→"Loading"`, `"无结果"→"No Results"`, `"请稍候"→"Please wait"`, etc. |
| Settings | `"中文"→"Chinese"`, `"日语"→"Japanese"`, `"英语"→"English"`, `"法语"→"French"`, `"俄语"→"Russian"`, etc. |
| PdfToolbar | `"上一页"→"Previous Page"`, `"下一页"→"Next Page"`, `"页面调整"→"Page Adjustment"`, `"适合宽度"→"Fit Width"`, `"适合页面"→"Fit Page"`, `"搜索"→"Search"`, `"搜索文档"→"Search Document"`, `"搜索结果"→"Search Results"`, `"未找到结果"→"No results found"`, `"添加书签"→"Add Bookmark"`, `"书签"→"Bookmarks"`, `"形状"→"Shape"`, `"自由绘图"→"Free Draw"`, `"高亮"→"Highlight"`, `"下划线"→"Underline"`, `"删除线"→"Strikethrough"`, `"波浪线"→"Squiggly"`, etc. — all aria-label strings fixed to colon format |
| Translate | `"翻译"→"Translate"`, `"翻译中..."→"Translating..."`, `"翻译引擎"→"Translation Engine"`, `"翻译失败"→"Translate Failed"`, `"翻译结果"→"Translation Result"`, `"自动检测"→"Auto Detect"` |

**Page/Chapter pattern replacements** (5 template literal patterns):
- `"第"+W+"页"` → `"Page "+W` — for page number display
- `"第"+Z+"章"` → `"Chapter "+Z` — for chapter number display
- `"第"+(W+1)+"页"` → `"Page "+(W+1)` — for 0-indexed page numbers
- `` `第${x}页` `` → `` `Page ${x}` ``
- `` `第${s}部分` `` → `` `Part ${s}` ``

### 7. Remaining CJK Characters

~2081 CJK characters remain in `index.js`. These are exclusively **simplified ↔ traditional Chinese conversion data pairs** (e.g. `学學`, `汉漢`) — data, not visible UI. Only 1 non-pair string left: `零一二三四五六七八九十百千` (number parsing regex data in the EPUB reader).

## Key Technical Decisions

1. **Node.js `new Function()` for syntax validation**: Python's `compile()` compiles as Python, not JavaScript. Always use `node -e "new Function(fs.readFileSync(...))"` for JS syntax checking.

2. **Two-layer translation default**: Changed at both function-parameter level (`f="vi"` in translate functions) AND component-init level (`b=i0(...)`). Neither alone is sufficient — the component init overrides the function default on first call.

3. **Hidden div vs null ref**: Replacing the Membership `<ul>` with an empty `null` caused a Vue runtime error because the template's `ref:B` binding was broken. Using `<div style="display:none">` preserves the ref.

4. **Non-minified build for name mapping**: Single-letter minified names (`p`, `b`, `B`, `C`, etc.) are scope-reused thousands of times. Safe global rename via `replaceAll` is impossible. The non-minified build (`10.6 MB`, `minify: false`) maps names.

5. **Bulk string vs regex replacement**: Used Python `str.replace()` (not regex) for targeted string replacement in the minified bundle. Careful to only replace actual UI strings, not the conversion data pairs in lines 616+ of `all_chinese_context.txt`.

## Minified Name Mappings

| Minified name | Original name | Description |
|---------------|---------------|-------------|
| `e4e` | `translateGoogle` | Google Translate engine |
| `i4e` | `translateAzure` | Azure Translate engine |
| `r4e` | `translateYandex` | Yandex Translate engine |
| `n4e` | `translateAiFree` | AI Free Translate engine |
| `o4e` | `translateAiSiyuan` | AI Siyuan Translate engine |
| `vz` | `engines` | Translation engines array |
| `M3e` | `TranslatePanel` | Translate panel component |
| `U4e` | `SettingsPanel` | Settings panel component |
| `Y3e` | `MarkPanel` | Mark panel component |

## Critical Code Locations in `index.js`

| Location | What |
|----------|------|
| `can()` bypass | Class `j1`, `static can(f,c)` → `return!0` |
| Activation no-op | `_openLicenseContent` → `()=>{}` |
| Upgrade no-op | `showUpgrade` → `L=>{}` |
| Membership section | `O("ul",{ref_key:"licenseRef"...})` → hidden div |
| Target language init | `b=i0(window.__sireader_tgt||"vi")` |
| Target language persist | `onClick:te=>{window.__sireader_tgt=H,b.value=H,...}` |
| Source-combo target array | `[["vi","Tiếng Việt"],["en","English"],...` (first entry = default) |
| Translate functions (5) | Function signatures default `f` parameter to `"vi"` |
| Language options list | `[["auto","Auto Detect"],["zh-CN","Chinese"],["en","English"]...` |
| Engine names | `"AI Translate (Free)"` and `"AI Translate (Siyuan)"` |

## Backup

- `index.js.bak` — backup of current working state at `D:\Logging\data\plugins\reader-book-lite\`

## Known Issues

- Target language resets to `"vi"` on page/plugin reload (persists via `window.__sireader_tgt` global — session-only, not saved to SiYuan settings)
- Translation component unmounts/remounts on every panel toggle (inherent to `v-if` usage in source)
- Remaining ~2081 CJK characters are conversion data pairs and should NOT be modified

## Bug Fixes

### PDF Text Select Mode Not Working on First Open

**Root cause**: `PdfToolbar.vue` function `applyMode()` has an early-return guard (`if (toolMode.value === mode) return`) that skips `applyContainerMode()` when the mode hasn't changed. Since `toolMode` ref initializes to `'text'` and default settings also store `toolMode: 'text'`, the first `applyMode('text')` call (from `applyToolbarSettings`) returns immediately without calling `applyContainerMode('text')`. The PDF viewer container never gets explicit `user-select:text` / `cursor:text` inline styles, making text selection non-functional until the user toggles to hand tool and back.

**Fix** (applied to `index.js`, function `Je` = `applyMode`):
```
Before: Je=(ge,ue=!0)=>{Q.value!==ge&&(Q.value=ge,qe(ge),ze(ge),ue&&ge!=="ink"&&ge!=="shape"&&k0())}
After:  Je=(ge,ue=!0)=>{qe(ge),Q.value!==ge&&(Q.value=ge,ze(ge),ue&&ge!=="ink"&&ge!=="shape"&&k0())}
```
`qe(ge)` (`applyContainerMode`) is moved outside the conditional guard so it always executes, ensuring container styles (`user-select`, `cursor`, scroll-drag listeners) are set on every call regardless of whether the mode value changed.

## Next Potential Work

1. Expose the English-only i18n toggle as a setting option
2. Save target language preference to SiYuan `settings.translation.targetLang` for permanent persistence
3. Update translator engine API keys handling (Google/Azure/Yandex keys)
4. Add more CJK→non-CJK conversion data mappings if needed
5. Test all features end-to-end in SiYuan desktop app
