# Selected Test Cases

本文档记录了本项目从 DataRaceBench 中选取的测试用例，以及每个测试用例所代表的并发错误类型或正确同步模式。  
我们的小型实验目标是：在较小规模下覆盖几类典型的 OpenMP 数据竞争场景，并同时包含 **有 race** 与 **无 race** 两类程序。

## 选取原则

本项目选择测试用例时主要考虑以下几点：

1. **覆盖不同类型的并发错误模式**
   - 循环相关依赖
   - 间接访问引起的共享冲突
   - OpenMP 子句缺失
   - 同步缺失
   - reduction 缺失
   - 正确同步（atomic / reduction / firstprivate / lastprivate）

2. **程序规模较小**
   - 适合 ThreadSanitizer 运行
   - 适合 LLM 阅读和判断

3. **同时包含 yes / no 两类样例**
   - yes：已知存在 data race
   - no：已知不存在 data race

---

## 一、存在 Data Race 的测试用例

### 1. DRB001-antidep1-orig-yes.c
- **标签**：YES
- **错误类型**：循环携带反依赖（loop-carried anti-dependence）
- **说明**：这是一个非常典型的并行循环竞态示例。文件名中的 `antidep1` 表明该程序涉及循环迭代之间的反依赖，DataRaceBench 文件注释中也明确把它描述为一个带有 loop-carried anti-dependence 的例子。
- **适合测试的原因**：结构简单，特别适合作为 data race 入门样例。

### 2. DRB005-indirectaccess1-orig-yes.c
- **标签**：YES
- **错误类型**：间接数组访问导致的共享冲突（indirect access race）
- **说明**：这类程序的问题不再是简单的 `a[i]` 与 `a[i+1]` 这种规则访问，而是通过间接索引访问共享数组，多个线程可能落到同一位置。
- **适合测试的原因**：能测试 LLM 是否能识别“索引映射后可能重叠”的情况。

### 3. DRB009-lastprivatemissing-orig-yes.c
- **标签**：YES
- **错误类型**：OpenMP 子句缺失（missing `lastprivate`）
- **说明**：文件名中的 `lastprivatemissing` 说明这个程序的问题来自 `lastprivate` 子句缺失。也就是说，本应在并行循环结束后保留最后一次迭代值的变量，没有被正确声明。
- **适合测试的原因**：很适合测试模型对 OpenMP clause 的理解能力。

### 4. DRB013-nowait-orig-yes.c
- **标签**：YES
- **错误类型**：同步缺失 / barrier 缺失（missing synchronization caused by `nowait`）
- **说明**：`nowait` 会移除某些 OpenMP 结构末尾的隐式同步。如果后续代码假设前一个并行区域已经全部完成，就可能产生竞态。
- **适合测试的原因**：能测试 LLM 是否理解隐式 barrier 的作用。

### 5. DRB022-reductionmissing-var-yes.c
- **标签**：YES
- **错误类型**：reduction 子句缺失
- **说明**：文件名中的 `reductionmissing` 表明程序对某个共享累加变量进行了并行更新，但没有使用 `reduction` 进行正确归约。
- **适合测试的原因**：这是 OpenMP 中最常见的竞态类型之一，也是课堂展示时很容易解释的一类错误。

### 6. DRB073-doall2-orig-yes.c
- **标签**：YES
- **错误类型**：并行 do-all 循环中的数据竞争
- **说明**：从仓库文件名可见它是一个 `doall2` 的 yes 样例，属于并行循环看似可以独立执行，但实际仍存在共享访问冲突的情形。
- **适合测试的原因**：可以补充一种“表面像 do-all，实际并不安全”的错误模式。

---

## 二、不存在 Data Race 的测试用例

### 7. DRB045-doall1-orig-no.c
- **标签**：NO
- **错误类型**：无，属于正确的 do-all 并行循环
- **说明**：这是一个典型的“每次迭代互不干扰”的并行循环程序。
- **适合测试的原因**：可作为 no 类基线，检查模型和 TSan 是否会误报。

