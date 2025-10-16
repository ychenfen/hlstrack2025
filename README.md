# 嵌赛FPGA赛道-AMD 命题式基础赛道 - 初赛

# Vitis Library HLS算法优化竞赛评分规则手册

## 1. 竞赛概述

本次竞赛要求学生对 Vitis Libraries 中的三个 L1 算法进行 HLS 优化，目标是在保持功能正确性的前提下，最小化算法的执行延迟（Latency）。

### 1.1 竞赛题目

| 题目编号        | 算法名称                             | 测试路径                                          | 核心头文件                                |
| --------------- | ------------------------------------ | ------------------------------------------------- | ----------------------------------------- |
| **题目1** | SHA-256                              | `security/L1/tests/sha224_256/sha256/`          | `sha224_256.hpp`                        |
| **题目2** | LZ4 Compress                         | `data_compression/L1/tests/lz4_compress/`       | `lz4_compress.hpp`, `lz_compress.hpp` |
| **题目3** | Cholesky (Complex Fixed-Point ARCH0) | `solver/L1/tests/cholesky/complex_fixed_arch0/` | `cholesky.hpp`                          |

### 1.2 目标平台

- **FPGA 器件**：Zynq-7000 (xc7z020-clg484-1)
- **工具版本**：Vitis HLS 2024.2

---

## 2. 优化约束条件

### 2.1 必须遵守的规则

1. **时钟频率固定**：不允许修改工程默认时钟频率

   - 所有算法使用相同的时钟约束
   - 评分时以默认时钟配置运行 Co-simulation
2. **功能正确性**：优化后的算法必须通过 Co-simulation 验证

   - C Simulation (csim) 必须 PASS
   - Co-simulation (cosim) 必须 PASS
   - 输出结果与参考实现完全一致
3. **允许修改的文件**：仅允许修改指定的头文件

   - 不得修改测试文件 (test_*.cpp)
   - 不得修改 Makefile 或 Tcl 脚本中的时钟配置
4. **资源约束**：优化后的设计必须能在 xc7z020 器件上实现

   - LUT、FF、BRAM、DSP 使用量不得超过器件容量

---

## 3. 评分公式

### 3.1 单题评分标准

每道题目的得分基于 **Co-simulation Latency**（单位：时钟周期数）计算：

#### 3.1.1 Latency 归一化

对于每道题目，定义：

- **L_baseline**：官方未优化版本的 Co-simulation Latency（基准值）
- **L_student**：学生优化版本的 Co-simulation Latency
- **L_best**：所有提交中的最优 Latency

计算**归一化得分**：

```
Score_normalized = (L_baseline - L_student) / (L_baseline - L_best)
```

#### 3.1.2 单题得分公式

每道题目满分 **100 分**，按以下规则计分：

```
Score_single = 100 × Score_normalized
```

**特殊情况处理**：

1. **未通过验证**：

   - 若 C Simulation 未通过 → 0 分
   - 若 Co-simulation 未通过 → 0 分
   - 若结果错误 → 0 分
2. **无优化或负优化**：

   - 若 L_student ≥ L_baseline → 基础分 10 分（功能正确但无优化）
   - 若 L_student > L_baseline（性能退化）→ 5 分
3. **最优提交**：

   - 若 L_student = L_best → 满分 100 分
4. **得分上限**：

   - 单题得分不超过 100 分，不低于 0 分

---

### 3.2 总分计算

#### 3.2.1 加权总分

三道题目采用**加权平均**方式计算总分：

```
Total_Score = w1 × Score_SHA256 + w2 × Score_LZ4 + w3 × Score_QRD
```

**推荐权重分配**（根据算法难度）：

| 算法             | 权重 | 理由                                       |
| ---------------- | ---- | ------------------------------------------ |
| SHA-256          | 30%  | 基础哈希算法，适合入门优化                 |
| LZ4 Compress     | 35%  | 中等难度，涉及数据依赖和流水线             |
| Cholesky (ARCH0) | 35%  | 较高难度，涉及定点运算、复数运算和矩阵分解 |

