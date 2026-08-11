# ls-base

基于 **LangChain / LangGraph + FastAPI** 的智能体应用——**私人厨师（Personal Chief）**：接收用户的食材照片或清单，调用 Web 搜索检索菜谱，从营养与难度维度打分排序，输出结构化推荐方案，并通过流式接口与前端交互。

项目同时保留了从零学习 LLM 智能体的 **Jupyter Notebook 系列**（见下方「学习路线」），覆盖模型接入、消息与多模态、提示词工程、工具、记忆等核心主题，是理解本项目 `app/` 源码的基础。

## 功能特性

- 🧑‍🍳 **私人厨师 Agent**：识别食材照片（多模态）、评估食材、检索菜谱、量化打分排序、输出结构化报告
- 🌐 **Web 搜索工具**：基于 Tavily 实时检索菜谱
- 🧠 **记忆持久化**：LangGraph Checkpointer + SQLite，按 `thread_id` 保存 / 查询 / 清空多轮会话
- ⚡ **流式对话**：SSE（Server-Sent Events）逐 token 输出
- 🖼️ **图片上传**：阿里云 OSS 预签名 URL，前端直传后以图片 URL 参与对话
- 🌍 **FastAPI 后端**：自带 CORS、静态资源托管（内置 Next.js 构建的前端 SPA）

## 技术栈

- **语言 / 环境**：Python >= 3.14，`uv` 包管理（`pyproject.toml` + `uv.lock`）
- **后端**：FastAPI、Uvicorn
- **智能体框架**：LangChain 1.x（`langchain`、`langchain-openai`、`langchain-tavily`）、LangGraph（`langgraph-checkpoint-sqlite`）
- **模型**：阿里云百炼 DashScope 通义千问多模态模型（`qwen3.7-plus`，经 OpenAI 兼容接口接入，支持图片 + 文本）
- **工具**：Tavily 搜索（`langchain-tavily`）
- **存储**：SQLite（对话 checkpointer）、阿里云 OSS（图片直传）
- **前端**：Next.js 构建产物（`app/static/`，随后端托管）

## 目录结构

```
ls-base/
├── app/                       # 主应用（FastAPI + LangGraph Agent）
│   ├── main.py                # FastAPI 入口（CORS、路由、静态托管），端口 8001
│   ├── agents/
│   │   └── personal_chief.py  # 私人厨师 Agent：模型 / Tavily 工具 / SQLite 记忆 / 流式对话
│   ├── api/v1/
│   │   ├── chat.py            # 对话接口（流式、历史、清空）
│   │   └── oss.py             # OSS 预签名上传接口
│   ├── common/logger.py       # 日志配置
│   ├── models/schemas.py      # Pydantic 请求模型（ChatRequest）
│   ├── db/                    # SQLite 数据库（personal_chief.db，自动创建）
│   └── static/                # 前端构建产物（Next.js SPA，随后端托管）
├── notebooks/                 # 学习 Notebook 系列（共 8 个，见「学习路线」）
│   └── resources/             # 资源文件（checkpoint 数据库、示例图片）
├── src/ls_base/               # 项目包（uv 脚手架占位）
├── langgraph.json             # LangGraph 平台配置（chief_agent 图）
├── pyproject.toml             # uv 项目配置与依赖声明
├── uv.lock                    # 依赖锁文件
└── .env                       # 环境变量（已被 .gitignore 忽略）
```

## 快速开始

```bash
# 1. 安装依赖（使用 uv，需 Python >= 3.14）
uv sync

# 2. 创建 .env 文件并填入自己的密钥（见下方「环境变量」表）

# 3. 启动后端服务
uv run python -m app.main
# 或直接使用虚拟环境：
# .venv/Scripts/python.exe -m app.main

# 4. 浏览器访问
#    http://127.0.0.1:8001
```

启动后 `http://127.0.0.1:8001` 会托管前端 SPA，`/docs` 可查看 Swagger 接口文档。

## API 文档

### 1. 流式对话

```
POST /api/v1/chat/stream
```

`Content-Type: application/json`，返回 `text/event-stream`（SSE 逐 token 输出）。

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `message` | string | 是 | 用户输入（文字 / 提问） |
| `image_url` | string | 否 | 图片 URL（如 OSS 上传后的地址），传入后作为多模态输入 |
| `thread_id` | string | 是 | 会话 ID，用于区分不同会话的记忆 |

