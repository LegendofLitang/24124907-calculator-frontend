# 计算器系统 · 前端

学号：24124907 ｜ 作者：LegendofLitang

原生 **HTML / CSS / JavaScript** 实现的计算器界面，零第三方依赖、零构建步骤，
通过 `fetch` 调用后端 REST API 完成计算与历史记录管理。

> **前后端分离**：本仓库只负责界面与交互，**不做任何数学计算**。
> 用户输入的表达式原样发给后端，计算结果与历史记录全部来自后端 API。
> 验证方法：关闭后端服务后，本前端仍可操作界面，但**无法算出任何新结果**。

## 技术栈

| 层 | 技术 |
| ---- | ---- |
| 结构 | HTML5（语义化标签 + ARIA 无障碍属性） |
| 样式 | CSS3（CSS 变量主题、Grid 布局、响应式断点） |
| 逻辑 | 原生 JavaScript（ES5 语法、IIFE 封装、无框架） |
| 通信 | Fetch API + Promise，HTTP + JSON |

## 运行环境

| 项 | 要求 |
| ---- | ---- |
| 浏览器 | Chrome / Edge / Firefox / Safari 任一现代浏览器 |
| 后端 | 需先启动后端服务（见后端仓库 README），默认 `http://localhost:8080` |
| 本地静态服务（可选） | Python 3（用 `python -m http.server`）或 VS Code Live Server 插件 |

本前端**不需要** Node.js、不需要 npm install、不需要打包构建。

## 文件结构

```
├── index.html    页面结构：计算区 / 历史区 / 提示条
├── style.css     样式：主题变量、响应式布局、按键面板
├── config.js     部署配置：后端 API 地址（唯一需要改的配置文件）
├── app.js        逻辑：API 调用、历史渲染、虚拟键盘、表单交互
├── README.md     本文件
└── codestyle.md  代码规范文档
```

## 安装与启动

### 1. 启动后端

先按**后端仓库的 README** 启动后端，确认 `http://localhost:8080` 可以访问。

### 2. 启动前端

三种方式任选其一：

**方式一（推荐，独立部署）**

```bash
python -m http.server 5500
# 浏览器访问 http://localhost:5500
```

**方式二**：用 VS Code 的 **Live Server** 插件，右键 `index.html` → Open with Live Server。

**方式三**：直接双击 `index.html` 用浏览器打开（file:// 协议下也能工作）。

> ⚠️ 方式一/二/三都属于"前端与后端不同源"，此时必须保证后端地址配置正确（见下节）。

## 配置说明

前端**唯一的配置项**是后端 API 地址，集中在 **`config.js`** 一个文件里：

```javascript
// config.js
window.API_BASE = "";     // 留空 = 自动判断；填地址 = 使用该地址
```

`config.js` 在 `index.html` 中先于 `app.js` 加载，`app.js` 据此决定请求地址。

| 场景 | `config.js` 中填什么 | 说明 |
| ---- | ---- | ---- |
| **由后端一并提供页面（同源）** | 留空 `""` | 走相对路径 `/api/...`。**本项目线上即为此模式** |
| 用 `file://` 直接双击打开 html | 留空 `""` | 自动回退到 `http://localhost:8080` |
| 本地用 Live Server / http.server 调试 | 填 `"http://localhost:8080"` | 页面与后端不同端口，必须显式指定 |
| 前端独立部署到公网 | 填 `"http://111.230.148.219"` | 指向后端公网地址 |

**部署时的修改方法**（只改这一行）：

```javascript
// config.js
window.API_BASE = "http://111.230.148.219";
```

改完提交推送，托管平台会自动重新部署。

> 未配置 `window.API_BASE` 时的解析规则：
> 页面以 `file://` 打开 → `http://localhost:8080`；
> 其余情况（含 80 / 443 这类默认端口的同源部署）→ 走相对路径 `/api/...`。

## 数据库初始化

**本前端不直接访问数据库，也没有任何本地数据存储**（不使用 LocalStorage、不使用 IndexedDB、不使用内存缓存）。

所有历史记录都保存在**后端的 H2 数据库**中，前端每次渲染历史列表都会重新请求
`GET /api/calculations`。因此：

- 数据库初始化方法请见**后端仓库 README** 的「数据库初始化」章节
- 只要后端数据库正常，刷新页面、关闭再打开前端、甚至换一台设备，历史记录都不会丢失

## 前后端对接方式

```
本前端（index.html + app.js）
      │  fetch  HTTP + JSON
      ▼
后端 REST API  http://localhost:8080/api/calculations
      │
      ▼
H2 数据库
```

### 调用的接口

| 前端动作 | 方法 | 路径 | 请求体 |
| ---- | ---- | ---- | ---- |
| 点击"计算" | POST | `/api/calculations` | `{"expression":"(1+2)*3"}` |
| 加载历史列表 | GET | `/api/calculations?page=0&size=10` | — |
| 删除单条记录 | DELETE | `/api/calculations/{id}` | — |
| 清空全部历史 | DELETE | `/api/calculations` | — |

### 请求 / 响应示例

