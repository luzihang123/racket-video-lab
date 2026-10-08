# 技术调研与依据

调研日期：2026-10-07。以下优先引用项目或标准的官方资料；“建议”是针对 Racket Video Lab 的设计选择，并非官方规定。

| 主题 | 官方资料与要点 | 本项目建议 |
| --- | --- | --- |
| FastAPI 模块化 | [Bigger Applications](https://fastapi.tiangolo.com/tutorial/bigger-applications/) 展示多文件及 APIRouter。 | 按视频、分析、健康检查拆路由，入口保持薄。 |
| Web 前端 | [Vite Getting Started](https://vite.dev/guide/) 提供 React + TypeScript 模板。 | `web/` 使用 React、TypeScript、Vite，按训练、导入、任务、片段、证据复核划分功能目录。 |
| Web 静态资源 | [FastAPI Static Files](https://fastapi.tiangolo.com/tutorial/static-files/) 支持挂载静态目录。 | 开发时由 Vite 代理 `/api`；部署时 API 提供 Web 构建产物，单页路由回退另行实现。 |
| 视频拖动播放 | [Starlette FileResponse](https://www.starlette.io/responses/) 支持 HTTP Range。 | 媒体经受控 API 读取；验证原片和证据短片的 Range 请求，不公开数据目录。 |
| Python 项目布局 | [src layout](https://packaging.python.org/en/latest/discussions/src-layout-vs-flat-layout/) 可避免测试意外导入工作树副本；[pyproject.toml 指南](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/)说明包元数据与工具配置。 | 一个 `src/racketlab` 包，根目录一个 `pyproject.toml`。 |
| MCP | [MCP Python SDK](https://py.sdk.modelcontextprotocol.io/)支持 tools、resources、prompts 和 stdio/HTTP 传输。 | 先用本机 stdio 工具入口，封装 `application` 用例；与目标聊天客户端实测。 |
| 长任务 | [FastAPI Background Tasks](https://fastapi.tiangolo.com/tutorial/background-tasks/)对较重计算提示可使用独立工具。 | API 落库并立即返回任务 ID；独立 worker 执行和恢复。 |
| 媒体处理 | [ffprobe](https://ffmpeg.org/ffprobe.html)用于探测媒体信息；[FFmpeg](https://ffmpeg.org/ffmpeg.html)处理抽帧与裁剪。 | 将命令调用集中封装，保存输入、参数、版本和输出路径。 |
| 数据库 | [SQLite WAL](https://sqlite.org/wal.html)支持并发读与单写，且不适用网络文件系统；[Alembic](https://alembic.sqlalchemy.org/en/latest/tutorial.html)管理迁移。 | 单机 SQLite 起步，短事务；先建迁移体系，后续按负载决定是否换 PostgreSQL。 |
| 模型缓存 | [Hugging Face 缓存变量](https://huggingface.co/docs/huggingface_hub/main/package_reference/environment_variables)允许设置模型缓存目录。 | 将模型移到专用数据目录，并记录模型版本和许可证。 |
| Mac mini 常驻 | [Apple launchd 指南](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLaunchdJobs.html)介绍用户 Agent 和系统 Daemon。 | 初期给 API 与 worker 各一个用户级 launchd 模板。 |

## 仍需实验验证

1. 目标 Mac mini 的芯片、内存、磁盘与视频分辨率对处理吞吐的影响。
2. 先验证正手候选检出率、漏检率及动作判断的一致性，再对新增动作类型逐项验证；采用用户授权视频和可复核标注。
3. WorkBuddy 当前版本具体支持的 MCP 启动方式、工具调用与 Skill 安装格式。
4. 选用的姿态模型对拍摄角度、遮挡、不同持拍手和多人入镜的表现。
