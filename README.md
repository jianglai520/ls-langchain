# ls-base

基于 **LangChain / LangGraph** 的大语言模型（LLM）智能体（Agent）学习项目，使用 Jupyter Notebook 循序渐进地演示如何构建、调用和扩展 AI 智能体。

> 项目以系列 Notebook 的形式记录学习过程，覆盖模型接入、消息与多模态、提示词工程、工具（Tool）、记忆（Memory）等核心主题，模型主要使用阿里云百炼（DashScope）的通义千问与 DeepSeek。

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

## 技术栈

- **语言 / 环境**：Python >= 3.14，`uv` 包管理（`pyproject.toml` + `uv.lock`）
- **框架**：LangChain 1.x（`langchain`、`langchain-core`、`langchain-community`、`langchain-openai`、`langchain-deepseek`）、LangGraph（含 `langgraph-checkpoint-sqlite`、`langgraph-checkpoint-postgres`）
- **模型提供方**：阿里云百炼 DashScope（通义千问 `qwen-max` / `qwen3.7-max` / `qwen3.7-plus`）、DeepSeek（`deepseek-chat` / `deepseek-v4-flash` / `deepseek-v4-pro`）
- **工具**：Tavily 搜索（`langchain-tavily`）
- **交互**：Jupyter Notebook（`notebook`）、`ipywidgets`（文件上传组件）

## 目录结构

```
ls-base/
├── notebooks/                  # 学习 Notebook 系列（共 7 个）
│   ├── 1.OpenAI.ipynb
│   ├── 2.langchain入门.ipynb
│   ├── 3.models.ipynb
│   ├── 4.messages.ipynb
│   ├── 5.prompt.ipynb
│   ├── 6.tools.ipynb
│   ├── 7.memory.ipynb
│   └── resources/              # 资源文件（SQLite checkpoint 数据库、图片）
├── src/ls_base/                # 项目包（入口为 hello world 示例）
├── pyproject.toml              # uv 项目配置与依赖声明
├── uv.lock                     # 依赖锁文件
└── .env                        # 环境变量（已被 .gitignore 忽略）
```

## 快速开始

```bash
# 1. 安装依赖（使用 uv，需 Python >= 3.14）
uv sync

# 2. 创建 .env 文件并填入自己的密钥
#    DASHSCOPE_API_KEY=<阿里云百炼 API Key>
#    DASHSCOPE_BASE_URL=<百炼兼容 OpenAI 的 Base URL>
#    TAVILY_API_KEY=<Tavily API Key>

# 3. 启动 Jupyter Notebook
uv run jupyter notebook
```

依次运行 `notebooks/` 目录下的 Notebook 即可跟随学习路线逐步实践。

## 环境变量

| 变量 | 用途 | 说明 |
|------|------|------|
| `DASHSCOPE_API_KEY` | 阿里云百炼 API Key | 访问通义千问系列模型 |
| `DASHSCOPE_BASE_URL` | 百炼 OpenAI 兼容接口地址 | 配合 `model_provider="openai"` 使用 |
| `TAVILY_API_KEY` | Tavily 搜索 API Key | `6.tools.ipynb` 预定义工具 |
| `DEEPSEEK_API_KEY` | DeepSeek API Key | `1.OpenAI.ipynb` 使用 |

> 提示：Notebook 中通过 `os.getenv` 读取环境变量（部分 Notebook 使用 `load_dotenv` 加载 `.env`），请确保在运行前已正确配置。`.env` 已被 `.gitignore` 忽略，请勿提交密钥。

## 注意事项

- 各 Notebook 依赖真实 API 调用，需要有效的模型服务密钥；部分模型（如通义千问多模态）可能有额外费用，请留意用量。
- 模型名称（如 `qwen3.7-max`、`deepseek-v4-pro`）为学习环境中的可用版本，请以你的服务商实际支持的模型列表为准。
- 记忆持久化使用 SQLite（`notebooks/resources/checkpoint.db`），生产环境建议使用数据库支持的 Checkpointer（如 `langgraph-checkpoint-postgres`）。

## 许可

仅用于学习与演示。
