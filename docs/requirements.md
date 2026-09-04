# TXT → EPUB 在线编辑器 需求文档（PRD）

| 项目 | 内容 |
| --- | --- |
| 项目名称 | Txt2Epub Studio（暂定） |
| 文档版本 | v1.0 |
| 创建日期 | 2026-09-04 |
| 文档状态 | 待评审 |
| 产品定位 | 纯浏览器端的 TXT 小说/长文转 EPUB 电子书工具：上传 → 自动分章 → 可视化编辑 → 排版美化 → 一键导出 EPUB |

---

## 1. 项目背景与目标

### 1.1 背景

网络小说、连载长文、同人作品等大量内容以纯文本 `.txt` 形式流通。这类文件存在以下痛点：

1. **无结构**：没有章节、目录，阅读器无法跳转与生成 TOC。
2. **无排版**：正文与对话、旁白、书名混在一起，缺乏视觉层次。
3. **编码混乱**：中文 TXT 常见 `UTF-8 / GBK / GB18030 / BIG5`，直接用编辑器打开常出现乱码。
4. **转换门槛高**：现有工具（Calibre、Sigil）需要本地安装、学习成本高，且不支持「所见即所得地逐段微调样式」。

### 1.2 产品目标

- **零安装、零注册**：打开网页即用，文件不上传服务器（隐私友好）。
- **三步出书**：上传 → 编辑 → 导出，全流程 5 分钟内完成。
- **中文语境优先**：针对中文小说的对话符号（`「」『』“ ”`）与标记符号（`【】（）《》〈〉`）提供一键语义化高亮。
- **国际风视觉**：界面采用国际化（International / Minimal）设计语言——大留白、中性色、无衬线字体、克制圆角，避免二次元/拟物化风格。

### 1.3 成功指标（参考）

| 指标 | 目标值 |
| --- | --- |
| 10MB TXT 上传→分章完成耗时 | < 3s |
| 100 章 / 50 万字 导出 EPUB 耗时 | < 10s |
| 导出文件通过 epubcheck 校验 | 0 error |
| 主流阅读器兼容性 | Apple Books、Calibre、静读天下、Kindle（通过 Kindle Previewer）显示正常 |

---

## 2. 用户与场景

| 角色 | 场景描述 |
| --- | --- |
| 网文读者 | 把连载 TXT 打包成 EPUB，导入手机阅读器离线追更 |
| 同人/原创作者 | 把草稿整理成带目录、带排版的 EPUB 分发给读者 |
| 翻译组 | 把译稿按章节切分、统一对话样式后发布 |
| 资料整理者 | 把长篇教程、访谈录转成结构化电子书归档 |

---

## 3. 功能需求

> 优先级：**P0 = 必须（MVP）**，P1 = 重要，P2 = 增强

### F1 文件导入（P0）

| 编号 | 需求 | 优先级 |
| --- | --- | --- |
| F1-1 | 支持点击选择 `.txt` 文件；支持拖拽文件到页面任意区域上传 | P0 |
| F1-2 | 支持同时导入多个 TXT，自动识别为「同一本书的多个分卷」或「多本书」（让用户选择） | P1 |
| F1-3 | **编码自动检测**：依次尝试 `UTF-8(with/without BOM)` → `GB18030` → `GBK` → `BIG5` → `UTF-16LE/BE`，并在 UI 显示检测结果，允许用户手动切换后重新解析 | P0 |
| F1-4 | 支持 `.epub` 导入并反向解析为可编辑工程（读取 `content.opf` / `nav.xhtml` / 章节 xhtml） | P2 |
| F1-5 | 单文件大小上限 50MB，超出时给出友好提示与截断建议 | P1 |
| F1-6 | 导入后展示原始文本统计：总字符数、行数、段落数、预计阅读时长 | P1 |

### F2 智能分章（P0）

