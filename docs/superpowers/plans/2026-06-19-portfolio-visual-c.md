# 个人主页方向 C 生成式极光美学 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 把 `docs/index.html` 升级为「生成式极光美学」全页改造：C1 SQL→Token 背景、放大 hero、玻璃拟态成绩卡、stat strip、成长时间线、玻璃拟态项目卡。

**Architecture:** 单文件静态页（无构建/框架/依赖）。新增全屏 `<canvas>` 背景层 + 一段 IIFE 驱动 token 上浮；CSS 用玻璃拟态（`backdrop-filter`）重构卡片；新增 stat strip 与横向时间线两块结构。所有动效尊重 `prefers-reduced-motion`，`backdrop-filter`/渐变在不支持时有实色回退。

**Tech Stack:** HTML + CSS + 原生 JS（Canvas 2D, requestAnimationFrame, IntersectionObserver）。

**验证方式:** 本仓库无测试框架，且为静态页——验证以「浏览器实测 + 降级检查」替代单测（YAGNI，不引入测试工具）。每个 Task 末尾 `open docs/index.html` 目检 + 提交。

---

### Task 1: C1 生成式背景（canvas + token 上浮）

**Files:**
- Modify: `docs/index.html`（`:root` 下方加 canvas/背景 CSS；`<body>` 首行加 `<canvas>`；`<script>` 内加 IIFE）

- [ ] **Step 1: 加 canvas 元素**

在 `<body>` 的 `<div class="wrap">` 之前插入：

```html
<canvas id="bg" aria-hidden="true"></canvas>
```

- [ ] **Step 2: 加 canvas CSS**

在 `<style>` 内 `body::before` 规则之前插入：

```css
#bg { position: fixed; inset: 0; z-index: -3; pointer-events: none; }
```

并把现有 `body::before` 的 `z-index: -2`、`body::after` 的 `z-index: -1` 保持不变（canvas 在最底 -3）。

- [ ] **Step 3: 加 token 上浮 IIFE**

在 `<script>` 内、`document.getElementById('year')...` 之后插入：

```javascript
// C1：SQL / token 从底部上浮溶解（reduced-motion / 隐藏标签页时不跑）
(function () {
  if (reduce) return;
  var c = document.getElementById('bg'), x = c.getContext('2d');
  var words = ['SELECT','FROM','WHERE','JOIN','GROUP BY','name','users','COUNT(*)','ORDER BY','token','schema','LLM','attention','t2sql','LIMIT','AS','ON','▢ ▢ ▢'];
  var parts = [], dpr = 1;
  function resize() { dpr = Math.min(window.devicePixelRatio || 1, 2); c.width = innerWidth * dpr; c.height = innerHeight * dpr; x.setTransform(dpr,0,0,dpr,0,0); }
  resize(); addEventListener('resize', resize);
  function spawn() {
    return { txt: words[(Math.random()*words.length)|0], xx: Math.random()*innerWidth, yy: innerHeight+20,
      v: .25+Math.random()*.55, size: 10+Math.random()*7, life: 0, max: innerHeight*(.6+Math.random()*.4),
      col: Math.random()<.4 ? '#ff8a33' : '#3fe9cf', op: 0 };
  }
  for (var i=0;i<26;i++){ var p=spawn(); p.yy=Math.random()*innerHeight; parts.push(p); }
  var running = true;
  document.addEventListener('visibilitychange', function(){ running = !document.hidden; if (running) frame(); });
  function frame() {
    if (!running) return;
    x.clearRect(0,0,innerWidth,innerHeight);
    for (var i=0;i<parts.length;i++){
      var p=parts[i]; p.yy-=p.v; p.life+=p.v;
      var prog=p.life/p.max;
      p.op = prog<.15 ? prog/.15 : (prog>.75 ? (1-prog)/.25 : 1); if (p.op<0) p.op=0;
      x.font='600 '+p.size+'px ui-monospace, Menlo, monospace';
      x.fillStyle=p.col; x.globalAlpha=p.op*.5;
      x.fillText(p.txt, p.xx, p.yy);
      if (p.yy < -20 || prog>=1) parts[i]=spawn();
    }
    x.globalAlpha=1;
    requestAnimationFrame(frame);
  }
  frame();
})();
```

（`reduce` 变量已在脚本顶部定义：`var reduce = window.matchMedia('(prefers-reduced-motion: reduce)').matches;`）

- [ ] **Step 4: 目检 + 提交**

```bash
open docs/index.html   # 确认背景有 token 上浮、不卡
git add docs/index.html && git commit -m "feat: 加入 C1 生成式 SQL→Token 背景"
```

---

### Task 2: Hero 放大 + 玻璃拟态 BIRD 成绩卡

**Files:**
- Modify: `docs/index.html`（`header h1` CSS 放大；`.highlight` 改为玻璃卡 `.hero-card`；把成绩卡从 `<main>` 移到 `header` 内 about 之后）

