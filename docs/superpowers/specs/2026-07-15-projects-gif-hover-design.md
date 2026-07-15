# Projects GIF Hover 放大跳转 — 设计文档

日期：2026-07-15
项目：cyk-homepage（`D:\Projects\cyk-homepage\index.html`，纯 HTML 单文件，GSAP 3.12.5 + ScrollTrigger）

## 目标

Projects 区左上角格子展示 `images/projects/ai-naming.gif`：

- 桌面：鼠标悬停时 GIF 从格子平滑"长大"到屏幕中央（占视口大部分），移开缩回；点击大图新标签打开 GitHub。
- 手机/触屏：格子内显示 GIF，点击直接新标签打开 GitHub，不做放大。
- 跳转目标：`https://github.com/sky-k111/ai-name`

## 现状

- Projects 区（`#projects`，index.html 596-615 行）：15 个 `.archive-placeholder` 占位格（桌面 5×3，移动端 3 列），4:5 比例。
- 滚动动画（1075-1092 行）：ScrollTrigger scrub + pin，15 格从中心散开到网格位；timeline 用函数式 x/y/scale/rotate 控制 `.archive-placeholder` 的 transform。
- 关键约束：**scrub timeline 拥有格子的 transform，hover 动画不能直接动格子本身**，否则滚动中悬停会打架。

## 方案（已选：方案 C 固定克隆层）

对比过的方案：

| 方案 | 说明 | 结论 |
|---|---|---|
| A. GSAP Flip 插件 | 官方插件做 FLIP 位移 | 多一个 CDN 依赖，且移动的是格子本身，与 scrub 冲突 |
| B. 直接放大格子 | 格子原地 scale/位移 | 最简单，但与 scrub timeline 抢 transform，滚动中悬停会乱 |
| **C. 固定克隆层（选定）** | 格子不动，克隆一份 GIF 在 fixed 层做 FLIP 动画 | 与滚动动画零冲突，无新依赖 |

## 实现设计

### 1. HTML 改动

- 第一个 `.archive-placeholder`（左上角，index 0）：`<div>` 改为 `<a>`，保留原 class（timeline 选择器 `.archive-placeholder` 继续匹配）：

```html
<a id="gifTile" class="archive-placeholder overflow-hidden"
   href="https://github.com/sky-k111/ai-name" target="_blank" rel="noopener noreferrer">
  <img id="gifTileImg" src="images/projects/ai-naming.gif" alt="AI 智能取名"
       class="w-full h-full object-cover pointer-events-none" />
</a>
```

- `</body>` 前追加克隆层（初始全部隐藏）：

```html
<div id="gifBackdrop" class="fixed inset-0 z-[90] bg-black/85 opacity-0 pointer-events-none"></div>
<img id="gifClone" src="images/projects/ai-naming.gif" alt="AI 智能取名"
     class="fixed z-[91] hidden cursor-pointer object-contain" />
```

### 2. CSS 改动（~10 行）

- `#gifTile` 悬停时轻微高亮边框（视觉提示可交互）。
- `#gifClone` `will-change: transform`。

### 3. JS 改动（~50 行，放现有 script 内、timeline 定义之后）

启用条件：`matchMedia('(hover: hover) and (pointer: fine)')` 为真才绑定悬停逻辑；否则只有 `<a>` 原生点击跳转。

桌面悬停流程：

1. `mouseenter` #gifTile：
   - `getBoundingClientRect()` 量格子当前视口位置（滚动中位置也准确）。
   - 克隆图定位到该矩形（top/left/width/height），显示。
   - 目标尺寸：GIF 原始宽高比 fit 进 `88vw × 85vh` 盒子，屏幕居中。
   - `gsap.to()` 克隆图飞到目标（~0.45s，power3.out）；幕布 `autoAlpha` 到 1。
   - 原格子 img `opacity: 0`（观感：同一张图长大）。
2. 鼠标从格子移到大图上：保持展开（克隆图自己也算悬停区）。
3. `mouseleave` 克隆图（且不在格子上）：
   - 重新量格子当前矩形（可能滚动过），克隆图飞回，完成后隐藏；幕布淡出；原格子 img 恢复。
4. `click` 克隆图：`window.open('https://github.com/sky-k111/ai-name', '_blank')`。
5. 防抖：进出动画用同一个 tween 引用，`overwrite: true` 防快速进出叠加。

### 4. 不改动的部分

- 15 格散开 scrub 动画、`#projectsWord`、底部 "coming soon" 文案、其余 14 个占位格：零改动。
- 克隆层与 ScrollTrigger 互不干涉（克隆是独立 fixed 元素）。

## 错误处理

- GIF 加载失败：`<img>` 落空显示 alt 文本，格子保持原占位底色；克隆逻辑照常但飞出的是空图——可接受（本地资源，风险低）。
- CDN（GSAP）未加载：现有代码已有 `typeof` 安全检查惯例，悬停逻辑包在同一保护内；`<a>` 原生跳转不依赖 JS。

## 验证标准

1. 桌面 Chrome：滚动到 Projects → 左上角格子显示 GIF → 悬停平滑放大到屏幕中央大部分 → 移开缩回原位 → 点击大图新标签打开 `github.com/sky-k111/ai-name`。
2. 滚动中途（格子在散开动画半程）悬停/移开：动画从当前位置起飞/落回，不跳变。
3. 手机模拟（触屏）：无放大，点格子直接新标签跳转。
4. 其余 14 格散开动画与改动前一致。
