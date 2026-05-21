# LLM 实验结果：Claude Sonnet 4.5 + Prompt B (AP2)

- **模型**：Claude Sonnet 4.5  
- **Prompt**：`prompts/prompt_B_step1.txt` + `prompts/prompt_B_step2.txt`（论文 AP2，两步推理）  
- **输入**：`benchmark_subset/cases/*.c`（实验时使用短文件名 `DRBxxx.c`）  
- **日期**：2026-05-21  

## 归档文件

| 步骤 | 文件 |
|------|------|
| Step 1 依赖分析 | [llm_claude_sonnet_4.5_prompt_B_step1.md](llm_claude_sonnet_4.5_prompt_B_step1.md) |
| Step 2 竞态判定汇总 | [llm_claude_sonnet_4.5_prompt_B_step2_summary.md](llm_claude_sonnet_4.5_prompt_B_step2_summary.md) |

## Step 2 汇总表

| 文件（实验） | 完整文件名 | Ground truth | 预测 | 正确 |
|--------------|------------|--------------|------|------|
| DRB001.c | DRB001-antidep1-orig-yes.c | yes | **yes** | ✓ |
| DRB005.c | DRB005-indirectaccess1-orig-yes.c | yes | yes | ✓ |
| DRB009.c | DRB009-lastprivatemissing-orig-yes.c | yes | yes | ✓ |
| DRB013.c | DRB013-nowait-orig-yes.c | yes | yes | ✓ |
| DRB022.c | DRB022-reductionmissing-var-yes.c | yes | yes | ✓ |
| DRB073.c | DRB073-doall2-orig-yes.c | yes | **no** | ✗ |
| DRB045.c | DRB045-doall1-orig-no.c | no | no | ✓ |
| DRB048.c | DRB048-firstprivate-orig-no.c | no | no | ✓ |
| DRB051.c | DRB051-getthreadnum-orig-no.c | no | no | ✓ |
| DRB059.c | DRB059-lastprivate-orig-no.c | no | no | ✓ |
| DRB065.c | DRB065-pireduction-orig-no.c | no | no | ✓ |
| DRB108.c | DRB108-atomic-orig-no.c | no | no | ✓ |

**检出存在 data race（Step 2）**：DRB001、DRB005、DRB009、DRB013、DRB022（5/12）

## 混淆矩阵与指标

|  | 预测 yes | 预测 no |
|--|----------|---------|
| **实际 yes** | TP = 5 | FN = 1 |
| **实际 no** | FP = 0 | TN = 6 |

| 指标 | Prompt B | Prompt A（对照） |
|------|----------|------------------|
| Accuracy | **0.917** | 0.750 |
| Precision | **1.000** | 0.800 |
| Recall | **0.833** | 0.667 |
| F1 | **0.909** | 0.727 |

结构化结果见 `llm_results.csv`、`metrics_summary.csv`。

## 与 Prompt A 的差异

| 用例 | Prompt A | Prompt B | 说明 |
|------|----------|----------|------|
| DRB001 | no ✗ | **yes** ✓ | Step1 明确写出迭代 i-1 读 `a[i]` 与迭代 i 写 `a[i]` 冲突 |
| DRB051 | yes ✗ | **no** ✓ | Step2 采纳隐式 barrier，纠正误报 |
| DRB073 | no ✗ | no ✗ | 两步均判「每线程不同行」，持续漏报 |
| 其余 9 例 | 与 B 一致 | — | — |

## Step 1 摘要（各用例 Potential Data Race）

| 文件 | Step1 判定 | Step2 最终 |
|------|------------|------------|
| DRB001 | YES | YES |
| DRB005 | YES | YES |
| DRB009 | YES | YES |
| DRB013 | YES | YES |
| DRB022 | YES | YES |
| DRB045 | NO | NO |
| DRB048 | NO | NO |
| DRB051 | MAYBE | NO |
| DRB059 | NO | NO |
| DRB065 | NO | NO |
| DRB073 | NO | NO |
| DRB108 | NO | NO |

DRB051 在 Step1 标为 MAYBE（内存可见性疑虑），Step2 结合「隐式 barrier」判 **NO**，与 ground truth 一致。

## 结论

在本项目 12 例子集上，**AP2 两步 prompt 优于 BP1 单轮 prompt**（F1 0.909 vs 0.727），与论文趋势一致：先依赖分析再判定，有助于修复 DRB001 漏报与 DRB051 误报；**DRB073** 仍为共同难点。
