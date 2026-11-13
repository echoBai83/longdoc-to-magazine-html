---
name: longdoc-to-magazine-html
description: 把长文档（粘贴文本 / PDF / Word / Markdown）转成单文件杂志风 HTML，同时满足浏览器预览与 A4 打印另存 PDF。当用户要求"排版成杂志风""转成可打印 HTML""长文做成白皮书/报告外观""导出 A4 PDF""网页和打印都要好看"时使用。核心保证：零遗漏保留原文、打印分页不撕裂、内网离线可交付。
license: MIT
version: 2.2.0
---

# 长文档转杂志风 HTML（完美打印版 V2.2｜封板生产版）

> 单一源真相：本文件是唯一规范源。`references/prompt.md` 是从本文件末尾萃取出的执行 Prompt，
> 修改时先改本文件，再同步 prompt。两处语义必须一致。
>
> 版本：V2.2，支持粘贴文本 / PDF / Word / Markdown 附件解析；网页浏览 + A4-PDF 双目标输出。

## 一、核心执行原则（防遗漏与忠实原文）

1. **零遗漏法则**：绝对禁止对原文的段落、列表、表格备注、括号说明、附件、脚注、图注进行概括、合并或省略。100% 保留原文信息。
2. **先提取，后套壳**：必须先将原文提取为完整的 DOM 结构（标题层级、表格行列、合并单元格、多级编号），确认节点无遗漏后再套用 CSS 类。
   > 特别提示：遇到合并单元格、深度嵌套多级列表（1.1、1.1.1），禁止扁平化改写，忠实还原 DOM 结构。
3. **智能布局匹配**（AI 生成阶段执行，产物为静态 HTML，运行时不依赖 AI / JS）
   - 比较/并列数据 👉 用 `<table>` 表格；超宽表格外层必须包裹 `.table-wrapper`
   - 长文段落/引言 👉 字符 300-1200 且预估模块高度小于单页 A4：双栏 `.layout-columns` + 首字下沉；超出阈值直接降级单栏
   - 核心金句/话术 👉 居中放大 `.layout-pullquote`；禁止放置在分页临界位置
   - 步骤/流程/编年史 👉 有序条目 ≥2 使用大数字列表 `.layout-index` + `.layout-index-item`
   - 场景/特征碎片化信息 👉 用卡片网格 `.card`；**禁止把大段正文强行塞入卡片**
   - 图文对照案例 👉 `.layout-split` 全宽分屏；尽量控制模块在单 A4 高度内
   - 主内容+简短补充备注来源 👉 `.layout-sidebar-note` 侧边旁注；备注过长降级普通段落

### 📌 附件输入解析规则（支持 PDF / Word / Markdown）

1. **解析优先保真原则**
   - PDF / Word：完整提取正文、表格、合并单元格、列表层级、标题层级、图片、图注、脚注、备注；保留表格合并关系、多级编号。丢弃自动页眉页脚、水印、自动页码、修订痕迹；**手动批注、脚注、尾注必须保留为正文备注**。
   - Markdown：保留原有标题层级、表格、列表、代码块、链接、图片引用，不擅自改写 Markdown 语义。
2. **附件解析失败降级策略**
   - 如果附件解析不全、表格错乱、图片丢失：**必须主动告知用户解析缺陷，不要脑补补全内容，禁止编造原文不存在的文字**。
3. **图片来源区分**
   - Word/PDF 内提取出的内嵌图片：遵循离线图片规则，AI 只输出 base64 占位标签，不生成超长 base64 字符串，交由人工替换编码。
   - Markdown 外部图片链接：联网环境保留 http 链接；离线内网场景替换为 base64 占位标签。
4. **扫描图片 PDF 限制：AI 不做 OCR 识别；收到扫描图片 PDF 直接告知用户无法解析，请提供可复制文本版本。**

### 📌 离线环境重要约束

