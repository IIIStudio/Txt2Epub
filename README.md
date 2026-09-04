# Txt2Epub Studio

纯浏览器端的 TXT → EPUB 在线编辑器。上传 → 自动分章 → 可视化编辑 → 排版美化 → 一键导出 EPUB，全程**数据不出本机**。

![Txt2Epub Studio 编辑器界面](./image/image.png)

## 特性

- **智能分章**：一条正则搞定，支持 `中文网文` / `出版书籍` / `英文原著` / `自定义正则` 预设，可实时预览、手动增删断点
- **编码自动检测**：`UTF-8(BOM)` → `GB18030` → `GBK` → `BIG5` → `UTF-16`，检测结果显示在 UI 上，可手动切换
- **语义高亮**：`「」『』“”""` 对话绿色下划线、`【】` 琥珀提示、`（）` 灰蓝斜体、`《》` 靛蓝波浪线……规则全部可编辑，导出时自动降级兼容旧阅读器
- **所见即所得**：导出 CSS 原样注入编辑区，改排版/规则/自定义 CSS 即刻可见
- **图片支持**：拖拽/粘贴插入，内容指纹去重，存 IndexedDB 刷新不丢，可设为封面
- **工程保存**：自动存草稿到 IndexedDB，支持工程 JSON 导入导出
- **导出 EPUB 3.0**：`nav.xhtml` + `toc.ncx` 双目录、独立 `style/main.css`、外链图片、导出前自检

## 使用

直接用浏览器打开 `index.html` 即可，无需安装、无需构建、无后端。

1. 点击「导入 TXT」或把文件拖到页面任意位置
2. 在弹出的分章预览中确认章节断点与编码，进入编辑器
3. 编辑正文、调整排版与高亮规则
4. 点击「导出 EPUB」下载成品

## 目录结构

```
├── index.html          # 单文件应用（HTML + CSS + JS 全部内联）
├── docs/requirements.md # 需求文档（PRD）
├── image/image.png     # 界面截图
└── LICENSE             # MIT
```

## 浏览器要求

最新两版 Chrome / Edge / Safari / Firefox。以 `file://` 打开时部分浏览器会限制 IndexedDB，建议用本地静态服务器访问：

```bash
python3 -m http.server 8000
# 然后访问 http://localhost:8000
```

## License

[MIT](./LICENSE) © 2026 IIIStudio
