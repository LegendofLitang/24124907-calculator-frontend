# 前端代码规范（HTML / CSS / JavaScript）

> **规范来源**
> 本文档以 **[Google JavaScript Style Guide](https://google.github.io/styleguide/jsguide.html)** 为主要依据，
> 样式部分参考 **[Google HTML/CSS Style Guide](https://google.github.io/styleguide/htmlcssguide.html)**，
> 并结合 **[Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)** 中的工程实践条目整理。
> 本文所列规则为上述规范在本项目中的**落地约定**，与来源文档冲突时以来源文档为准。

---

## 1. 通用约定

- 一律使用 **UTF-8** 编码，HTML 中通过 `<meta charset="UTF-8">` 显式声明。
- 缩进使用 **2 个空格**（HTML / CSS / JS 统一），**禁止使用 Tab**。
- 换行符统一为 **LF**（`\n`）。
- 文件末尾保留**一个**换行符。

---

## 2. JavaScript 规范

### 2.1 严格模式与作用域

- **所有** JS 文件必须启用严格模式，且放在文件或函数的第一行。
- 全局变量**禁止**污染 `window`。本项目所有逻辑封装在 **IIFE** 中：

```javascript
(function () {
  "use strict";
  // 模块内部逻辑
})();
```

### 2.2 变量声明

- 统一使用 `var`（ES5 兼容写法，本项目不引入构建工具）。
- **禁止**隐式声明全局变量（不加 `var` 直接赋值）。
- 变量声明尽量靠近首次使用处。

```javascript
// 正确
var currentPage = 0;
var pageSize = 10;

// 错误：隐式全局变量
currentPage = 0;
```

### 2.3 命名规范

| 类型 | 规则 | 示例 |
|---|---|---|
| 变量 / 函数 | **lowerCamelCase** | `currentPage`、`loadHistory`、`renderHistory` |
| 常量（约定不变的值） | **UPPER_SNAKE_CASE** | `API_BASE`、`PAGE_SIZE` |
| 类 / 构造函数 | **UpperCamelCase** | `HistoryItem` |
| 私有函数（模块内部） | lowerCamelCase，不加下划线前缀 | `insertIntoInput` |
| DOM 元素变量 | 名词 + `El` / `Btn` 后缀 | `listEl`、`prevBtn`、`calcBtn` |

**禁止**拼音命名、单字母变量（循环变量 `i` 除外）、无意义缩写。

### 2.4 字符串

- 统一使用**双引号**。
- 需要拼接时使用 `+`，本项目不引入模板字符串以保持 ES5 兼容。

```javascript
var API_BASE = "http://localhost:8080";
errorBox.textContent = "加载历史失败：" + err.message;
```

### 2.5 函数

- 函数名用**动词开头**：`show*` / `hide*` / `load*` / `render*` / `remove*`。
- 单一职责，一个函数只做一件事。
- 函数之间空一行。

```javascript
function showError(msg) {
  errorBox.textContent = msg;
  errorBox.hidden = false;
}

function hideError() {
  errorBox.hidden = true;
}
```

### 2.6 比较与判断

- 一律使用 **`===` / `!==`**，禁止 `==` / `!=`（避免隐式类型转换）。
- 判断变量是否有值，使用显式判断：

```javascript
// 正确
if (typeof start !== "number" || typeof end !== "number") { ... }
if (!expr) { ... }

// 错误
if (start == undefined) { ... }
```

### 2.7 DOM 操作与 XSS 防护

**强制要求**：渲染任何来自后端或用户的数据，必须使用 `textContent`，**禁止**使用 `innerHTML` 拼接内容。

```javascript
// 正确：安全，用户输入不会被解析为 HTML
var expr = document.createElement("div");
expr.textContent = item.expression;

// 禁止：存在 XSS 风险
listEl.innerHTML += "<div>" + item.expression + "</div>";
```

`innerHTML` **仅允许**用于清空容器：`listEl.innerHTML = "";`

### 2.8 事件绑定

- 优先使用 **事件委托**（在父容器上绑一次），避免为每个子元素重复绑定。
- 事件处理函数优先使用具名函数，便于调试。

```javascript
var keypadEl = document.querySelector(".calc-keypad");
if (keypadEl) {
  keypadEl.addEventListener("click", function (ev) {
    var btn = ev.target && ev.target.closest ? ev.target.closest(".key-btn") : null;
    if (!btn) { return; }
    // ...
  });
}
```

### 2.9 异步请求

- 统一使用 `fetch` + Promise 链。
- **每个**请求都必须有 `.catch()` 处理失败分支。
- 请求前后必须复位 UI 状态（如按钮禁用）。

```javascript
fetch(API_BASE + "/api/calculations", { method: "POST", headers: {...}, body: ... })
    .then(function (res) { /* ... */ })
    .catch(function (err) { showError(err.message); })
    .finally(function () { calcBtn.disabled = false; });
```

### 2.10 注释

- 文件头写模块说明。
- 复杂逻辑块前写单行注释，说明**为什么**。
- 分区块用 `// ---------------- 区块名 ----------------` 分隔。

---

## 3. HTML 规范

- 使用 HTML5 文档类型 `<!DOCTYPE html>`。
- 必须声明 `<html lang="zh-CN">` 与 `<meta charset="UTF-8">`。
- 标签、属性名**全部小写**；属性值使用**双引号**。
- 自闭合标签以 ` />` 结尾（如 `<meta ... />`）。
- 通过 `<link>` 引入 CSS 放在 `<head>`，`<script>` 放在 `</body>` 前。
- 交互元素必须有可访问性属性：`aria-label`、`role`、`title`。

```html
<button type="button" class="key-btn fn" data-key="clear" aria-label="清空输入">C</button>
```

- 表单输入必须有关联的 `<label for="...">`。
- **禁止**使用行内样式 `style="..."` 和行内事件 `onclick="..."`，统一放在 CSS / JS 中。

---

## 4. CSS 规范

### 4.1 命名

- 类名使用 **kebab-case**（短横线小写），语义化。
- 采用近似 **BEM** 的命名：`块-元素`，如 `.history-item`、`.history-expr`、`.del-btn`。

### 4.2 格式

- 每条声明**独占一行**，以分号结尾。
- 选择器与 `{` 之间空一格；属性名与值之间冒号后空一格。
- 多个选择器共用一个规则时，每个选择器独占一行。

```css
.history-item {
  display: flex;
  align-items: center;
  padding: 10px 6px;
}
```

### 4.3 变量与主题

- 颜色、圆角等复用值必须提取为 **CSS 变量**，定义在 `:root` 中。

```css
:root {
  --primary: #4f46e5;
  --radius: 12px;
}
```

### 4.4 布局与响应式

- 优先使用 **Flexbox / Grid**，避免 `float` 布局。
- 必须提供移动端断点适配（本项目为 `760px` 与 `480px`）。

```css
@media (max-width: 760px) {
  .layout { grid-template-columns: 1fr; }
}
```

### 4.5 禁止事项

- 禁止使用 `!important`（除非覆盖第三方样式且已注释说明原因）。
- 禁止无意义的选择器嵌套过深（不超过 3 层）。
- 禁止遗留未使用的样式规则。

---

## 5. 文件组织

```
项目根目录/
├── index.html    页面结构：语义化标签，无行内样式与脚本
├── style.css     样式：CSS 变量主题 + 响应式布局
├── app.js        逻辑：API 调用、渲染、事件绑定，IIFE 封装
├── README.md     项目说明
└── codestyle.md  本文件
```

职责分离：**结构 / 表现 / 行为三者不得混写**。
