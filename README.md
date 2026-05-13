# RAG 示例

基于本地文档的简单 RAG 流水线：对 `doc.md` 分段、向量化写入 **ChromaDB**，检索后用 **CrossEncoder** 重排，再通过 **Google Gemini** 生成回答。主要逻辑在 [`main.ipynb`](main.ipynb)。

## 环境要求

- **Python**：`>= 3.14`（见 [`pyproject.toml`](pyproject.toml)）
- **[uv](https://docs.astral.sh/uv/)**：用于安装依赖与运行命令

## 快速开始

```bash
cd rag
uv sync
```

### 1. API 密钥（生成步骤必需）

在项目根目录创建 `.env`（该文件已加入 `.gitignore`，勿提交到仓库），例如：

```env
GEMINI_API_KEY=你的密钥
```

密钥来自 [Google AI Studio](https://aistudio.google.com/apikey)。`main.ipynb` 使用 `google-genai` 的 `genai.Client()`，会读取环境变量中的 Gemini API 配置。

### 2. 启动 Jupyter

```bash
uv run --with jupyter jupyter lab
```

在浏览器中打开 `main.ipynb`，按从上到下顺序执行单元格即可。

> 首次运行会从 Hugging Face 下载嵌入模型与 CrossEncoder 权重，需要能访问外网或使用镜像（见下文）。

## 访问 Hugging Face 失败时

若出现连接超时（例如 `WinError 10060`），可先设置镜像再启动 Jupyter：

**Git Bash / Linux / macOS**

```bash
export HF_ENDPOINT="https://hf-mirror.com"
uv run --with jupyter jupyter lab
```

**PowerShell**

```powershell
$env:HF_ENDPOINT = "https://hf-mirror.com"
uv run --with jupyter jupyter lab
```

可选：在 [Hugging Face](https://huggingface.co/settings/tokens) 创建只读 Token 后设置 `HF_TOKEN`，有利于提高下载速率与限额。

## 项目结构说明

| 路径 | 说明 |
|------|------|
| `main.ipynb` | 分段、嵌入、入库、检索、重排、调用 Gemini |
| `doc.md` | 被索引的示例文档 |
| `chroma_db/` | Chroma 持久化数据（本地生成，勿提交敏感内容时可按需忽略） |

嵌入模型：`shibing624/text2vec-base-chinese`。重排模型：`cross-encoder/mmarco-mMiniLMv2-L12-H384-v1`。生成模型：notebook 中为 `gemini-2.5-flash`，可按需在代码中替换。

## 重复执行 notebook 的说明

向 Chroma 重复 `add` 相同 `id` 可能报错。若需重新建库，可删除本地目录 `chroma_db/` 后重新运行写入向量相关的单元格。