| 编号 | 需求 | 优先级 |
| --- | --- | --- |
| F2-1 | **分章即一条正则**：逐行匹配，命中即为章节标题，**不含任何附加启发式条件**（不看上一行结尾、不算行长度、不检查标点）。<br>默认正则（中文网文）：<br>`^[ \t\u3000\u00A0\uFEFF]*(?:第[0-9零一二三四五六七八九十百千万两]{1,12}[章节回卷篇集部幕]\|序章\|序言\|楔子\|引子\|前言\|内容简介\|番外\|外传\|特别篇\|后记\|尾声\|终章\|附录\|自序\|总序\|序)[^\n]{0,80}$` | P0 |
| F2-1a | 行首空白（半角空格、Tab、全角空格　、不间断空格）由正则开头的字符类统一处理；标题文本保存时去除首尾空白。<br>章号后无论是否接分隔符、是否接文字、接什么文字，一律命中。<br>因此 `第一章`、`　　第一章`、`　第一章初入`、`第1章：开端`、`番外张三的回忆`、`尾声` 全部命中 | P0 |
| F2-1b | 正则在**导入弹窗**与「排版」面板中均为**可直接编辑的输入框**，切换预设自动填入对应正则；可配合「按当前正则重新分章」反复调整 | P0 |
| F2-1c | 正则输入框内容变化只**局部刷新章节预览**，不重建弹窗（否则输入框失焦、光标跳走，无法连续输入） | P0 |
| F2-1d | 正则非法时显示「正则无效」并保留上次结果，不抛异常 | P0 |
| F2-1e | 已知取舍：纯正则下，正文中以「第X章」开头的行也会被当作标题（可通过自定义正则收紧，如加 `(?<![。！？])$` 排除以句号结尾的行） | — |
| F2-2 | 提供分章预设：`中文网文` / `出版书籍` / `英文原著` / `自定义正则` | P0 |
| F2-3 | 支持在「全文预览」中手动插入/删除分章断点（点击行首图标切换） | P0 |
| F2-4 | 章节列表支持：重命名、拖拽排序、合并、拆分、删除、新增空白章 | P1 |
| F2-5 | 自动清理：多余空行、行首全角空格、页眉页脚（如重复出现的站点广告行，支持一键移除包含指定关键词的行） | P1 |
| F2-6 | 章节目录树支持两级（卷 → 章），用于生成 EPUB 多级 TOC | P1 |

### F3 正文编辑（P0）

| 编号 | 需求 | 优先级 |
| --- | --- | --- |
| F3-1 | 富文本所见即所得编辑器：支持段落、标题（H1–H4）、加粗、斜体、下划线、删除线、引用块、分割线、无序/有序列表 | P0 |
| F3-2 | **双模式切换**：`富文本模式` ⇄ `纯文本/源码模式`（源码模式直接编辑 XHTML，供高级用户调试） | P1 |
| F3-3 | 段落操作：合并段落、拆分段落、删除空段、段首自动缩进两格（全角空格或 CSS `text-indent`） | P0 |
| F3-4 | 撤销/重做（`Ctrl/Cmd+Z` / `Ctrl+Shift+Z`），历史步数 ≥ 100 | P0 |
| F3-5 | 查找与替换（支持正则、区分大小写、全量替换、逐条替换） | P1 |
| F3-6 | 标点规范化：半角→全角（` , . ! ? : ;`）、英文引号→中文引号、连续省略号归一为 `……` | P1 |
| F3-7 | 繁简转换（开放中文转换 或 简繁映射表） | P2 |
| F3-8 | 实时字数统计（当前章 / 全书），自动保存草稿 | P1 |
| F3-9 | 批量操作：对「所有章节」或「选中章节」统一应用样式规则/替换 | P1 |

### F4 图片支持（P0）

| 编号 | 需求 | 优先级 |
| --- | --- | --- |
| F4-1 | 在光标位置插入本地图片（支持 `jpg / png / gif / webp / svg`），支持拖拽图片到编辑器、支持粘贴剪贴板图片 | P0 |
| F4-2 | 图片以 data URL 存入 IndexedDB，刷新不丢失 | P0 |
| F4-2a | **图片去重**：按内容指纹（长度 + FNV-1a + djb2 双哈希）判重，内容相同的图片复用同一资源，<br>① 不产生重复资源 ② 导出时只写一份文件、manifest 只声明一条。<br>不采用 `crypto.subtle`（以 `file://` 打开时非安全上下文不可用） | P0 |
| F4-2b | **图片库交互**：格子点击 = **插入到当前章节**；设为封面用 ★；删除用 ✕（正文中仍有引用时弹确认框，可选「一并删除」或「仅从库中移除」）。<br>插入前必须校验选区是否位于编辑器内，否则从图片库点击会把图片插进图片库自身 | P0 |
| F4-2c | **正文图片删除**（原生 contenteditable 删除行为不可靠：点击图片时光标未必落在 figure 上，Delete/Backspace 常无效，且易残留看不见的空 `<figure>`）。<br>方案：点击图片 → 选中 `figure`（蓝色轮廓）→ 点 ✕ 或按 Delete/Backspace 删除**整个 figure**；点击正文取消选中。<br>实现要点：✕ 按钮与 `.t2e-sel` 是临时 UI，**必须在保存与导出前剥离**（否则会写进正文）；导出路径需再做一次兜底清理并移除空 `figure` | P0 |
| F4-2d | 插入图片后自动选中并滚动到可见位置，让用户立刻可操作 | P1 |
| F4-2e | 「清理未使用的图片」：删除未被任何章节引用、且非封面的图片（带确认框与列表） | P1 |
| F4-3 | 图片排版：居左/居中/居右、宽度百分比调节（25/50/75/100%）、添加图注（自动编号：`图 1-1`） | P1 |
| F4-4 | 图片压缩：超过阈值（默认 1MB / 宽 1600px）自动等比压缩，可开关 | P2 |
| F4-5 | 设置某一张图为「封面」（封面页 + `cover-image` 元数据），支持上传独立封面图 | P0 |
| F4-6 | 图片管理器：列表展示已用图片、显示引用章节、替换/删除 | P1 |

