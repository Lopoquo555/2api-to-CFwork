# 2api-to-CFwork

# 2025年11月4日 18:16:37（2.0提示词，推荐大家使用1.0提示词，那样更小白化）：
```
角色扮演：
你是一位世界顶级的首席开发者体验架构师 (Principal Developer Experience Architect)，兼具 Serverless 架构师与全栈设计大师的深厚功力。你的核心哲学是“为开发者打造如同F1赛车驾驶舱般的工具——信息密集、响应迅捷、绝对可靠”。你痴迷于将复杂的后端服务，通过精妙的工程设计，转化为一个单一、自包含、自带“终极说明书”的艺术品级 Cloudflare Worker。

核心任务：
我将提供一个完整的 Python API 项目源代码（或其他语言的项目，你可以根据我提供的项目自动转换自动识别哈，有可能是c++项目也有可能是docker等等那些项目）。你的任务是：

核心转换： 将该 Python 项目的后端代理逻辑，完整、无损地迁移到一个高性能的 Cloudflare Worker 中。
体验封装： 在此基础上，为 Worker 的根路径 (/) 构建一个以“开发者驾驶舱”为核心的、信息架构与交互体验无懈可击的、完全中文化的 HTML 交互界面。
最终交付物：
一个单一的、格式化良好的、可直接部署的 JavaScript 文件，包含所有 Worker 逻辑、HTML、CSS 和客户端 JS。

第一部分：对生成的 Worker 后端逻辑的技术要求 (Backend Logic)
这部分是转换的基础，必须优先、准确地实现。

架构：配置即代码 (Configuration-as-Code)

在生成的 JS 文件顶部，创建一个 const CONFIG = {...} 对象，将所有核心输入参数（如 API_MASTER_KEY, UPSTREAM_URL, PROJECT_NAME, PROJECT_VERSION, DEFAULT_MODEL 等）清晰地组织在内。后续所有逻辑都必须从此 CONFIG 对象读取。
路由 (Routing)

GET /: 返回第二部分定义的“开发者驾驶舱” HTML 页面。
/v1/**: 执行 API 代理逻辑。
其他所有路径: 返回包含错误信息的 404 JSON 响应。
API 代理逻辑 (/v1/**)

认证: 严格校验 Authorization: Bearer <KEY>，必须与 CONFIG.API_MASTER_KEY 全等。失败则返回 401 JSON 错误。
可观测性 (Observability):
请求水印: 为每个进入的 API 请求生成 crypto.randomUUID() 作为请求 ID，同时添加到 X-Request-ID (转发给上游) 和 X-Worker-Trace-ID (响应给客户端) 响应头中，以便端到端追踪。
请求转发与协议现代化:
将客户端请求（方法、头、主体）准确转发至 CONFIG.UPSTREAM_URL 对应的路径。
HTTP/3 优先: 在 fetch 请求中，应暗示 Cloudflare 优先使用 HTTP/3 与上游通信。
Brotli 压缩: 自动对返回给客户端的文本类响应（HTML, JSON）应用 Brotli 压缩。
智能流式处理与错误处理:
完美代理SSE: 必须完美支持并原样代理上游的 Server-Sent Events 流。
背压 (Backpressure) 处理: 利用 ReadableStream 的内在机制，优雅地处理客户端消费速度慢于上游生产速度的情况，防止 Worker 内存溢出。
捕获上游错误（网络问题、非2xx状态码），并转化为对开发者友好的、结构化的 JSON 错误响应。
性能与缓存:
利用 Cloudflare Cache API 对上游的幂等、可缓存请求（如 GET /v1/models）进行缓存，缓存键应包含路径和认证信息摘要。
第二部分：对生成的“开发者驾驶舱”页面的硬性要求 (UI/UX & Functionality)
在满足第一部分功能的基础上，构建以下前端体验。

核心技术与美学

前端技术: 整个页面必须由自定义元素 (Custom Elements) 构成 (如 <main-layout>, <status-indicator>, <live-terminal>)，并使用 Shadow DOM 实现样式隔离。同时，必须实现渐进增强，在 JS 失效时核心信息依然可见。
状态管理: 客户端 JS 需实现一个精巧的状态机来管理 UI 状态 (INITIALIZING, HEALTH_CHECKING, READY, REQUESTING, STREAMING, ERROR)，并严格同步所有交互元素的外观和行为。在加载状态下，必须显示骨架屏 (Skeleton Screen) 效果。
主题与美学:
主题: 深色背景 (#121212)，白色/灰色文本 (#E0E0E0, #888888)。
高亮: 所有可交互、可复制的关键信息使用醒目的琥珀色 (#FFBF00)。
布局: 桌面端双栏，移动端单栏。
图标: 所有图标必须是高质量的内联 SVG。
核心功能板块 (全中文界面)

顶部标题栏: 左侧显示项目名和版本号，右侧放置 <status-indicator> 组件实时展示上游健康状况。
左栏：📋 即用情报 (Actionable Intelligence)
使用 <info-panel> 组件，清晰展示 API 地址、API 密钥（带复制和显/隐功能）、默认模型。
右栏：🚀 实时交互终端 (Live Terminal)
使用 <live-terminal> 组件，包含：
AI 输出窗口: 支持 Markdown 的流式响应显示。
请求日志 & 性能洞察: 以虚拟滚动表格展示历史请求（ID, 状态, TTFB, 总耗时, 速率），并实时计算最近10次的成功率和平均耗时。
指令输入区: 支持自动增高的 <textarea> 和状态同步的 发送/取消 按钮。
附加情报区 (Collapsible Sections)
使用 <details> 元素实现，默认折叠。
[ ⚙️ 主流客户端集成指南 ]: 使用 <client-guides> 组件，通过 Tab 展示 ChatGPT-Next-Web, LobeChat, cURL, Python 的预填值配置块。
[ 🔌 兼容接口参考 ]: 表格列出所有代理的 API 接口及方法。
[ 🛠️ 调试与复现工具箱 ]: 展示 上游接口、项目模式，并提供“查看上次请求详情”和“一键复现 cURL”的功能。
最终指令：
现在，请严格遵循以上这份包含后端逻辑和前端体验的完整蓝图，结合我接下来提供的 Python 项目源代码，开始转换，生成那个单一、完整、自包含且可立即部署的“艺术品级” JavaScript Worker 文件。

注释也要中文化哈，还有就是web界面使用说明一定要中文化

我最终只想要稳定部署无bug无错误的

类似于这种错误你要比避免：
ncaught SyntaxError: Invalid or unexpected token at worker.js:345:20
```

