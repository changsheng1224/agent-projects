# DSH Case Compiler 测试与评测报告

## 离线端到端验证

2026-09-05 使用 Node.js `v24.16.0`、pnpm `11.19.0` 执行项目内置离线 Demo：

```powershell
./examples/run-f15-demo.ps1
```

实际输出：

```text
=== F15 offline baseline demos ===
Running hermetic import -> minimize -> bundle -> replay tests...

RUN  v4.1.8
Test Files  2 passed (2)
Tests       3 passed (3)
Duration    2.46s

=== F15 demo completed: no network, no credentials, no real writes ===
```

### Synthetic Search Loop

该 Case 从包含三次重复 `search_docs` 调用和冗余规划文本的 JSONL Session 开始，完整执行：

```text
Session import
  -> CaseIR normalization
  -> baseline replay
  -> hierarchical DDMin
  -> Observed 1-minimal verification
  -> Case Bundle write/load
  -> final replay
  -> repaired negative replay
```

验证结果：

- 最终状态为 `observed-one-minimal`；
- 无关 History 被删除，估算输入 Token 减少 `50%+`；
- Bundle 写入后能够重新加载并通过一致性校验；
- 原始行为命中 Failure Oracle；
- 修复后行为不再命中同一 Oracle。

### Realistic Wrong Tool

该 Case 模拟 Agent 错误选择具有外部写语义的 `delete_ticket`。Tool Policy 将其固定为 fixture replay，验证结果为：

- 修复前命中 forbidden-tool Oracle；
- 修复后改为 `search_docs`，Oracle 不再命中；
- 真实删除处理器调用次数为 `0`；
- Fixture Miss、执行错误和权限拒绝均不计为 Oracle 命中。

## 真实模型最小化验证

2026-09-01 在显式凭据和预算下，以 DeepSeek `deepseek-chat` 执行真实模型回归；外部写 Tool 仍由 Fixture 拦截。

| 项目                 |                   结果 |
| -------------------- | ---------------------: |
| Baseline Oracle 命中 |                  `5/5` |
| Final Oracle 命中    |                `10/10` |
| 最终状态             | `observed-one-minimal` |
| 被删除 History 单元  |                  `3/3` |
| 真实外部写入         |                    `0` |

真实模型负责生成 Tool Call，Replay Engine 负责策略检查、Fixture 执行和 Oracle 判定。最终 Case 保留不可缩减的 System Prompt、当前请求、Context 与 Tool Surface，删除全部三个可缩减 History 单元。

## 工程验证入口

项目使用以下仓库门禁检查格式、类型、测试、覆盖率、构建和发布包完整性：

```bash
pnpm format:check
pnpm lint
pnpm typecheck
pnpm test
pnpm run ci
pnpm run release:check
```

普通 CI 不会隐式调用真实模型。Live 验证必须由用户显式提供凭据和预算，且不会执行真实本地或外部写 Tool。
