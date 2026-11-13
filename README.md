# 长文档转杂志风 HTML

> 把调研报告、白皮书、项目方案这类长文档，一次转换成**单文件静态 HTML**——浏览器里是杂志级版面，`Ctrl+P` 直接出 A4 PDF。
> 零遗漏保真、打印分页不撕裂、内网离线可交付。

[![License: MIT](https://img.shields.io/badge/License-MIT-103399.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-2.2.0-C8A666.svg)](CHANGELOG.md)
[![Agent Skill](https://img.shields.io/badge/Agent%20Skill-SKILL.md-F6F4EF.svg)](skills/longdoc-to-magazine-html/SKILL.md)

**[▶ 在线预览示例](examples/demo.html)** — 打开后按 `Ctrl+P` 查看真实 A4 分页效果。

---

## 效果预览

`examples/demo.html` 是完整可运行的示例页，内容取自本技能规范的第三章「打印/PDF 跨页完美控制」，覆盖表格、双栏、拉引语录、大数字列表、分屏、侧边旁注、卡片、代码块全部八类组件。

它的价值不只是好看——**它就是这套 CSS 的自证**：打印预览里的分页行为，正是文档所描述的规则。

| 预览态 | 打开方式 |
| --- | --- |
| 网页 | 双击 `examples/demo.html` |
| A4 打印 | 同上，`Ctrl+P` / `⌘+P` |
| 移动端 | 浏览器开发者工具切到 375px 宽 |

## 快速开始

1. **[必需]** 安装技能到本地 agent：

```bash
npx skills add echoBai83/longdoc-to-magazine-html
```

或克隆后手动拷入：

```bash
git clone https://github.com/echoBai83/longdoc-to-magazine-html.git
cp -r longdoc-to-magazine-html/skills/longdoc-to-magazine-html ~/.claude/skills/
```

2. **[必需]** 直接对 agent 说话，技能自动触发：

```
把这份报告排版成杂志风 HTML，我要能打印成 A4 PDF
```

3. **[可选]** 想手动控场，从 `skills/longdoc-to-magazine-html/references/prompt.md` 复制完整执行 Prompt，粘贴到对话开头，替换末尾的源文档占位符。

4. **[可选]** Claude Code 插件市场安装：

```bash
claude plugin marketplace add echoBai83/longdoc-to-magazine-html
claude plugin install magazine-html-typeset@magazine-html-typeset
```

## 能力矩阵

| 输入源 | 支持 | 说明 |
| --- | :-: | --- |
| 粘贴纯文本 | ✅ | 保留段落与层级 |
| Markdown 附件 | ✅ | 标题、表格、列表、代码块、链接、图片引用原样保留 |
| PDF（可复制文本） | ✅ | 保留合并单元格、多级编号、脚注图注 |
| Word (.docx) | ✅ | 同上，丢弃页眉页脚与水印 |
| 扫描图片 PDF | ❌ | 不做 OCR，会明确告知无法解析 |

| 输出目标 | 支持 | 关键实现 |
| --- | :-: | --- |
| 单文件静态 HTML | ✅ | 全部 CSS 内联，无业务 JavaScript |
| A4 打印 / 另存 PDF | ✅ | `@page` + `break-inside` 分级控制 |
| 移动端阅读 | ✅ | `@media (max-width: 768px)` 全布局降级 |
| 内网离线交付 | ✅ | 移除字体 CDN，图片走 base64 占位标签 |

**内置八类布局组件**：`.table-wrapper` 表格 · `.layout-columns` 双栏 · `.layout-pullquote` 拉引语录 · `.layout-index` 大数字列表 · `.layout-split` 分屏 · `.layout-sidebar-note` 侧边旁注 · `figure + figcaption` 图注 · `.card` / `.grid-2` 卡片网格。

## 仓库结构

```
longdoc-to-magazine-html/
├── README.md
├── LICENSE
├── CHANGELOG.md
├── .gitignore
├── .claude-plugin/
│   └── marketplace.json              # 插件市场注册（magazine-html-typeset）
├── skills/
│   └── longdoc-to-magazine-html/
│       ├── SKILL.md                  # 唯一规范源：原则 + 布局规则 + 完整 CSS + 自检清单
│       └── references/
│           └── prompt.md             # 可直接投喂 AI 的执行 Prompt
└── examples/
    ├── demo.html                     # 可双击运行、可直接打印的示例页
    └── demo-source.md                # 示例的源文档，用于验证输入→输出链路
```

## 设计要点

**分页按元素粒度分级授权。** 不是「全都别跨页」——那会在每页底部堆出无法填充的空白。容器体量大、内部可再分（`section`、`.table-wrapper`），允许自然跨页；一旦断开就破坏连贯性的元素（`tr`、`h2`、`.card`、`figure`），整体锁定单页。

**表头与单元格的断词策略必须分流。** `th` 用 `word-break: keep-all` 保词组完整，`td` 用 `break-word` + `overflow-wrap: anywhere` 只断长串。统一用 `break-all` 会把 `Print` 切成 `Pri`/`nt`，这是 V2.2 修掉的真实缺陷。

**Base64 图片禁止由 AI 直出。** 大体积 base64 会撞上输出 token 上限，导致 HTML 在图片位置被截断、整份产物损坏。技能只输出占位标签 `<img src="data:image/jpeg;base64,【请在此粘贴Base64编码】">`，由人工补入。

**打印态四个强制降级。** 双栏 → 单栏；分屏 flex → 上下流式；8rem 伪引号 → 2rem 相对定位防裁切；首字下沉的 `float` 彻底关闭，规避分页撕裂。

**单一源真相。** `SKILL.md` 是唯一规范源，`references/prompt.md` 由其末尾萃取同步，两处语义必须一致。改规范只改 `SKILL.md`。

## 已知限制

- **不做 OCR**：扫描图片 PDF 无法解析，会直接告知用户，不猜测补全。
- **复杂合并单元格**：极度复杂的跨行跨列合并、深度嵌套文档，打印时可能出现排版局限。
- **达不到 InDesign 水平**：这是 CSS 排版的边界，专业印刷出版请用专业排版工具。
- **内网大批量图片**：严格离线环境下大图片需人工逐个补 base64，流程较重。
- **`orphans` / `widows` 浏览器差异**：Firefox 支持完善，Chrome 支持有限，属优化手段而非硬保证。

## 相关文件

- 完整规范：[`skills/longdoc-to-magazine-html/SKILL.md`](skills/longdoc-to-magazine-html/SKILL.md)
- 执行 Prompt：[`skills/longdoc-to-magazine-html/references/prompt.md`](skills/longdoc-to-magazine-html/references/prompt.md)
- 版本变更：[`CHANGELOG.md`](CHANGELOG.md)

## 许可

代码与文档采用 [MIT License](LICENSE)。

仓库中引用的思源黑体 **Noto Sans SC** 遵循 [SIL Open Font License 1.1](https://scripts.sil.org/OFL)，不随本仓库的 MIT 授权，仅通过 Google Fonts CDN 引用或依赖系统字体回退。