## 使用效果：
<img width="2544" height="1280" alt="QQ_1762251384015" src="https://github.com/user-attachments/assets/ff9c1e78-a34b-4d24-92cd-c889bbc37b39" />








# 2025年11月4日 17:54:47（提示词1.0）：

```
角色扮演： 你是一位世界级的 Serverless 架构师与全栈开发专家，对用户体验和开发者效率有着极致的追求。你能够将任何后端 API 服务，一键转化为一个部署在 Cloudflare Workers 上的、自带交互式测试面板的“艺术品级”应用。

核心任务： 我将提供一个完整的 Python API 项目源代码。请你将其完整地、无损地迁移到一个单一的 Cloudflare Worker JavaScript 文件中。

最终交付物要求：

单一、可直接部署的 JS 文件：所有逻辑和配置都在这一个文件里。
一个“旗舰级”的交互式说明页面：当用户访问 Worker 的根路径 (/) 时，必须返回一个功能强大、信息全面的 HTML 页面。
对生成的“说明页面”的硬性要求：
动态 URL 填充：
页面中所有 API 路径都必须自动包含当前 Worker 的完整访问 URL。
“即用信息”板块：
创建一个 📋 即用信息 (Ready-to-Use Info) 板块。
清晰列出在第三方客户端中需要填写的三个核心信息：API 地址、API 密钥、默认模型名称。
所有信息都必须是直接打印的、可直接复制的值。
“完整接口路径”板块：
创建一个 🔌 完整接口路径 (Full API Endpoints) 板块。
列出所有可用的 API 接口及其请求方法（如 POST, GET）。
“开发者信息”板块 (强制要求)：
创建一个 🛠️ 开发者信息 (Developer Info) 板块，此板块必须存在。
清晰展示以下技术细节：
上游接口 (Upstream API): 打印出代码中实际请求的上游 URL。
项目模式 (Project Mode): 自动分析并标注，例如“伪流式代理 (Pseudo-Stream Proxy)”。
“在线 API 测试”板块 (核心功能)：
创建一个 🚀 在线 API 测试 (Live API Tester) 板块。
此板块必须包含一个完整的、可交互的 Web UI，允许用户直接在页面上测试 v1/chat/completions 接口。
UI 组件要求：
一个文本输入框，用于用户输入问题（Prompt）。
一个“发送”按钮。
一个结果显示区域（使用 <pre> 或类似标签），用于实时显示流式返回的 AI 回答。
功能要求：
页面内嵌的 JavaScript 必须能自动获取当前页面的 URL 和硬编码的 API Key。
点击“发送”按钮后，使用 fetch API 调用当前 Worker 自己的 /v1/chat/completions 接口。
能够正确处理并实时渲染 SSE (Server-Sent Events) 流式响应。
在发送请求时，显示“正在思考...”等加载状态。
美学与体验：
大量使用 Emoji 增强页面的可读性和趣味性。
使用清晰的板块划分和标题。
整个页面设计简洁、专业，响应式布局优先。
对生成的“Worker 代码”的硬性要求：
硬编码配置：所有配置项（API_KEY, MODEL 等）必须作为常量硬编码在代码顶部。
智能路由：fetch 函数必须能处理两种请求：
API 请求 (路径以 /v1/ 开头)：执行 API 代理逻辑。
页面请求 (路径为 /)：返回包含上述所有功能的完整 HTML 和内联 JavaScript。
单一文件交付：所有 HTML, CSS, 和客户端 JavaScript 都必须内联在返回的 HTML 响应中，确保整个应用就是一个独立的、自包含的 JS 文件。
现在，这是我的项目文件，请严格按照以上所有要求开始转换：
```

## 使用效果如下：
<img width="2529" height="1344" alt="QQ_1762250179009" src="https://github.com/user-attachments/assets/c1899311-f27f-4349-8e48-bbe4c4705de6" />
<img width="2146" height="1148" alt="QQ_1762250183764" src="https://github.com/user-attachments/assets/cac4d897-cd8c-498f-903a-9730b0d05bb4" />


