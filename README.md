# Multi-Agent Workflow — 三智能体协作的软件工程流水线

本项目用 3 个 Agent 协作完成需求、设计、编码的迷你软件工程流程，通过共享状态（黑板）实现 Agent 间通信。

> 📌 本仓库为多模型版本：三个 Agent 分别使用智谱 AI 的三个模型。其后续版本已迁移到火山方舟豆包大模型（单模型、多角色），见 `doubao-multi-agent-workflow` 仓库。

## 1. 项目目标

实现一条完整的 ***\*Mini SE 流程\****：

```
需求 Agent  →  设计 Agent  →  编码 Agent
（需求文档）   （技术设计）    （代码骨架）
```

- 至少 2 个 Agent 协作完成完整流程 ✅（实际 3 个）
- Agent 间消息传递 / 状态共享 ✅（共享状态黑板 + 通信日志）
- 基本流程控制（顺序执行）✅（`Workflow.run` 顺序编排）
- MVP 可运行即可 ✅（默认接入智谱 AI 的 3 个不同免费模型；未设 Key 时自动降级 mock，无需任何配置即可跑通）

## 2. 架构与流程图

```
flowchart LR
    U([用户/需求]) --> S[(共享状态 SharedState)]
    S --> R[需求Agent]
    R --> S
    S --> D[设计Agent]
    D --> S
    S --> C[编码Agent]
    C --> S
    S --> O([产出: 需求/设计/代码])
```

- ***\*共享状态（Blackboard）\****：`state.py` 提供 `publish`/`read`/`history`。
- 上游 Agent 把结果 `publish` 到共享状态；
- 下游 Agent 通过 `read` 获取上游产物，完成 ***\*Agent 间信息互通\****；
- `history` 记录完整通信链路，满足「执行日志与状态追踪」。

## 3. 三个 Agent 接入的三个不同免费模型（并已在代码中默认启用）

本项目 `LLMClient`（`llm.py`）统一抽象了调用层，内置 ***\*多个 OpenAI 兼容的免费 LLM 引擎\****。
为保证「3 个 Agent 用 3 个不同且完全免费的模型」，默认让 3 个 Agent 接入 ***\*智谱 AI 平台上的 3 个不同免费模型\****（glm-4-air / glm-4-flash / glm-3-turbo，均经实测免费可用且稳定）：

| #    | 平台（模型）                     | 免费方式                   | 用于 Agent | 所需 Key（环境变量） |
| ---- | -------------------------------- | -------------------------- | ---------- | -------------------- |
| 1    | ***\*智谱 AI\****（glm-4-air）   | 注册送免费额度，国内可访问 | 需求 Agent | `ZHIPU_API_KEY`      |
| 2    | ***\*智谱 AI\****（glm-4-flash） | 注册送免费额度，国内可访问 | 设计 Agent | `ZHIPU_API_KEY`      |
| 3    | ***\*智谱 AI\****（glm-3-turbo） | 注册送免费额度，国内可访问 | 编码 Agent | `ZHIPU_API_KEY`      |

> 三个 Agent 使用三个***\*不同的免费模型\****（均为智谱 AI：glm-4-air / glm-4-flash / glm-3-turbo），满足「3 个免费 Agent 协作」；把 `ZHIPU_API_KEY` 填入 `.env` 即可启用真实模型。
> 若 Key 未填，三个 Agent 会自动降级为 mock 模板，流程仍能跑通。
> `llm.py` 采用 OpenAI 兼容接口，也可接入其它兼容平台（在 `main.py` 的 `PLATFORMS` 中改配置）。

> 实测说明：原计划用 DeepSeek 官方（deepseek-chat / reasoner），但该账户实测返回 ***\*402 需付费\****，非免费，故弃用，三个 Agent 改用智谱 AI 的三个免费模型。选择理由：智谱国内可访问、注册即送免费额度、OpenAI 兼容易调用，上手成本低，最适合快速验证。

## 4. 使用方法

```
# 1. 安装依赖
pip install -r requirements.txt

# 2. 直接运行（默认接入智谱 AI 的三个免费模型；未设 Key 则自动降级 mock）
python main.py
```

运行时首先进行***\*环境检测\****：检查 `ZHIPU_API_KEY` 是否设置；
已设置则三个 Agent 使用各自的真实模型生成内容，未设置则自动降级为 mock 模板并给出提示，流程仍可跑通（无 Key 也能开箱跑通）。

运行后根据提示***\*输入一段产品需求描述\****（回车使用默认示例需求），
三个 Agent 将依次运行，完整产出直接输出在终端：需求文档 → 技术设计 → 代码骨架。

### 配置智谱 AI（推荐：国内可访问、免费、无需下载软件/大模型）

API Key 存入项目根目录的 `.env` 文件，`main.py` 启动时会***\*自动加载\****，无需每次手动 `set`。
三个 Agent 共用同一个 Key（注册免费、国内可访问）：

