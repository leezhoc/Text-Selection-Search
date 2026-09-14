// ==UserScript==
// @name         Text Selection Search
// @namespace    http://tampermonkey.net/
// @version      0.2.1
// @description  Search selected text using various search engines.
// @author       You
// @match        *://*/*
// @noframes
// @grant GM_getValue
// @grant GM_setValue
// ==/UserScript==

(function () {
    'use strict';

    const DEFAULT_ENGINES = [
        { name: '百度',     url: 'https://www.baidu.com/s?wd=' },
        { name: 'Bing',    url: 'https://www.bing.com/search?q=' },
        { name: '大同',     url: 'https://datong.info/ui/search.html?q=' },
        { name: 'Google',  url: 'https://www.google.com/search?q=' },
        { name: '知乎',     url: 'https://www.zhihu.com/search?q=' },
        { name: '哔哩哔哩', url: 'https://search.bilibili.com/all?keyword=' },
        { name: '豆瓣',    url: 'https://www.douban.com/search?source=suggest&q=' },
        { name: 'Bangumi', url: 'https://bgm.tv/subject_search/' },
        { name: '萌娘百科', url: 'https://zh.moegirl.org.cn/index.php?search=' },
        { name: '小红书',   url: 'https://www.xiaohongshu.com/search_result?keyword=' },
        { name: 'YouTube', url: 'https://www.youtube.com/results?search_query=' },
        { name: 'Github',  url: 'https://github.com/search?q=' },
        { name: 'Twitter', url: 'https://x.com/search?q=' },
        { name: 'Yandex',  url: 'https://yandex.com/search/?text=' },
    ];

    /* ---------- 可调项 ---------- */
    const DURATION = '0.15s';
    const HANDLE_REST_OPACITY = 0.33;
    const HANDLE_SIZE = 24;
    const MENU_PADDING_X = 8;   // 菜单文字离卡片左右边缘的距离(px)，调大更宽松、调小更紧凑
    const STORAGE_KEY = 'tss_engines_v1';

    const pageWin = (typeof unsafeWindow !== 'undefined') ? unsafeWindow : window;

    /* ---------- 配置读写 ---------- */
    function normalize(list) {
        if (!Array.isArray(list)) return null;
        const out = list
            .filter(e => e && typeof e.name === 'string' && typeof e.url === 'string' && e.name && e.url)
            .map(e => ({ name: e.name, url: e.url, hidden: !!e.hidden }));
        return out.length ? out : null;
    }

    function defaultEngines() {
        return DEFAULT_ENGINES.map(e => ({ name: e.name, url: e.url, hidden: false }));
    }

    let engines;
    try {
        engines = normalize(JSON.parse(GM_getValue(STORAGE_KEY) || 'null')) || defaultEngines();
    } catch (err) {
        engines = defaultEngines();
    }

    function saveEngines() {
        try { GM_setValue(STORAGE_KEY, JSON.stringify(engines)); } catch (err) {}
    }

    let selectedText = '';

    /* ---------- DOM ---------- */
    const panel = document.createElement('div');
    panel.id = 'tss-panel';
    Object.assign(panel.style, {
        position: 'fixed',
        left: '0', top: '50%',
        transform: 'translateY(-50%)',
        zIndex: '2147483647',
        display: 'none',
        flexDirection: 'row',
        alignItems: 'center',
        pointerEvents: 'none',
        font: '12px/1.4 sans-serif',
    });

    const handle = document.createElement('div');
    handle.textContent = '🔍';
    Object.assign(handle.style, {
        flex: `0 0 ${HANDLE_SIZE}px`,
        width: HANDLE_SIZE + 'px',
        height: HANDLE_SIZE + 'px',
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center',
        background: 'rgba(0,0,0,.38)',
        color: '#fff',
        fontSize: '11px',
        borderRadius: '50%',
        cursor: 'pointer',
        pointerEvents: 'auto',
        opacity: String(HANDLE_REST_OPACITY),
        transition: `opacity ${DURATION} ease, background ${DURATION} ease`,
    });

    const menu = document.createElement('div');
    Object.assign(menu.style, {
        flex: '0 0 auto',
        marginLeft: `-${HANDLE_SIZE / 2}px`,
        width: 'max-content',
        maxHeight: '90vh',
        overflowY: 'auto',
        overflowX: 'hidden',
        scrollbarWidth: 'thin',
        background: '#fff',
        border: '1px solid #e0e0e0',
        borderRadius: '8px',
        boxShadow: '0 4px 14px rgba(0,0,0,.18)',
        padding: '4px 0',             // 水平必须为 0：高亮要顶到卡片两边
        pointerEvents: 'none',
        opacity: '0',
        transform: 'translateX(-8px)',
        transition: `opacity ${DURATION} ease, transform ${DURATION} ease`,
        userSelect: 'none',
        webkitUserSelect: 'none',
    });

    function buildSearchUrl(url, text) {
        const enc = encodeURIComponent(text);
        return url.includes('%s') ? url.replace('%s', enc) : url + enc;
    }

    function makeEngineButton(eng) {
        const btn = document.createElement('button');
        btn.textContent = eng.name;
        Object.assign(btn.style, {
            display: 'block',
            width: '100%',             // button 的 block 不会自动满宽，必须显式指定
            boxSizing: 'border-box',   // 100% 含 padding，高亮正好顶到卡片两边的边框
            whiteSpace: 'nowrap',
            margin: '1px 0',           // 不加左右边距
            padding: `4px ${MENU_PADDING_X}px`,
            border: 'none',
            borderRadius: '0',
            background: 'transparent',
            font: 'inherit',
            textAlign: 'left',
            cursor: 'pointer',
        });
        btn.addEventListener('click', () => {
            if (selectedText) {
                window.open(buildSearchUrl(eng.url, selectedText), '_blank');
            }
        });
        return btn;
    }

    function makeManageRow() {
        const row = document.createElement('button');
        row.className = 'tss-manage';
        row.textContent = '⚙ 管理引擎';
        Object.assign(row.style, {
            display: 'block',
            width: '100%',
            boxSizing: 'border-box',   // 顶部 1px 分隔线也计入宽度，不溢出
            whiteSpace: 'nowrap',
            margin: '4px 0 0',
            padding: `5px ${MENU_PADDING_X}px 4px`,
            border: 'none',
            borderTop: '1px solid',    // 颜色交给样式表（跟主题）
            borderRadius: '0',
            background: 'transparent',
            font: 'inherit',
            fontSize: '11px',
            textAlign: 'center',
            cursor: 'pointer',
            opacity: '.75',
        });
        row.addEventListener('click', openManager);
        return row;
    }

    function renderMenu() {
        menu.textContent = '';
        engines.filter(e => !e.hidden).forEach(eng => menu.appendChild(makeEngineButton(eng)));
        menu.appendChild(makeManageRow());
    }
    renderMenu();

    panel.appendChild(handle);
    panel.appendChild(menu);
    document.body.appendChild(panel);

    /* ---------- 暗色模式 ---------- */
    const mql = window.matchMedia('(prefers-color-scheme: dark)');

    function applyTheme() {
        const dark = mql.matches;
        menu.style.background = dark ? '#2b2d31' : '#fff';
        menu.style.borderColor = dark ? '#45474d' : '#e0e0e0';
        menu.style.color = dark ? '#d4d4d8' : '#333';
        menu.style.boxShadow = dark ? '0 4px 14px rgba(0,0,0,.5)' : '0 4px 14px rgba(0,0,0,.18)';
        menu.style.colorScheme = dark ? 'dark' : 'light';
        handle.style.background = dark ? 'rgba(255,255,255,.32)' : 'rgba(0,0,0,.38)';
    }
    applyTheme();
    if (typeof mql.addEventListener === 'function') mql.addEventListener('change', applyTheme);
    else if (typeof mql.addListener === 'function') mql.addListener(applyTheme);

    /* ---------- 样式表：hover / 分隔线 / 管理面板配色 ---------- */
    const HOVER_CSS = `
        #tss-panel button:hover { background: #eef4fb !important; }
        #tss-panel .tss-manage { border-top-color: #e6e6e6 !important; }
        #tss-mgr .tss-card { background: #fff !important; border-color: #e0e0e0 !important; color: #333 !important; }
        #tss-mgr input { background: #fff !important; border-color: #ccc !important; color: #333 !important; }
        #tss-mgr input:focus { border-color: #4a90d9 !important; }
        #tss-mgr button { border-color: #ccc !important; color: #333 !important; }
        #tss-mgr button:hover { background: #eef4fb !important; }
        @media (prefers-color-scheme: dark) {
            #tss-panel button:hover { background: #3a3d45 !important; }
            #tss-panel .tss-manage { border-top-color: #3f4147 !important; }
            #tss-mgr .tss-card { background: #26282c !important; border-color: #3f4147 !important; color: #d4d4d8 !important; }
            #tss-mgr input { background: #1e2023 !important; border-color: #4a4d52 !important; color: #d4d4d8 !important; }
            #tss-mgr input:focus { border-color: #5b9bd5 !important; }
            #tss-mgr button { border-color: #4a4d52 !important; color: #d4d4d8 !important; }
            #tss-mgr button:hover { background: #3a3d45 !important; }
        }`;
    let cssOK = false;
    try {
        if (typeof pageWin.CSSStyleSheet === 'function') {
            const sheet = new pageWin.CSSStyleSheet();
            sheet.replaceSync(HOVER_CSS);
            document.adoptedStyleSheets = [...document.adoptedStyleSheets, sheet];
            cssOK = true;
        }
    } catch (e) {}
    if (!cssOK) {
        try {
            const style = document.createElement('style');
            style.textContent = HOVER_CSS;
            document.head.appendChild(style);
        } catch (e) {}
    }

    /* ---------- 展开 / 收起 ---------- */
    function openMenu() {
        menu.style.opacity = '1';
        menu.style.transform = 'translateX(0)';
        menu.style.pointerEvents = 'auto';
        handle.style.opacity = '0';
    }
    function closeMenu() {
        menu.style.opacity = '0';
        menu.style.transform = 'translateX(-8px)';
        menu.style.pointerEvents = 'none';
        handle.style.opacity = String(HANDLE_REST_OPACITY);
    }

    panel.addEventListener('mouseenter', openMenu);
    panel.addEventListener('mouseleave', closeMenu);
    panel.addEventListener('mousedown', e => e.preventDefault());

    const show = () => (panel.style.display = 'flex');
    const hide = () => {
        if (panel.style.display === 'none') return;
        panel.style.display = 'none';
        closeMenu();
    };

    /* ---------- 管理面板（懒构建） ---------- */
    let mgr = null;

    function styleInput(inp) {
        Object.assign(inp.style, {
            display: 'block',
            width: '100%',
            boxSizing: 'border-box',
            margin: '2px 0',
            padding: '3px 7px',
            border: '1px solid #ccc',
            borderRadius: '5px',
            background: '#fff',
            font: 'inherit',
            fontSize: '12px',
        });
    }

    function opBtn(label, title, fn) {
        const b = document.createElement('button');
        b.textContent = label;
        b.title = title;
        Object.assign(b.style, {
            height: '26px',
            minWidth: '26px',
            padding: '0 7px',
            border: '1px solid #ccc',
            borderRadius: '5px',
            background: 'transparent',
            font: 'inherit',
            fontSize: '12px',
            cursor: 'pointer',
            flex: 'none',
        });
        b.addEventListener('click', fn);
        return b;
    }

    function buildManager() {
        const backdrop = document.createElement('div');
        backdrop.id = 'tss-mgr';
        Object.assign(backdrop.style, {
            position: 'fixed',
            inset: '0',
            zIndex: '2147483647',
            display: 'none',
            alignItems: 'center',
            justifyContent: 'center',
            background: 'rgba(0,0,0,.4)',
            padding: '24px',
        });

        const card = document.createElement('div');
        card.className = 'tss-card';
        Object.assign(card.style, {
            width: '460px',
            maxWidth: '92vw',
            maxHeight: '82vh',
            overflowY: 'auto',
            borderRadius: '10px',
            border: '1px solid #e0e0e0',
            background: '#fff',
            color: '#333',
            boxShadow: '0 12px 40px rgba(0,0,0,.35)',
            padding: '12px 14px',
            font: '13px/1.5 sans-serif',
        });

        const head = document.createElement('div');
        Object.assign(head.style, { display: 'flex', alignItems: 'center', gap: '8px' });
        const title = document.createElement('b');
        title.textContent = '管理搜索引擎';
        const closeBtn = opBtn('✕', '关闭', closeManager);
        closeBtn.style.marginLeft = 'auto';
        head.append(title, closeBtn);

        const hint = document.createElement('div');
        hint.textContent = '链接中的 %s 会被替换成选中的文字，没有 %s 则追加到末尾；隐藏的引擎不出现在菜单里，可随时恢复。';
        Object.assign(hint.style, { fontSize: '11px', opacity: '.7', margin: '4px 0 8px' });

        const list = document.createElement('div');

        const foot = document.createElement('div');
        Object.assign(foot.style, { display: 'flex', gap: '8px', marginTop: '10px' });
        const addBtn = opBtn('＋ 添加搜索引擎', '添加', addEngine);
        const resetBtn = opBtn('恢复默认', '恢复默认引擎列表', resetEngines);
        resetBtn.style.marginLeft = 'auto';
        foot.append(addBtn, resetBtn);

        card.append(head, hint, list, foot);
        backdrop.appendChild(card);
        document.body.appendChild(backdrop);

        backdrop.addEventListener('mousedown', e => {
            if (e.target === backdrop) closeManager();
        });
        document.addEventListener('keydown', e => {
            if (e.key === 'Escape' && mgr && mgr.backdrop.style.display === 'flex') closeManager();
        });

        return { backdrop, list };
    }

    function openManager() {
        if (!mgr) mgr = buildManager();
        renderManagerRows();
        mgr.backdrop.style.display = 'flex';
        closeMenu();
    }

    function closeManager() {
        if (!mgr) return;
        mgr.backdrop.style.display = 'none';
        update(null);
    }

    function renderManagerRows() {
        const list = mgr.list;
        list.textContent = '';
        engines.forEach((eng, i) => {
            const row = document.createElement('div');
            Object.assign(row.style, { display: 'flex', gap: '6px', alignItems: 'flex-start', margin: '7px 0' });

            const fields = document.createElement('div');
            Object.assign(fields.style, { flex: '1', minWidth: '0' });

            const nameIn = document.createElement('input');
            nameIn.type = 'text';
            nameIn.value = eng.name;
            nameIn.placeholder = '名称';
            styleInput(nameIn);

            const urlIn = document.createElement('input');
            urlIn.type = 'text';
            urlIn.value = eng.url;
            urlIn.placeholder = 'https://…/search?q=%s';
            styleInput(urlIn);

            nameIn.addEventListener('change', () => {
                const v = nameIn.value.trim();
                if (v) { eng.name = v; saveEngines(); renderMenu(); }
            });
            urlIn.addEventListener('change', () => {
                const v = urlIn.value.trim();
                if (v) { eng.url = v; saveEngines(); renderMenu(); }
            });
            fields.append(nameIn, urlIn);

            const ops = document.createElement('div');
            Object.assign(ops.style, { display: 'flex', gap: '4px', paddingTop: '2px' });
            ops.append(
                opBtn('↑', '上移', () => moveEngine(i, -1)),
                opBtn('↓', '下移', () => moveEngine(i, 1)),
                opBtn(eng.hidden ? '取消隐藏' : '隐藏', '', () => {
                    eng.hidden = !eng.hidden;
                    saveEngines(); renderManagerRows(); renderMenu();
                }),
                opBtn('✕', '移除', () => {
                    if (engines.length > 1 && confirm(`移除「${eng.name}」？`)) {
                        engines.splice(i, 1);
                        saveEngines(); renderManagerRows(); renderMenu();
                    }
                })
            );

            row.append(fields, ops);
            list.appendChild(row);
        });
    }

    function moveEngine(i, dir) {
        const j = i + dir;
        if (j < 0 || j >= engines.length) return;
        [engines[i], engines[j]] = [engines[j], engines[i]];
        saveEngines(); renderManagerRows(); renderMenu();
    }

    function addEngine() {
        engines.push({ name: '新引擎', url: 'https://www.example.com/search?q=%s', hidden: false });
        saveEngines(); renderManagerRows(); renderMenu();
        const last = mgr.list.lastElementChild;
        if (last) {
            const inp = last.querySelector('input');
            inp.focus();
            inp.select();
        }
    }

    function resetEngines() {
        if (!confirm('恢复默认引擎列表？当前所有修改（含自定义引擎）将丢失。')) return;
        engines = defaultEngines();
        saveEngines(); renderManagerRows(); renderMenu();
    }

    /* ---------- 读选区：普通文档 / 输入框 / Shadow DOM ---------- */
    const watchedRoots = new Set();

    function findShadowRoots(e) {
        const roots = [];
        if (e && typeof e.composedPath === 'function') {
            for (const node of e.composedPath()) {
                if (node instanceof ShadowRoot) roots.push(node);
            }
        }
        let el = document.activeElement;
        while (el && el.shadowRoot) {
            roots.push(el.shadowRoot);
            el = el.shadowRoot.activeElement;
        }
        return roots;
    }

    function readInputSelection() {
        let el = document.activeElement;
        while (el) {
            const tag = el.tagName;
            if (tag === 'TEXTAREA' ||
                (tag === 'INPUT' && /^(text|search|url|tel)$/.test(el.type || 'text'))) {
                const s = el.selectionStart, t = el.selectionEnd;
                return s !== null && s !== t ? el.value.slice(s, t) : '';
            }
            el = el.shadowRoot ? el.shadowRoot.activeElement : null;
        }
        return '';
    }

    function getSelectedText(e) {
        const docSel = document.getSelection();
        if (docSel && !docSel.isCollapsed) {
            const t = docSel.toString().trim();
            if (t) return t;
        }
        const inputText = readInputSelection().trim();
        if (inputText) return inputText;
        for (const root of findShadowRoots(e)) {
            watchRoot(root);
            if (typeof root.getSelection === 'function') {
                const sel = root.getSelection();
                if (sel && !sel.isCollapsed) {
                    const t = sel.toString().trim();
                    if (t) return t;
                }
            }
        }
        return '';
    }

    function anySelection() {
        const docSel = document.getSelection();
        if (docSel && !docSel.isCollapsed) return true;
        for (const root of watchedRoots) {
            if (!root.isConnected) { watchedRoots.delete(root); continue; }
            if (typeof root.getSelection !== 'function') continue;
            const sel = root.getSelection();
            if (sel && !sel.isCollapsed) return true;
        }
        return false;
    }

    const onSelectionChange = () => { if (!anySelection()) hide(); };
    document.addEventListener('selectionchange', onSelectionChange);

    function watchRoot(root) {
        if (watchedRoots.has(root)) return;
        watchedRoots.add(root);
        root.addEventListener('selectionchange', onSelectionChange);
    }

    /* ---------- 事件 ---------- */
    function update(e) {
        if (mgr && mgr.backdrop.style.display === 'flex') return;
        const text = getSelectedText(e);
        if (text) {
            selectedText = text;
            show();
        }
    }

    document.addEventListener('mouseup', e => {
        if (e.target && panel.contains(e.target)) return;
        update(e);
    }, true);

    document.addEventListener('keyup', e => {
        const k = e.key;
        if (k === 'a' || k === 'A' || (k && k.startsWith('Arrow'))) update(e);
    }, true);

    document.addEventListener('mousedown', e => {
        if (!e.target || panel.contains(e.target)) return;
        hide();
    }, true);
})();
