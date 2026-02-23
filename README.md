## M-project：把你所有的“过去积累”一网打尽地召回

M-project 是一个围绕「长期记忆召回」打造的个人知识基础设施。

它把 Obsidian 笔记、n8n 自动化、RAGFlow 检索（以及可选的 MinerU、S-project 等外部信息源）串成一条流水线，让你只需要问一个问题，就能把自己过去写过、看过、收藏过的所有相关内容“一网打尽”地调出来。

---

## 解决什么问题

- 笔记太多记不住  
  写了无数笔记、文章、学习记录，但写完就“进博物馆”，几乎不会再翻。真正需要的时候，只记得“好像写过/看过类似的东西”，却根本找不到。

- 关键时刻想不起来自己曾经的积累  
  准备面试、做复盘、规划项目时，其实以前系统地学过、做过项目、写过总结，但事后只剩模糊印象，很难完整调出当时的思路和证据。

- 想“一键召回所有相关旧知识”  
  当你输入一个问题时，希望不是只搜到一两篇散落的笔记，而是把你过去所有相关的思考、实验记录、学习笔记、收藏文章，一次性都翻出来，按重要程度排好，并能直接点击跳转源头。

- 想快速打包分享“过去所有的记录”  
  想给别人展示你在某个方向的长期积累（比如某门技术、某个岗位、某个主题），希望只说一句“帮我整理一下关于 XXX 的所有记录”，系统就能自动汇总成一份结构化的材料。

- 不只是自己的笔记，还包括曾经看过的好内容  
  配合 S-project，把你在网上冲浪时收藏/点赞/关注过的高质量内容也纳入知识库。比如你曾经在小红书、博客、论坛上收藏过大量面经、实验结果、实战文章，只需要输入“某一个岗位的面经”，系统就会从“自己的笔记 + 曾经收藏过的面经和记录”中，一网打尽地收集并汇总出来。

M-project 的目标是：让你不再为“我好像以前学过/看过这个，但具体在哪？”而焦虑，而是把过去所有的积累变成可随时召回、可复用的长期资产。

---

## 系统架构总览

从整体上看，M-project 由四类组件组成：

- Obsidian 端
  - 通过 Local REST API 插件开放本地 API。
  - 使用 Templater 插件编写查询模板脚本，在当前光标位置触发“问问题 → 等待 → 回填结果”。

- 自动化编排层：n8n
  - Write Path：监控本地文件变更（如笔记创建/修改），经过防抖和预处理后，送入 RAGFlow 做摄入和索引更新。
  - Read Path：作为 Webhook 服务，接收来自 Obsidian 的查询请求，调用 RAGFlow 检索，经过 Code 节点格式化，再把结果回传给 Obsidian。

- 检索与知识引擎：RAGFlow
  - 负责文档解析、文本分块、向量化和索引构建（ingestion pipeline）。
  - 支持多种检索方式（向量检索、关键字检索 BM25、Hybrid Search、Rerank、RAPTOR 等）。

- 可选增强组件
  - MinerU：高质量 PDF/图片 OCR，作为 RAGFlow PDF parser 的补充，提高复杂文档的解析质量。
  - S-project：收集你在互联网上“看过/收藏过”的高质量内容（如小红书面经、博客收藏等），统一汇入同一知识空间，让“外部冲浪记录”也能被召回。

整体数据流可以概括为：

- 写入路径：文件/笔记变化 → n8n 工作流 → RAGFlow 摄入与索引 → 可被检索。
- 读取路径：Obsidian 快捷键 → n8n Webhook → RAGFlow 检索 → n8n 格式化 → 回填到 Obsidian 光标位置。

---

## 功能特性与典型场景

- 防抖自动同步（Write Path）
  - 监控本地笔记库（例如 Obsidian vault）文件变化。
  - 通过 n8n 的 Local File Trigger 等节点，在短时间内合并多次变更，避免重复摄入。
  - 自动将新笔记/更新笔记推送到 RAGFlow，对应知识库及时更新。

- 灵感即时查询（Read Path）
  - 在 Obsidian 中按下一个快捷键（如 `Alt + Q`），弹出输入框。
  - 输入问题后，系统一键调用 n8n → RAGFlow，从你的长期知识库中检索相关内容。
  - 前若干条返回内容为“高质量长摘要”，后续条目为带 Obsidian 跳转链接的“线索列表”，方便深入追踪。

- 多源知识统一召回
  - 本地 Obsidian 笔记。
  - 通过 S-project 等管道收集的外部内容（例如你收藏/点赞/关注过的面经、实测记录、技术文章等）。
  - 若这些内容被摄入到 RAGFlow 对应知识库，检索时会统一被召回。