1. AI 布局选择仅发生在 HTML 生成阶段，输出产物是**纯静态 HTML，不含业务 JavaScript**，保存本地后断网可直接打开、打印 PDF。
2. 联网交付：使用 Google Fonts CDN 思源黑体；图片允许输出 http 链接地址。
3. **内网/完全离线交付场景强制规则**
   - 移除外部 Google Fonts 的 link 标签，或字体做 Base64 内嵌 CSS；依靠系统字体回退作为兜底；
   - 图片禁止直接输出 http 外部 URL；
   - **Base64 图片处理提示：禁止 AI 直接生成大体积 Base64 长串（极易触发输出 token 上限造成 HTML 截断损坏）。AI 仅输出占位标签：`<img src="data:image/jpeg;base64,【请在此粘贴Base64编码】">`，由人工完成 Base64 粘贴替换。**

## 二、全局视觉与排版规范（Editorial 杂志风）

- **字体**：全局强制使用 **思源黑体** (Noto Sans SC)。

```html
<link href="https://fonts.googleapis.com/css2?family=Noto+Sans+SC:wght@400;500;700;900&display=swap" rel="stylesheet">
```

```css
font-family: "Noto Sans SC", "Source Han Sans SC", "思源黑体", "PingFang SC", sans-serif;
```

- **CSS 变量配色**

```css
:root {
    --primary: #103399;
    --gold: #C8A666;
    --bg-color: #F6F4EF;
    --white: #ffffff;
    --border-color: #E2DDD3;
    --text-main: #1a1a1a;
    --text-light: #555555;
}
```

- **字号约束（拒绝 1.5 倍等比缩放）**
  - **Web 端**：正文 `1rem`，H3 `1.25rem`，H2 `2rem`，H1 `clamp(2.5rem, 5vw, 4rem)`。
  - **Print 端（@media print）**：正文 `11pt`，H3 `14pt`，H2 `20pt`，H1 `32pt`。
- **间距视觉基准**：卡片内边距 24px-30px，组件垂直间距 32px-40px，章节间距 60px-80px；大面积留白，避免拥挤。

## 三、打印/PDF 跨页完美控制（核心 CSS）

> 原则：大区块允许跨页（防留白），细粒度元素防截断（防断行）
> 兼容提示：`orphans / widows` Firefox 支持完善，Chrome/Chromium 支持有限，仅作为排版优化手段，不能保证 100% 消除孤行。

```css
@media print {
    @page { size: A4; margin: 12mm; }
    body {
        background-color: #ffffff !important;
        -webkit-print-color-adjust: exact;
        print-color-adjust: exact;
        font-size: 11pt !important;
    }
    .container { padding: 0 !important; max-width: 100% !important; }

    /* 大区块允许自然跨页 */
    section, .table-wrapper, .layout-columns { page-break-inside: auto !important; break-inside: auto !important; }

    /* 细粒度元素防截断 */
    tr, h2, h3, .card, .layout-index-item, .layout-split, figure, .layout-sidebar-note {
        page-break-inside: avoid !important;
        break-inside: avoid !important;
    }

    /* 表头跨页自动重复，标题不吊底 */
    thead { display: table-header-group; }
    h2, h3 { page-break-after: avoid; break-after: avoid; }

    /* 尝试抑制孤行孤字（浏览器存在兼容差异） */
    p { orphans: 3; widows: 3; }

    /* 打印字号覆盖 */
    h1 { font-size: 32pt !important; }
    h2 { font-size: 20pt !important; margin-top: 20px !important; margin-bottom: 15px !important; }
    h3 { font-size: 14pt !important; margin-top: 15px !important; margin-bottom: 10px !important; }
    section { margin-bottom: 30px !important; }
    th, td { padding: 8px 12px !important; word-break: break-word !important; overflow-wrap: break-word !important; }

    /* ==========布局打印降级补丁========== */
    /* 双栏打印强制单栏（优先在 DOM 生成阶段就控制）*/
    .layout-columns { column-count: 1 !important; }
    /* 分屏 flex 打印改为上下流式 */
    .layout-split { display: block !important; }
    .layout-split .split-img,
    .layout-split .split-text { flex: none; width: 100%; }
    /* 拉引语录大号引号打印降级，防止裁切 */
    .layout-pullquote::before {
        font-size: 2rem !important;
        position: relative !important;
        top: auto;
        left: auto;
    }
    /* 打印关闭首字下沉，规避 float 分页撕裂灾难 */
    .layout-columns p::first-letter {
        font-size: inherit !important;
        float: none !important;
        margin-right: 0 !important;
        color: inherit !important;
        font-weight: inherit !important;
    }
}
```

