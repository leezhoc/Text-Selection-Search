# Text Selection Search

![Version](https://img.shields.io/badge/version-0.2.1-blue.svg)
![License](https://img.shields.io/badge/license-AGPL--3.0-blue.svg)
![Tampermonkey](https://img.shields.io/badge/Tampermonkey-supported-green.svg)

> 选中网页上的任意文字，左侧边缘滑出搜索引擎菜单，一键在新标签页搜索。
> Select any text on a page, and a search-engine menu slides out from the left edge — search it in one click.

中文 | [English](#english)

---

## 简介

平时查一个词，流程是：选中 → 复制 → 新开标签 → 粘贴 → 挑站点。这个脚本把中间几步压成一次点击。

选中文字后，屏幕左边缘会浮出一个半透明的 🔍 圆点，鼠标移上去立即展开引擎列表，点哪个就用哪个搜索。面板贴在左边缘、垂直居中，静止时几乎透明，既不遮挡正文，也不打断阅读。

全部逻辑在本地运行：没有任何网络请求、没有统计上报，配置只写进油猴的本地存储。

## 功能特性

- **14 个内置搜索引擎**：百度、Bing、大同、Google、知乎、哔哩哔哩、豆瓣、Bangumi、萌娘百科、小红书、YouTube、Github、Twitter、Yandex
- **选区识别范围广**：正文选区、`<textarea>` 与 `input[type=text/search/url/tel]` 内的选区，甚至 Shadow DOM 里的选区都能识别
- **完全可自定义**：增删引擎、改名、改搜索地址、↑↓ 调整顺序、隐藏暂时不用的引擎、一键恢复默认
- **悬停展开，不打断阅读**：24px 圆点静止时只有 33% 不透明度，鼠标移入后 0.15s 滑出菜单，移出即收起
- **跟随系统深色模式**：无需手动切换，系统配色变化时面板跟着变
- **不污染页面样式**：优先通过 `adoptedStyleSheets` 注入，失败才降级到 `<style>`，所有规则都限定在自有 ID / class 内
- **配置持久化**：引擎列表存于油猴本地存储，换页面、关浏览器都不丢
- **零追踪**：无网络请求、无统计、无远端配置

## 安装

1. 浏览器安装 [Tampermonkey](https://www.tampermonkey.net/)（Violentmonkey 等支持 `GM_getValue` / `GM_setValue` 的管理器理论可用，未做测试）。
2. 打开安装链接，油猴会弹出安装页，点「安装」：

   `https://raw.githubusercontent.com/leezhoc/Text-Selection-Search/main/Text-Selection-Search.user.js`

3. 刷新已打开的页面即可生效。

**手动安装**：油猴面板 → 添加新脚本 → 清空模板 → 粘贴 `Text-Selection-Search.user.js` 全文 → `Ctrl/Cmd + S` 保存。

> 升级时重新打开上面的安装链接覆盖安装即可；引擎配置存在本地存储中，通常不受覆盖安装影响。

## 使用说明

1. **选中文字** —— 鼠标拖选，或 `Ctrl/Cmd + A` 全选、按住 `Shift` 用方向键扩展选区。
2. **移到左侧圆点** —— 屏幕左边缘垂直居中处出现 🔍，鼠标移上去，菜单向右展开。
3. **点击引擎** —— 在新标签页打开对应搜索页，关键词已自动 URL 编码。
4. **收起** —— 鼠标移出菜单自动收起；点击页面任意空白处也会隐藏。

菜单底部固定一行「⚙ 管理引擎」，点开是引擎管理面板：

| 按钮 | 作用 |
| --- | --- |
| ↑ / ↓ | 上移 / 下移该引擎，改变菜单里的排列顺序 |
| 隐藏 / 取消隐藏 | 从菜单中暂时移除该引擎，配置仍保留，随时可恢复 |
| ✕ | 删除该引擎（只剩一条时不可删） |
| ＋ 添加搜索引擎 | 在列表末尾追加一条，可直接编辑 |
| 恢复默认 | 清空所有自定义，回到 14 个内置引擎（有二次确认） |

面板右上角 `✕`、`Esc` 键、点击面板外的遮罩都能关闭。名称或链接输入框修改后失焦即保存，不需要额外的保存按钮。

## 自定义搜索引擎

每条引擎由「名称 + 链接模板」组成，模板里可以用 `%s` 占位：

- **模板含 `%s`** → `%s` 被替换为 URL 编码后的选中文字
- **模板不含 `%s`** → 选中文字直接拼接在末尾

举例：

| 引擎 | 链接模板 | 实际访问 |
| --- | --- | --- |
| 百度 | `https://www.baidu.com/s?wd=%s` | 关键词作为 `wd` 参数 |
| Github | `https://github.com/search?q=%s` | 关键词作为 `q` 参数 |
| Bangumi | `https://bgm.tv/subject_search/` | 关键词拼接在路径末尾 |

想接入自建搜索、内部 wiki、论坛搜索页，只要把那条 URL 里关键词所在的位置换成 `%s` 填进去即可。

## 常见问题

**Q：iframe 里的文字选中后没有反应？**

脚本带 `@noframes`，只在顶层页面运行，内嵌框架里的选区不会触发面板。

**Q：某些页面完全不出现圆点？**

浏览器内置页（`chrome://`、`about:`）、扩展商店、内置 PDF 阅读器、`view-source:` 等页面不允许注入用户脚本，属于浏览器限制。

**Q：手机上能用吗？**

不能。展开菜单依赖 `mouseenter` / `mouseleave` 悬停事件，触屏设备没有 hover，脚本面向桌面端。

**Q：圆点挡住了原文怎么办？**

圆点贴在视口最左边缘，仅 24px 且静止态 33% 不透明，通常不会压住正文；若确实碍事，点一下页面空白处即可隐藏。

**Q：配置存在哪里？卸载会丢吗？**

存在油猴的本地存储中，键名 `tss_engines_v1`，内容是 `{name, url, hidden}` 的 JSON 数组。卸载脚本会一并清除，需要保留的话请先到油猴的「存储」页导出。

**Q：菜单太长超出屏幕？**

菜单最高 90vh，超出部分在内部滚动，不会跑出视口。

**Q：想改圆点大小、动画速度或菜单留白？**

脚本顶部「可调项」区集中了这些常量：`HANDLE_SIZE`（圆点直径）、`HANDLE_REST_OPACITY`（静止透明度）、`DURATION`（动画时长）、`MENU_PADDING_X`（文字左右留白）。

## 许可证

[AGPL-3.0](./LICENSE)

---

## English

### What it is

A Tampermonkey userscript that turns text selection into a search action. Highlight anything on a page, hover the small handle on the left edge, and pick an engine — the query opens in a new tab. Everything runs locally: no network requests, no telemetry, no remote config.

### Features

- **14 built-in engines** — Baidu, Bing, Datong, Google, Zhihu, Bilibili, Douban, Bangumi, Moegirlpedia, Xiaohongshu, YouTube, GitHub, Twitter, Yandex.
- **Broad selection support** — document selections, `<textarea>` and text-like `<input>` selections, and even selections inside Shadow DOM.
- **Fully customizable** — add, rename, re-point, reorder, hide, or reset engines at any time.
- **Non-intrusive UI** — a 24px handle at 33% opacity sits on the left edge, vertically centered; the menu slides out on hover and collapses when the pointer leaves.
- **Automatic dark mode** — follows `prefers-color-scheme`.
- **Style isolation** — injected via `adoptedStyleSheets` with a `<style>` fallback; every rule is scoped to the script's own IDs and classes.
- **Persistent config** — the engine list lives in userscript storage.

### Install

1. Install [Tampermonkey](https://www.tampermonkey.net/) (or any manager supporting `GM_getValue` / `GM_setValue`).
2. Open the install link — Tampermonkey shows its install page:

   `https://raw.githubusercontent.com/leezhoc/Text-Selection-Search/main/Text-Selection-Search.user.js`

3. Reload any open tab.

Manual install: Tampermonkey dashboard → add a new script → replace the template with the full contents of `Text-Selection-Search.user.js` → save.

### Usage

1. Select text (drag, `Ctrl/Cmd + A`, or `Shift` + arrow keys).
2. Hover the 🔍 handle at the left edge — the menu expands to the right.
3. Click an engine to open the results in a new tab.
4. Move the pointer away, or click anywhere on the page, to dismiss.

The last row of the menu, **⚙ 管理引擎** ("Manage engines"), opens the editor: rename, edit the URL template, move up/down, hide, remove, add, or reset to defaults. Edits save on blur; close with `Esc`, the `✕` button, or a click on the backdrop.

### Custom engines

An engine is a name plus a URL template. If the template contains `%s`, it is replaced with the URL-encoded selection; otherwise the selection is appended to the end.

| Engine | Template | Result |
| --- | --- | --- |
| Baidu | `https://www.baidu.com/s?wd=%s` | query in `wd` |
| GitHub | `https://github.com/search?q=%s` | query in `q` |
| Bangumi | `https://bgm.tv/subject_search/` | query appended to the path |

Any internal wiki, forum search page, or self-hosted search endpoint works the same way — just put `%s` where the keyword belongs.

### FAQ

- **Nothing happens inside an iframe** — the script uses `@noframes` and runs only in the top-level document.
- **No handle on some pages** — browser-internal pages (`chrome://`, `about:`), extension stores, the built-in PDF viewer and `view-source:` do not allow userscripts.
- **Mobile / touch devices** — not supported; expansion relies on hover.
- **Where is the config stored?** Userscript storage under the key `tss_engines_v1`, as a JSON array of `{name, url, hidden}`. Uninstalling the script clears it — export it first if you want to keep your list.
- **Menu taller than the screen?** It caps at 90vh and scrolls internally.

### License

[AGPL-3.0](./LICENSE)