- [ ] **Step 1: 放大标题 CSS**

把 `header h1` 的 `font-size` 改为 `clamp(52px, 11vw, 96px)`、`letter-spacing: -2.5px`，并加 `filter: drop-shadow(0 6px 40px rgba(255,122,26,.22));`

- [ ] **Step 2: 玻璃卡 CSS**

把现有 `.highlight` 整段替换为：

```css
.hero-card {
  display: flex; align-items: center; gap: 14px; flex-wrap: wrap;
  margin: 26px 0 0; max-width: 580px;
  border: 1px solid rgba(255,255,255,.12); border-radius: 18px; padding: 18px 22px;
  background: linear-gradient(135deg, rgba(255,138,51,.12), rgba(63,233,207,.06));
  backdrop-filter: blur(14px); -webkit-backdrop-filter: blur(14px);
  box-shadow: 0 20px 60px rgba(0,0,0,.4);
  font-family: var(--mono); transition: transform .2s, box-shadow .2s, border-color .2s;
}
.hero-card:hover { transform: translateY(-2px); border-color: rgba(255,138,51,.5); box-shadow: 0 26px 70px rgba(0,0,0,.5); }
.hero-card .hl-tag { color: var(--accent2); font-size: 11px; letter-spacing: 1px; border: 1px solid var(--line); border-radius: 6px; padding: 2px 9px; white-space: nowrap; }
.hero-card .hl-text { color: var(--ink); font-size: 14px; flex: 1; min-width: 200px; line-height: 1.6; }
.hero-card .hl-text b { color: var(--accent); font-weight: 700; }
.hero-card .hl-go { color: var(--accent); font-size: 13px; white-space: nowrap; }
```

并把 `.highlight:focus-visible` 选择器改为 `.hero-card:focus-visible`。

- [ ] **Step 3: 移动成绩卡到 header**

把 `<main>` 内的 `<a class="highlight" id="featured" ...>...</a>` 块剪切，改 class 为 `hero-card`，粘到 `header` 内 `.now` 段之后（仍带 `id="featured"`，保留 CTA 锚点）。

- [ ] **Step 4: 目检 + 提交**

```bash
open docs/index.html
git add docs/index.html && git commit -m "feat: hero 放大 + 玻璃拟态 BIRD 成绩卡"
```

---

### Task 3: Stat strip（关键数字）

**Files:**
- Modify: `docs/index.html`（CSS 加 `.stats`；`<main>` 顶部 `.lede` 之后插入结构）

- [ ] **Step 1: stat CSS**

在 `.lede` 规则之后插入：

```css
.stats { display: grid; grid-template-columns: repeat(4, 1fr); gap: 12px; margin: 8px 0 36px; }
.stat {
  border: 1px solid rgba(255,255,255,.1); border-radius: 14px; padding: 16px 14px; text-align: center;
  background: linear-gradient(180deg, rgba(60,65,79,.5), rgba(43,47,58,.3));
  backdrop-filter: blur(8px); -webkit-backdrop-filter: blur(8px);
}
.stat .v { font-size: clamp(18px, 3vw, 24px); font-weight: 800; color: var(--accent); letter-spacing: -.5px; }
.stat .l { font-family: var(--mono); font-size: 11px; color: var(--muted); margin-top: 4px; }
@media (max-width: 640px) { .stats { grid-template-columns: repeat(2, 1fr); } }
```

- [ ] **Step 2: stat 结构**

在 `<main>` 内 `.lede` 段之后插入（全部真实事实）：

```html
<div class="stats" aria-label="关键信息">
  <div class="stat"><div class="v">清华</div><div class="l">毕业院校</div></div>
  <div class="stat"><div class="v">BIRD</div><div class="l">单模型榜上榜</div></div>
  <div class="stat"><div class="v">11</div><div class="l">开源项目</div></div>
  <div class="stat"><div class="v">3</div><div class="l">技术领域</div></div>
</div>
```

- [ ] **Step 3: 目检 + 提交**

```bash
open docs/index.html
git add docs/index.html && git commit -m "feat: 加入关键数字 stat strip"
```

---

### Task 4: 成长时间线（安全 → AI 安全 → 大模型）

**Files:**
- Modify: `docs/index.html`（CSS 加 `.timeline`；把 `.lede` 的 `.arc` 替换为时间线结构，放在 stats 之后/项目组之前）

- [ ] **Step 1: timeline CSS**