| 平台             | 注册地址                 | 用于                                                         | 环境变量        |
| ---------------- | ------------------------ | ------------------------------------------------------------ | --------------- |
| 智谱 AI 开放平台 | https://open.bigmodel.cn | 三个 Agent（glm-4-air / glm-4-flash / glm-3-turbo 三个不同免费模型） | `ZHIPU_API_KEY` |

> 注：DeepSeek 官方因该账户返回 402 需付费，未采用。如需使用其它 OpenAI 兼容平台，可在 `main.py` 的 `PLATFORMS` 中修改配置。

配置步骤：

1. 在智谱 AI 开放平台注册并创建 API Key（免费送额度）；
2. 复制 `.env.example` 为 `.env`（若已存在 `.env` 直接编辑即可）；
3. 用记事本打开 `.env`，把 `ZHIPU_API_KEY=` 后面填上对应的值；
4. 重新运行 `python main.py`，检测通过即使用真实模型。

> 未填 Key 时，三个 Agent 会自动降级为 mock 模板，不影响整体流程跑通。
> 安全提示：`.env` 含密钥，切勿提交到公开仓库（本项目已加入 `.gitignore`）。

### 运行实测示例（真实模型：此前 SiliconFlow 额度充足时跑通，以下为真实生成证据）

> 注：本示例记录了早期使用硅基流动等平台真实模型端到端跑通的产出，用于证明 3 个 Agent 真实生成且互通。当前版本改用智谱三模型后，只需填入 `ZHIPU_API_KEY` 即可获得同等真实产出。

***\*输入需求描述：\****

```
帮我写一个学生成绩管理系统，能录入学生成绩、自动计算每门课的平均分，并按总分从高到低排名。
```

***\*终端过程（节选，来自真实模型生成）：\****

```
>>> 步骤：运行 需求Agent（引擎：DeepSeek 官方 / deepseek-chat）
    [通信] 需求Agent -> 状态 'requirement' (写入 1417 字符)
>>> 步骤：运行 设计Agent（引擎：智谱 AI / glm-4-flash）
    [通信] 设计Agent -> 状态 'design' (写入 3016 字符)
>>> 步骤：运行 编码Agent（引擎：阿里百炼 / qwen-turbo）
    [通信] 编码Agent -> 状态 'code' (写入 6115 字符)
Workflow 完成：3 个 Agent 已通过共享状态完成信息互通。
```

***\*产出（终端输出，均为真实模型内容）：\****

- 需求文档：标准化软件需求规格说明书，含产品概述、功能需求（成绩录入 / 平均分 / 排名）、验收标准
- 技术设计：模块划分（用户 / 成绩 / 排名 / 报表管理）+ 核心接口（如 `POST /grades`、`GET /courses/:courseId/average`、`GET /students/rankings`），***\*由需求推导而来\**** → 互通证据
- 代码骨架：Flask + SQLAlchemy 代码骨架，开头「***\*基于上述技术设计\****」实现上述接口 → 互通证据

> 该次运行完整验证了：3 个 Agent 顺序协作、真实模型生成、共享状态传递、端到端跑通。

## 5. 文件结构

```
multi-agent-workflow
├── llm.py                  # 统一 LLM 客户端（mock + 多个 OpenAI 兼容平台）
├── state.py                # 共享状态 / 消息通信（Blackboard）
├── agents.py               # 3 个 Agent：需求 / 设计 / 编码
├── workflow.py             # 顺序流程编排
├── main.py                 # 入口（含环境检测、平台配置 PLATFORMS）
├── test_workflow.py        # 自动化测试（12 项检查）
├── run_test.bat            # 一键运行测试（Windows）
├── sample_requirements.txt # 示例需求描述
├── requirements.txt
├── .env.example            # 密钥模板（复制为 .env 后填入自己的 Key）
├── docs/
│   ├── Design.md           # 设计文档（架构 / 流程图 / Agent 设计 / 技术选型）
│   └── TEST.md             # 测试文档（方法 + 测试数据 + 判定）
└── README.md
```

> 运行：在项目根目录执行 `pip install -r requirements.txt` 后 `python main.py`。
> `.env` 含密钥，已加入 `.gitignore`，切勿提交到公开仓库。

## 6. 设计要点对照

| 维度       | 实现情况                                                     |
| ---------- | ------------------------------------------------------------ |
| 流程完整性 | Mini SE 流程端到端跑通：需求描述 → 需求 → 设计 → 代码        |
| Agent 协作 | 3 个 Agent 通过共享状态互通，通信链路由 `history` 记录       |
| 可扩展性   | 新增 Agent 只需继承 `BaseAgent` 并加入 `Workflow.agents`；流程可改并行/分支 |
| 文档       | 本 README + 流程图 + 使用说明 + TEST.md 测试文档             |
