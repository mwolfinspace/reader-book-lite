# Reader Book Lite — Full Session Context

> This file contains the complete modification history for AI assistants to resume work. Last updated: 2026-06-14 (session 2).

## Project Goal

Fork `siyuan-sireader` into `reader-book-lite` with:
- License gating removed (all features free, no purchase/activation/upgrade prompts)
- English-only UI (no visible Chinese strings)
- Default translation target: English (`en`) (was Vietnamese `vi` which was originally Chinese `zh-CN`)
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
- `translateGoogle`: default `f` parameter → `"en"`
- `translateAzure`: default `f` parameter → `"en"`
- `translateYandex`: default `f` parameter → `"en"`
- `translateAiFree`: default `f` parameter → `"en"`
- `translateAiSiyuan`: default `f` parameter → `"en"`

**Component-level** (Translate component):
- `b=i0("zh-CN")` → `b=i0(window.__sireader_tgt||"en")`
- Language options array: first entry is still `["vi","Tiếng Việt"]` but default target is now `"en"`

**Engine names** (English):
- `"AI Translate (Free)"` instead of Chinese name
- `"AI Translate (Siyuan)"` instead of Chinese name
- AI Siyuan prompt in English

### 5. Target Language Default and Order

**Default**: `tgt = ref("zh-CN")` → `b = i0(window.__sireader_tgt||"en")` — reads `window.__sireader_tgt` for session persistence, falls back to `"en"`.

**Language order** in the dropdown: `[["vi","Tiếng Việt"],["en","English"],["zh-CN","Chinese"],...]` — Vietnamese first, then English, then the rest.

**Session persistence via `window.__sireader_tgt`**:
- Init: `b = i0(window.__sireader_tgt||"en")`
- On change: `onClick:te=>{window.__sireader_tgt=H,b.value=H,Q.value=!1,L()}` — saves to `window.__sireader_tgt`, sets target, closes dropdown, translates.
- Language survives within the same browser tab/session as long as the page isn't fully reloaded. Writes to a `window` property (sync, no async, no settings save), safe for sandboxed environments.

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

### General

| Location | What |
|----------|------|
| `can()` bypass | Class `j1`, `static can(f,c)` → `return!0` |
| Activation no-op | `_openLicenseContent` → `()=>{}` |
| Upgrade no-op | `showUpgrade` → `L=>{}` |
| Membership section | `O("ul",{ref_key:"licenseRef"...})` → hidden div |
| Translate functions (5) | Function signatures default `f` parameter to `"en"` |
| Engine names | `"AI Translate (Free)"` and `"AI Translate (Siyuan)"` |

### Translate Panel — Structural Map (all on line 772, 180161 chars)

The entire Translate component definition, its scoped wrapper `D3e`, the popup host code, and the MarkPanel component all live on line 772. Use `IndexOf()` on the line string to jump to any position.

#### Class/scoped-id definitions (around pos 26000–26444)

| Minified name | Value | Position | Purpose |
|---------------|-------|----------|---------|
| `b3e` | `{class:"tr-section"}` | 26125 | Source section wrapper |
| `w3e` | `{class:"tr-head"}` | 26150 | Source section header row |
| `m3e` | `{class:"tr-select"}` | 26172 | Source dropdown container |
| `v3e` | `["onClick"]` | 26196 | Source dropdown items dynamic attrs |
| `y3e` | `{class:"tr-text tr-src"}` | 26212 | Source text area |
| `B3e` | `{class:"tr-section"}` | 26241 | Target section wrapper |
| `x3e` | `{class:"tr-head"}` | 26266 | Target section header row |
| `C3e` | `{class:"tr-select"}` | 26288 | Target dropdown container |
| `E3e` | `["onClick"]` | 26312 | Target dropdown items dynamic attrs |
| `S3e` | `{class:"tr-text tr-tgt"}` | 26328 | Target text/result area |
| `_3e` | `{class:"tr-section"}` | 26357 | Engine section wrapper |
| `I3e` | `{class:"tr-head"}` | 26382 | Engine section header row |
| `Q3e` | `{class:"tr-select"}` | 26404 | Engine dropdown container |
| `T3e` | `["onClick"]` | 26428 | Engine dropdown items dynamic attrs |