- 典型应用场景
  - 面试准备：输入“某某岗位的面经”，系统自动汇总你写过的面试记录 + 收藏过的面经帖子 + 实测结果，一网打尽。
  - 主题复盘：输入“我之前学 XXX 的全部记录”，自动聚合课堂笔记、实验记录、复盘、收藏文章等。
  - 输出前的“拉全史”：准备做一次分享/输出，事先只需要抛出主题问题，让系统帮你把“过去在这件事上到底干了什么”全部找齐。

---

## 环境要求

> 以下是当前实践环境的典型组合，仅作参考，你可以按需调整。

- 操作系统
  - Windows 10/11（建议搭配 WSL2 使用 Docker）。

- 必备软件与组件
  - Obsidian（需要安装 Local REST API 和 Templater 插件）。
  - n8n（建议使用 Docker 部署）。
  - RAGFlow（Docker 部署，默认 Web UI 与 API 端口建议配置为：
    - CPU 版本：`http://localhost:8090`
    - GPU 版本（如有）：`http://localhost:9390`
  - Docker Desktop + WSL2
    - 建议通过 `.wslconfig` 为 WSL2 分配足够内存和 CPU，以防止 RAGFlow 等服务在处理大文档时被 OOM 或表现为“CPU 0% 假死”。

- 可选组件
  - MinerU：用于高质量 PDF/图片 OCR，作为 RAGFlow PDF parser 的补充。
  - S-project：用于从各类外部平台收集你“看过/收藏过”的内容，并统一输出给 RAGFlow。

---

## 快速开始

这部分以“能跑起来”为目标，只描述关键步骤。详细配置请参考后续章节以及各官方文档。

### 1. 准备 Docker 与 WSL2

- 安装 Docker Desktop 并启用 WSL2 集成。
- 在用户目录下创建或编辑 `.wslconfig`（可选但强烈推荐），为 WSL2 分配：
  - 足够内存（例如 10–12GB，用于 RAGFlow 等重负载服务）。
  - 合理的 CPU 核数（例如 6–7 核）。
  - 合适的 swap 和 `virtioFiles` 以优化 I/O 与资源回收。

### 2. 启动 RAGFlow

- 按照 RAGFlow 官方文档完成安装与启动（支持 CPU 或 GPU）。
- 确保 Web UI/接口可访问，如：
  - CPU：`http://localhost:8090`
  - GPU：`http://localhost:9390`
- 如果你希望直接复用本仓库中随 RAGFlow 官方发行包一并提供的 Docker 配置，可以使用根目录下的 `RAGFLOW config/` 目录：
  - `docker-compose.yml`、`docker-compose-base.yml`：官方基础 compose 配置。
  - `docker-compose-gpu.yml`：在官方配置基础上调整了 GPU 版本的启动方式，并预留了 MinerU 相关环境变量。
  - `.env`：需要根据你的本机环境修改 RAGFlow、向量库、MinerU 等服务的环境变量。
  - `.env.example`：已为你准备好的示例配置，可复制为 `.env` 后再按需修改敏感信息。
  启动 RAGFlow 时，建议以该目录下的 compose 文件和 `.env` 为模板，具体说明见 `RAGFLOW config/readme.md`。
  例如，在命令行中可以这样一键启动（以 CPU 版本为例）：

  ```bash
  cd "RAGFLOW config"
  # 第一次使用时，从示例文件复制一份环境变量配置
  cp .env.example .env      # Windows PowerShell 可使用：Copy-Item .env.example .env

  # 启动 CPU 版本 RAGFlow（使用 docker-compose.yml）
  docker compose --profile cpu up -d

  # 如需启动 GPU 版本（确保已正确安装并配置 NVIDIA 驱动）：
  # docker compose --profile gpu up -d
  ```
- 在 RAGFlow 中创建至少一个知识库，用于接收来自 Obsidian/n8n 的文档。

### 3. 部署并配置 n8n

- 使用 Docker 或其他方式启动 n8n 服务，确保可以访问其 Web UI。
- 创建两个核心工作流（推荐之后导出为 JSON 保存到仓库）：
  - 防抖自动同步流（Write Path）：
    - 监听笔记目录变更，将文件内容送入 RAGFlow 对应知识库进行摄入和索引。
  - 灵感即时查询流（Read Path）：
    - Webhook 入口（`POST /webhook/rag-query`）→ HTTP Request 调用 RAGFlow 检索 → Code 节点格式化结果 → Respond to Webhook 返回 Markdown 文本。

### 4. 配置 Obsidian 插件

- 安装 Local REST API 插件，并按照插件文档完成证书和接口设置。
- 安装 Templater 插件：
  - 创建模板文件夹（例如：`Templates/Templater`）。
  - 在模板中创建一个 RAG 查询脚本（如 `RAG_Query`），脚本逻辑大致为：
    - 弹出输入框获取 query。
    - 在当前光标位置插入“正在检索”的提示。
    - 向 n8n Webhook 发送 `POST` 请求（Body 中包含 query）。
    - 拿到 n8n 返回的 Markdown 文本后，替换掉 loading 提示。
