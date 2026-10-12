# 路 C 机器初审提供者接入范例:open-code-review

本文件示范如何把一个代码审查 CLI 接入 aicodeline 的路 C(机器初审)。任何满足接口约定的工具都可以替换——接口约定见各模式手册的路 C 节:**输入=分支 diff;输出=分级缺陷清单(critical/high/medium/low,行级定位);运行要求=独立进程、独立凭证、会话标识每次新生成不得复用**。

## 工具

open-code-review(`ocr` 命令):调用独立 LLM 端点审查分支 diff,输出 JSON 分级问题清单。LLM 配置以本机 `~/.opencodereview/config.json` 为准(aicodeline 不管理其配置);输出语言经其全局配置设为中文(`language=中文`)。

## 命令形态

**派发引擎(普通 shell,在工作树目录内执行)**:

```bash
cd <工作树> && OCR_LLM_EXTRA_HEADERS="x-opencode-session=<新生成uuid>" \
  ocr review --from <主分支> --to <工作树分支> --audience agent \
  --format json --output <QA目录>/OCR-<票号>-R<N>.json
```

- 可附 `--background "<业务背景>"`(仅中立事实、取自 ticket 原文);
- 会话标识(uuid)**每次新生成、不得复用**(例:`node -e "console.log(crypto.randomUUID())"`);
- 连通性自检:`OCR_LLM_EXTRA_HEADERS="x-opencode-session=<新uuid>" ocr llm test`。

**工作流引擎(world.run,固定在主工作区执行、无 shell 语法)**:

```ts
// 先取会话标识(每次新生成):
world.run("node", ["-e", "console.log(crypto.randomUUID())"]);
// 再跑审查(env 包装注入环境变量;路径全部用绝对路径):
world.run("env", [
  "OCR_LLM_EXTRA_HEADERS=x-opencode-session=<新uuid>", "ocr", "review",
  "--from", "<主分支>", "--to", "<工作树分支>", "--audience", "agent",
  "--format", "json", "--output", "<绝对输出路径>.json",
]);
```

- world.run 的 cwd 固定为主工作区、无法 `cd` 进工作树——本例不受影响(diff 已由 `--from/--to` 分支界定,输出用绝对路径);若你的工具必须在目标目录内运行,优先找它自带的目录参数,实在没有则在工作树内放一个一次性包装脚本再调用;
- 启动前建议用 EvalWorkflowSnippet 实测一遍真实命令。

## 登记进 defaults.md

环境适配或口述更新时,把下面两行写进 defaults.md 的"机器初审提供者(路C)"段(以实际命令为准):

```markdown
- 审查命令模板: cd <工作树> && OCR_LLM_EXTRA_HEADERS="x-opencode-session=<新uuid>" ocr review --from <基线分支> --to <目标分支> --audience agent --format json --output <输出路径>.json
- 连通性自检: OCR_LLM_EXTRA_HEADERS="x-opencode-session=<新uuid>" ocr llm test
```

## 结果消费(对任何提供者通用)

- critical/high=候选阻塞逐条核实;medium=候选建议;low 默认忽略(噪音多);
- 行号可作两路报告行号存疑时的参照基准;
- 摘录(命令、会话标识、提供者、分级清单)转录进 QA 路C 区块,完整输出留档;
- 失败/超时按降级处置留痕,不阻塞验收——路 A+路 B+管理员核对已满足双审硬规则;
- 同一轮连续 2 次失败(工作流批量场景按票计) → 停下重跑连通性自检(多为额度/网络问题),修复或经用户确认免跑。

欢迎为其它工具(openreview、自建 lint 管线等)提交同款接入范例 PR。
