## MetaGPT 代码库新手导览

这份指南面向初次接触 MetaGPT 的同学，帮助你快速理解仓库结构、核心运行机制，以及下一步学习路径。

### 1）整体认知（Big Picture）

MetaGPT 是一个多智能体（Multi-Agent）框架：不同角色（如产品经理、架构师、工程师、数据分析师）在同一环境中协作，把一句需求逐步产出为文档、设计与代码。

典型运行流程：

1. CLI 或 Python 入口接收用户想法（idea）。
2. 基于配置创建 `Context`（上下文）。
3. 创建 `Team` 并雇佣多个 `Role`。
4. 各个角色通过 `Environment` 里的消息机制协作。
5. 每个角色选择并执行自己的 `Action`。
6. 循环迭代，直到系统空闲（idle）或达到预算/轮次限制。

### 2）仓库结构总览

- `metagpt/`：框架核心代码。
  - `software_company.py`：CLI 入口与高层流程编排。
  - `team.py`：`Team` 抽象与项目运行主循环。
  - `environment/`：运行环境、消息路由及扩展环境。
  - `roles/`：角色定义和角色运行逻辑。
  - `actions/`：动作抽象及具体执行步骤。
  - `provider/`：LLM 提供方对接。
  - `configs/`、`config2.py`：配置模型、加载与合并逻辑。
  - `rag/`：RAG（检索增强生成）能力。
  - `ext/`：可选扩展模块（AFlow、安卓助手、代码评审等）。
  - `tools/`、`utils/`：通用工具和基础能力。
- `examples/`：各类可运行示例（DI、RAG、Debate、AFlow 等）。
- `tests/`：单元测试与集成测试。
- `config/`：示例配置文件。
- `docs/`：文档、教程、资源。

### 3）必须理解的运行时概念

#### Context 与配置

`Context` 保存全局运行状态（配置、成本管理器、LLM 实例创建逻辑等）。`Config.default()` 会合并环境变量、`config/config2.yaml` 和 `~/.metagpt/config2.yaml`，其中用户 Home 目录配置优先级更高。

#### Team 与 Environment

`Team` 是顶层协调器：负责组建角色、预算控制和回合推进。`Environment` 负责消息分发和角色调度。

#### Role 与 Action

`Role` 可以理解为一个“有身份和记忆的智能体”，通常包含：

- 角色设定（profile/goal/constraints）
- 记忆（memory）
- 消息缓冲区（message buffer）
- 一个或多个 `Action`
- 反应模式（`react` / `by_order` / `plan_and_act`）

`Action` 是最小执行单元，一般负责调用 LLM / 工具并产出结果，推动流程向前。

### 4）推荐阅读顺序

建议按下面顺序看源码：

1. `README.md`：先建立项目定位和使用方式。
2. `metagpt/software_company.py`：看入口如何启动完整流程。
3. `metagpt/team.py`：理解团队生命周期。
4. `metagpt/environment/base_env.py`：看消息路由和调度。
5. `metagpt/roles/role.py`：理解角色内部机制。
6. `metagpt/actions/action.py`：理解动作抽象与扩展接口。
7. `metagpt/config2.py` + `metagpt/configs/*`：掌握配置系统。

然后再进入和你目标最接近的 `examples/`。

### 5）上手建议（实用）

- 先跑通 CLI：执行 `metagpt --init-config`，配置好 API Key，再跑一个简单需求。
- 优先参考示例：`examples/hello_world.py`、`examples/di/`、`examples/rag/` 是不错的起点。
- 扩展功能时先判断放在哪一层：
  - 单个步骤能力增强 -> 新增 `Action`
  - 新角色/新协作策略 -> 新增 `Role`
  - 外部系统/游戏环境接入 -> 扩展 `environment/` 或 `ext/`
- 调试阶段重点关注预算与轮次（`investment`、`n_round`）。

### 6）下一步学习路线

- **自定义 Agent**：先看 `examples/build_customized_agent.py`。
- **多 Agent 协作模式**：看 `examples/build_customized_multi_agents.py` 与 `team.py`。
- **Data Interpreter**：在 `examples/di/` 学习“代码生成 + 工具使用”模式。
- **RAG 管线**：从 `examples/rag/rag_pipeline.py` 和 `metagpt/rag/` 入手。
- **研究型工作流（AFlow/SPO）**：看 `examples/aflow/`、`examples/spo/` 与 `metagpt/ext/aflow/`。

### 7）建议保留的心智模型

可以把 MetaGPT 拆成三层来理解：

- **控制平面（Control Plane）**：`Team` + `Environment` + `Role` 的协作循环
- **执行平面（Execution Plane）**：`Action` 调用 LLM/工具完成任务
- **配置平面（Configuration Plane）**：`Config` / `Context` 决定运行参数与依赖

遇到问题时，先判断属于哪一层，再深入对应模块，定位效率会高很多。