- 在 Templater 设置中为该模板配置快捷键（例如 `Alt + Q`），方便随时触发。

### 5. 做一次首个检索测试

- 在你的 Obsidian 笔记中打开任意页面。
- 按下快捷键（如 `Alt + Q`），输入一个你曾经写过/收藏过的主题问题，例如“某某岗位的面经”。
- 若一切正常，你会看到：
  - 页面中出现一个“正在调动知识库”的提示。
  - 几秒后，该区域被替换为：
    - 前若干条的详细内容（适合直接阅读和复制）。
    - 后续条目的列表，每一项都带有 `obsidian://` 跳转链接，可以直接打开对应原始笔记。

---

## 组件与配置指南（概览）

> 这一部分只给出配置要点，具体细节可以拆到独立文档或 n8n/RAGFlow 的导出配置中维护。

### Obsidian 端

- Local REST API 插件
  - 负责在本地开放可供脚本调用的 API。
  - 配置证书与信任链后，可通过 HTTPS 与 Obsidian 通信。

- Templater + RAG_Query 模板
  - 负责交互体验：弹窗获取 query → 显示 loading → 接收并插入 Markdown 结果。
  - 建议：
    - 对 n8n Webhook URL、端口等做成易于修改的变量。
    - 对错误情况（如 n8n 连接失败）给出清晰提示。

### n8n 工作流

- 防抖自动同步流（Write Path）
  - Local File Trigger（或其他监控节点）：
    - 监听本地挂载路径（如 `/data/notes`），忽略不需要的文件或频繁变动的缓存。
  - 中间可以增加过滤、去抖动逻辑：
    - 合并短时间内的多次文件变更。
    - 按文件类型/路径路由到不同的 RAGFlow 知识库（例如“手册类 PDF”单独进入 manual 知识库）。
  - 最终通过 HTTP Request 节点调用 RAGFlow 的摄入接口，传送文件和元数据。

- 灵感即时查询流（Read Path）
  - Webhook 节点：
    - `HTTP Method`：`POST`
    - `Path`：`rag-query`
    - `Response Mode`：`When Last Node Finishes`
  - HTTP Request 节点：
    - 调用 RAGFlow 检索接口，Body 中包含来自 Webhook 的 `query`。
  - Code 节点：
    - 对 RAGFlow 返回的结果数组进行处理：
      - 前若干条（如 Top 10）：提取核心内容（如 `content_with_weight`），组成“深度参考”段。
      - 后续条目（如 11–50）：只保留标题/简短摘要 + Obsidian 跳转链接（`obsidian://open?vault=...&file=...`）。
    - 返回合并后的 Markdown 字符串。
  - Respond to Webhook 节点：
    - 将 Code 节点生成的 Markdown 文本作为 Response 返回给 Obsidian。

### RAGFlow 配置

- ingestion pipeline（摄入流水线）
  - 解析器（Parser）：
    - 根据不同文件类型选择不同的解析策略：
      - PDF：可选用 RAGFlow 的 DeepDoc，或通过 HTTP 调用 MinerU 输出结果后再交给 RAGFlow。
      - Word / PPTX：一般默认解析表现良好，可根据实践调整。
      - Markdown（特别是 Obsidian 笔记）：
        - 推荐启用 AST 切片策略，识别标题、列表、代码块。
        - 将 `Chunk token number` 调整到 512–800，以保证每个小标题下的内容尽可能被完整保留为一个语义块。
  - 转换器（Converter）：
    - 分块器与文本清洗逻辑，可根据文档类型做不同配置。
  - 索引器（Indexer）：
    - 支持多路径召回，可配置向量索引、关键字索引等。

- 检索配置
  - Hybrid Search：
    - 同时开启向量检索和关键词检索（BM25）。
    - 建议初始权重：
      - 向量检索权重：`0.7`
      - 关键字检索权重：`0.3`
  - 相似度阈值：
    - 默认通常较低（如 0.2），可能导致结果太杂。
    - 如果你希望“每一条都是真的相关”，可以尝试调高到 `0.3–0.35`。
  - Rerank 模型：
    - 推荐开启，如使用类似 `bge-reranker` 一类的模型，对初筛的 Top K 结果做二次排序。
  - 返回条数（Top N）：
    - 视自己的使用习惯确定，可采用“前 10 条深度 + 后 40 条线索”的模式。

- 高级功能
  - DeepDoc OCR：
    - 用于解析包含图片、扫描件、流程图、思维导图的 PDF 或截图。
  - RAPTOR（递归摘要树检索）：
    - 适用于长篇笔记或大量文档的高层级查询，通过摘要层级聚合，提高对“大问题”的回答能力。
  - Auto-Keyword / Auto-Question：
    - 自动提取关键词与潜在问答对，提高检索命中率和语义召回能力。

