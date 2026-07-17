# Footer 白色钢笔手写签名 — 设计文档

日期：2026-07-17
项目：cyk-homepage（`D:\Projects\cyk-homepage\index.html`，纯 HTML 单文件，GSAP + ScrollTrigger）

## 目标

Footer 右下角出现白色钢笔风格的手写签名，滚动进入 footer 后自动一笔一划勾勒写出，只播一次。

## 需求（已确认）

- 笔迹：白色钢笔——细笔触、白色描边、圆头圆角、无填充、无发光
- 位置：右下角，`right 8% / bottom 20%`，`rotate(-7deg)`，宽 `min(340px, 44vw)`，移动端缩小
- 触发：footer 进入视口 60% 时自动播放（`once: true`），不跟滚动 scrub，总时长约 2.5-3.5 秒
- 逐笔顺序：按 path 顺序依次描边，每笔时长按路径长度分配，`power1.inOut`
- **无预显示圆点**：旧版每笔起点会漏出圆头笔帽的小圆点。本版每条 path 动画前 `visibility: hidden` + dashoffset 多留 1px，起笔瞬间才显示
- 签名内容：用户之后在 draw.html 重新手写（内容写时定）；先用 stash 旧版笔迹路径作占位调试动画

## 实现

1. **CSS**（`</style>` 前）：`.sig` 定位/倾斜/宽度；`.sig path { stroke-width: 8 }`（viewBox 1200×500 坐标系下≈2px 视觉粗细）；移动端 media query
2. **HTML**（footer `#footerHand` 之后）：`<svg id="chenSig" class="sig" viewBox="-30 -30 1260 560">`，单组 `<g fill="none" stroke="#fff" stroke-linecap="round" stroke-linejoin="round">` 包 19 条占位 path（提取自 stash，不 pop stash 本身）
3. **JS**（GSAP 块内、footerTimeline 之后）：每条 path `getTotalLength()` 设 dasharray/offset(+1) 并隐藏；独立 timeline，`trigger: '#footer', start: 'top 60%', once: true`，逐笔 `.set(visibility)` + `.to(strokeDashoffset: 0)`

## 边界

- `prefers-reduced-motion` / GSAP 未加载：整个动画 JS 不执行 → path 未被隐藏 → 签名静态显示
- 与现有 `footerTimeline`（scrub）独立，互不干扰

## 后续

用户 draw.html 手写新签名（笔粗调细）→ 导出 path → 替换 `#chenSig` 内的占位 path，动画代码不变。