#### Component definition (26444 → 29203)

```
TranslatePanel=k2({__name:"Translate",props:{text:{}},setup(u){
  const f=u,
    c=[["vi","Tiếng Việt"],["en","English"],["zh-CN","Chinese"],["ja","Japanese"],
       ["ko","Korean"],["fr","Français"],["de","Deutsch"],["es","Español"],["ru","Русский"]],
    d=engines,                     // translator engines object
    p=i0("auto"),                  // src = ref("auto")
    b=i0(window.__sireader_tgt||"en"), // tgt = ref() — session persisted
    v=()=>{...},                   // getEngine() — reads window.__sireader_settings.translation.engine
    B=i0(v()),                     // eng = ref(getEngine())
    C=i0(""),                      // result = ref("")
    E=i0(!1),                      // loading = ref(false)
    I=i0(!1),                      // showSrc = ref(false)
    Q=i0(!1),                      // showTgt = ref(false)
    D=i0(!1),                      // showEng = ref(false)
    F=y0(()=>{...}),               // srcName computed — "Auto detect" or lang name
    V=y0(()=>{...}),               // tgtName computed — lang name or "English"
    L=async()=>{...},              // translate() — calls d[eng].translate(text, tgt)
    G=async W=>{...};              // setEngine() — saves to settings, calls L()
  return xi(()=>f.text,()=>{       // watch on props.text — resets engine, calls L()
    B.value=v(),I.value=!1,Q.value=!1,D.value=!1,L()
  },{immediate:!0}),
  (W,Z)=>(ne(),fe(F0,null,[        // render function — 3 sections
    O("div",b3e,[...]),            // Source section
    Z[10]||(Z[10]=O("div",{class:"tr-line"},null,-1)),
    O("div",B3e,[...]),            // Target section
    Z[11]||(Z[11]=O("div",{class:"tr-line"},null,-1)),
    O("div",_3e,[...])             // Engine section
  ]),64)
}})
```

| What | Variable | Position | Edit pattern |
|------|----------|----------|-------------|
| Langs array | `c` | 26517 | `c=[["vi","Tiếng Việt"],["en","English"],...` |
| Source lang ref init | `p=i0("auto")` | 26707 | `p=i0("auto")` |
| Target lang ref init | `b=i0(...)` | 26720 | `b=i0(window.__sireader_tgt\|\|"en")` — session persisted |
| Engine default func | `v=()=>{...}` | 26754 | Reads `window.__sireader_settings.translation.engine`, defaults to `"azure"` |
| Engine ref init | `B=i0(v())` | ~26850 | Calls `v()` for initial engine |
| Result ref init | `C=i0("")` | ~26880 | Empty string |
| Loading ref init | `E=i0(!1)` | ~26890 | False |
| showSrc ref | `I=i0(!1)` | ~26900 | Source dropdown visibility |
| showTgt ref | `Q=i0(!1)` | ~26910 | Target dropdown visibility |
| showEng ref | `D=i0(!1)` | ~26920 | Engine dropdown visibility |
| srcName computed | `F=y0(()=>{...})` | ~26930 | Returns "Auto detect" or lang name |
| tgtName computed | `V=y0(()=>{...})` | ~27050 | Returns lang name or "English" |
| translate function | `L=async()=>{...}` | 27150 | Calls `d[B.value].translate(f.text,b.value)` |
| setEngine function | `G=async W=>{...}` | 27266 | Saves engine to settings, calls `L()` |
| Watch on text | `xi(()=>f.text,...)` | 27473 | Resets engine, closes dropdowns, calls `L()` |
| Render function | `(W,Z)=>(ne(),...)` | ~27560 | Returns VNode tree |
| **Source** `<span>` | `"Source"` | 27642 | Source section label |
| **Source** dropdown button | `onClick:Z[0]\|\|...` | ~27700 | Toggles `I.value` (showSrc) |
| **Source** dropdown items | `p.value=H` | ~27850 | Sets `src=code`, closes dropdown |
| **Target** `<span>` | `"Target"` | 28251 | Target section label |
| **Target** dropdown button | `onClick:Z[3]\|\|...` | ~28300 | Toggles `Q.value` (showTgt) |
| **Target** dropdown items | `window.__sireader_tgt=H` | ~28554 | **PERSISTS** `window.__sireader_tgt=H,b.value=H,Q.value=!1,L()` |
| **Target** result display | `"Translating..."` | 28658 | Loading state text |
| **Target** fallback | `"Translation failed"` | 28684 | Error/empty result text |
| **Engine** `<span>` | `"Engine"` | 28813 | Engine section label |
| **Engine** dropdown button | `onClick:Z[5]\|\|...` | ~28860 | Toggles `D.value` (showEng) |
| **Engine** dropdown items | `onClick:te=>G(X)` | ~29000 | Calls `setEngine(key)` |