### MinerU（可选）

- 适用场景
  - 对复杂 PDF（尤其是多列排版、公式密集、扫描件）有更高的 OCR 和结构化解析要求。
- 典型用法
  - 使用 Python/conda 环境安装 MinerU。
  - 启动 MinerU API 服务，并在 RAGFlow 的 PDF 解析配置中，指定通过 HTTP 调用 MinerU 完成前置解析。

---

## 调优与最佳实践

- 检索策略选择
  - 多数实际场景下，单纯的语义向量检索未必最佳，关键字检索（BM25）对于专有名词、项目代号等尤其有效。
  - 建议使用 Hybrid Search，结合两者的优势，并配合 Rerank 提高最终排序质量。

- 文档解析策略
  - PDF：
    - 区分“手册类”“论文类”“扫描类”等不同类型，选择对应解析模板或送入不同解析器。
  - Markdown（Obsidian）：
    - 尽量保持结构清晰（标题层级、列表、代码块），有利于 AST 切片。
    - 如有自定义分隔符（例如 `---`），可在解析配置中显式声明。

- Windows + WSL + Docker 环境
  - 通过 `.wslconfig` 为 WSL2 分配足够的内存和 CPU，避免容器因 OOM 或调度问题被“假死”。
  - 关闭 Docker Desktop 中部分自动资源优化选项，改为显式配置资源，提升稳定性。

---

## 常见问题（FAQ）

- Q：为什么 Docker 中 RAGFlow 的 CPU 使用率一直显示 0%，服务似乎卡死？  
  A：可能是 WSL2 资源不足或调度问题导致容器进程被暂停或长时间等待调度。建议：
  - 在 `.wslconfig` 中为 WSL2 分配足够内存和 CPU。
  - 合理配置 swap 与 `virtioFiles`。
  - 在 Docker Desktop 中查看容器日志，确认是否存在 OOM 或挂起。

- Q：n8n 的 Local File Trigger 为什么用不了？  
  A：在较新版本中，该节点可能默认被关闭或需要额外配置。可以：
  - 检查 docker-compose 配置，确保相关节点插件启用。
  - 如有需要，可考虑使用社区文件节点（例如 `n8n-nodes-fs`）替代。

- Q：Windows 下 n8n 里要写什么路径？  
  A：如果你在 Docker 中把本地路径挂载为 `/data/notes`，那么在 n8n 的文件相关节点里，应该使用 Linux 风格路径，例如 `/data/notes`，而不是 `C:\...`。

- Q：RAGFlow API 返回字段和示例不一样，Code 节点报错怎么办？  
  A：RAGFlow 不同版本之间返回结构可能有微调。遇到问题时：
  - 在 n8n 中打开 RAGFlow 返回的原始 JSON。
  - 根据实际字段名（如 `doc_name`、`document_name`、`content_with_weight` 等）调整 Code 节点的访问逻辑。

- Q：如何在手机端 Obsidian 使用这个功能？  
  A：需要将 n8n 的 Webhook 端口（如 5678）从内网暴露到公网或局域网可访问：
  - 可使用 cpolar、frp 等内网穿透工具。
  - 注意安全性与鉴权配置。

---

## 项目状态与 Roadmap（示例）

> 下面仅为示例，可根据实际进度在本仓库中维护。

- [ ] 整理并提交 n8n 工作流的示例导出文件（JSON）。
- [ ] 整理 RAGFlow ingestion/retrieval 配置示例。
- [ ] 抽取 Obsidian 端 Templater 模板为独立示例文件。
- [ ] 集成 S-project 的输出，打通“外部收藏内容 → RAGFlow”的全链路。
- [ ] 探索引入 GraphRAG 等更高级检索方式，增强跨笔记关系推断能力。

---

## 推荐的仓库结构与示例文件（可选）

> 以下为推荐结构，不一定全部存在于当前仓库，可按需逐步补充。

- `project_doc/`：内部工程文档（需求、规格、架构说明、测试计划等）。
- `config/`
  - `n8n/`：存放防抖自动同步流、灵感即时查询流的导出 JSON 示例。
  - `ragflow/`：存放 ingestion pipeline 与检索配置的示例 JSON。
  - `docker/`：存放 n8n、RAGFlow 等服务的示例 docker-compose 文件。
- `obsidian/`
  - `templates/`：示例 Templater 模板（如 `RAG_Query.md`）。
  - `scripts/`：如果有，将复杂的脚本抽为独立 JS 文件。
- `scripts/`
  - 一键启动/停止 RAGFlow、n8n、MinerU 等服务的脚本
