# 执行 Prompt｜长文档转杂志风 HTML

> 本文件由 `SKILL.md` 萃取，语义必须与其保持一致。修改时先改 `SKILL.md`，再同步此处。
> 用法：从下面的代码块复制全部内容，粘贴到对话开头，替换末尾【此处粘贴你的源文档内容/上传附件】。

```
【任务：长文档转杂志风 HTML｜V2.2 生产封板版】
支持输入源：直接粘贴文本 / 上传 PDF、Word、Markdown 附件；扫描图片 PDF 不支持解析。
你需要把输入文档转换为单文件静态杂志风 HTML，支持网页浏览 + A4 纸张打印另存 PDF。
---
## 核心铁则（严禁违反）
1. 零遗漏：原文所有段落、表格、备注、合并单元格、多级编号、脚注、图注全部保留，禁止概括、扁平化改写。先还原完整 DOM 结构，再套 CSS 类。
> 附件解析规则：
> • PDF/Word：提取正文、表格合并关系、标题层级、内嵌图片；丢弃自动页眉页脚、水印、自动页码、修订痕迹；手动批注/脚注必须保留为备注。
> • Markdown：保留标题、表格、列表、链接、图片引用，不篡改原有语义。
> • 扫描图片 PDF：直接告知用户无法解析，不要强行生成。
> • 解析出现缺失/错乱，必须输出解析警告，禁止脑补编造原文内容。
2. 产物为纯静态 HTML，无业务 JS；AI 布局判断仅在生成阶段完成，运行不依赖 AI。
3. 配色固定：主色 #103399 深蓝，辅色 #C8A666 香槟金，背景 #F6F4EF 暖米，正文 #1a1a1a，辅助文字 #555555，卡片背景 #ffffff。
4. 字体优先思源 Noto Sans SC；联网引入 Google Fonts；内网离线场景移除该 CDN 链接。

## 🧠 智能布局选择规则（生成 DOM 时执行）
- 对比并列数据 → <table>，外层必须包 .table-wrapper
- 正文段落 300-1200 字、预估高度小于 1 页 A4 → .layout-columns 双栏 + 首字下沉；超限直接输出单栏
- 原文金句/核心摘抄 → .layout-pullquote 拉引语录，禁止放在分页临界点
- 有序步骤、时间线、排行 ≥2 条 → .layout-index + .layout-index-item 大数字编年史，优先替代卡片
- 碎片化并列特征 → .card 卡片，禁止把大段正文塞进卡片
- 图文对照案例 → .layout-split 全宽分屏，尽量控制在单 A4 高度
- 主文本+简短备注来源 → .layout-sidebar-note 侧边旁注；备注过长降级普通段落

## ⚠️ 离线/内网特殊规则
1. 联网环境：图片允许 http 链接。
2. 离线内网：禁止输出 http 图片 URL；禁止 AI 直接输出大段 Base64 字符串（会触发输出截断损坏 HTML），仅输出占位：<img src="data:image/jpeg;base64,【请在此粘贴Base64编码】">，由人工替换编码。
> 附件提取的内嵌图片同样遵守本条占位规则。

## 🖨 打印 & 浏览器兼容硬性约束（CSS 必须完整写入 <style>）
1. @page 设置 A4、margin:12mm；区分大区块允许跨页，卡片、条目、figure 等细粒度元素 break-inside:avoid 防截断。thead 实现表格跨页表头重复；h2/h3 设置 break-after:avoid 防止标题孤悬底部。body 必须同时声明 -webkit-print-color-adjust: exact 与 print-color-adjust: exact，否则 Chrome 会丢弃深蓝与香槟金配色。
2. 打印媒体中：双栏强制单栏；分屏布局改为上下流式；拉引语录伪引号缩小改为相对定位；打印环境强制关闭首字下沉，规避 float 分页撕裂 bug。
3. th 表头使用 word-break:keep-all 保持词组完整；td 单元格使用 word-break:break-word; overflow-wrap:anywhere。禁止粗暴 word-break:break-all，防止英文单词中间硬切断。
4. p 设置 orphans:3; widows:3，知晓 Chrome 对此支持有限，仅作优化。
5. 增加移动端 @media (max-width:768px) 响应式：双栏→单栏、分屏改为块级、大数字列表自适应、网格变为单列。

## 📐 字号与排版规范
- Web 端：正文 1rem；H3 1.25rem；H2 2rem；H1 clamp(2.5rem, 5vw, 4rem)
- Print (@media print pt 单位)：正文 11pt；H3 14pt；H2 20pt；H1 32pt
- 视觉基准值：卡片内边距 24px-30px，组件之间垂直间距 32px-40px，章节间 60px-80px。确保大面积留白，避免拥挤。

## 🧩 必须内置的组件 class
.table-wrapper / .layout-columns / .layout-pullquote / .layout-split / .layout-index / .layout-index-item / .layout-sidebar-note / figure+figcaption 图注 / .card / .card-gold / .grid-2

## ✅ 输出前自检清单
- [ ] 原文信息无丢失；合并单元格、多级列表 DOM 忠实还原；附件解析有缺失要给出警告
- [ ] 长文本没有强行塞入卡片；表格外层带 .table-wrapper
- [ ] 双栏仅用于短模块，超 A4 高度直接 DOM 层面输出单栏
- [ ] 离线模式图片输出 base64 占位标签，不输出长 base64 串
- [ ] 打印降级 CSS 全部包含，关闭打印端首字下沉；表头不断词、单元格智能断词
- [ ] 包含移动端响应式媒体查询

## 输出格式要求
1. 先输出【文档大纲与布局分配说明】，使用普通自然语言文本，**不要写 HTML 代码**；逐条说明每一个章节/模块对应的选用布局，用于人工校验布局逻辑。如果附件解析存在缺陷，在此处增加解析警告。
2. 再输出完整独立 HTML 文件代码，所有 CSS 全部内联在内部 <style> 标签。不要输出 markdown 解释，直接输出 ```html ... ``` 代码块。

> 边界提醒：该方案达不到 InDesign 专业印刷；极度复杂合并单元格文档会存在排版局限；扫描图片 PDF 无法解析文本。
---
现在处理下面这份文档/上传附件，输出最终 HTML：
【此处粘贴你的源文档内容/上传附件】
```
