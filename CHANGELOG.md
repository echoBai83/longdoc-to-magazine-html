# Changelog

本项目的所有重要变更都记录在此文件。

格式基于 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/)，
版本号遵循 [语义化版本 SemVer](https://semver.org/lang/zh-CN/)。

版本历史放在仓库根目录、**不放入 `skills/` 技能目录**——这样 agent 执行技能时不会把版本记录加载进上下文。

## [2.2.0] - 封板生产版

本版本为 V2.2 封板生产版，首次以 Agent Skill 形式发布到 GitHub。

### Added

- **附件输入解析规则**：支持 PDF / Word / Markdown 三类附件，含解析失败降级策略与「禁止脑补补全」硬约束
- **离线 / 内网交付强制规则**：移除字体 CDN、禁止 http 图片外链、Base64 图片仅输出占位标签（避免 AI 直出长串触发输出截断）
- **移动端响应式**：`@media (max-width: 768px)` 覆盖双栏、分屏、侧边旁注、大数字列表、卡片网格五类布局
- **打印态降级补丁**：双栏强制单栏、分屏转上下流式、伪引号缩位防裁切、关闭首字下沉规避 float 分页撕裂
- **字号双轨制**：Web 端 rem、Print 端 pt，拒绝 1.5 倍等比缩放
- 官方示例 `examples/demo.html`：以「第三章 打印/PDF 跨页完美控制」为内容，可双击运行、可直接 Ctrl+P
- 仓库治理文件：README、LICENSE、.gitignore、`.claude-plugin/marketplace.json`（插件名 `magazine-html-typeset`）
- 执行 Prompt 拆分为 `references/prompt.md`，避免全量规范重复进入上下文

### Fixed

- **表头英文粗暴断词**：旧版对 `th` 与 `td` 统一使用 `word-break: break-all`，导致 `Print` 被切成 `Pri`/`nt`、`A4-PDF` 被切成 `A4-P`/`DF`。现分流为 `th { word-break: keep-all }` 与 `td { word-break: break-word; overflow-wrap: anywhere }`
- **Google Fonts 链接失效**：旧文档中 `<link href>` 被 Markdown 链接语法污染成 `href="[https://...](https://...)"`，浏览器请求非法 URL，字体加载静默失败。现改为裸 URL
- **打印规则指向未定义类**：打印媒体查询引用了 `.quote-block`、`.list-block` 两个从未在组件库中定义的 class，已移除
- **多栏填充行为不确定**：`.layout-columns` 的 `column-fill: auto` 在不定高容器中浏览器表现不一致，改为 `balance`

### Changed

- 单一源真相原则明确化：`SKILL.md` 为唯一规范源，`references/prompt.md` 由其萃取同步，两处语义必须一致

## [2.1] - 内部版本

### Added

- 附件输入解析规则雏形（PDF / Markdown）

### Changed

- 布局匹配规则补充「备注过长降级普通段落」等例外分支

## [2.0] - 内部版本

### Added

- 智能布局匹配规则：按内容类型自动选择表格、双栏、拉引语录、大数字列表、分屏、旁注、卡片
- 标准组件库 CSS：八类布局组件 + 移动端响应式
- 输出前自检清单

### Changed

- 从「固定模板套用」改为「先提取完整 DOM，再按规则套壳」

## [1.0] - 初始版本

### Added

- A4 打印分页控制基础策略：`@page`、`break-inside` 分级、`thead` 跨页重复
- 杂志风视觉基底：深蓝 `#103399` + 香槟金 `#C8A666` + 暖米 `#F6F4EF` 配色体系
