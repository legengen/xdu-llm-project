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
  工具在该次运行中报告的数据竞争数量。  
  对于 ThreadSanitizer，这一列表示检测到的 race 数目。

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
因此我们可以将两者进行比较，用于统计：
- 是否检测成功
- 是否存在漏报
- 是否存在误报

后续在 `metrics_summary.csv` 中，我们基于这些结果进一步计算 Accuracy、Precision、Recall 和 F1-score。
