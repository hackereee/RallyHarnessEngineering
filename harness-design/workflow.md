# Harness Engineering workflow

## session-start.py
### 1.Harness检查:
- 整个Harness目录与文档是否完备? 完备 - 继续， 不完备 - 补全所有的目录与文档
- 检查所有Harness脚本是否完备? 不完备 - 报错返回大模型，不允许继续执行
- 检查开发环境: 构建cli,单元测试cli 等等, 不强控, 返回错误信息给大模型决策能否正常安装
### 2. workflow-state bootstrap
1. 如果 `work/workflow-state.json` 已存在，只校验，不修改。
2. 如果 `work/workflow-state.json` 不存在且没有 active plan，从 `.harness/templates/workflow-state.template.json` 创建首个 L0/L1 state。
3. 如果 `work/workflow-state.json` 不存在但存在 active plan，阻断并返回大模型做语义恢复；脚本不猜测当前 workflow。
### 3. validate-state.py
1. 检查 workflow-state.json 状态
### 4. session audit
1. 写入 `work/sessions/YYYY-MM-DD/session-<id>.md`。
2. 记录 lint、validate、环境、git status 和当前 `workflow-state.nextAction`。
3. 会话文件只做审计证据，不作为 workflow 真相源。