### 8. DRB048-firstprivate-orig-no.c
- **标签**：NO
- **错误类型**：无，正确使用 `firstprivate`
- **说明**：DataRaceBench 仓库中该文件名直接表明它是 `firstprivate` 的正确示例。`firstprivate` 会为每个线程创建变量私有副本，并用进入并行区前的值初始化。
- **适合测试的原因**：适合测试 LLM 会不会把正确的私有化误判成共享变量竞态。

### 9. DRB051-getthreadnum-orig-no.c
- **标签**：NO
- **错误类型**：无，线程编号相关的安全访问
- **说明**：这类程序通常通过线程号控制各线程访问不同的数据区域，因此不会形成冲突写入。
- **适合测试的原因**：程序逻辑通常比较直观，适合做无竞态样例。

### 10. DRB059-lastprivate-orig-no.c
- **标签**：NO
- **错误类型**：无，正确使用 `lastprivate`
- **说明**：这个程序和 `DRB009-lastprivatemissing-orig-yes.c` 正好形成对照：一个是 `lastprivate` 缺失导致问题，一个是 `lastprivate` 正确使用。
- **适合测试的原因**：很适合在报告中做“成对对比分析”。

### 11. DRB065-pireduction-orig-no.c
- **标签**：NO
- **错误类型**：无，正确使用 reduction
- **说明**：从文件名可以看出它是一个 `pi reduction` 示例，通常用于演示在并行求和中正确使用 `reduction` 的方式。
- **适合测试的原因**：可以和 `DRB022-reductionmissing-var-yes.c` 形成对照，突出 reduction 的作用。

### 12. DRB108-atomic-orig-no.c
- **标签**：NO
- **错误类型**：无，正确使用 `atomic`
- **说明**：文件名说明这是一个 `atomic` 的正确示例。`atomic` 能保证对共享变量的更新具有原子性，从而避免竞态。
- **适合测试的原因**：适合测试 LLM 是否理解 `#pragma omp atomic` 是一种同步机制。

---

## 三、为什么这组样例适合小型实验

本项目没有选择过多 benchmark，而是优先选择：

- **错误模式有代表性**
- **程序规模较小**
- **便于人工分析与展示**
- **适合 TSan 和 LLM 同时测试**

具体来说，这 12 个样例覆盖了以下几类核心场景：

| 类别 | 代表用例 |
|---|---|
| 循环依赖 / 反依赖 | DRB001 |
| 间接访问冲突 | DRB005 |
| OpenMP clause 缺失 | DRB009 |
| barrier / 同步缺失 | DRB013 |
| reduction 缺失 | DRB022 |
| 看似可并行但存在冲突的 do-all | DRB073 |
| 正确的并行 do-all | DRB045 |
| 正确的 firstprivate | DRB048 |
| 正确的线程分工访问 | DRB051 |
| 正确的 lastprivate | DRB059 |
| 正确的 reduction | DRB065 |
| 正确的 atomic | DRB108 |

---

## 四、实验中的作用

在本项目中，这些测试用例会被用于两类实验：

1. **ThreadSanitizer 基线测试**
   - 运行 benchmark
   - 记录是否检测到 race

2. **LLM prompt 检测实验**
   - 将 benchmark 代码输入大语言模型
   - 让模型判断是否存在 data race
   - 比较不同 prompt 的效果

最终，我们会基于这些样例统计：
- Accuracy
- Precision
- Recall
- F1-score

---

## 五、备注

本文档中的“错误类型”主要根据：
- DataRaceBench 官方仓库中的 benchmark 文件名命名规则
- 具体 benchmark 文件的注释说明
- OpenMP 编程模型中的常见竞态模式

后续如果实验过程中发现某些 benchmark 对 TSan 或 LLM 不够稳定，可以在不改变总体 yes/no 平衡的前提下替换为同类型样例。