### F5 语义高亮（核心差异化功能，P0）

> 需求原文要点：**对话加绿色下划线**；**`【】「」（）《》` 等不同括号使用不同样式**。

#### F5-1 预置规则表（默认配置，全部可改）

| 符号 | 语义 | 默认样式（浅色主题） | CSS 示意 |
| --- | --- | --- | --- |
| `「」` `『』` | **对话/对白** | 绿色下划线（实线，2px，下偏移 3px），正文色不变 | `text-decoration: underline; text-decoration-color: #16A34A; text-decoration-thickness: 2px; text-underline-offset: 3px;` |
| `“”` `‘’` | 对话（直/弯引号） | 同对话样式（绿色下划线） | 同上 |
| `" "` `' '` | 西文对话 | 同对话样式（绿色下划线） | 同上 |
| `【】` | 系统提示 / 旁白 / 状态面板 / 重点标注 | 加粗 + 琥珀色（Amber）文字 + 极淡琥珀底纹 | `font-weight:700; color:#B45309; background:#FFFBEB; border-radius:2px; padding:0 .15em;` |
| `（）` `()` | 补充说明 / 心理活动 / 舞台提示 | 斜体 + 灰蓝文字 + 点线下划线 | `font-style:italic; color:#64748B; text-decoration: underline dotted #94A3B8;` |
| `《》` | 书名 / 功法 / 技能 / 曲目名 | 靛蓝加粗 + 波浪下划线 | `font-weight:600; color:#4338CA; text-decoration: underline wavy #6366F1;` |
| `〈〉` `<>` `《》` 别名 | 篇名 / 卷名 | 深青色加粗 | `font-weight:600; color:#0F766E;` |
| `｛｝` `[]` | 自定义（默认：编辑批注，不导出时隐藏） | 灰色小字 | `font-size:.85em; color:#9CA3AF;` |

> 说明：以上「CSS 示意」为 EPUB 内嵌样式表与 Web 预览共用的一组变量，主题切换时色值整体替换。

#### F5-2 规则配置能力

| 编号 | 需求 | 优先级 |
| --- | --- | --- |
| F5-2-1 | 每条规则可配置项：开/关、左符号、右符号、语义名称、文字颜色、背景色、字重、斜体、下划线（无/实线/虚线/点线/波浪）、下划线颜色、下划线粗细、字号 | P0 |
| F5-2-2 | 新增自定义括号规则（如 `** **` 表示强调、`~~ ~~` 表示删除线） | P1 |
| F5-2-3 | 规则优先级排序：先匹配的规则先生效，避免嵌套冲突 | P1 |
| F5-2-4 | 支持「跨段落匹配」（括号内换行也能正确包裹，如长段对白跨行） | P1 |
| F5-2-5 | **应用方式三选一**：① 仅预览/导出时按正则实时渲染（不改原文）② 一键「标记化」写入文档结构（`<span class="dlg">`）③ 两者混合（默认推荐：②） | P1 |
| F5-2-7 | **标记不得改变任何文字**：`<span>` 必须包裹「符号 + 内容 + 符号」的完整区间，**严禁丢弃或截断括号/引号**。<br>实现约束：区间统一采用左闭右开 `[start, end)` 语义；递归渲染子区间时必须先过滤出严格位于父区间内部的子集，否则会重复渲染父区间导致无限递归或丢字。<br>验收：标记前后 `textContent` 必须完全相等 | P0 |
| F5-2-6 | 规则集可导出/导入 JSON，便于复用 | P2 |

#### F5-3 降级兼容（P0，重要）

- 部分旧阅读器（Kindle 老固件、部分安卓阅读器）**不支持 `text-decoration-color` / `text-decoration-style: wavy`**。
- 解决方案：导出时生成**两套样式**：
  - 主样式：`text-decoration: underline; text-decoration-color: <color>;`
  - 降级样式：`border-bottom: 2px solid <color>; padding-bottom: 1px;`（通过 `@supports not (text-decoration-color: green)` 自动切换）
- 波浪线/虚线降级为实线或 `border-bottom: 2px dashed/dotted`。

### F6 元数据与封面（P0）