## 四、标准组件库 CSS 模板（含移动端响应式，V2.2 修复表头断词）

```css
* { box-sizing: border-box; margin: 0; padding: 0; }
body {
    background-color: var(--bg-color);
    color: var(--text-main);
    font-family: "Noto Sans SC", "Source Han Sans SC", "思源黑体", "PingFang SC", sans-serif;
    line-height: 1.6;
}
.container { max-width: 1000px; margin: 0 auto; padding: 60px 40px 100px; }

/* 1. 基础表格，修复英文粗暴断词，改用智能换行 */
.table-wrapper { width: 100%; background: var(--white); border-radius: 8px; border: 1px solid var(--border-color); overflow-x: auto; margin-bottom: 30px; }
table { width: 100%; border-collapse: collapse; font-size: 0.95rem; }
th, td { padding: 16px 20px; border-bottom: 1px solid var(--border-color); vertical-align: middle; text-align: left; }
th { background-color: var(--primary); color: var(--white); font-weight: 700; word-break: keep-all; }
td { word-break: break-word; overflow-wrap: anywhere; }
tbody tr:nth-child(even) { background-color: #faf9f6; }

/* 2. 多栏阅读 (适合大段文字，Web 保留首字下沉，打印环境已经关闭首字下沉) */
.layout-columns { column-count: 2; column-gap: 40px; column-fill: balance; margin-bottom: 40px; border-top: 1px solid var(--border-color); padding-top: 30px; }
.layout-columns p { margin-top: 0; color: var(--text-light); text-align: justify; }
.layout-columns p::first-letter { font-size: 3.5rem; float: left; line-height: 1; margin-right: 8px; color: var(--primary); font-weight: 900; }

/* 3. 拉引语录 (适合核心金句) */
.layout-pullquote { text-align: center; margin: 50px 0; position: relative; padding: 0 40px; }
.layout-pullquote::before { content: "\201C"; font-size: 8rem; color: var(--gold); opacity: 0.3; position: absolute; top: -40px; left: 0; font-family: serif; }
.layout-pullquote blockquote { font-size: 1.8rem; font-weight: 700; color: var(--primary); line-height: 1.4; margin: 0; }
.layout-pullquote cite { display: block; margin-top: 20px; font-size: 0.9rem; color: var(--gold); font-weight: 700; font-style: normal; letter-spacing: 0.1em; }

/* 4. 全宽分屏 (适合案例展示、图文对照) */
.layout-split { display: flex; align-items: stretch; margin-bottom: 40px; background: var(--white); }
.split-img { flex: 1; background: var(--primary); min-height: 250px; } /* 如果有图片用 img 标签代替 */
.split-text { flex: 1; padding: 40px; display: flex; flex-direction: column; justify-content: center; }

/* 5. 大数字列表 (适合步骤、编年史) min-width 兼容两位数序号 01-12 */
.layout-index { margin-bottom: 40px; }
.layout-index-item { display: flex; align-items: flex-start; border-bottom: 1px solid var(--border-color); padding: 24px 0; }
.index-num { font-size: 4rem; font-weight: 900; color: var(--gold); line-height: 1; min-width: 120px; flex-shrink: 0; font-family: sans-serif; }

/* 6. 侧边旁注布局 主次信息分离 */
.layout-sidebar-note { display: grid; grid-template-columns: 1fr 3fr; gap: 32px; margin-bottom: 40px; align-items: start; }
.layout-sidebar-note .note-side { color: var(--gold); font-size: 0.9rem; font-weight: 500; }
.layout-sidebar-note .note-main { color: var(--text-main); }

/* 7. 图片与图注 (防止撑破版面) */
figure { margin: 30px 0; }
figure img { width: 100%; max-width: 100%; height: auto; border-radius: 8px; }
figcaption { margin-top: 10px; font-size: 0.85rem; color: var(--text-light); text-align: center; font-style: italic; }

/* 8. 卡片 (用于特征并列) */
.grid-2 { display: grid; grid-template-columns: repeat(auto-fit, minmax(350px, 1fr)); gap: 24px; margin-bottom: 24px; }
.card { background: var(--white); padding: 30px; border-radius: 8px; border-top: 3px solid var(--primary); }
.card-gold { border-top-color: var(--gold); }

/* =====================移动端响应式 768px 以下===================== */
@media (max-width: 768px) {
    .container { padding: 20px 16px 40px; }
    .layout-columns { column-count: 1 !important; }
    .layout-split { display: block !important; }
    .layout-split .split-img, .layout-split .split-text { flex: none; width: 100%; }
    .layout-sidebar-note { grid-template-columns: 1fr !important; }
    .layout-index-item { flex-wrap: wrap; }
    .index-num { min-width: auto; font-size: 2.5rem; margin-bottom: 8px; }
    .grid-2 { grid-template-columns: 1fr; }
}
```

