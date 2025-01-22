# Quick Bookmarks Retrieval 快速书签检索

一个简洁高效的 Chrome 扩展程序，帮助你快速检索和访问书签。通过优雅的界面设计和智能的搜索功能，让书签管理变得更加轻松。

## 功能特点

- 🚀 快速搜索：实时过滤和匹配书签
- 🎯 快捷键支持：使用 `Ctrl + B` 快速唤起搜索界面
- 🎨 优雅界面：现代化 UI 设计，流畅的动画效果
- 🔍 智能匹配：支持模糊搜索，快速定位目标书签
- 🖼️ 网站图标：自动获取并显示网站 favicon

## 技术栈

- **前端框架**：Vue 3 + Vite
  - 使用 Vue 3 的 Composition API 构建响应式界面
  - Vite 提供快速的开发体验和优化的构建流程

- **Chrome Extension API**
  - 使用 `chrome.bookmarks.getTree()` API 获取书签数据
  - 采用 Manifest V3 规范开发

- **现代化 UI 设计**
  - 使用 CSS3 实现流畅动画效果
  - 响应式布局，优雅的视觉体验

## 实现原理

### 书签数据获取

```javascript
// 使用 Chrome Bookmarks API 获取书签树
const bookmarkTree = await chrome.bookmarks.getTree()

// 递归遍历书签树，提取所需数据
const traverseBookmarks = (nodes) => {
  for (const node of nodes) {
    if (node.url) {
      // 收集书签信息
      bookmarks.push({
        id: node.id,
        title: node.title,
        url: node.url
      })
    }
    // 继续遍历子节点
    if (node.children) {
      traverseBookmarks(node.children)
    }
  }
}
```

### 网站图标获取

使用 Google 的 favicon 服务获取网站图标：

```javascript
// 从 URL 中提取域名并获取对应的 favicon
icon: `https://www.google.com/s2/favicons?domain=${new URL(node.url).hostname}&sz=32`
```

## 安装和使用

1. 克隆项目到本地：
   ```bash
   git clone [项目地址]
   cd quick-bookmarks-retrieval
   ```

2. 安装依赖：
   ```bash
   npm install
   ```

3. 开发模式：
   ```bash
   npm run dev
   ```

4. 构建扩展：
   ```bash
   npm run build
   ```

5. 在 Chrome 中加载扩展：
   - 打开 Chrome 扩展管理页面 (`chrome://extensions/`)
   - 开启「开发者模式」
   - 点击「加载已解压的扩展程序」
   - 选择项目的 `dist` 目录

## 使用方法

1. 安装扩展后，点击工具栏中的扩展图标即可打开搜索界面
2. 使用快捷键 `Ctrl + B` 快速唤起搜索界面
3. 在搜索框中输入关键词，即可实时过滤匹配的书签
4. 点击书签项即可在新标签页中打开对应网站

## 贡献

欢迎提交 Issue 和 Pull Request！

## 联系方式

WeChat: daxin261