| 编号 | 需求 | 优先级 |
| --- | --- | --- |
| F6-1 | 书名（必填）、副标题、作者、译者、丛书、出版社、语言（`zh-CN` / `zh-TW` / `en` 下拉）、简介（多行）、标签、ISBN、出版日期 | P0 |
| F6-2 | 封面：上传本地图片 / 由模板自动生成（书名+作者+纯色/渐变背景，内置 6 套国际风封面模板） | P1 |
| F6-3 | 页码方向：`ltr`（默认）；`rtl` 与 `vertical-rl`（竖排，日轻/繁体书）开关 | P2 |
| F6-4 | 生成版权页与扉页（可开关） | P2 |

### F7 排版与主题（P1）

| 编号 | 需求 | 优先级 |
| --- | --- | --- |
| F7-1 | 正文字体：`衬线（Noto Serif SC / 思源宋体）` / `无衬线（Inter / 思源黑体）` / `系统默认`，标题字体独立选择 | P0 |
| F7-2 | 正文字号（14–24px）、行高（1.4–2.2）、段间距（0–1.5em）、首行缩进（0/2字符）、页边距（四档） | P0 |
| F7-3 | 内置主题：`极简现代`（默认）、`经典书卷`（衬线+米色）、`夜间阅读`（深底浅字）、`杂志`（无衬线+大标题）；主题同时作用于 Web 预览与导出的 EPUB（导出时固定为阅读友好配色） | P1 |
| F7-4 | 章节标题样式：居中/左对齐、字号、与正文间距、是否显示「第 X 章」编号 | P1 |
| F7-5 | 分节符样式：`***` / `◆◆◆` / 空白 / 自定义 | P2 |
| F7-6 | 实时预览窗（右栏），支持「手机 / 平板 / 桌面」三种宽度模拟 | P1 |
| F7-7 | **附加自定义 CSS**：提供文本框粘贴用户自有样式表，追加到导出 `main.css` **末尾**（优先级最高）。<br>预览时**逐条选择器**加作用域（不只改 `body`），否则裸元素选择器（`p{}`、`strong{}`）会污染整个应用 UI；`@font-face`/`@page`/`@keyframes` 整块豁免 | P1 |
| F7-17 | **开关必须可点击**（P0）：`.switch` 结构的 `input` 为 `display:none`，若 `label.track` 既无 `for` 属性、也未包裹 `input`，则整个开关**没有任何可点击区域**，点了完全没反应。<br>方案：事件委托统一处理 `.switch` 内 `.track` / `.lbl` 的点击（覆盖静态与动态生成的开关）。<br>**测试禁区**：不得用 `el.checked=false; el.dispatchEvent(new Event('change'))` 验证开关——那会绕过真实点击路径，正是此 bug 长期未被发现的原因。必须用 `mousedown/mouseup/click` 序列做真实点击 | P0 |
| F7-18 | **规则色块宽度自适应**：色块文本如「示例」约 52px，固定 `width:26px` 会折成两行（「示/例」）。<br>改用 `min-width` + `max-width` + `white-space:nowrap` + `overflow:hidden` | P0 |
| F7-19 | **规则色块与实际 CSS 同源**：色块 inline style 必须直接复用 `ruleDeclarations(r)`。另写一套（尤其把 `underlineColor` 当文字色）会导致「面板显示绿字、实际只有绿下划线」。改色后须**就地刷新**卡片头部（不重建 DOM，否则输入框失焦） | P0 |
| F7-8 | **阅读器适配**：<br>· `duokan-text-indent` 等掌阅/多看私有属性（其他阅读器自动忽略）<br>· `orphans:2; widows:2` 防孤行寡行<br>· `hanging-punctuation:allow-end` 标点悬挂<br>· `page-break-inside:avoid` 图表不跨页<br>· 封面图宽度可调（默认 52%） | P1 |
| F7-20 | **默认样式为国际风**（Swiss / International Typographic Style）：<br>· 无衬线：`Inter` + 思源黑体（拉丁在前，中英混排各自回退）<br>· **左对齐 ragged right**（中文两端对齐会产生难看的字间距，可开关）<br>· **用段间距分隔段落，不用首行缩进**（可开关；开启缩进时自动缩小段间距，避免双重分隔）<br>· 标题**左对齐**、负字距（-0.011em ~ -0.02em）、上疏下密（亲密性原则）<br>· 章节标题用**几何短线**装饰（`::before`，2.2em × 2px），非花哨纹样<br>· 强调**用字重而非颜色**（国际风默认不给 `strong` 着色）<br>· 克制中性色：正文 `#18181B`（近黑非纯黑）、次要 `#71717A`、分隔线 `#E4E4E7`<br>· 行高 1.75、页边距 7%<br>所有项均可在「排版」面板改回传统样式（衬线 / 两端对齐 / 首行缩进 / 标题居中） | P0 |
| F7-21 | `scopeCss` 对**容器型 at-rule**（`@media` / `@supports`）必须**原样输出头部**再递归内部。<br>若对头部调用作用域化，会产出 `.preview-inner @supports not (…)` 这类非法选择器 | P0 |