## 五、HTML 骨架模板

> 将上面全部 CSS 变量、组件样式、打印媒体、移动端媒体完整复制进 `<style>` 标签；头部按需保留/移除 Google Fonts link。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>文档标题</title>
<!-- 离线内网请删除下面 link -->
<link href="https://fonts.googleapis.com/css2?family=Noto+Sans+SC:wght@400;500;700;900&display=swap" rel="stylesheet">
<style>
/* 粘贴上面完整 CSS：root 变量、全局样式、全部组件、@media print 打印、@media (max-width:768px) 移动端 */
</style>
</head>
<body>
<div class="container">
    <!-- 页面头部、章节、各布局组件由 AI 按照智能布局匹配规则生成 -->
</div>
</body>
</html>
```

## 六、AI 执行校验清单（生成 HTML 输出前必须自检）

- 原文档的每一句话、每一个数据、备注、合并单元格、多级编号、脚注图注是否完整保留，无删减扁平化？附件解析存在缺陷必须输出警告
- 是否存在长文本、长列表被强行塞入小卡片？长列表优先使用 `.layout-index`
- 表格全部包裹在 `.table-wrapper`；表头使用智能断词，无粗暴英文切割；超宽表格具备横向滚动
- 使用双栏 `.layout-columns` 时，确认模块预估高度；超过单页 A4，DOM 直接输出单栏
- 拉引语录不放在分页临界位置；分屏 `.layout-split` 尽量控制在单 A4 高度内
- 图片全部用 `figure + figcaption`，图片设置 `width:100%; height:auto`
- 打印模式大区块允许跨页，细粒度元素设置防 `break-inside:avoid`
- **离线交付专项校验：** 内网离线场景，确认无外部字体 CDN 依赖；图片输出 base64 占位标签，不直接生成 base64 长串，无 http 图片链接

## 📌 能力边界声明

✅ 适合：调研报告、项目方案、白皮书、业务分析文档，网页预览 + A4-PDF 交付；输入支持粘贴文本、可复制文本 PDF、docx、Markdown。

❌ 不适合：扫描图片 PDF；大量极度复杂合并单元格、深度嵌套文档；严格内网下大图片批量处理；专业印刷出版（无法达到 InDesign 精细排版级别）。

---

## 执行入口

需要直接投喂给 AI 的完整执行 Prompt（含任务说明、铁则、布局规则、输出格式要求），见 `references/prompt.md`。本文件与 prompt 两处语义必须保持一致。
