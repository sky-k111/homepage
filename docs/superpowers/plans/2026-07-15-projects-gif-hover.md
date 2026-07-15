# Projects GIF Hover 放大跳转 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Projects 区左上角格子展示 ai-naming.gif，桌面 hover 时 GIF 从格子 FLIP 放大到屏幕中央（88vw × 85vh 内 contain），点击新标签打开 GitHub；触屏直接点击跳转。

**Architecture:** 方案 C「固定克隆层」——格子本身不做 hover 动画（scrub timeline 拥有 `.archive-placeholder` 的 transform），另建一个 `position: fixed` 的克隆 `<img>` + 幕布层，hover 时量格子实时矩形，克隆图从该矩形 GSAP 飞到屏幕中央。与滚动动画零冲突。

**Tech Stack:** 纯 HTML 单文件（`D:\Projects\cyk-homepage\index.html`）、Tailwind CDN、GSAP 3.12.5（已有，不加新依赖）。

**Spec:** `docs/superpowers/specs/2026-07-15-projects-gif-hover-design.md`

## Global Constraints

- 不新增任何 CDN 依赖（不引入 GSAP Flip 插件）。
- 不改动 15 格散开 scrub timeline（index.html `projectsTimeline`）及其余 14 个占位格。
- 左上角格子必须保留 class `archive-placeholder`（timeline 选择器依赖）。
- 跳转链接精确为 `https://github.com/sky-k111/ai-name`，`target="_blank"` + `rel="noopener noreferrer"`。
- GIF 路径：`images/projects/ai-naming.gif`（项目内已存在）。
- 悬停逻辑只在 `matchMedia('(hover: hover) and (pointer: fine)')` 为真时启用。
- 悬停 JS 放在现有 `if (!reducedMotion && typeof gsap !== 'undefined')` 大块内（reduced-motion 用户自动跳过动画，原生 `<a>` 跳转不受影响）。
- 本项目无测试框架，遵循项目惯例验证：结构 grep + JS 语法检查 + 浏览器手动验证（用户偏好记录在案）。
- 每个 task 结束 git commit。

## File Structure

只改一个文件：

- Modify: `D:\Projects\cyk-homepage\index.html`
  - ① 第 602 行附近：Projects 网格第一个占位 `<div>` → `<a>` + `<img>`（GIF 格子）
  - ② `</style>`（约 434 行）前：追加 ~6 行 CSS
  - ③ `<script>`（约 934 行）前：追加克隆层 HTML（幕布 + 克隆 img）——必须在 `<script>` 之前，脚本立即执行时元素需已存在
  - ④ `projectsTimeline` 定义（约 1092 行 `.fromTo('#projectsWord', ...)` 之后）：追加悬停 JS ~55 行

---

### Task 1: GIF 格子 + 克隆层 HTML + CSS（无 JS，先让静态部分成立）

**Files:**
- Modify: `D:\Projects\cyk-homepage\index.html`（三处：网格第一格、`</style>` 前、`<script>` 前）

**Interfaces:**
- Produces: `#gifTile`（`<a>` 格子）、`#gifTileImg`（格内 GIF）、`#gifClone`（克隆图，初始 `display:none`）、`#gifBackdrop`（幕布，初始透明且 `pointer-events:none`）。Task 2 的 JS 按这四个 id 取元素。

- [ ] **Step 1: 改造左上角格子**

找到 Projects 区网格（`<div class="archive-grid">` 内，第 602 行附近）：

```html
<div class="archive-placeholder"></div><div class="archive-placeholder"></div><div class="archive-placeholder"></div>
```

把**第一个** `<div class="archive-placeholder"></div>` 替换为（该行其余两个 div 不动）：

```html
<a id="gifTile" class="archive-placeholder block overflow-hidden" href="https://github.com/sky-k111/ai-name" target="_blank" rel="noopener noreferrer"><img id="gifTileImg" src="images/projects/ai-naming.gif" alt="AI 智能取名" class="w-full h-full object-cover pointer-events-none" /></a>
```

