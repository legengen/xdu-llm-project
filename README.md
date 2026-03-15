# LLM辅助并发程序修复

## 1. 项目简介
本项目围绕论文 **Data Race Detection Using Large Language Models** 展开，基于开源 benchmark **DataRaceBench**，复现论文中“使用大语言模型判断 OpenMP 程序是否存在 data race”的核心实验，并与传统动态检测工具 **ThreadSanitizer (TSan)** 进行对比分析。

本项目的目标是：
- 理解 DataRaceBench 的 benchmark 结构与作用
- 使用 ThreadSanitizer 对选定测试用例进行检测
- 使用大语言模型（LLM）对同一批测试用例进行 yes/no 判断
- 统计并比较不同方法的检测效果

---

## 2. 论文与项目背景

### 2.1 论文名称
**Data Race Detection Using Large Language Models**

### 2.2 研究问题
论文主要研究：
- 大语言模型能否判断 OpenMP 程序是否存在 data race
- 不同 prompt 设计对检测结果有何影响
- LLM 与传统 data race 检测工具相比各有哪些优缺点

### 2.3 开源项目
本项目使用的开源 benchmark 是 **DataRaceBench**。  
它是一个用于评估并发程序数据竞争检测工具效果的 benchmark suite，包含大量带 race 和不带 race 的 OpenMP 微基准程序。

---

## 3. 项目分工

### 成员 1
- 环境配置
- DataRaceBench 搭建
- ThreadSanitizer 测试
- TSan 结果整理

### 成员 2
- Prompt A 设计
- LLM 检测实验
- 结果整理

### 成员 3
- Prompt B 设计
- LLM 检测实验
- 指标统计与结果分析

### 成员 4
- LLM 检测实验
- 指标统计与结果分析

### 成员 5
- PPT以及大报告制作
- 上台成果展示

---

## 4. 测试用例选择
本项目从 DataRaceBench 中选取了少量代表性 benchmark 进行小型实验，包含 **有 data race** 和 **无 data race** 两类程序。

```text
DRB001-antidep1-orig-yes.c
DRB005-indirectaccess1-orig-yes.c
DRB009-lastprivatemissing-orig-yes.c
DRB013-nowait-orig-yes.c
DRB022-reductionmissing-var-yes.c
DRB073-doall2-orig-yes.c
DRB045-doall1-orig-no.c
DRB048-firstprivate-orig-no.c
DRB051-getthreadnum-orig-no.c
DRB059-lastprivate-orig-no.c
DRB065-pireduction-orig-no.c
DRB108-atomic-orig-no.c
```
## 5. 项目结构

本项目仓库结构如下：

```text
datarace-llm-project/
├── README.md
├── benchmark_subset/
│   ├── selected_cases.md
│   └── cases/
├── prompts/
│   ├── prompt_A.txt
│   └── prompt_B.txt
├── results/
│   ├── tsan_results.csv
│   ├── llm_results.csv
│   └── metrics_summary.csv
└── docs/
    ├── topic_ppt.pdf
    ├── final_ppt.pdf
    └── report.pdf
```

各目录和文件说明如下：

- **README.md**
  项目的总体说明文件，包括研究背景、实验目标、运行方式、结果说明等内容。

- **benchmark_subset/**
  存放本项目选取的 DataRaceBench 测试用例。
  - **selected_cases.md**：每个测试用例的详细介绍，包括错误类型和选择原因。
  - **cases/**：从 DataRaceBench 中复制出的源代码文件。

- **prompts/**
  存放 LLM 检测实验中使用的 prompt。
  - **prompt_A.txt**：基础版 prompt。
  - **prompt_B.txt**：改进版或分步骤分析 prompt。

- **results/**
  存放实验结果文件。

- **docs/**
  存放课程作业相关文档。

