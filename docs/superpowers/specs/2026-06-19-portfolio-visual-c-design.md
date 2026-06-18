# 个人主页「方向 C · 生成式极光美学」全页改造设计

- 日期：2026-06-19
- 文件：`docs/index.html`（单文件、零依赖、GitHub Pages）
- 目标受众：综合名片（谁都可能看：HR、面试官、同行、社区）
- 出彩维度（用户优先级）：视觉惊艳 > 内容说服力 > 独特交互

## 1. 目标与约束

把现有「深色终端」主页升级为方向 C「生成式极光美学」，在保持单文件零依赖的前提下，做到一打开就 wow，同时把大模型实力（BIRD 单模型榜 JD-UNION-Text2sql-32B）摆到更有说服力的位置。

约束：
- 单一 `docs/index.html`，无构建、无框架、无外部依赖。
- 真实性优先：不编造排名、分数、年份、指标。可用的真实事实：清华大学、BIRD 单模型榜（JD-UNION-Text2sql-32B）、11 个开源项目、3 个领域（安全 / ML / 大模型）。
- 性能与无障碍不退化：`prefers-reduced-motion`、`forced-colors`、`:focus-visible`、语义地标、SEO/OG/JSON-LD 全部保留。

## 2. 模块设计

### 2.1 生成式背景（C1 · SQL→Token 流）—— 已确认
- 新增一个全屏 `<canvas id="bg">`，z-index 置于内容之下、极光之上。
- 粒子为 SQL 关键词 / token 文本（SELECT、FROM、WHERE、JOIN、token、schema、attention、t2sql…），从底部缓慢上浮、到顶溶解（透明度淡入淡出）。橙(`--accent`)/青(`--accent2`)双色，低透明度（≤ .55）。
- 现有极光辉光（`body::after`）与点阵网格（`body::before`）保留，作为更深层背景。
- 性能：`requestAnimationFrame` 驱动；`devicePixelRatio` 封顶 2；约 24–28 个粒子循环复用对象池；`document.hidden` 时暂停动画。
- 无障碍：`prefers-reduced-motion: reduce` 时不初始化 canvas（隐藏），只保留静态极光；`aria-hidden` 装饰层不进入辅助技术。

### 2.2 Hero 首屏
- 放大渐变标题 `Li Gao`（clamp 到更大字号，加 drop-shadow 辉光）。
- 保留 `$ whoami` 终端 eyebrow + 打字机 tagline + 闪烁光标。
- CTA：★ 看核心工作（主按钮）/ GitHub / Email（保留邮箱一键复制）。
- **玻璃拟态 BIRD 成绩卡**：把现有 `.highlight` 横幅升级为 `backdrop-filter: blur` 的发光玻璃卡，置于 hero 内显眼位置，突出 `JD-UNION-Text2sql-32B` 与「BIRD 单模型榜」，链接官方榜单。

### 2.3 Stat strip（关键数字）
- 一行 4 个数字卡：`清华` · `BIRD 单模型榜` · `11 开源项目` · `3 领域`。
- 玻璃拟态小卡，等宽栅格，移动端可换行。全部为真实事实。

### 2.4 成长时间线
- 把 `网络安全 → AI for 安全 → 大模型` 做成横向三段可视化时间线（节点 + 连线 + 阶段标题 + 一句代表性描述）。
- 定性三段，不编造年份；每段挂 1–2 个代表项目名作为佐证。
- 移动端竖向堆叠。

### 2.5 项目区
- 保留三组（tools & apps / security / machine learning）、编号 [01]–[11]、计数。
- 卡片升级为玻璃拟态（半透明 + blur + 细描边），保留现有 hover 辉光、扫描线、3D 倾斜、滚动入场（IntersectionObserver 错位渐显）。

### 2.6 交互
- 保留 ⌘K 命令面板（help/ls/open/whoami/github/email/art/clear）与右下角 FAB。
- 保留邮箱点击复制（回退 mailto）。

## 3. 数据流 / 结构
纯静态，无数据流。DOM 结构：`header`（hero + 成绩卡 + stat strip）→ `main`（时间线 + 三组项目 + more）→ `footer` → ⌘K 面板。canvas 与背景为 `body` 直接子层。

## 4. 错误处理与降级
- 无 `navigator.clipboard` → 邮箱回退 `mailto:`。
- `prefers-reduced-motion` → 关闭 canvas/极光/打字机/入场动画，直接显示终态。
- `forced-colors: active` → 渐变标题回退为 `currentColor`，玻璃效果回退为实色描边。
- 不支持 `backdrop-filter` 的浏览器 → 回退为半透明实色背景（仍可读）。

## 5. 验证标准
- 桌面浏览器实测：背景动效流畅、不掉帧；hero/成绩卡/stat strip/时间线/项目区布局正确。
- 移动端（≤640px）：栅格单列、时间线竖排、不溢出。
- `prefers-reduced-motion` 开启时：无动画、内容完整可读。
- 键盘 Tab 焦点可见；⌘K 正常开关；邮箱复制正常。
- HTML 仍为单文件、零外部依赖。

## 6. 明确不做（YAGNI）
- 不加后端、不加构建工具、不拆多文件。
- 不编造任何排名 / 分数 / 年份 / 量化指标。
- 不做与本次目标无关的重构。
