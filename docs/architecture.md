# 工程目录与架构

设计目标：GitHub 开源友好、Mac mini 单机部署、Web 承载训练与多动作复盘。采用**一个仓库、一个 Web 前端工程、一个 Python 包、API 与 worker 两个后端进程**；MCP Server 是同一组应用用例的另一入口。分析能力先用网球正手验证，产品结构不以正手命名或限制其他动作。Web 的详细设计见 [Web 产品与项目结构](web-project-structure.md)。

## 推荐目录

```text
racket-video-lab/
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── .gitignore
├── .env.example
├── pyproject.toml
├── web/                         React + TypeScript + Vite 前端工程
│   ├── package.json
│   ├── vite.config.ts
│   ├── index.html
│   └── src/
│       ├── app/                 入口、路由、全局状态
│       ├── pages/               训练列表、训练详情、动作复盘
│       ├── features/            视频导入、任务进度、动作片段、证据复核
│       └── shared/              API 客户端、通用 UI、时间码等工具
├── src/racketlab/
│   ├── core/                  config.py, logging.py, errors.py
│   ├── domain/                session、video、analysis、action_segment、evidence
│   ├── application/           导入、提交分析、查询片段、复核证据等用例
│   ├── infrastructure/
│   │   ├── db/                session.py, tables.py, repositories.py
│   │   ├── media/             ffprobe.py, ffmpeg.py, local_store.py
│   │   └── vision/            pose.py, model_loader.py
│   ├── pipeline/              解码、候选切分、动作分类、分析、证据渲染
│   ├── adapters/
│   │   └── tennis/            网球动作分类与各动作分析模块；先实现正手
│   ├── api/                   app.py, dependencies.py, schemas.py,
│   │   └── routes/            sessions、videos、analyses、segments、media 等
│   ├── mcp/                   server.py, tools.py, formatting.py
│   ├── workers/               main.py, sqlite_queue.py
│   └── cli.py
├── migrations/                env.py, versions/
├── tests/                     后端单元、集成与接口契约验证
├── docs/                      产品、架构、拍摄说明、输出契约
├── integrations/workbuddy/    联调说明；Skill 待验证
├── deploy/launchd/            API 与 worker 模板；Web 构建产物由 API 提供
├── deploy/docker/             后续跨平台部署
└── scripts/                   环境检查与示例导入
```

这棵树是**目标布局**，当前仓库只创建文档。开始编码时按实际用例逐步创建文件，不预建空模块。

## 职责与依赖

| 边界 | 职责 |
| --- | --- |
| `core` | 小范围通用运行配置、日志、异常；不当作杂项目录。 |
| `web` | 训练记录、导入、任务状态、动作时间轴、证据复核；通过 HTTP API 读写，不实现分析算法。 |
| `domain` | 训练、视频、分析任务、动作片段、观察、证据和人工复核等业务概念；定义仓储及媒体接口，不依赖框架。 |
| `application` | 导入、提交、执行和查询用例；协调分析与持久化。 |
| `infrastructure` | SQLite、FFmpeg、文件系统和模型加载的具体实现。 |
| `pipeline` | 从视频到候选动作片段、动作类别、可验证特征和证据素材的处理步骤。 |
| `adapters` | 运动项目的术语、分类、规则和参数。先实现网球；第二运动项目出现时再提取共用协议。 |
| `api`、`mcp`、`cli` | 不同入口，共用 `application`，不各自实现分析逻辑。 |
| `workers` | 领取任务、状态、超时恢复和调用用例；不放动作算法。 |
| `storage` | 指运行数据布局，不另建与 `infrastructure` 重叠的代码目录。 |
| `tests` | 单元测试侧重时间轴和动作规则；集成测试覆盖数据库、FFmpeg、API/MCP 契约。 |
| `docs`、`scripts` | 用户说明、工程决策及可重复的维护命令。 |

`web → api → application → domain`；`mcp/cli → application → domain`。外部能力由 `infrastructure` 实现 `domain/ports.py` 所定义的接口；`domain` 不导入 FastAPI、MCP、ORM 或 OpenCV。Web 的 API 类型从后端 OpenAPI 契约生成或校验，避免维护两份不一致的手写模型。

## MVP 工作流

创建训练记录 → 从 Web 导入视频 → ffprobe 校验 → 建立 `queued` 任务 → worker 领取 → 筛选候选动作片段 → 对已支持的动作执行分析 → 保存证据和结果 → Web 或 MCP 复核与查询。任务状态为 `queued → running → completed/failed`，记录阶段、进度、错误、分析版本及模型版本。worker 重启时可回收过期的 `running` 任务。未支持或无法判断的动作保留片段与原因，不伪装成已完成诊断。

最小入口建议：`POST /api/v1/sessions`、`POST /api/v1/videos`、`POST /api/v1/analyses`、`GET /api/v1/analyses/{id}`、`GET /api/v1/analyses/{id}/segments`、`GET /api/v1/segments/{id}`、`GET /api/v1/capabilities`；片段复核与媒体读取入口见 Web 文档。MCP 的片段查询也应使用通用动作语义。分析返回时间戳、动作类型、观察值、证据、置信度和无法判断的原因，而不只是一段教练式文案。

## 运行数据

```text
<RACKETLAB_DATA_DIR>/
├── racketlab.sqlite3
├── media/<video_id>/
│   ├── original.mp4
│   └── analyses/<analysis_id>/{clips,frames,result.json}
├── tmp/
├── models/
└── logs/
```

原片、数据库、模型及临时文件不进入 Git。先用 SQLite WAL 和单 worker；数据库留在 Mac mini 本地盘。配置从环境变量读取，仓库保留 `.env.example`。从第一张表开始使用 Alembic 迁移。模型文件需记录来源、许可证、版本和校验值。

## 后续演进

- 正手经实测后逐步增加反手、发球、截击和步法模块；Web 通过能力清单展示各动作当前的可分析状态。
- 第二个运动项目出现时再抽取共用的运动适配接口。
- 单机队列确实成为瓶颈时更换任务后端；多用户并发写入增加时迁移 PostgreSQL。
- WorkBuddy 的 Skill 目录与配置应在确认其当前接入规范并完成联调后落地。
- `launchd` 服务 Mac mini 本地部署；Docker 留给跨平台或服务器环境。