即：

```
Total_Score = 0.30 × Score_SHA256 + 0.35 × Score_LZ4 + 0.35 × Score_Cholesky
```

#### 3.2.2 完整性加分

为鼓励学生完成所有题目，设置**完整性加分**：

- 完成 1 道题目：总分 × 1.0（无加分）
- 完成 2 道题目：总分 × 1.05（加 5%）
- 完成 3 道题目：总分 × 1.10（加 10%）

最终得分：

```
Final_Score = Total_Score × Completeness_Bonus
```

**注意**：最终得分上限为 110 分。

---

## 4. 评分流程

### 4.1 自动化评分脚本

评审系统将执行以下步骤：

#### 步骤 1：环境准备

```bash
source /opt/xilinx/Vitis_HLS/2024.2/settings64.sh
export XPART=xc7z020-clg484-1
```

#### 步骤 2：运行每道题目

以 SHA-256 为例：

```bash
cd Vitis_Libraries/security/L1/tests/sha224_256/sha256
make clean
make run TARGET=csim    # 验证功能
make run TARGET=cosim   # 获取 Latency
```

Cholesky 测试：

```bash
cd Vitis_Libraries/solver/L1/tests/cholesky/complex_fixed_arch0
make clean
make run TARGET=csim
make run TARGET=cosim
```

#### 步骤 3：提取 Latency

从 Co-simulation 报告中提取关键指标：

```
proj_*/solution1/sim/report/*_cosim.rpt
```

关键字段：

- **Latency (cycles)**：总延迟（时钟周期数）
- **Interval (cycles)**：吞吐间隔

评分使用 **Latency** 值。

#### 步骤 4：计算得分

根据第 3 节公式计算各题得分和总分。

---

### 4.2 排名规则

1. **主排名**：按 `Final_Score` 从高到低排序
2. **平局处理**（若总分相同）：
   - 优先比较 Cholesky 得分（难度最高）
   - 其次比较 LZ4 Compress 得分
   - 最后比较 SHA-256 得分
   - 若仍相同，则比较提交时间（早提交者优先）

---

## 5. 评分示例

### 5.1 示例数据

假设三道题目的 Baseline 和 Best Latency 如下：

| 算法         | Baseline (周期) | Best (周期) | 学生A (周期) | 学生B (周期) |
| ------------ | --------------- | ----------- | ------------ | ------------ |
| SHA-256      | 10000           | 3000        | 5000         | 8000         |
| LZ4 Compress | 50000           | 20000       | 30000        | 25000        |
| Cholesky     | 5000            | 2000        | 3000         | 2500         |

### 5.2 学生 A 得分计算

#### SHA-256

```
Score_normalized = (10000 - 5000) / (10000 - 3000) = 5000 / 7000 = 0.714
Score_SHA256 = 100 × 0.714 = 71.4 分
```

#### LZ4 Compress

```
Score_normalized = (50000 - 30000) / (50000 - 20000) = 20000 / 30000 = 0.667
Score_LZ4 = 100 × 0.667 = 66.7 分
```

#### Cholesky

```
Score_normalized = (5000 - 3000) / (5000 - 2000) = 2000 / 3000 = 0.667
Score_Cholesky = 100 × 0.667 = 66.7 分
```

#### 加权总分

```
Total_Score = 0.30 × 71.4 + 0.35 × 66.7 + 0.35 × 66.7
            = 21.42 + 23.35 + 23.35
            = 68.12 分
```

#### 完整性加分（完成 3 题）

```
Final_Score = 68.12 × 1.10 = 74.93 分
```

---

### 5.3 学生 B 得分计算

#### SHA-256

```
Score_normalized = (10000 - 8000) / (10000 - 3000) = 2000 / 7000 = 0.286
Score_SHA256 = 100 × 0.286 = 28.6 分
```

#### LZ4 Compress

