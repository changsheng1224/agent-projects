# FixLoop 评测摘要

## 评测目标

验证受控 Agent 编排是否能够稳定完成代码修复，并与 Single-Agent 基线比较修复成功率、补丁精度、耗时和 Token 成本。

## 实验设计

- Case：10 个，覆盖 TypeError、ImportError、AttributeError、逻辑、配置与复合错误；
- 变体：FixLoop 完整编排（full）与 Single-Agent 基线（single）；
- 重复：每个变体、每个 Case 运行 3 次，共 60 runs；
- 判定：补丁必须通过独立 Verifier，模型自述完成不计为成功；
- 记录：保存修复状态、补丁规模、耗时、Token 与回归结果。

## 项目级结果

| 指标 | FixLoop full | Single-Agent |
| --- | ---: | ---: |
| 修复通过 | **30/30（100%）** | 29/30（96.7%） |
| Patch 精度 | **1.22** | 0.94 |
| 平均耗时 | 31.8 s | **19.7 s** |
| 平均 Token | 5,182 | **2,581** |
| 引入回归率 | **0%** | 0% |

结果表明，完整编排承担了更高的调用成本，但在重复评测中保持 30/30 通过，并以更高的 Patch 精度减少无关修改。FixLoop 因此不默认用更多 Agent 换取形式上的复杂度，而是通过独立检查、验证与失败反馈提升长链路修复的稳定性。

## 快速复现

无需模型 API 的单 Case 链路检查：

```bash
python -m src.cli eval --case case_001 --fake --markdown --verbose
```

运行完整消融评测需要配置模型凭据与预算：

```bash
python -m src.cli ablation --all \
  --variant full --variant single \
  --repetitions 3 \
  --output eval_results/ablation \
  --markdown --verbose
```

评测会输出 JSON 和 Markdown 报告，便于保留同配置复跑结果并比较不同 Runtime、Prompt 或编排版本。
