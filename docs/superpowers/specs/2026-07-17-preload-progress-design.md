# 首访全量预载 + 加载进度 — 设计文档

日期：2026-07-17
项目：cyk-homepage（`D:\Projects\cyk-homepage\index.html`）

## 目标

首次打开时在loader阶段预载全部图片+视频，右下角显示"加载中 xx%"；进入页面后所有区域即开即看。电脑/手机同一逻辑。

## 已确认决策

- 进度按**文件数**计（54个资源：53张图+ai-naming.mp4），非字节
- **等100% + 15秒超时放行**：超时进入后剩余资源后台继续加载
- 进度显示在loader**右下角**，替换"Loading presence"占位文案
- 二次访问靠浏览器缓存自然秒过，不做localStorage标记

## 实现

1. **HTML**：loader-meta右侧span → `<span id="loadPct">加载中 0%</span>`
2. **timeline拆分**：onLoadTimeline拆成 introTl（0~1.88s开场：mark/letters/tag/meta/brand/scanline/strips）+ exitTl（paused：loader上滑、heroWord、nav、heroCopy，相对偏移保持原重叠关系：heroWord 0 / loader滑出0.10 / nav 0.24 / heroCopy 0.66）
3. **门闩**：`introDone && assetsDone` 双条件都满足才 `exitTl.play()`；intro的onComplete置introDone；预载完成或15s超时置assetsDone
4. **预载**：收集全部`img[src]`去重 + mp4；图用`new Image()`并发预载，mp4用`fetch().blob()`警热缓存；每个onload/onerror都+1计数并更新百分比；错误也计数永不卡死；`finished`标志防超时与100%双触发
5. **不动**：页面`loading="lazy"`属性保留（缓存已热滚到即显）；减动效/无GSAP兜底路径直接进不等待；滚动恢复、签名等loader逻辑不变

## 验证

- 硬刷新+DevTools禁缓存：loader开场→右下角百分比爬升→100%掀开→快速滚全页图片全部已就绪
- DevTools限速Slow 3G：15秒左右放行，进入后图片继续补
- 二次刷新：百分比瞬间跑满，等待感≈0
- 减动效模式：直接进，无报错
