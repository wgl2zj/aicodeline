# 平台要求(platform.md)

## 子智能体派发引擎(modes/aicode-mainother*.md,不含 -workflow)

需要 agent 环境具备:

- **SKILL.md 技能格式**(本包的载体);
- **子智能体机制**:可指定类型派发子任务、子智能体有独立上下文与文件/命令工具;
- **非阻塞派发**:子任务可后台运行,主会话可继续发消息唤醒/答复(如 run_in_background + SendMessage 类机制);
- **AskUserQuestion 类结构化问答**。

ZCode 原生满足以上全部。Claude Code 的 Task/Skill 结构接近但未实测,欢迎反馈。

## 工作流引擎(modes/*-workflow.md)

**仅支持 ZCode**,依赖其动态工作流(dynamic-workflows)能力:

- `CreateWorkflow`:脚本化编排,阶段图(phase)与确认框;
- **per-agent 模型指定**(`agent(name, { model })` + `model()` 名册,启动锁定;模型 id 可附 `$<level>` 指定思考强度,默认各模型最高档)——三路审查的异构性由此成立,这是工作流版的硬前提;
- **escalation 与 ResolveWorkflowQuestion**:子智能体运行中直问管理员——替代派发引擎的 QA 停机轮转;
- `world.run` 确定性命令闸门(exit code 可分支);
- **AmendWorkflow / ResumeWorkflowRun**:失败后的缓存回放与续跑;
- `ListModels`(环境适配用)、`EvalWorkflowSnippet`(路 C 提供者实测用)。

其它平台没有上述等价物(尤其是 per-agent 模型与 escalation),不做跨平台适配层;如你想移植,编排原语清单即上一段,欢迎以 PR 形式提交你的适配经验。

## 版本要求

ZCode 的 per-agent 模型与模型名册于 2026-10 前后加入,早于该版本的 ZCode 无法保证工作流引擎三路异构成立(会退化为同模型独立上下文)——自查方法:打开你本机的 dynamic-workflows 技能文档(通常在 ZCode 安装目录的 bundled-skills 内),搜 `agent(` 看是否支持第二参数指定模型;不支持时请使用派发引擎的两种方式,跳过两本 -workflow 手册。