要点：class 保留 `archive-placeholder`（scrub timeline 选择器继续匹配，index 0 行为不变）；`pointer-events-none` 让事件都落在 `<a>` 上。

- [ ] **Step 2: 追加 CSS**

在 `</style>`（约 434 行）之前插入：

```css
/* ---------- Projects GIF hover ---------- */
#gifTile { transition: border-color 0.3s; }
#gifTile:hover { border-color: rgba(255,255,255,0.5); }
#gifClone { object-fit: contain; will-change: left, top, width, height; }
```

- [ ] **Step 3: 追加克隆层 HTML**

在 `<script>`（约 934 行，页面主脚本开始处）**之前**插入：

```html
<!-- Projects GIF hover 放大克隆层 -->
<div id="gifBackdrop" class="fixed inset-0 z-[90] bg-black/85 opacity-0 pointer-events-none"></div>
<img id="gifClone" src="images/projects/ai-naming.gif" alt="AI 智能取名" class="fixed z-[91] hidden cursor-pointer" />
```

位置必须在 `<script>` 之前：主脚本顶层立即执行，元素必须先于脚本存在。

- [ ] **Step 4: 结构验证**

```powershell
Select-String -Path D:\Projects\cyk-homepage\index.html -Pattern 'gifTile|gifClone|gifBackdrop' | Measure-Object | Select-Object Count
```

Expected: Count ≥ 6（gifTile×2 处、gifTileImg、gifClone、gifBackdrop 各出现）。

```powershell
(Select-String -Path D:\Projects\cyk-homepage\index.html -Pattern 'archive-placeholder' -AllMatches).Matches.Count
```

Expected: 17（CSS 定义 2 处 + 网格 15 格，改造后仍 15 格：14 div + 1 a）。

- [ ] **Step 5: 浏览器验证（静态部分）**

打开 `D:\Projects\cyk-homepage\index.html`，滚动到 Projects 区：
- 左上角格子显示 ai-naming.gif（散开动画正常，GIF 随格子从中心飞到左上角位）
- 点击 GIF 格子 → 新标签打开 `github.com/sky-k111/ai-name`（原生 `<a>`，此时无放大逻辑）
- 其余 14 格及滚动动画与改动前一致
- Console 无新增报错

- [ ] **Step 6: Commit**

```bash
cd /d/Projects/cyk-homepage
git add index.html
git commit -m "feat: Projects左上角格子展示ai-naming.gif并链接GitHub"
```

---

### Task 2: 桌面 hover 克隆层 FLIP 放大 JS

**Files:**
- Modify: `D:\Projects\cyk-homepage\index.html`（`projectsTimeline` 段之后插入）

**Interfaces:**
- Consumes: Task 1 的 `#gifTile` / `#gifTileImg` / `#gifClone` / `#gifBackdrop`；现有全局 `gsap`。
- Produces: 无（终端功能）。

- [ ] **Step 1: 插入悬停 JS**

定位 `/* ---------- 4 · Projects ---------- */` 段落，在 `.fromTo('#projectsWord', { y: '8vh', scale: 1.08 }, { y: '-8vh', scale: 0.98, ease: 'none' }, 0);`（约 1092 行，projectsTimeline 链式调用结尾）之后插入：