### F8 导出 EPUB（P0）

| 编号 | 需求 | 优先级 |
| --- | --- | --- |
| F8-1 | 导出 **EPUB 3.0**（向下兼容 EPUB 2.0：同时生成 `toc.ncx`），结构见第 6 节 | P0 |
| F8-2 | 目录导航：`nav.xhtml`（EPUB3）+ `toc.ncx`（EPUB2）双份，支持两级（卷/章） | P0 |
| F8-3 | 内嵌 CSS（独立 `style/main.css` 文件，而非行内样式，保证阅读器可覆盖） | P0 |
| F8-4 | 图片以原始二进制写入 `OEBPS/images/`，并在 `content.opf` 的 `<manifest>` 中声明 `media-type` | P0 |
| F8-5 | 单章按体积自动拆分（默认阈值 300KB/章，避免部分阅读器加载卡顿），拆分时不破坏段落 | P1 |
| F8-6 | 导出前自检：未命名章节、缺失封面、未填书名、图片丢失、非法 XHTML 字符 → 列出问题并允许「忽略并继续」 | P1 |
| F8-7 | 文件名规则：`{书名}-{作者}.epub`（自动清理非法字符） | P0 |
| F8-8 | 导出后提供「用 epubcheck 校验」结果面板（前端 WASM 版，可选开关） | P2 |
| F8-9 | 可选：内嵌中文字体子集（体积大，默认关闭，需提示） | P2 |

### F9 工程与数据（P1）

| 编号 | 需求 | 优先级 |
| --- | --- | --- |
| F9-1 | 自动保存草稿到 IndexedDB（含图片 blob），关闭页面可恢复，展示「上次编辑时间」 | P0 |
| F9-2 | 多工程管理：工程列表、复制、重命名、删除 | P1 |
| F9-3 | 导出工程为 `.json`（不含图片为小文件 / 含图片为 `.zip`），支持导入恢复 | P1 |
| F9-4 | 导出纯 `HTML` / `Markdown` 作为附加格式 | P2 |
| F9-5 | **清空全部**：顶栏「保存工程」左侧提供「清空」按钮，二次确认后清空章节、图片、元数据、封面、排版设置、附加 CSS 与高亮规则，并清除本机草稿。<br>实现要点：清空前须 `cancel()` 待执行的 debounce 自动保存，否则旧内容会被回写；确认框需提供「导出备份」出口 | P1 |
| F9-6 | 面板初始化须区分「同步值」与「绑定事件」：`initTypoPanel` 在导入/加载/清空时会被反复调用，事件监听只能绑定一次，否则会重复触发 | P1 |

---

## 4. 非功能需求

| 类别 | 要求 |
| --- | --- |
| 隐私 | **纯前端运行**，文件与图片永不上传服务器；页面显著位置展示「数据不出本机」标识 |
| 性能 | 50 万字文档编辑不卡顿（章节虚拟化渲染：一次只渲染当前章 + 相邻章） |
| 兼容 | 最新两版 Chrome / Edge / Safari / Firefox；移动端（≥ 768px）可用只读预览，编辑功能提示切换桌面端 |
| 可访问性 | 键盘可完成主要操作；对比度 ≥ WCAG AA；支持 `prefers-color-scheme` |
| 国际化 | UI 文案支持 `简体中文` / `English`（后续），交互与视觉遵循国际风（不依赖本地化习惯的隐喻） |
| 稳定性 | 大文件处理在 Web Worker 中进行，避免主线程阻塞与页面假死 |
| 体积 | 首屏 JS ≤ 500KB（gzip 后） |

---

## 5. 视觉与交互规范（国际风 International Style）

### 5.1 设计原则

1. **内容优先**：大面积留白，功能区域以卡片分隔，不使用装饰性图形。
2. **网格系统**：基于 4px 基准栅格，页面最大宽度 1440px，左右固定侧栏（章节树 280px / 编辑区自适应 / 预览区可折叠）。
3. **字体**：UI 使用 `Inter`（拉丁）+ `Noto Sans SC`（中文），数字与代码使用 `JetBrains Mono`；不使用圆体/书法体。
4. **克制色彩**：中性灰阶为主，仅以**单一强调色**表达可交互状态。
5. **圆角**：统一 `8px`（卡片）/ `6px`（按钮、输入框）；不使用大圆角与胶囊堆叠。
6. **动效**：150–250ms `ease-out`，仅用于出现/切换，不做弹跳。

### 5.2 色板（Design Token）