```
Score_normalized = (50000 - 25000) / (50000 - 20000) = 25000 / 30000 = 0.833
Score_LZ4 = 100 × 0.833 = 83.3 分
```

#### Cholesky

```
Score_normalized = (5000 - 2500) / (5000 - 2000) = 2500 / 3000 = 0.833
Score_Cholesky = 100 × 0.833 = 83.3 分
```

#### 加权总分

```
Total_Score = 0.30 × 28.6 + 0.35 × 83.3 + 0.35 × 83.3
            = 8.58 + 29.16 + 29.16
            = 66.90 分
```

#### 完整性加分（完成 3 题）

```
Final_Score = 66.90 × 1.10 = 73.59 分
```

**结论**：学生 A 排名更高（74.93 > 73.59）

---

## 6. 提交要求

### 6.1 提交材料

学生需提交完整的 hlstrack2025 仓库，包含：

1. **修改后的头文件**（每道题目对应的 `*.hpp` 文件）
2. **reports 目录**：每道题目的测试目录下需包含
   - `csim.log` - C 仿真日志
   - `cosim.log` - Co-simulation 日志
   - `csynth.log` - 综合报告
3. **prompts 目录**：大模型使用记录（`llm_usage.md`）

### 6.2 提交检查清单

- [ ] 所有题目均通过 C Simulation
- [ ] 所有题目均通过 Co-simulation
- [ ] 已提取并保存 cosim.log
- [ ] 已填写 LLM 使用记录（如使用）
- [ ] 仓库结构完整（未删除其他文件）

---

## 7. 常见问题

### Q1: 如果只完成部分题目怎么办？

**A**: 可以提交，但：

- 未完成的题目得 0 分
- 无法获得完整性加分（或仅获得部分加分）
- 建议至少完成 2 道题目以获得基础加分

### Q2: 可以修改时钟频率吗？

**A**: **不可以**。所有提交必须使用相同的时钟配置，以确保公平性。修改时钟配置的提交将被判为违规（0 分）。

### Q3: Co-simulation 很慢怎么办？

**A**: Co-simulation 确实耗时较长，建议：

- 先用 C Simulation 快速验证功能
- 优化完成后再运行 Co-simulation
- 仅提交一次 Co-simulation 结果

### Q4: 资源超限怎么办？

**A**: 若优化后的设计超过 xc7z020 资源限制：

- 该题目得 0 分（视为未通过验证）
- 需要权衡 Latency 与资源使用
- 建议使用 `csynth.log` 提前检查资源使用情况

### Q5: 如何确认自己的 Latency 值？

**A**: 在 Co-simulation 报告中查找：

```
|         Latency (cycles)         |
|    min   |    max   |    avg    |
|----------|----------|-----------|
|   XXXX   |   XXXX   |   XXXX    |
```

评分使用 **max** 值（最坏情况延迟）。

---

## 8. 附录：Baseline 参考值

> **注意**：以下为示例值，实际 Baseline 值以竞赛公布为准。

| 算法             | Baseline Latency (cycles) | 预期优化目标 |
| ---------------- | ------------------------- | ------------ |
| SHA-256          | 690                       | 400          |
| LZ4 Compress     | 4784                      | 2500         |
| Cholesky (ARCH0) | 7015                      | 3500         |

**优化建议**：

- **SHA-256**：重点优化循环展开、数据流水线
- **LZ4 Compress**：关注数据依赖、内存访问模式
- **Cholesky (ARCH0)**：
  - 复数定点运算优化
  - 循环流水线设计（关注对角线计算和非对角线更新）
  - 数组分区减少访存冲突
  - 平方根运算延迟优化
  - 利用矩阵对称性减少计算量

---

## 9. 联系与支持

- **技术问题**：请在QQ竞赛群 1022632722 提问
- **提交问题**：检查 SUBMISSION_GUIDE 文档

---

**祝各位同学取得好成绩！**

---

*更新日期：2025-10-18*
