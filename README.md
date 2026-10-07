# Racket Video Lab

Mac mini 上运行的本地优先小球运动视频分析项目。第一阶段聚焦网球正手：导入视频、筛选候选击球、生成带时间戳的证据与分析结果，再通过 HTTP API 或 MCP 在聊天工具中查询。

**仓库名：** `racket-video-lab`  
**工作名：** RacketLab  
**状态：** 项目规划；目前只有调研和工程目录设计，尚无可运行的分析软件。

## 从哪里读起

- [竞品分析](docs/competitor-analysis.md)：延续前期对话中的 10 个竞品，区分已核实信息与产品判断。
- [工程目录与架构](docs/architecture.md)：推荐的单仓库目录树、模块职责和依赖方向。
- [技术调研](docs/technical-research.md)：FastAPI、MCP、任务执行、媒体处理与 Mac mini 部署的资料。

## 第一阶段范围

输入一段本地网球训练视频，输出候选正手片段、时间戳、少量有证据支持的动作观察和无法判断的原因。普通单机位视频没有标定时，不输出未经验证的厘米级空间测量。

建议技术起点：Python、FastAPI、FFmpeg/ffprobe、视觉处理 pipeline、SQLite、独立 worker、MCP Server。WorkBuddy 接入格式以实际客户端文档和联调结果为准。

运行数据、用户视频、模型文件及数据库应放在仓库外，由 `RACKETLAB_DATA_DIR` 指定。