```js

  /* ---------- 4b · Projects GIF hover 放大 ---------- */
  (function () {
    var fine = window.matchMedia('(hover: hover) and (pointer: fine)').matches;
    var tile = document.getElementById('gifTile');
    var tileImg = document.getElementById('gifTileImg');
    var clone = document.getElementById('gifClone');
    var backdrop = document.getElementById('gifBackdrop');
    if (!fine || !tile || !tileImg || !clone || !backdrop) return;

    var expanded = false;

    /* GIF 原始比例 fit 进 88vw × 85vh，屏幕居中 */
    function targetBox() {
      var vw = window.innerWidth, vh = window.innerHeight;
      var maxW = vw * 0.88, maxH = vh * 0.85;
      var ratio = (clone.naturalWidth || 4) / (clone.naturalHeight || 3);
      var w = maxW, h = w / ratio;
      if (h > maxH) { h = maxH; w = h * ratio; }
      return { left: (vw - w) / 2, top: (vh - h) / 2, width: w, height: h };
    }

    function expand() {
      if (expanded) return;
      expanded = true;
      var r = tile.getBoundingClientRect();
      gsap.set(clone, { display: 'block', left: r.left, top: r.top, width: r.width, height: r.height });
      gsap.set(tileImg, { opacity: 0 });
      var t = targetBox();
      gsap.to(clone, { left: t.left, top: t.top, width: t.width, height: t.height, duration: 0.45, ease: 'power3.out', overwrite: true });
      gsap.to(backdrop, { autoAlpha: 1, duration: 0.35, overwrite: true });
    }

    function collapse() {
      if (!expanded) return;
      expanded = false;
      var r = tile.getBoundingClientRect();
      gsap.to(clone, {
        left: r.left, top: r.top, width: r.width, height: r.height,
        duration: 0.4, ease: 'power3.inOut', overwrite: true,
        onComplete: function () {
          if (!expanded) { gsap.set(clone, { display: 'none' }); gsap.set(tileImg, { opacity: 1 }); }
        }
      });
      gsap.to(backdrop, { autoAlpha: 0, duration: 0.35, overwrite: true });
    }

    tile.addEventListener('mouseenter', expand);
    clone.addEventListener('mouseleave', collapse);
    clone.addEventListener('click', function () {
      window.open('https://github.com/sky-k111/ai-name', '_blank', 'noopener');
    });
  })();
```

设计要点（实现者须知）：
- 展开瞬间克隆图就盖在鼠标下 → 关闭只需监听**克隆图**的 `mouseleave`，不监听格子的（格子的 mouseleave 会在克隆图盖上来时立刻触发，监听它会导致刚展开就收回）。
- `overwrite: true` 防快速进出时动画叠加。
- `collapse` 时重新量格子矩形——滚动中格子位置会变，飞回实时位置不跳变。
- `expanded` 标志防重入；`onComplete` 里再查一次防"收回动画中又展开"时误隐藏。
- 动画 left/top/width/height 而非 transform scale：img 非等比缩放会变形，单元素动画性能足够。

- [ ] **Step 2: JS 语法验证**

```powershell
$html = Get-Content D:\Projects\cyk-homepage\index.html -Raw; $m = [regex]::Match($html, '(?s)<script>(.*)</script>'); Set-Content -Path $env:TEMP\cyk-check.js -Value $m.Groups[1].Value -Encoding utf8; node --check $env:TEMP\cyk-check.js
```

Expected: 无输出（语法通过）。报错则修复后重跑。

- [ ] **Step 3: 浏览器验证（完整流程）**

打开页面，滚动到 Projects 区，逐项确认：
1. 悬停左上角 GIF 格 → GIF 从格子平滑飞到屏幕中央放大（占视口大部分，完整不裁切），背景变暗
2. 鼠标在大图上移动 → 保持展开
3. 移出大图 → 飞回格子原位，幕布淡出，格内 GIF 恢复显示
4. 快速反复进出 → 无闪烁、无叠加、无卡死
5. 滚动到散开动画半程悬停再移开 → 从当前位置起飞/落回，不跳变
6. 点击大图 → 新标签打开 `github.com/sky-k111/ai-name`
7. DevTools 切手机模拟（触屏）刷新 → 悬停无效果，点格子直接跳转
8. Console 无报错；其余 sections 动画正常

- [ ] **Step 4: Commit**

```bash
cd /d/Projects/cyk-homepage
git add index.html
git commit -m "feat: Projects GIF桌面hover克隆层放大动画"
```