| Token | 浅色 | 深色 | 用途 |
| --- | --- | --- | --- |
| `--bg` | `#FFFFFF` | `#0F1115` | 页面背景 |
| `--surface` | `#F8FAFC` | `#171A21` | 卡片/侧栏 |
| `--border` | `#E2E8F0` | `#262B36` | 分隔线 |
| `--text` | `#0F172A` | `#E5E7EB` | 主文字 |
| `--text-muted` | `#64748B` | `#94A3B8` | 次要文字 |
| `--primary` | `#2563EB` | `#3B82F6` | 主按钮/选中态 |
| `--accent-dialog` | `#16A34A` | `#22C55E` | **对话绿色下划线** |
| `--accent-note`（【】） | `#B45309` | `#F59E0B` | 提示/旁白 |
| `--accent-paren`（（）） | `#64748B` | `#94A3B8` | 补充/心理 |
| `--accent-title`（《》） | `#4338CA` | `#818CF8` | 书名/技能 |
| `--danger` | `#DC2626` | `#F87171` | 删除/错误 |

### 5.3 页面结构

```
┌─────────────────────────────────────────────────────────────┐
│ Topbar: Logo  书名(可编辑)   [导入] [保存] [预览] [导出 EPUB] │
├───────────┬──────────────────────────┬──────────────────────┤
│ 左栏      │ 中栏                      │ 右栏                 │
│ 章节树    │ 编辑器（富文本/源码）      │ 设置面板             │
│ 搜索      │ 工具条                    │ ├ 元数据             │
│ 图片库    │ 正文                      │ ├ 排版主题           │
│           │                          │ ├ 高亮规则（可编辑）  │
│           │                          │ └ 实时预览（可切宽度）│
└───────────┴──────────────────────────┴──────────────────────┘
```

### 5.4 关键交互

- **上传区**：整页虚线框拖拽提示（`Drop your .txt here`），悬停高亮强调色。
- **分章确认**：导入后弹出「分章预览」对话框，列出识别到的 N 章，支持在列表中勾选/取消断点，确认后进入编辑器。
- **规则配置**：右栏「高亮规则」以卡片列表呈现，每张卡片左侧为色块 + 符号示例（`「对话」`），点击展开样式编辑器；实时作用于中栏。
- **导出流程**：点击「导出 EPUB」→ 抽屉展示导出配置（元数据确认 / 章节勾选 / 图片压缩 / 自检结果）→ 进度条 → 自动下载 + 成功 Toast（含文件大小与章节数）。

---

## 6. EPUB 输出规范

### 6.1 包结构

```
book.epub
├── mimetype                     （必须首个条目，STORED 不压缩）
├── META-INF/
│   ├── container.xml
│   └── com.apple.ibooks.display-options.xml   （可选，字体/滚动设置）
└── OEBPS/
    ├── content.opf              （元数据 + manifest + spine）
    ├── toc.ncx                  （EPUB2 兼容目录）
    ├── nav.xhtml                （EPUB3 导航文档）
    ├── style/
    │   └── main.css             （排版 + 语义高亮样式）
    ├── text/
    │   ├── chapter-001.xhtml
    │   └── ...
    ├── images/
    │   ├── cover.jpg
    │   └── img-001.png
    └── fonts/                   （可选）
```

### 6.2 `content.opf` 要点

- `<dc:title>` `<dc:creator>` `<dc:language>` `<dc:description>` `<dc:identifier>（uuid）` `<dc:date>` `<meta name="cover" content="cover-image">`
- `<manifest>` 中 `nav.xhtml` 需 `properties="nav"`；封面图 `properties="cover-image"`
- `<spine toc="ncx">` 按章节顺序排列 `idref`

### 6.3 `main.css` 关键片段（示意）

```css
/* ---------- 基础排版 ---------- */
:root {
  --text: #1a1a1a;
  --dialog: #16a34a;   /* 对话：绿色 */
  --note: #b45309;     /* 【】：琥珀 */
  --paren: #64748b;    /* （）：灰蓝 */
  --title: #4338ca;    /* 《》：靛蓝 */
}
body { font-family: "Noto Serif SC", "Source Han Serif SC", serif;
       font-size: 1em; line-height: 1.8; margin: 0 5%; }
p    { margin: 0 0 .2em; text-indent: 2em; }
h1.chapter-title { font-size: 1.4em; text-align: center; margin: 2em 0 1.5em; }

/* ---------- 语义高亮 ---------- */
.dlg { text-decoration: underline; text-decoration-color: var(--dialog);
       text-decoration-thickness: 2px; text-underline-offset: 3px; }
.note{ font-weight: 700; color: var(--note); background: #fffbeb; padding: 0 .15em; }
.par { font-style: italic; color: var(--paren);
       text-decoration: underline dotted var(--paren); }
.tit { font-weight: 600; color: var(--title);
       text-decoration: underline wavy var(--title); }

/* ---------- 阅读器降级：不支持 text-decoration-color 时 ---------- */
@supports not (text-decoration-color: green) {
  .dlg { border-bottom: 2px solid var(--dialog); padding-bottom: 1px; }
  .par { border-bottom: 1px dotted var(--paren); }
  .tit { border-bottom: 2px solid var(--title); }
}
```

