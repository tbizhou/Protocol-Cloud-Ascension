# 《云端飞升协议》技术栈规划 (Technology Stack)

**版本:** 1.0  
**核心理念:** 前端重逻辑 (Vue3 + Worker) / 后端重并发 (Go) / 极简部署 (Docker)

---

## 1. 前端架构 (Frontend - The Client)

基于 Web 端（H5）开发，强调组件化复用和高性能数值计算。

### 1.1 核心框架
* **Framework**: **Vue 3** (Composition API)
    * *理由*: 响应式系统非常适合游戏 UI 的高频数据绑定（如实时跳动的算力数值），且开发效率高。
* **Build Tool**: **Vite**
    * *理由*: 秒级热更新，极佳的开发体验。
* **Language**: **TypeScript** (强烈推荐)
    * *理由*: 游戏逻辑涉及复杂的对象结构（玩家存档、装备属性、技能接口），强类型能避免 90% 的 `undefined` 运行时错误。

### 1.2 状态管理与逻辑核心
* **State Management**: **Pinia**
    * *理由*: 比 Vuex 更轻量，完美支持 TS，适合管理庞大的玩家存档数据 (SaveData)。
* **Multithreading**: **Web Workers** (**关键**)
    * *理由*: 挂机游戏的“挖矿循环”和“数值计算”必须放在 Worker 线程中运行。
    * *好处*: 避免主线程渲染 UI 时造成计算卡顿；确保浏览器标签页切换到后台时，游戏逻辑依然能准确运行（避免 Chrome 的后台节流机制）。
* **Big Number Library**: **break_infinity.js** 或 **decimal.js**
    * *理由*: JS 原生 `Number` 在 $1.79 \times 10^{308}$ 后会溢出。修仙游戏后期数值极易膨胀（如 $10^{1024}$），必须使用大数库处理。

### 1.3 UI 与 视觉特效
* **UI Framework**: **Tailwind CSS**
    * *理由*: 原子化 CSS，非常适合快速构建“网格布局”和“仪表盘”，且方便自定义赛博朋克配色（Neon Colors）。
* **Visual Effects**:
    * **CRT Shader**: 使用纯 CSS `animation` 和 `box-shadow` 模拟老式显示器的扫描线、辉光和屏幕曲率。
    * **Glitch Effect**: 使用 CSS `clip-path` 制作受击时的故障特效。
    * **Combat Rendering**: 建议使用 **HTML DOM + CSS Transform**（性能足够且易调试）。如果后期弹幕量极大，可引入 **Pixi.js** (WebGL)。

### 1.4 数据持久化
* **Local Storage**: **LocalStorage** + **LZ-String**
    * *理由*: 使用 LZ-String 对存档 JSON 进行压缩和 Base64 编码。既节省空间，又防止玩家轻易修改 LocalStorage 数值（防君子不防小人）。
* **Offline Calculation**: 记录 `lastSaveTime`，玩家上线时计算 `(Date.now() - lastSaveTime)` 差值，模拟离线收益。

---

## 2. 后端架构 (Backend - The Cloud)

用于防作弊校验、云存档同步、排行榜以及实时聊天。

### 2.1 服务端 (Server)
* **Language**: **Go (Golang)**
    * *理由*: 高并发，低内存占用，非常适合处理大量即时的“握手”请求或聊天广播。
* **Web Framework**: **Gin**
    * *理由*: 轻量、高性能，路由处理极快，社区生态成熟。
* **Protocol**:
    * **HTTP/REST**: 用于账号注册、云存档上传/下载 (`POST /save`)、排行榜拉取。
    * **WebSocket**: 用于“世界频道”聊天室、实时“宗门战”通知。

### 2.2 数据库与缓存 (Data)
* **Database**: **PostgreSQL** 或 **MySQL**
    * *理由*: 存储用户账号基础信息、加密后的存档字符串（Blob/Text）。
* **Cache / Leaderboard**: **Redis** (**关键**)
    * *理由*: Redis 的 `Sorted Set` (ZSET) 是实现全服排行榜（算力榜、层数榜）的最佳方案，查询速度极快 (O(log(N)))。


---

## 4. 开发路线图 (Tech Roadmap)

### Phase 1: 原型机 (Prototype) - *纯前端*
* [ ] 初始化 Vue3 + Vite + Tailwind 项目。
* [ ] 引入 `break_infinity.js`，实现基础的点击 `count++`。
* [ ] 编写 `GameLoop` (Web Worker)，实现每秒自动增加资源。
* [ ] 实现 `Save/Load` 功能 (LocalStorage)。

### Phase 2: 核心玩法 (Core Gameplay)
* [ ] 定义 **Player** 和 **Skill** 的数据结构 (TypeScript Interfaces)。
* [ ] 开发 **战斗系统** (时间轴碰撞逻辑)。
* [ ] 制作核心 UI：仪表盘、日志窗口、CRT 滤镜效果。

### Phase 3: 联网功能 (Online Features) - *引入 Go 后端*
* [ ] 搭建 Gin 服务，实现 JWT 登录注册。
* [ ] 实现云存档接口 (`POST /api/save`, `GET /api/load`)。
* [ ] 接入 Redis，实现 `GET /api/leaderboard` 排行榜功能。

---

## 5. 目录结构建议 (Project Structure)

```text
Protocol-Cloud-Ascension/
├── client/                 # 前端工程 (Vue 3)
│   ├── src/
│   │   ├── assets/         # 字体、图片、CSS
│   │   ├── components/     # UI组件 (TerminalWindow, StatBar...)
│   │   ├── core/           # 游戏核心逻辑 (纯TS，不依赖UI框架)
│   │   │   ├── gameLoop.ts # 主循环
│   │   │   ├── combat.ts   # 战斗计算公式
│   │   │   └── math.ts     # 大数库封装
│   │   ├── stores/         # Pinia 状态管理 (usePlayerStore)
│   │   ├── workers/        # Web Workers (挖矿/离线计算)
│   │   └── App.vue
│   ├── vite.config.ts
│   └── package.json
├── server/                 # 后端工程 (Go)
│   ├── cmd/
│   │   └── main.go         # 入口文件
│   ├── internal/
│   │   ├── api/            # Gin Handlers (Controllers)
│   │   ├── models/         # DB Models (Structs)
│   │   ├── service/        # 业务逻辑层
│   │   └── websocket/      # WS Hub
│   ├── go.mod
│   └── Dockerfile
├── docker-compose.yml      # 本地环境编排
└── README.md
