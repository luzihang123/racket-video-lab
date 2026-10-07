# 工程目录与架构

设计目标：GitHub 开源友好、Mac mini 单机部署、第一阶段只验证网球正手。采用**一个仓库、一个 Python 包、API 与 worker 两个进程**；MCP Server 是同一组应用用例的另一入口。

## 推荐目录

```text
racket-video-lab/
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── .gitignore
├── .env.example
├── pyproject.toml
├── src/racketlab/
│   ├── core/                  config.py, logging.py, errors.py
│   ├── domain/                models.py, enums.py, ports.py
│   ├── application/           ingest_video.py, submit_analysis.py,
│   │                          run_analysis.py, get_analysis.py
│   ├── infrastructure/
│   │   ├── db/                session.py, tables.py, repositories.py
│   │   ├── media/             ffprobe.py, ffmpeg.py, local_store.py
│   │   └── vision/            pose.py, model_loader.py
│   ├── pipeline/              types.py, decode.py, detect_candidates.py,
│   │                          analyze_forehand.py, render_evidence.py
│   ├── adapters/
│   │   └── tennis/            forehand.py, terminology.py
│   ├── api/                   app.py, dependencies.py, schemas.py,
│   │   └── routes/            videos.py, analyses.py, health.py
│   ├── mcp/                   server.py, tools.py, formatting.py
│   ├── workers/               main.py, sqlite_queue.py
│   └── cli.py
├── migrations/                env.py, versions/
├── tests/                     unit/, integration/, fixtures/
├── docs/                      产品、架构、拍摄说明、输出契约
├── integrations/workbuddy/    联调说明；Skill 待验证
├── deploy/launchd/            API 与 worker 模板
├── deploy/docker/             后续跨平台部署
└── scripts/                   环境检查与示例导入
```

这棵树是**目标布局**，当前仓库只创建文档。开始编码时按实际用例逐步创建文件，不预建空模块。

## 职责与依赖

| 边界 | 职责 |
| --- | --- |
| `core` | 小范围通用运行配置、日志、异常；不当作杂项目录。 |
| `domain` | 视频、任务、击球、发现、置信度等业务概念；定义仓储及媒体接口，不依赖框架。 |
| `application` | 导入、提交、执行和查询用例；协调分析与持久化。 |
| `infrastructure` | SQLite、FFmpeg、文件系统和模型加载的具体实现。 |
| `pipeline` | 从帧到候选击球、动作特征、证据片段的处理步骤。 |
| `adapters` | 球类术语、规则和参数。先实现网球；第二球类出现时再提取共用协议。 |
| `api`、`mcp`、`cli` | 不同入口，共用 `application`，不各自实现分析逻辑。 |
| `workers` | 领取任务、状态、超时恢复和调用用例；不放动作算法。 |
| `storage` | 指运行数据布局，不另建与 `infrastructure` 重叠的代码目录。 |
| `tests` | 单元测试侧重时间轴和动作规则；集成测试覆盖数据库、FFmpeg、API/MCP 契约。 |
| `docs`、`scripts` | 用户说明、工程决策及可重复的维护命令。 |

`api/mcp/cli → application → domain`。外部能力由 `infrastructure` 实现 `domain/ports.py` 所定义的接口；`domain` 不导入 FastAPI、MCP、ORM 或 OpenCV。

## MVP 工作流

导入本地视频 → ffprobe 校验 → 建立 `queued` 任务 → worker 领取 → 抽帧与筛选正手候选 → 提取可验证特征 → 保存证据和结果 → HTTP/MCP 查询。任务状态为 `queued → running → completed/failed`，记录阶段、进度、错误、分析版本及模型版本。worker 重启时可回收过期的 `running` 任务。

最小入口建议：`POST /api/v1/videos`、`POST /api/v1/analyses`、`GET /api/v1/analyses/{id}`；MCP 提供 `submit_analysis`、`get_analysis`、`list_shots`、`get_shot_evidence`。分析应返回时间戳、观察值、置信度和无法判断的原因，而不只是一段教练式文案。

## 运行数据

```text
<RACKETLAB_DATA_DIR>/
├── racketlab.sqlite3
├── media/<video_id>/
│   ├── original.mp4
│   └── analyses/<analysis_id>/{clips,evidence,result.json}
├── tmp/
├── models/
└── logs/
```

原片、数据库、模型及临时文件不进入 Git。先用 SQLite WAL 和单 worker；数据库留在 Mac mini 本地盘。配置从环境变量读取，仓库保留 `.env.example`。从第一张表开始使用 Alembic 迁移。模型文件需记录来源、许可证、版本和校验值。

## 后续演进

- 正手经实测后增加反手、发球、截击和步法模块。
- 第二个运动项目出现时再抽取共用的运动适配接口。
- 单机队列确实成为瓶颈时更换任务后端；多用户并发写入增加时迁移 PostgreSQL。
- WorkBuddy 的 Skill 目录与配置应在确认其当前接入规范并完成联调后落地。
- `launchd` 服务 Mac mini 本地部署；Docker 留给跨平台或服务器环境。