```css
.timeline { display: grid; grid-template-columns: repeat(3, 1fr); gap: 0; margin: 0 0 44px; position: relative; }
.tl-step { position: relative; padding: 0 18px 0 0; }
.tl-step .dot { width: 12px; height: 12px; border-radius: 50%; background: var(--accent); box-shadow: 0 0 12px var(--accent); margin-bottom: 12px; }
.tl-step::before { content: ""; position: absolute; left: 6px; top: 6px; right: -6px; height: 2px; background: linear-gradient(90deg, var(--accent), var(--accent2)); opacity: .4; }
.tl-step:last-child::before { display: none; }
.tl-step h4 { font-family: var(--mono); font-size: 14px; color: var(--ink); margin-bottom: 6px; }
.tl-step p { font-size: 13px; color: var(--muted); line-height: 1.6; }
.tl-step .tag { font-family: var(--mono); font-size: 11px; color: var(--accent2); }
@media (max-width: 640px) {
  .timeline { grid-template-columns: 1fr; gap: 22px; }
  .tl-step { padding: 0 0 0 18px; }
  .tl-step::before { left: 5px; top: 12px; bottom: -22px; right: auto; width: 2px; height: auto; }
}
```

- [ ] **Step 2: timeline 结构**

把 `<p class="lede">…</p>` 整段替换为：

```html
<p class="lede">一条从安全到智能的主线 —— <b>下面是这条路上留下的项目。</b></p>
<div class="timeline" aria-label="成长主线">
  <div class="tl-step"><div class="dot" aria-hidden="true"></div><span class="tag">01</span><h4>网络安全</h4><p>恶意流量检测、工控系统漏洞、内容安全平台。</p></div>
  <div class="tl-step"><div class="dot" aria-hidden="true"></div><span class="tag">02</span><h4>AI for 安全</h4><p>对抗样本下的鲁棒检测、面向内容安全的机器学习。</p></div>
  <div class="tl-step"><div class="dot" aria-hidden="true"></div><span class="tag">03</span><h4>大模型</h4><p>生成式大模型与 Text-to-SQL：JD-UNION-Text2sql-32B 上 BIRD 单模型榜。</p></div>
</div>
```

（注：stats 与 timeline 顺序为 `成绩卡(header)` → `main` 内 `lede` + `stats` + `timeline` → 项目组。Task 3 的 stats 放在 lede 之后、本 timeline 之前。）

- [ ] **Step 3: 目检 + 提交**

```bash
open docs/index.html
git add docs/index.html && git commit -m "feat: 加入成长时间线"
```

---

### Task 5: 项目卡玻璃拟态

**Files:**
- Modify: `docs/index.html`（`.proj` 的 `background` 改为半透明 + `backdrop-filter`）

- [ ] **Step 1: 改 .proj 背景**

把 `.proj` 规则里的 `background: linear-gradient(180deg, var(--card), var(--bg2));` 改为：

```css
  background: linear-gradient(180deg, rgba(60,65,79,.55), rgba(43,47,58,.35));
  backdrop-filter: blur(10px); -webkit-backdrop-filter: blur(10px);
```

保留现有 hover 辉光、扫描线、3D 倾斜、入场动画规则不变。

- [ ] **Step 2: 目检 + 提交**

```bash
open docs/index.html
git add docs/index.html && git commit -m "feat: 项目卡玻璃拟态"
```

---

### Task 6: 降级与回退检查 + 收尾提交

**Files:**
- Modify: `docs/index.html`（`prefers-reduced-motion` 与 `forced-colors` 段补充）

- [ ] **Step 1: reduced-motion 补充**

确认 `@media (prefers-reduced-motion: reduce)` 段包含 `#bg { display: none; }`（canvas 已在 JS 层 `return`，此为双保险隐藏静态画布）。

- [ ] **Step 2: forced-colors 回退**

在 `@media (forced-colors: active)` 段补充：玻璃元素回退实色——

```css
.hero-card, .stat, .proj { background: Canvas; backdrop-filter: none; -webkit-backdrop-filter: none; }
```

- [ ] **Step 3: 全面目检**

- 桌面：背景动效流畅；hero/成绩卡/stats/timeline/项目区布局正确；⌘K、邮箱复制正常。
- 窄屏（拖窄到 ≤640px）：stats 两列、timeline 竖排、grid 单列、不溢出。
- 系统开启「减弱动态效果」后刷新：无动画、内容完整。

```bash
open docs/index.html
```

- [ ] **Step 4: 收尾提交并推送**

```bash
git add docs/index.html && git commit -m "feat: 完成方向 C 全页改造的降级与回退"
git push
```

---

## 自检（Self-Review）

- **Spec 覆盖**：2.1 背景→T1；2.2 hero+成绩卡→T2；2.3 stat strip→T3；2.4 时间线→T4；2.5 项目卡→T5；2.6 交互（⌘K/邮箱）保留不动→T6 目检；第 4 节降级→T1(reduce)/T6(forced-colors/backdrop 回退)。无遗漏。
- **占位符**：无 TBD/TODO；每个改动给了具体代码。
- **一致性**：成绩卡 class 统一为 `.hero-card`（含 focus-visible 与 forced-colors 回退）；`id="featured"` 保留以匹配「★ 看核心工作」CTA；stats/timeline 在 `main` 内顺序明确（lede → stats → timeline → 项目组）。
- **真实性**：stats 与 timeline 仅用真实事实，无编造数字/年份。
