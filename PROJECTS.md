# Markdown Editor - 项目分析

## 项目概述

这是一个纯前端实现的 Markdown 编辑器，基于单 HTML 文件架构（`index.html`），无需构建工具或服务器即可运行。

## 技术栈

| 技术 | 版本/来源 | 用途 |
|------|-----------|------|
| HTML5 | - | 页面结构 |
| CSS3 | - | 样式与主题（CSS 变量） |
| JavaScript (ES6+) | - | 交互逻辑 |
| marked.js | `lib/marked.min.js` | Markdown 解析与渲染 |
| Mermaid.js | `lib/mermaid.min.js` | 图表渲染（流程图、时序图等） |

## 核心功能模块

### 1. 编辑器核心 (`index.html:250-254`)
- **行号显示**：实时渲染行号，当前行高亮（`renderLineNumbers`）
- **光标位置**：显示当前行/列（`updateCursorPos`）
- **字数统计**：实时更新字数（`updatePreview`）
- **Debounce 防抖**：预览渲染采用 150ms 防抖，提升输入响应速度

### 2. 三种视图模式 (`index.html:223-234`)
- **编辑模式** (`edit`)：仅显示编辑器
- **预览模式** (`preview`)：仅显示渲染结果
- **分屏模式** (`split`)：左右分屏，编辑器与预览并排

### 3. 主题系统 (`index.html:10-46`)
- **暗色主题**：默认主题，深色背景
- **亮色主题**：浅色背景
- **CSS 变量**：通过 `data-theme` 属性切换
- **localStorage 持久化**：保存主题偏好至本地存储

### 4. 分屏滚动同步 (`index.html:878-1255`)
- **源码行号 → 预览元素映射**：`buildBlockLineRanges` 构建块级元素行范围
- **双向同步**：编辑器滚动时预览区同步，反之亦然
- **细粒度映射**：支持列表项 `<li>` 和表格行 `<tr>` 的行号标注

### 5. ASCII Art 转 SVG (`index.html:288-506`)
- **盒形字符检测**：识别 `│┌┐└┘├┤┬┴┼` 等 Box Drawing 字符
- **列匹配算法**：`matchColsByPosition` 实现竖线对齐
- **SVG 渲染**：将 ASCII 图转换为矢量图形

### 6. Mermaid 图表渲染 (`index.html:1360-1415`)
- **自动检测**：识别 `graph`/`flowchart` 开头的代码块
- **异步渲染**：支持 Promise 式 API，避免重复渲染
- **主题适配**：跟随编辑器主题切换
- **扩展图表类型**：支持 sequence、classDiagram、stateDiagram 等

### 7. 思维导图 (`index.html:1527-1638`)
- **标题解析**：`parseHeadingsToTree` 提取 `#` ~ `######` 标题
- **树形结构**：转换为 Mermaid 图语法
- **缩放控制**：放大、缩小、重置

### 8. 文件操作 (`index.html:1439-1462`)
- **上传**：支持 `.md`、`.markdown`、`.txt` 文件
- **下载**：导出为 `document.md`
- **打印**：生成打印友好的 HTML 页面
- **拖拽上传**：拖放文件到编辑器区域自动加载

### 9. 拖拽调整大小 (`index.html:734-876`)
- **编辑区拖拽**：`editResizeHandle` 调整编辑区宽度
- **分屏拖拽**：`splitResizeHandle` 调整分屏比例

### 10. 查找替换 (`index.html:1618-1701`)
- **查找**：高亮匹配项，支持导航上一个/下一个
- **替换**：替换当前匹配项或全部替换
- **快捷键**：`Ctrl+F` 打开查找，`Ctrl+H` 打开替换，`Esc` 关闭