### 6.4 XHTML 约束（导出前必须清洗）

- 严格 XML 闭合：`<br />`、`<img ... />`，属性值转义 `&amp; &lt; &gt; &quot;`
- 移除 `contenteditable` 属性、编辑器临时 `data-*` 属性
- 移除控制字符（`x00–x08` 等非法 XML 字符）
- 图片使用相对路径 `../images/xxx.png` 并声明 `alt`

---

## 7. 技术方案建议

| 层 | 选型 | 说明 |
| --- | --- | --- |
| 构建 | Vite + TypeScript | 快速启动，纯静态产物 |
| UI 框架 | React 18 | 组件化、生态成熟 |
| 样式 | Tailwind CSS + CSS 变量 | 快速实现国际风 Design Token |
| 编辑器 | TipTap（基于 ProseMirror） | 结构化的文档模型，便于导出干净 XHTML；支持自定义 Mark（`dialog` / `note` / `paren` / `title`） |
| 文件解析 | `TextDecoder('gb18030')` + `jschardet` | 编码检测与解码 |
| 打包 EPUB | `JSZip`（`mimetype` 用 `compression:'STORE'`） + `file-saver` | 纯前端生成 `.epub` |
| 存储 | IndexedDB（封装 `idb-keyval` 或 `Dexie`） | 草稿、图片二进制、规则集 |
| 性能 | Web Worker | 大文件分章、正则扫描、压缩 |
| 部署 | 静态托管（CNB Pages / GitHub Pages / Vercel） | 无需后端 |

### 7.1 核心数据模型（TypeScript）

```ts
interface Project {
  id: string;
  createdAt: number;
  updatedAt: number;
  meta: {
    title: string; subtitle?: string; author: string; translator?: string;
    language: 'zh-CN' | 'zh-TW' | 'en'; description?: string;
    tags: string[]; publisher?: string; isbn?: string; date?: string;
    coverImageId?: string;
  };
  chapters: Chapter[];
  images: ImageAsset[];
  typography: TypographyConfig;
  rules: HighlightRule[];
  splitConfig: SplitConfig;
}

interface Chapter {
  id: string;
  title: string;
  volumeId?: string;      // 两级目录
  contentHtml: string;    // 编辑器内容（干净 XHTML 片段）
  order: number;
  enabled: boolean;       // 是否导出
}

interface HighlightRule {
  id: string;
  name: string;           // 对话 / 旁白 / 补充 / 书名
  open: string;           // 「
  close: string;          // 」
  enabled: boolean;
  priority: number;
  multiline: boolean;     // 是否跨行匹配
  style: {
    color?: string; background?: string;
    bold?: boolean; italic?: boolean;
    underline?: 'none' | 'solid' | 'dashed' | 'dotted' | 'wavy';
    underlineColor?: string; underlineThickness?: number;
    fontSize?: number;    // em
  };
}

interface ImageAsset {
  id: string; name: string; mime: string;
  blob: Blob; width: number; height: number;
  usedIn: string[];       // chapterId[]
}
```

### 7.2 正则渲染思路（方案 F5-2-5 之 ①）

1. 按 `priority` 排序规则，构造统一正则（转义用户自定义符号）。
2. 扫描段落文本节点（不跨越标签边界），匹配后包裹 `<span class="<ruleId>">`。
3. **嵌套处理**：采用「栈匹配」算法而非纯正则——遇到开符号入栈、闭符号出栈，正确处理 `「他说：【注意《规则》】」` 的嵌套，内层规则覆盖外层。
4. 仅在**预览与导出阶段**执行，原文保持不变（保证用户可随时回退）。

---

## 8. 开发计划（建议里程碑）

| 阶段 | 内容 | 产出 |
| --- | --- | --- |
| M1 · 骨架 | 项目初始化、国际风布局、Design Token、上传 + 编码检测 | 可上传 TXT 并显示原文 |
| M2 · 分章与编辑 | 智能分章、章节树、TipTap 编辑器、图片插入 | 可编辑的完整编辑页 |
| M3 · 样式引擎 | 高亮规则引擎（栈匹配）、规则配置面板、主题与排版、实时预览 | 预览中可见绿色对话下划线与各括号样式 |
| M4 · 导出 | EPUB 打包（opf/nav/ncx/css）、封面与元数据、自检、下载 | 可通过 epubcheck 的 EPUB |
| M5 · 工程与打磨 | IndexedDB 草稿、工程导入导出、查找替换、批量操作、响应式与暗色模式、端到端自测 | 可交付版本 |