请求：

```json
{ "expression": "10 / (2 + 3)" }
```

成功响应（HTTP 201）：

```json
{
  "id": 1,
  "expression": "10 / (2 + 3)",
  "result": 2.0,
  "calculatedAt": "2026-10-03T14:30:00",
  "success": true,
  "errorMessage": null
}
```

错误响应（HTTP 400）：

```json
{
  "timestamp": "2026-10-03T14:30:00",
  "status": 400,
  "error": "Bad Request",
  "message": "除数不能为零",
  "path": "/api/calculations"
}
```

前端读取 `message` 字段，展示在错误提示框中。

### 跨域说明

后端 `WebConfig.java` 已对 `/api/**` 放开 CORS（`allowedOriginPatterns("*")`），
因此本前端**无论部署在哪个域名或端口**，都能直接调用后端接口，无需额外配置代理。

## 功能

- **计算**：四则运算、取模、括号、小数、负数，支持运算优先级与一元正负号
  （计算结果**全部由后端产生**）
- **幂运算**（扩展功能）：`^` 运算符，右结合，支持负指数与开方，如 `2^3`、`2^3^2`、`2^-1`、`4^0.5`
- **输入**：虚拟按键面板（含 C 清空、⌫ 退格、xʸ 幂运算）+ 键盘输入，回车即算
- **历史**：分页浏览（每页 10 条）、**按表达式关键字搜索**（扩展功能，300ms 防抖）、
  点击回填表达式、单条删除、全部清空
- **反馈**：成功显示结果卡片，失败显示统一错误提示，操作结果用 Toast 提示
- **数值显示**：自动消除二进制浮点噪声，`1.1 + 2.2` 显示 `3.3` 而非 `3.3000000000000003`
- **安全**：历史列表全部使用 `createElement` + `textContent` 渲染，用户输入不会被解析为 HTML

## 界面说明

| 区域 | 内容 |
| ---- | ---- |
| 左侧计算区 | 表达式输入框、清空按钮、4×6 虚拟键盘（含 xʸ 幂运算键）、计算按钮、结果/错误提示 |
| 右侧历史区 | 搜索框 + 重置按钮、记录条数统计、上一页/下一页、清空全部、历史列表（含成功/失败状态标记） |
| 底部 | Toast 提示条（4 秒自动消失） |

## 部署

### 🟢 评测期间在线地址

```
http://111.230.148.219
```

**本前端当前采用「由后端一并提供」的部署方式**：后端仓库的 `src/main/resources/static/`
中包含本前端的副本，因此访问上面的地址即可看到界面，无需单独部署本仓库。

| 项 | 值 |
| ---- | ---- |
| 云服务商 | 腾讯云 轻量应用服务器（广州 · 2核2G · Ubuntu） |
| 后端监听端口 | 80 |
| 访问方式 | 浏览器直接打开 `http://111.230.148.219` |

> 由于前后端同源，`config.js` 中的 `window.API_BASE` **保持为空**即可，
> 请求会自动走相对路径 `/api/...`。

### 独立部署本仓库（可选）

本前端也可独立部署到静态托管平台，更能体现前后端分离：

| 平台 | 步骤 |
| ---- | ---- |
| **Vercel** | 导入本仓库 → Framework 选 `Other` → Build Command 留空 → Output Directory 填 `.` → Deploy |
| **Netlify** | 拖拽整个文件夹到 Netlify Drop，或连接仓库后 Publish directory 填 `.` |
| **Cloudflare Pages** | 连接仓库 → 构建命令留空 → 输出目录 `.` |
| **GitHub Pages** | 仓库 Settings → Pages → Source 选 `main` 分支根目录 |

**独立部署后必做**：把 `config.js` 的 `window.API_BASE` 改成后端公网地址，否则会去请求
`localhost`（在访问者电脑上不存在）：

```javascript
window.API_BASE = "http://111.230.148.219";
```

## 交付物

| 文件 | 说明 |
| ---- | ---- |
| `index.html` | 页面结构 |
| `style.css` | 样式 |
| `config.js` | 部署配置（后端 API 地址） |
| `app.js` | 交互逻辑与 API 调用 |
| `README.md` | 本文件：项目简介、技术栈、运行环境、安装启动、配置说明、前后端对接 |
| `codestyle.md` | 代码规范文档（基于 Google JavaScript Style Guide） |

## 关联仓库与规范文档

| 项目 | 地址 |
| ---- | ---- |
| 前端仓库 | `24124907-calculator-frontend`（本仓库） |
| 后端仓库 | `24124907-calculator-backend` |
| 前端代码规范 | 本仓库中的 [codestyle.md](codestyle.md) |
| 后端代码规范 | `24124907-calculator-backend` 仓库中的 `codestyle.md` |

## 代码规范

本项目遵循 **[Google JavaScript Style Guide](https://google.github.io/styleguide/jsguide.html)**，
HTML / CSS 部分参考 [Google HTML/CSS Style Guide](https://google.github.io/styleguide/htmlcssguide.html)，
详见 [codestyle.md](codestyle.md)。