### 11. TOC 目录导航 (`index.html:1703-1760`)
- **自动提取标题**：从 Markdown 内容中提取 h1-h6 标题
- **侧边栏显示**：可折叠的目录面板，显示标题层级
- **点击跳转**：点击目录项跳转到对应行
- **实时更新**：编辑内容时自动更新目录
- **快捷键**：`Ctrl+T` 切换目录显示

### 12. 快捷键系统 (`index.html:1600-1613`)
| 快捷键 | 功能 |
|--------|------|
| `Ctrl+B` | 插入粗体 |
| `Ctrl+I` | 插入斜体 |
| `Ctrl+K` | 插入链接 |
| `Ctrl+S` | 保存内容 |
| `Ctrl+F` | 查找 |
| `Ctrl+H` | 替换 |
| `Ctrl+T` | 目录导航 |
| `Esc` | 关闭查找栏/目录 |

### 13. localStorage 持久化
- **内容保存**：自动保存编辑器内容，刷新页面后恢复
- **主题保存**：记住用户选择的主题

## CSS 架构

### 主题变量 (CSS Custom Properties)
```css
--bg-base, --bg-panel, --bg-editor, --bg-elevated  /* 背景色 */
--border, --text-primary, --text-muted, --text-dim  /* 边框与文字 */
--accent, --accent-text                              /* 强调色 */
--stroke-ascii, --text-ascii, --bg-ascii            /* ASCII 图表色 */
--transition                                        /* 过渡动画 */
```

### 布局系统
- **Flexbox**：主布局、工具栏、分屏
- **Position**：思维导图面板（fixed）、拖拽手柄（absolute）
- **Fixed**：查找栏、拖拽覆盖层、TOC 侧边栏

### 打印样式 (`@media print`)
- 隐藏工具栏和编辑区
- 白底黑字
- Mermaid 图表无填充、细边框

## JavaScript 函数清单

### 初始化与模式切换
| 函数 | 位置 | 说明 |
|------|------|------|
| `init()` | 717 | 页面初始化 |
| `setMode(mode)` | 785 | 切换编辑/预览/分屏模式 |
| `getActiveEditor()` | 768 | 获取当前激活的编辑器 |

### 编辑器操作
| 函数 | 位置 | 说明 |
|------|------|------|
| `onEditorInput()` | 772 | 编辑器输入事件处理（含防抖） |
| `insertTable()` | 1430 | 插入表格 |
| `insertCodeBlock()` | 1431 | 插入代码块 |
| `insertHeading(level)` | 1433 | 插入标题 |
| `insertBold()` | 1434 | 插入粗体 |
| `insertItalic()` | 1435 | 插入斜体 |
| `insertLink()` | 1436 | 插入链接 |
| `insertList()` | 1437 | 插入列表 |
| `clearEditor()` | 1417 | 清空编辑器 |
| `handleEditorKeydown(e)` | 1600 | 快捷键处理 |

### 行号与光标
| 函数 | 位置 | 说明 |
|------|------|------|
| `renderLineNumbers()` | 655 | 渲染行号（diff 更新） |
| `highlightActiveLine()` | 672 | 高亮当前行 |
| `getCurrentLine()` | 683 | 获取当前行号 |
| `getCurrentCol()` | 690 | 获取当前列号 |
| `updateCursorPos()` | 698 | 更新状态栏光标位置 |
| `syncLineScroll()` | 706 | 同步行号滚动 |

### 预览渲染
| 函数 | 位置 | 说明 |
|------|------|------|
| `updatePreview()` | 1258 | 更新预览区 |
| `renderPreview(content, container)` | 1270 | 渲染 Markdown 为 HTML |
| `buildBlockLineRanges(md)` | 887 | 构建块级元素行范围 |
| `injectBlockMarkers(md, blocks)` | 984 | 注入行号标记注释 |
| `processRenderedHTML(html)` | 1004 | 后处理 HTML 属性 |

