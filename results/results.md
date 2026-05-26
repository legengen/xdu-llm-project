## Results 文件说明

本项目使用 DataRaceBench 的 `check-data-races.sh` 脚本运行 ThreadSanitizer，并在 `results/` 目录下生成结果文件。  
其中 `results/tsan-clang.csv` 记录了 ThreadSanitizer 在选定 benchmark 上的检测结果。

### `tsan-clang.csv` 字段说明

- **tool**  
  使用的检测工具名称。  
  例如本项目中为 `tsan-clang`，表示使用 Clang 版本的 ThreadSanitizer。

- **id**  
  benchmark 的编号。  
  例如 `001` 对应 `DRB001-antidep1-orig-yes.c`。这个编号来自 DataRaceBench 文件命名规则中的 `DRBxxx` 编号。

- **filename**  
  测试用例源文件名，即实际执行的 microbenchmark 文件。

- **haverace**  
  benchmark 的真值标签（ground truth）。  
  - `true`：该测试用例已知存在 data race  
  - `false`：该测试用例已知不存在 data race

- **threads**  
  运行该测试时使用的线程数。  
  例如本项目中大多为 `8`。

- **dataset**  
  数据集大小或参数规模。  
  某些 benchmark 有可变参数时，这里会显示具体数值；如果该程序没有可变数据规模，则显示 `N/A`。

- **races**  
  **TSan 的检测输出**：该次运行报告的数据竞争条数（`0` = 未报 race，`>0` = 检出）。  
  与 **`haverace`（标准答案）** 对比才能算 TP/FP/FN/TN；二者不可混用。

- **elapsed-time(seconds)**  
  本次实验运行耗时，单位为秒。

- **used-mem(KBs)**  
  本次实验运行过程中使用的内存大小，单位为 KB。

- **compile-return**  
  编译阶段的返回码。  
  - `0`：编译成功  
  - 非 `0`：编译失败或发生异常

- **runtime-return**  
  运行阶段的返回码。  
  - `0`：程序正常运行结束  
  - 非 `0`：程序运行异常结束  
  例如 `11` 通常表示程序因信号中断退出（如段错误或其他运行时异常）。

### 字段含义总结

可以把这些字段分成四类理解：

1. **程序基本信息**  
   - `id`
   - `filename`
   - `haverace`

2. **运行配置**  
   - `threads`
   - `dataset`

3. **检测结果**  
   - `races`

4. **性能与执行状态**  
   - `elapsed-time(seconds)`
   - `used-mem(KBs)`
   - `compile-return`
   - `runtime-return`

### 说明

在本项目中，`haverace` 是 DataRaceBench 提供的标准答案，`races` 是 ThreadSanitizer 的检测输出。

**TSan 结果来源**：在官方仓库 `dataracebench/` 中配置 `list.def`（12 用例）与 `tool.def`（`tsan-clang`），运行 `./check-data-races.sh --customize c` 后，将 `dataracebench/results/tsan-clang.csv` 复制到本目录（可用 `dataracebench/scripts/copy_tsan_results_to_xdu.sh`）。Docker 方式见 `dataracebench/docker/README.md` 与 `dataracebench/scripts/run_subset_tsan_docker.sh`。

因此我们可以将 `races` 与 `haverace`、以及 `llm_results.csv` 比较，用于统计是否检测成功、漏报与误报。汇总指标见 `metrics_summary.csv`。

---

## LLM 实验结果

### 文件

| 文件 | 说明 |
|------|------|
| `llm_results.csv` | 每条 benchmark 的 ground truth、模型预测、是否正确 |
| `metrics_summary.csv` | 按 model + prompt 汇总的 TP/FP/TN/FN 与 Accuracy/Precision/Recall/F1 |
| `llm_<model>_<prompt>.md` | 该次实验的分用例说明与原始输出归档 |

### `llm_results.csv` 字段说明

- **model**：使用的 LLM（如 `claude-sonnet-4.5`）
- **prompt**：`prompt_A` / `prompt_B` 等
- **filename**：测试用例文件名
- **ground_truth**：由文件名 `-yes` / `-no` 得到（`yes` / `no`）
- **prediction**：模型最终判定（以 Summary 表或明确最终结论为准）
- **correct**：`prediction` 是否与 `ground_truth` 一致
- **notes**：漏报/误报原因、自我修正等备注

### 已完成的实验

| 模型 | Prompt | 报告 | F1 |
|------|--------|------|-----|
| Claude Sonnet 4.5 | prompt_A (BP1) | [llm_claude_sonnet_4.5_prompt_A.md](llm_claude_sonnet_4.5_prompt_A.md) | 0.727 |
| Claude Sonnet 4.5 | prompt_B (AP2) | [llm_claude_sonnet_4.5_prompt_B.md](llm_claude_sonnet_4.5_prompt_B.md) | 0.909 |
| TSan (tsan-clang) | 动态检测（官方 DRB 脚本） | `tsan-clang.csv` | 0.714 |

### LLM 与 TSan 对比

均在 **同一 Ground truth**（文件名 `-yes`/`-no`）下计算，见 `metrics_summary.csv`：

| 方法 | F1 | Recall | 说明 |
|------|-----|--------|------|
| TSan | 0.714 | 0.833 | 官方 `check-data-races.sh --customize c`，12 例子集 |
| LLM Prompt B | 0.909 | 0.833 | Claude Sonnet 4.5 |
| LLM Prompt A | 0.727 | 0.667 | Claude Sonnet 4.5 |

TSan 漏报 DRB001（`races=0`）；误报 DRB059/065/108。LLM Prompt B 漏报 DRB073。

### TSan 误报说明（FP）

在本子集中，**DRB059、DRB065、DRB108** 的 ground truth 为 **no**（无用户级 data race），但 TSan 的 `races>0`，属于**误报**。详细日志见官方仓库 `dataracebench/results/log/*.tsan-clang.log`。

**共性原因**：TSan 基于 pthread 层面的 happens-before 分析，**不能完整理解 OpenMP 语义**（`atomic`、`reduction`、`lastprivate`、并行区结束时的隐式 barrier）。编译器与 `libomp` 生成的归约、收尾代码在 TSan 看来像「未同步的并发读写」，因而报警；按 OpenMP 标准这些访问仍有同步保证。

| 用例 | OpenMP 保护 | TSan 看到的现象 |
|------|-------------|-----------------|
| DRB108 | `#pragma omp atomic` | 工作线程 atomic 写 `a` 时，主线程已在 `printf` 中读 `a` |
| DRB059 | `lastprivate(x)` | 并行循环未完全结束时，主线程在 `printf` 读 `x` |
| DRB065 | `reduction(+:pi)` | 归约函数 / `__kmp_hyper_barrier_gather` 内线程间读写临时缓冲区 |

**结论**：上述误报**不代表 benchmark 写错**，而是 TSan 与 OpenMP 组合时的已知局限；与 LLM 对比时，应把 FP 单独说明，不宜把 TSan 输出当作标准答案。DataRaceBench 论文对各工具的 FP/FN 亦有类似讨论。