### 2. 获取历史消息

```
GET /api/v1/chat/messages?thread_id=<会话ID>
```

返回该会话的消息列表：

```json
{ "messages": [ { "role": "user", "content": "..." }, { "role": "assistant", "content": "..." } ] }
```

### 3. 清空历史消息

```
DELETE /api/v1/chat/messages?thread_id=<会话ID>
```

返回 `{ "success": true }`。

### 4. 获取 OSS 预签名上传 URL

```
GET /api/v1/oss/presign?filename=<文件名>
```

为前端生成直传 OSS 的预签名 URL（有效期 1 小时），并返回访问地址：

```json
{
  "uploadUrl": "https://...",
  "contentType": "image/jpeg",
  "accessUrl": "https://<bucket>.<endpoint>/<filename>"
}
```

## 环境变量

| 变量 | 用途 | 说明 |
|------|------|------|
| `DASHSCOPE_API_KEY` | 阿里云百炼 API Key | 访问通义千问多模态模型 |
| `DASHSCOPE_BASE_URL` | 百炼 OpenAI 兼容接口地址 | 配合 `model_provider="openai"` 使用 |
| `TAVILY_API_KEY` | Tavily 搜索 API Key | Web 搜索工具 |
| `OSS_ACCESS_KEY_ID` | 阿里云 AccessKey ID | OSS 预签名接口使用 |
| `OSS_ACCESS_KEY_SECRET` | 阿里云 AccessKey Secret | OSS 预签名接口使用 |
| `OSS_BUCKET` | OSS 存储桶名称 | 未配置时 `/oss/presign` 返回 500 |
| `OSS_ENDPOINT` | OSS Endpoint（可选） | 默认 `oss-cn-beijing.aliyuncs.com` |

> `.env` 已被 `.gitignore` 忽略，请勿提交密钥。模型名（如 `qwen3.7-plus`）请以你的服务商实际支持的模型列表为准。

## 学习路线

| # | Notebook | 主题 | 核心内容 |
|---|----------|------|----------|
| 1 | `1.OpenAI.ipynb` | OpenAI SDK 基础 | 创建客户端、访问模型、获取响应结果 |
| 2 | `2.langchain入门.ipynb` | LangChain 入门 | 定义工具、创建 Agent、发起调用 |
| 3 | `3.models.ipynb` | 模型访问 | `init_chat_model`、社区版 `ChatTongyi`、`invoke` / `stream` 调用、在智能体中使用模型 |
| 4 | `4.messages.ipynb` | 消息与多模态 | 消息类型（`SystemMessage` / `HumanMessage` / `AIMessage`）、在线图片、本地图片（base64） |
| 5 | `5.prompt.ipynb` | 提示词工程 | 系统提示词、角色与指令、对话示例、结构化输出（Pydantic） |
| 6 | `6.tools.ipynb` | 工具（Tool） | 自定义工具（`@tool` 装饰器 / 函数名与文档注释 / Pydantic 参数约束）、预定义工具 Tavily |
| 7 | `7.memory.ipynb` | 记忆（Memory） | 无记忆 vs 短期记忆（Checkpoint / `thread_id`）、SQLite 持久化、记忆管理策略（修剪 / 删除 / 总结摘要） |
| 8 | `8.实战.ipynb` | 实战 | 综合应用：多模态模型识别食材 + Tavily 检索菜谱 + SQLite 记忆，即 `app/` 应用的原型 |

## 注意事项

- Agent 依赖真实 API 调用，需要有效的模型服务密钥；多模态模型调用可能有额外费用，请留意用量。
- 记忆持久化使用 SQLite（`app/db/personal_chief.db`，启动时自动创建）；生产环境建议改用数据库支持的 Checkpointer（如 `langgraph-checkpoint-postgres`）。
- 项目基于 LangGraph 平台开发时，可通过 `langgraph.json` 的 `chief_agent` 图启动本地开发服务（`uv run langgraph dev`，API 位于 `http://127.0.0.1:2024`）。平台下持久化由 LangGraph 自动管理，代码会检测到 `LANGGRAPH_API_URL` 环境变量并自动跳过自定义 SQLite checkpointer；仅 FastAPI 直跑（`python -m app.main`）时使用本地 SQLite 记忆。

## 许可

仅用于学习与演示。
