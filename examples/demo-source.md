# 打印 / PDF 跨页完美控制

> 本文是 `longdoc-to-magazine-html` 技能的官方示例，内容取自技能规范的第三章「打印/PDF 跨页完美控制」，用于验证本套 CSS 在真实打印场景下的分页表现。

## 01 核心原则

大区块允许跨页，细粒度元素防截断。这条原则解决的是排版里两个互相拉扯的诉求：如果所有区块都禁止跨页，页面底部会堆出大量无法填充的空白；如果所有区块都允许跨页，一张卡片、一段语录就可能被撕成两半，横跨两张纸。

正确的做法是按元素粒度分级授权——`section`、`.table-wrapper`、`.layout-columns` 这类容器体积大、内部可再分，允许它们自然跨页；而 `tr`、`h2`、`h3`、`.card`、`.layout-index-item`、`.layout-split`、`figure`、`.layout-sidebar-note` 这些一旦断开就会破坏阅读连贯性的元素，必须整体锁定在单页内。下面这张表记录了当前版本的分级策略。

| 元素 / 选择器 | 分页策略 | 设定理由 | 违规后果 |
| --- | --- | --- | --- |
| `section` | 允许跨页 `break-inside: auto` | 章节体量大，内部段落与卡片可自然承接 | 强制不跨页会在大章节前留下整页空白 |
| `.table-wrapper` | 允许跨页 | 配合 `thead` 重复表头，长表格可延续 | 长表格被挤到下一页，前半页大面积留白 |
| `.layout-columns` | 允许跨页 | 打印态已强制降级为单栏，跨页即普通段落流 | — |
| `tr` | 禁止跨页 `break-inside: avoid` | 单行数据被撕开会丢失对应关系 | 金额与项目名分处两页，产生误读 |
| `h2` / `h3` | 禁止跨页 + `break-after: avoid` | 标题必须与后续正文同页 | 标题孤悬页底，正文全部跑到下页 |
| `.card` | 禁止跨页 | 卡片是信息自足单元 | 卡片边框被切开，视觉残缺 |
| `.layout-index-item` | 禁止跨页 | 大数字与说明文字必须成对 | 序号在上一页、释义在下一页 |
| `figure` | 禁止跨页 | 图片与图注不可分离 | 图注与图片错页，注释失去指向 |
| `.layout-split` | 禁止跨页 | 分屏图文对照必须同框 | 左右两栏被分页割裂成上下两页 |

## 02 浏览器兼容现实

`orphans` 与 `widows` 是控制孤行孤字的两个属性：`orphans` 约束页面底部的段落最少保留几行，`widows` 约束页面顶部的段落最少承接几行。Firefox 对这两个属性支持完善，Chrome / Chromium 的支持则相当有限。

这意味着不能把它们当作确定性保证。它们是排版优化手段，能改善平均表现，但**不能保证 100% 消除孤行**。在交付文档中如此声明，比承诺「绝对不会出现孤行」更诚实。

> 兼容性声明是交付质量的一部分。承诺确定的结果却做不到，比提前说明边界更伤害信任。

```css
@media print {
    p { orphans: 3; widows: 3; }
}
```

## 03 打印态强制降级补丁

Web 端为了视觉张力使用了双栏、分屏、超大引号和首字下沉。这些手法在屏幕上成立，在纸上会逐个失效：双栏在 A4 窄幅下每栏只剩约 90mm，中文阅读节奏被打碎；分屏的 flex 布局在分页时无法优雅断开；8rem 的巨型引号会被页边距裁切；首字下沉依赖 `float`，一旦落在分页线附近就会引发整块内容错位。

因此打印媒体查询中必须包含四个降级补丁：

1. `.layout-columns` 强制 `column-count: 1`，双栏回归单栏阅读。
2. `.layout-split` 改为 `display: block`，子元素宽度 100%，分屏转上下流式。
3. `.layout-pullquote::before` 降到 2rem 并改为相对定位，避免被裁切。
4. `.layout-columns p::first-letter` 恢复继承字号、取消 `float`，彻底关闭首字下沉。

```css
@media print {
    .layout-columns { column-count: 1 !important; }

    .layout-split { display: block !important; }
    .layout-split .split-img,
    .layout-split .split-text { flex: none; width: 100%; }

    .layout-pullquote::before {
        font-size: 2rem !important;
        position: relative !important;
        top: auto;
        left: auto;
    }

    .layout-columns p::first-letter {
        font-size: inherit !important;
        float: none !important;
        margin-right: 0 !important;
        color: inherit !important;
        font-weight: inherit !important;
    }
}
```

## 04 表头断词：V2.2 的关键修复

V2.2 修复了一个隐蔽但影响明显的问题：表头英文与数字被粗暴切断。旧版本在 `th` 和 `td` 上统一使用 `word-break: break-all`，浏览器为了填满单元格会从任意字符处断行，导致 `Print` 被切成 `Pri` 与 `nt`，`A4-PDF` 被切成 `A4-P` 与 `DF`。

修复方案是按元素分流：`th` 表头通常只有短词组，使用 `word-break: keep-all` 保持词组完整；`td` 单元格可能包含长 URL 或长英文串，使用 `word-break: break-word` 配合 `overflow-wrap: anywhere`，只在必要时断行，且不切断单词中间的字符。

```css
th { word-break: keep-all; }
td { word-break: break-word; overflow-wrap: anywhere; }
```

## 05 完整打印媒体查询

以下为可直接投入生产的完整打印样式块。注意 `-webkit-print-color-adjust: exact` 与 `print-color-adjust: exact` 必须同时声明，否则 Chrome 会丢弃深蓝表头与香槟金强调色，打印出灰白文档。

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

    section, .table-wrapper, .layout-columns {
        page-break-inside: auto !important;
        break-inside: auto !important;
    }

    tr, h2, h3, .card, .layout-index-item,
    .layout-split, figure, .layout-sidebar-note {
        page-break-inside: avoid !important;
        break-inside: avoid !important;
    }

    thead { display: table-header-group; }
    h2, h3 { page-break-after: avoid; break-after: avoid; }
    p { orphans: 3; widows: 3; }

    h1 { font-size: 32pt !important; }
    h2 { font-size: 20pt !important; }
    h3 { font-size: 14pt !important; }
}
```

## 06 交付前自检

按顺序核对以下七项，任何一项不通过都不得交付：

- 大区块是否允许跨页，细粒度元素是否全部锁定 `break-inside: avoid`
- `thead` 是否声明 `display: table-header-group`，长表格是否跨页重复表头
- `h2` / `h3` 是否设置 `break-after: avoid`，标题是否仍会孤悬页底
- 四个降级补丁是否全部写入打印媒体查询
- `th` 是否使用 `keep-all`，`td` 是否使用 `break-word` + `anywhere`
- `print-color-adjust: exact` 是否双前缀声明
- 移动端 `@media (max-width: 768px)` 是否覆盖双栏、分屏、旁注、网格四类布局

> 自检的价值不在于清单本身，而在于它把「我觉得没问题」换成「我逐条验证过」。

## 07 版本说明

本章内容对应 `longdoc-to-magazine-html` V2.2 封板生产版。打印分页策略自 V1.0 起保持稳定，V2.2 的主要变更集中在表头断词分流与移动端响应式补全。若需查看完整技能规范，请返回仓库根目录阅读 `SKILL.md`。