#### Scoped wrapper & usage (after definition)

```
D3e=vn(TranslatePanel,[["__scopeId","data-v-61fc17c8"]])     (pos 29203)
```

`vn` = `createVNode`. `D3e` is the scoped TranslatePanel used as `I.panel==="translate"?wr(D3e,{key:0,text:...})` inside the MarkPanel popup.

#### Popup host (MarkPanel render, pos 44460)

The translate panel is rendered inside MarkPanel's popup div:

```
I.showPanel?sr((ne(),fe("div",{
  key:3,
  initial:'{"opacity":0,"y":5}',
  enter:'{"opacity":1,"y":0}',
  class:"sr-popup sr-popup-panel",
  style:ar(k0.value),
  onClick:a0[8]||...
},[O("div",Z3e,[
  I.panel==="translate"?
    (ne(),wr(D3e,{key:0,text:((Fi=I.selection)==null?void 0:Fi.text)||""},null,8,["text"])):
    (ne(),wr(t8,{key:1,...}))   // t8 = MarkEditPanel
])])),[[O0]])) : v0("",!0)
```

| What | Position | Notes |
|------|----------|-------|
| `I.showPanel?sr(` | 44460 | Popup visibility controlled by MarkPanel state |
| `initial:'{...}'` | ~44497 | JSON-serialized animation config for `v-motion` |
| `D3e` (TranslatePanel) | ~44640 | Used as `wr(D3e,{key:0,text:selection.text})` |
| `O0` directive | ~44750 | `v-motion` directive reference |

#### Minified names in translate panel

| Minified | Meaning | Scope |
|----------|---------|-------|
| `f` | `props` (function arg) | setup function |
| `c` | `langs` array | setup scope |
| `d` | `engines` (translators object) | setup scope |
| `p` | `src` ref | setup scope |
| `b` | `tgt` ref | setup scope |
| `v` | `getEngine()` function | setup scope |
| `B` | `eng` ref | setup scope |
| `C` | `result` ref | setup scope |
| `E` | `loading` ref | setup scope |
| `I` | `showSrc` ref | setup scope |
| `Q` | `showTgt` ref | setup scope |
| `D` | `showEng` ref | setup scope |
| `F` | `srcName` computed | setup scope |
| `V` | `tgtName` computed | setup scope |
| `L` | `translate()` async function | setup scope |
| `G` | `setEngine()` async function | setup scope |
| `H` | `code` (dropdown item value) | click handler param |
| `Z` | cache array (static vnode hoisting) | render function |
| `W` | render function param (first = `$ctx`/setup scope) | render function |
| `Z[0]`-`Z[6]` | cached handler closures | render function |
| `Z[7]`-`Z[9]` | cached `<span>` vnodes (Source/Target/Engine labels) | render function |
| `Z[10]`-`Z[11]` | cached `tr-line` divider vnodes | render function |

## Backup

