# Learning Notes

Harness Engineering 学习过程中沉淀的设计思考与结论，每份笔记针对一个具体问题，给出判断框架而非流水账。

## Index

| 笔记 | 核心问题 |
|---|---|
| [scripts-vs-llm.md](./scripts-vs-llm.md) | 模型足够强时是否可摒弃脚本？脚本与 LLM 如何分工？ |
| [tasks-workflow-gates.md](./tasks-workflow-gates.md) | tasks.json 如何抽取、落盘、更新？testing/review 与 workflow gate 如何分工？ |

## Current Contract Notes

- Backlog intake 是 intake-side pending queue，未消费项落在 `work/backlog/backlogs.json`。
- 稳定契约位于 `.harness/schemas/backlogs.schema.json`，初始化样例位于 `.harness/templates/backlogs.template.json`；消费事件契约位于 `.harness/schemas/backlog-consumption-event.schema.json`。
- 被 workflow 或 plan 接管后的 item 经 `.harness/scripts/backlog-consume.py` 从 pending queue 删除，完整原 item 写入 `work/backlog/consumed.jsonl`。
- `queue` 记录普通后续工作；`preempt` 请求 LLM 评估是否插队。两者都不会自动修改 active workflow。
- `handoff.md` 是 L2/L3 active plan 的恢复摘要，结构规则见 `.harness/rules/handoff-rules.md`；真实 workflow/task 状态仍以 `workflow-state.json` 与 `tasks.json` 为准。
- session audit 文件由 `session-start.py` 写入，规则见 `.harness/rules/session-start.md`；它记录启动证据，不作为 workflow 或 task 真相源。