### 分屏滚动同步
| 函数 | 位置 | 说明 |
|------|------|------|
| `initSplitScroll()` | 1229 | 初始化分屏滚动同步 |
| `destroySplitScroll()` | 1249 | 销毁滚动同步 |
| `getSourceLineAtTop(textarea)` | 1060 | 获取编辑器顶部行号 |
| `getPreviewScrollForLine()` | 1071 | 计算预览区滚动位置 |
| `findElementForLine()` | 1103 | 查找行号对应的元素 |

### 图表渲染
| 函数 | 位置 | 说明 |
|------|------|------|
| `renderMermaid(container)` | 1360 | 渲染 Mermaid 图表（异步） |
| `refreshMermaidDiagrams()` | 570 | 刷新 Mermaid 主题 |
| `renderAsciiDiagram(container)` | 1308 | 渲染 ASCII Art |
| `convertAsciiToSvg(text, opts)` | 393 | ASCII 转 SVG |
| `renderBoxChar(ch, ...)` | 353 | 渲染盒形字符 |

### 主题系统
| 函数 | 位置 | 说明 |
|------|------|------|
| `applyTheme(theme)` | 533 | 应用主题 |
| `toggleTheme()` | 645 | 切换主题 |
| `refreshAsciiColors()` | 601 | 刷新 ASCII 颜色 |

### 思维导图
| 函数 | 位置 | 说明 |
|------|------|------|
| `toggleMindMap()` | 1527 | 切换思维导图面板 |
| `parseHeadingsToTree(content)` | 1534 | 解析标题为树结构 |
| `treeToMermaid(nodes, parentId)` | 1567 | 树转 Mermaid 语法 |
| `updateMindMapFromHeadings()` | 1590 | 更新思维导图 |
| `zoomMindMap(factor)` | 513 | 缩放思维导图 |

### 查找替换
| 函数 | 位置 | 说明 |
|------|------|------|
| `toggleFindBar(withReplace)` | 1618 | 切换查找替换栏 |
| `doFind()` | 1646 | 执行查找 |
| `findNext()` | 1670 | 查找下一个 |
| `findPrev()` | 1675 | 查找上一个 |
| `replaceOne()` | 1680 | 替换当前 |
| `replaceAll()` | 1688 | 全部替换 |

### TOC 目录导航
| 函数 | 位置 | 说明 |
|------|------|------|
| `toggleToc()` | 1713 | 切换 TOC 侧边栏 |
| `updateToc()` | 1720 | 更新目录内容 |
| `jumpToLine(lineNum)` | 1753 | 跳转到指定行 |

### 文件操作
| 函数 | 位置 | 说明 |
|------|------|------|
| `uploadFile()` | 1439 | 触发文件上传 |
| `handleFileUpload(event)` | 1440 | 处理文件上传 |
| `downloadFile()` | 1455 | 下载文件 |
| `printDocument()` | 1508 | 打印文档 |
| `doPrint(previewEl)` | 1464 | 执行打印 |

### 持久化
| 函数 | 位置 | 说明 |
|------|------|------|
| `saveContent()` | - | 保存编辑器内容到 localStorage |
| `saveTheme()` | - | 保存主题到 localStorage |

## 文件结构

```
markdownEditor/
├── index.html          # 主文件（HTML + CSS + JS 全部内联）
├── lib/
│   ├── marked.min.js   # Markdown 解析库
│   └── mermaid.min.js  # 图表渲染库
├── LICENSE             # BSD 2-Clause 许可证
└── README.md           # 项目说明
```

## 已知问题

1. **file:// 协议限制**：Mermaid 在本地直接打开时可能无法加载资源
2. **初始化时序**：`renderLineNumbers` 需要 `setTimeout` 延迟执行
3. **滚动同步精度**：大文档时可能有微小偏差

## 代码规模

- **总行数**：约 1900 行
- **CSS**：约 300 行
- **HTML**：约 130 行
- **JavaScript**：约 1470 行
- **依赖库**：2 个（marked.js, mermaid.js）
