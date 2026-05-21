# Prompts 说明

本目录存放 LLM 数据竞争检测实验用的 prompt 模板，设计依据论文 **Data Race Detection Using Large Language Models** 的结论：优先做 **S1（是否存在 data race 的二分类）**，避免一次性要求定位变量/行号/JSON（BP2 / S2–S3）导致漏报上升。

## 文件与论文对应关系

| 项目文件 | 论文 prompt | 任务层次 | 设计要点 |
|----------|-------------|----------|----------|
| `prompt_A.txt` | **BP1** | S1 | 仅 yes/no，首行作答；召回较高，适合基线 |
| `prompt_B_step1.txt` + `prompt_B_step2.txt` | **AP2** | S1（两步推理） | 先依赖分析，再判断 race；通常优于单次复杂 prompt |

未单独提供 **BP2**（同时输出变量对、行号、JSON）和 **AP1**（BP1 + 长定义）：论文显示 BP2 会显著增加 FN，AP1 对 GPT-3.5 未必优于 BP1。若需扩展实验，可在本目录新增 `prompt_BP2.txt` / `prompt_AP1.txt`。

## 占位符

| 占位符 | 含义 |
|--------|------|
| `{{SOURCE_CODE}}` | 替换为 `benchmark_subset/cases/*.c` 的完整源码（建议去掉文件头 LLNL 版权块以节省 token，保留 `main` 与 OpenMP 相关代码即可） |
| `{{STEP1_OUTPUT}}` | 仅用于 `prompt_B_step2.txt`，替换为 Step 1 模型的完整回复 |

## 使用流程

### Prompt A（单轮，对应 BP1）

1. 读取 `prompt_A.txt`，将 `{{SOURCE_CODE}}` 替换为目标 `.c` 文件内容。
2. 调用 LLM 一次。
3. 解析回复**首行**为 `yes` 或 `no`（见下方解析规则）。

### Prompt B（两轮，对应 AP2）

1. 用 `prompt_B_step1.txt` + 源码 → 得到 **Step 1 分析文本**。
2. 用 `prompt_B_step2.txt`，填入 `{{STEP1_OUTPUT}}` 与同一份 `{{SOURCE_CODE}}` → 得到最终 yes/no。
3. 仅将 Step 2 的首行作为预测标签，与 DataRaceBench 的 `yes`/`no` 文件名 ground truth 对比。

## 输出解析（评测用）

与论文一致，S1 实验只关心二分类标签。建议用正则取首行（不区分大小写）：

```python
import re

def parse_yes_no(text: str) -> str | None:
    first = text.strip().splitlines()[0].strip().lower()
    m = re.match(r"^(yes|no)\b", first)
    return m.group(1) if m else None
```

若首行无法解析，记为 `parse_error`，单独统计，勿默认当成 `no`（否则会人为抬高漏报相关指标）。

## 记录实验结果

建议在 `results/llm_results.csv` 中至少包含：

- `filename`, `prompt`（A 或 B）, `model`, `prediction`（yes/no）, `ground_truth`（由文件名 `-yes`/`-no` 得到）, `raw_response`（可选，便于复查）

`prompt_B` 可另存 `step1_response` 列，便于报告展示“依赖分析 → 判定”过程。

## 与 ThreadSanitizer 的对比

- **LLM（本目录）**：静态阅读源码的 S1 判断；Prompt A/B 对比 prompt 设计影响。
- **TSan（`results/tsan-clang.csv`）**：动态运行检测；`haverace` 为 benchmark 标签，`races` 为工具输出。

两者 ground truth 均来自 DataRaceBench 文件名与标注，指标统一用 Accuracy / Precision / Recall / F1。