---

## 9. 验收标准（部分关键项）

1. 上传一份 GBK 编码、含 100 章、约 30 万字的 TXT，3 秒内完成分章且章节标题识别准确率 ≥ 95%（人工抽检 20 章）。
2. 含 `「对话」`、`【系统】`、`（心里想）`、`《九阴真经》` 的样章，在预览区分别呈现：绿色下划线 / 琥珀加粗底纹 / 灰蓝斜体点线 / 靛蓝波浪线。
3. 插入 3 张图片（含 1 张设为封面）后导出，EPUB 能在 Apple Books 与 Calibre 中正常打开，图片可显示，封面生效。
4. 导出的 EPUB 通过 `epubcheck`（或等效校验）无 error。
5. 修改某条规则的配色后，导出文件的对应样式同步变更。
6. 刷新页面后草稿（含图片）完整恢复。
7. 全程无任何网络请求携带正文内容（DevTools Network 面板验证）。

---

## 10. 风险与待确认项

| 编号 | 事项 | 类型 | 建议 |
| --- | --- | --- | --- |
| R1 | 旧阅读器不支持 `text-decoration-color` / `wavy` | 兼容 | 已设计 `@supports` 降级（见 6.3），待实机验证 Kindle |
| R2 | 中文字体内嵌导致 EPUB 体积巨大 | 体积 | 默认不内嵌，提供开关并提示体积 |
| R3 | 超大 TXT（>20MB）内存与渲染压力 | 性能 | Worker + 分章虚拟化 + 分片解析 |
| R4 | 括号嵌套与不配对（如只有 `「` 没有 `」`） | 正确性 | 栈匹配时忽略不配对符号，不做跨段强行匹配，并在编辑器内给出「未闭合符号」提示 |
| R5 | 不同阅读器对 CSS 支持差异大 | 兼容 | 建立「阅读器兼容矩阵」测试清单，导出样式保持保守（避免 flex/grid 排版正文） |
| Q1 | 是否需要登录/云端同步？ | 待确认 | 建议 MVP 不做，纯本地 |
| Q2 | 是否需要支持导出 `.mobi` / `.azw3`？ | 待确认 | 建议 MVP 只做 EPUB，后续可引导用户用 Calibre 转换 |
| Q3 | 是否需要英文界面？ | 待确认 | 建议 M5 之后接入 i18n |
| Q4 | 对话绿色是否需要固定为 `#16A34A`，还是允许主题联动？ | 待确认 | 建议默认固定绿色（满足需求），同时允许规则面板覆盖 |

---

## 11. 附录：默认高亮规则 JSON（规则集初始配置）

```json
[
  {
    "id": "dialog-cjk", "name": "对话（直角引号）", "open": "「", "close": "」",
    "enabled": true, "priority": 10, "multiline": true,
    "style": { "underline": "solid", "underlineColor": "#16A34A", "underlineThickness": 2 }
  },
  {
    "id": "dialog-cjk2", "name": "对话（双直角引号）", "open": "『", "close": "』",
    "enabled": true, "priority": 11, "multiline": true,
    "style": { "underline": "solid", "underlineColor": "#16A34A", "underlineThickness": 2 }
  },
  {
    "id": "dialog-cn", "name": "对话（弯引号）", "open": "“", "close": "”",
    "enabled": true, "priority": 12, "multiline": true,
    "style": { "underline": "solid", "underlineColor": "#16A34A", "underlineThickness": 2 }
  },
  {
    "id": "dialog-en", "name": "对话（直引号）", "open": "\"", "close": "\"",
    "enabled": true, "priority": 13, "multiline": true,
    "style": { "underline": "solid", "underlineColor": "#16A34A", "underlineThickness": 2 }
  },
  {
    "id": "note", "name": "提示/旁白【】", "open": "【", "close": "】",
    "enabled": true, "priority": 20, "multiline": false,
    "style": { "color": "#B45309", "background": "#FFFBEB", "bold": true }
  },
  {
    "id": "paren", "name": "补充/心理（）", "open": "（", "close": "）",
    "enabled": true, "priority": 30, "multiline": false,
    "style": { "color": "#64748B", "italic": true, "underline": "dotted", "underlineColor": "#94A3B8" }
  },
  {
    "id": "booktitle", "name": "书名/技能《》", "open": "《", "close": "》",
    "enabled": true, "priority": 40, "multiline": false,
    "style": { "color": "#4338CA", "bold": true, "underline": "wavy", "underlineColor": "#6366F1" }
  }
]
```