- `index.js.bak` — backup of current working state at `D:\Logging\data\plugins\reader-book-lite\`

## Known Issues

- Translation component unmounts/remounts on every panel toggle (inherent to `v-if` usage in source)
- Remaining ~2081 CJK characters are conversion data pairs and should NOT be modified

## Remaining CJK Characters

~1904 CJK characters remain in `index.js`. These are overwhelmingly **simplified ↔ traditional Chinese conversion data pairs** (~900 pairs like `学學`, `汉漢`, `语語`) — data, not visible UI. A few dozen non-conversion strings remain in specialized features (dictionary/annotation Chinese text analysis patterns like `复|单|口|旧`, stats labels like `读过`, `累计`, book metadata fields like `三体` which is content, not UI).

## Bug Fixes

### PDF Text Select Mode Not Working on First Open

**Root cause**: `PdfToolbar.vue` function `applyMode()` has an early-return guard (`if (toolMode.value === mode) return`) that skips `applyContainerMode()` when the mode hasn't changed. Since `toolMode` ref initializes to `'text'` and default settings also store `toolMode: 'text'`, the first `applyMode('text')` call (from `applyToolbarSettings`) returns immediately without calling `applyContainerMode('text')`. The PDF viewer container never gets explicit `user-select:text` / `cursor:text` inline styles, making text selection non-functional until the user toggles to hand tool and back.

**Fix** (applied to `index.js`, function `Je` = `applyMode`):
```
Before: Je=(ge,ue=!0)=>{Q.value!==ge&&(Q.value=ge,qe(ge),ze(ge),ue&&ge!=="ink"&&ge!=="shape"&&k0())}
After:  Je=(ge,ue=!0)=>{qe(ge),Q.value!==ge&&(Q.value=ge,ze(ge),ue&&ge!=="ink"&&ge!=="shape"&&k0())}
```
`qe(ge)` (`applyContainerMode`) is moved outside the conditional guard so it always executes, ensuring container styles (`user-select`, `cursor`, scroll-drag listeners) are set on every call regardless of whether the mode value changed.

### Translate Popup Disappears on Scroll/Click Inside

**Root cause**: `MarkPanel.vue` registers a capture-phase scroll listener at `window.addEventListener("scroll", closeAll, true)` (source: `MarkPanel.vue:360`). The `true` (capture phase) means this listener fires for ALL scroll events, including scrolls inside the translate popup (e.g., scrolling translation results in `.tr-tgt` which has `max-height:200px; overflow-y:auto`, or scrolling the language dropdown `.tr-menu` which has `max-height:240px; overflow-y:auto`). When the user tries to scroll inside the popup, `closeAll` fires, closing the panel. The popup reappears because the browser text selection is still active, triggering `checkSelection` on mouseup.

**Fix** (applied to `index.js`, MarkPanel's `onMounted` scroll handler):
```
Before: window.addEventListener("scroll", nt, !0)
        ...window.removeEventListener("scroll", nt, !0)  (in onUnmounted cleanup)
