## MetaGPT Codebase Overview for Newcomers

This guide explains how the repository is organized and where to start when you're new.

### 1) Big picture

MetaGPT is a multi-agent framework where specialized roles (e.g., Product Manager, Architect, Engineer, Data Analyst) collaborate in a shared environment to turn a user requirement into artifacts like docs and code.

At runtime:

1. A CLI/library entrypoint receives an idea.
2. A `Context` is built from configuration.
3. A `Team` is created and hires multiple `Role` objects.
4. Roles communicate through an `Environment` via messages.
5. Each role chooses and runs `Action` objects.
6. The workflow iterates until the system becomes idle or budget/round constraints are reached.

### 2) Repository structure

- `metagpt/`: core framework code.
  - `software_company.py`: CLI entrypoint and high-level orchestration.
  - `team.py`: the `Team` abstraction and project loop.
  - `environment/`: messaging/runtime environment and extensions.
  - `roles/`: role definitions and role runtime logic.
  - `actions/`: action abstractions and concrete task steps.
  - `provider/`: LLM provider integrations.
  - `configs/`, `config2.py`: config schemas + loading/merging logic.
  - `rag/`: retrieval-augmented generation components.
  - `ext/`: optional feature packs (AFlow, Android assistant, code review, etc.).
  - `tools/`, `utils/`: shared utilities/tooling.
- `examples/`: runnable examples across use cases (DI, RAG, debate, aflow, etc.).
- `tests/`: unit/integration tests.
- `config/`: sample config files.
- `docs/`: docs, tutorials, and references.

### 3) Important runtime concepts

#### Context and config

`Context` stores global runtime state (selected config, cost manager, and LLM instance creation). `Config.default()` merges environment variables with `config/config2.yaml` and `~/.metagpt/config2.yaml`, where home config has higher precedence.

#### Team and environment

`Team` is the top-level coordinator. It creates an `Environment`, hires roles, enforces budget, and runs rounds. The environment handles message routing and role scheduling.

#### Roles and actions

A `Role` is an agent with:

- profile/goal/constraints,
- memory,
- a message buffer,
- one or more `Action`s,
- a reaction mode (`react`, `by_order`, or `plan_and_act`).

An `Action` is the executable unit that usually calls an LLM (or tools) and returns outputs to move the workflow forward.

### 4) Where to start reading code

Recommended reading order:

1. `README.md` (project purpose and quickstart).
2. `metagpt/software_company.py` (CLI + project startup path).
3. `metagpt/team.py` (team lifecycle).
4. `metagpt/environment/base_env.py` (message routing and role execution loop).
5. `metagpt/roles/role.py` (role internals).
6. `metagpt/actions/action.py` (action contract).
7. `metagpt/config2.py` and `metagpt/configs/*` (configuration model).

Then jump to a use case in `examples/` that matches your goal.

### 5) Practical onboarding tips

- Get CLI working first: run `metagpt --init-config`, set an API key, then run one simple prompt.
- Use examples instead of writing from scratch: `examples/hello_world.py`, `examples/di/`, and `examples/rag/` are good starting points.
- When extending behavior, decide where it belongs:
  - new behavior for one agent step -> new `Action`,
  - a new persona/workflow policy -> new `Role`,
  - external system/game integration -> environment extension under `environment/` or `ext/`.
- Watch costs and rounds while debugging (`investment`, `n_round`).

### 6) What to learn next

- **Custom agents:** build a custom role + action pair in `examples/build_customized_agent.py`.
- **Multi-agent collaboration patterns:** inspect `examples/build_customized_multi_agents.py` and `team.py`.
- **Data Interpreter workflows:** explore `examples/di/` for code generation + tool usage patterns.
- **RAG stack:** start from `examples/rag/rag_pipeline.py` and `metagpt/rag/`.
- **AFlow/SPO research workflows:** inspect `examples/aflow/`, `examples/spo/`, and `metagpt/ext/aflow/`.

### 7) Mental model to keep

Think of MetaGPT as:

- **Control plane:** `Team` + `Environment` + `Role` loop,
- **Execution plane:** `Action`s using LLM/tools,
- **Configuration plane:** `Config`/`Context`.

When debugging, first locate which plane the issue belongs to, then drill into that layer.
