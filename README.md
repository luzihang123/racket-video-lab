# Racket Video Lab

Mac mini 上运行的本地优先持拍运动视频分析项目。Web 是训练与动作复盘的主入口：导入视频、查看分析进度、按动作类型浏览片段、复核带时间戳的证据，并记录下一次训练重点。HTTP API 和 MCP 共用同一套分析用例。

**仓库名：** `racket-video-lab`  
**工作名：** RacketLab  
**状态：** 项目规划；目前只有调研和工程目录设计，尚无可运行的分析软件。

## 从哪里读起

- [竞品分析](docs/competitor-analysis.md)：延续前期对话中的 10 个竞品，区分已核实信息与产品判断。
- [工程目录与架构](docs/architecture.md)：包含 `web/` 前端与 Python 后端的单仓库目录、模块职责和依赖方向。
- [Web 产品与项目结构](docs/web-project-structure.md)：页面流程、前端目录、通用动作数据契约、接口与部署边界。
- [技术调研](docs/technical-research.md)：FastAPI、MCP、任务执行、媒体处理与 Mac mini 部署的资料。
- [实战分析经验与 Pipeline 启示](docs/video-analysis-insights.md)：基于 iPhone 真实 19 分钟实战网球视频沉淀的粗细漏斗处理、夜间 ROI 裁剪与生物力学规则库。
- [自拍机位与拍摄参数指南](docs/camera-setup-guide.md)：后侧 45° 黄金机位、正侧 90° 机位对比、手机高度/距离与 iPhone 60fps 拍摄避坑技巧。
- [AI 教练分析 Prompt 模板](docs/prompt-templates.md)：用于快速触发分析、工程化归档与专项问诊的可复用提示词库。
- [参考基准样例 (Example)](examples/forehand-groundstroke/README.md)：正手动作的完整证据包样本（短视频 clip、GIF、5 阶段关键帧与结构化 JSON）。

## 产品范围与第一阶段

产品面向完整训练视频与多类动作；首个运动项目是网球，动作类型可逐步覆盖正手、反手、发球、截击和步法。第一阶段的**分析能力验证**仍从正手开始：输入本地训练视频，输出候选片段、时间戳、少量有证据支持的观察和无法判断的原因。Web 的页面、路由和数据模型从一开始按通用动作设计，实际支持的动作类型由后端能力清单决定，不把未实现的分析显示为已支持。普通单机位视频没有标定时，不输出未经验证的厘米级空间测量。

建议技术起点：Web 采用 React、TypeScript、Vite；后端采用 Python、FastAPI、FFmpeg/ffprobe、视觉处理 pipeline、SQLite、独立 worker、MCP Server。WorkBuddy 接入格式以实际客户端文档和联调结果为准。

运行数据、用户视频、模型文件及数据库应放在仓库外，由 `RACKETLAB_DATA_DIR` 指定。