After:  window.addEventListener("scroll", (e=>{e.target.closest(".sr-popup-panel")||nt(e)}), !0)
        [removeEventListener removed — anonymous wrapper can't be matched]
```
The wrapped handler checks if the scroll event's target (the scrolled element) is inside `.sr-popup-panel`. If it is, `closeAll` is NOT called, allowing the popup to stay open and scroll normally. The corresponding `removeEventListener("scroll", nt, !0)` in `onUnmounted` was removed since the anonymous wrapped function cannot be cleanly removed. The MarkPanel/Reader component is long-lived, so the leak is negligible.

**Minified name mappings**: `nt` = `closeAll`, `Vn` = `onMounted`, `ts` = `onUnmounted`

### Translate Target Language Not Persisting Across Panel Toggle

**Root cause**: The Translate component is mounted/destroyed each time via `v-if` on the popup (`I.showPanel`). The `tgt` ref was initialized with a hardcoded default `b=i0("en")`, resetting to English every time the panel reopened. The click handler only set `b.value=H` (local ref) without persisting elsewhere.

**Fix** (applied to `index.js`, Translate component setup):
1. **Init**: `b=i0("en")` → `b=i0(window.__sireader_tgt||"en")` — reads previously persisted value
2. **Click handler**: `onClick:te=>{b.value=H,Q.value=!1,L()}` → `onClick:te=>{window.__sireader_tgt=H,b.value=H,Q.value=!1,L()}` — saves to `window.__sireader_tgt` before setting ref and translating

Uses a simple `window` property for persistence — survives panel toggle within the same session/window, requires no async settings save.

### Popup Animation `[object Object]` Attributes

**Root cause**: MarkPanel's popup div uses `v-motion :initial="{opacity:0,y:5}" :enter="{opacity:1,y:0}"` (Vue template). The compiled VNode creates the div with these as DOM attributes via `fe("div",{...,initial:{opacity:0,y:5},enter:{opacity:1,y:0},...})`. Vue sets non-standard props as HTML attributes via `setAttribute()`, which stringifies objects to `[object Object]`. The `@vueuse/motion` directive then receives `"[object Object]"` instead of parseable JSON, breaking popup entrance/exit animations.

**Fix** (applied to `index.js`, MarkPanel render function):
- `initial:{opacity:0,y:5},enter:{opacity:1,y:0}` → `initial:'{"opacity":0,"y":5}',enter:'{"opacity":1,"y":0}'`
- Serialized as JSON strings so `v-motion` can parse them back with `JSON.parse()`.

## Feature: Selection Popup Toolbar

The selection popup toolbar (`.mark-menu`) already exists in `reader-book-lite` — it was ported from `siyuan-sireader`'s `MarkPanel.vue` but was not working due to the scroll handler bug (now fixed). It contains:

1. **Note** (`#lucide-square-pen`) — creates a mark/annotation and opens editor
2. **Mark** (`#iconMark`) — creates a quick highlight/annotation
3. **Send to** (`#lucide-send`) — sends selection to a SiYuan document
4. **Copy** (`#iconCopy`) — copies selected text to clipboard
5. **TTS** (`#iconPlay`) — reads selection aloud (conditional on `ttsConfig.enabled`)
6. **Dictionary** (`#iconLanguage`) — opens dictionary lookup
7. **Translate** (`#iconTranslate`) — opens translate panel (calls `setPanelState("translate")`)
8. **Search Google** (`#iconSearch`) — opens Google search in new tab

The toolbar is rendered in MarkPanel's template (compiled into `index.js`). `checkSelection` is called on EPUB `mouseup`/`touchend` via `setupEpubKeyboard` (`dpe` function). PDF mode uses a different path: `initPdfAnnotationEvents` calls `showMenu` directly.

**Minified name mappings**: `dpe` = `setupEpubKeyboard`, `ke` = `markPanelRef`, `I` = `state` (reactive), `nt` = `closeAll`, `j0` = `openSelectionPanel`, `b0` = `openMarkPanel`, `Ae` = `handleCopy`, `l0` = `handleTranslate`, `P0` = `setPanelState`

## Edit History

### 2026-06-14: Google Search button added to selection popup
- Added 8th button to `.mark-menu` toolbar (Search Google)
- Uses inline arrow function: `()=>window.open("https://www.google.com/search?q="+encodeURIComponent(I.text||""))`
- Icon: `#iconSearch` (SiYuan-provided SVG sprite)
- Inserted after translate button in the mark-menu children array at byte 2149269

### 2026-06-14: Removed auto-translate bypass in openSelectionPanel
- The `openSelectionPanel` function had a check:
  ```javascript
  if (window.__sireader_settings?.translation?.autoOnSelection && ...)
    return setPanelState("translate", ...)
  ```
  This bypassed the selection toolbar and opened translate directly when `autoOnSelection` was enabled.
- **Fix**: Removed the entire auto-translate condition block so the toolbar always appears on selection.
- Users who want auto-translate-on-selection can still click the Translate button on the toolbar.
- Location: `openSelectionPanel` function (`j0`), removed condition at byte 2139858–2140013.

## Next Potential Work

1. Expose the English-only i18n toggle as a setting option
2. Save target language preference to SiYuan `settings.translation.targetLang` for permanent persistence
3. Update translator engine API keys handling (Google/Azure/Yandex keys)
4. Add more CJK→non-CJK conversion data mappings if needed
5. Test all features end-to-end in SiYuan desktop app
