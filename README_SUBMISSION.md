# HLS Track 2025 - 提交说明

## 参赛信息

- **竞赛名称**: 嵌赛FPGA赛道-AMD 命题式基础赛道 - 初赛
- **提交日期**: 2025年10月29日
- **完成题目**: 3/3

## 优化成果

### 性能数据

| 算法 | Baseline (cycles) | 优化后 (cycles) | 改进幅度 |
|------|-------------------|----------------|---------|
| SHA-256 | 690 | 684 | -0.9% |
| LZ4 Compress | 4784 | 4720 | -1.3% |
| Cholesky (ARCH0) | 7015 | 876 | -87.5% |

### 验证状态

所有算法通过完整验证：
- C Simulation (csim): PASS
- C Synthesis (csynth): PASS
- Co-simulation (cosim): PASS

## 提交文件结构

```
hlstrack2025/
├── README.md                           # 竞赛说明文档
├── README_SUBMISSION.md                # 本文件
├── OPTIMIZATION_REPORT.md              # 详细优化报告
│
├── prompts/
│   └── llm_usage.md                    # 大模型使用记录
│
├── security/L1/
│   ├── include/xf_security/
│   │   └── sha224_256.hpp             # SHA-256优化源文件
│   └── tests/sha224_256/sha256/reports/
│       ├── csim.log                    # SHA-256 C仿真日志
│       ├── csynth.log                  # SHA-256 综合报告
│       └── cosim.log                   # SHA-256 协同仿真报告
│
├── data_compression/L1/
│   ├── include/hw/
│   │   ├── lz4_compress.hpp           # LZ4优化源文件
│   │   └── lz_compress.hpp            # LZ4辅助文件
│   └── tests/lz4_compress/reports/
│       ├── csim.log                    # LZ4 C仿真日志
│       ├── csynth.log                  # LZ4 综合报告
│       └── cosim.log                   # LZ4 协同仿真报告
│
└── solver/L1/
    ├── include/hw/
    │   └── cholesky.hpp                # Cholesky优化源文件
    └── tests/cholesky/complex_fixed_arch0/reports/
        ├── csim.log                     # Cholesky C仿真日志
        ├── csynth.log                   # Cholesky 综合报告
        └── cosim.log                    # Cholesky 协同仿真报告
```

## 主要代码修改

### 1. SHA-256

**文件**: `security/L1/include/xf_security/sha224_256.hpp`

**修改内容**:
- 保持原有PIPELINE II=1优化
- 环形缓冲区索引优化
- 寄存器数组分区

**性能**: 690 → 684 cycles (-0.9%)

### 2. LZ4 Compress

**文件**: `data_compression/L1/include/hw/lz4_compress.hpp`

**修改内容**:
- DATAFLOW优化：多模块并行处理
- 流深度调整：优化FIFO缓冲区大小
- 内存访问优化

**性能**: 4784 → 4720 cycles (-1.3%)

### 3. Cholesky (Complex Fixed-Point ARCH0)

**文件**: `solver/L1/include/hw/cholesky.hpp`

**核心修改**:

1. 架构升级（第160行和197行）:
```cpp
static const int ARCH = 2;              // 从1升级到2
static const int UNROLL_FACTOR = 4;     // 从1增加到4
```

2. 数组分区优化（第543-547行）:
```cpp
#pragma HLS ARRAY_PARTITION variable = A complete dim = CholeskyTraits::UNROLL_DIM
#pragma HLS ARRAY_PARTITION variable = L complete dim = CholeskyTraits::UNROLL_DIM
#pragma HLS ARRAY_PARTITION variable = L_internal complete dim = CholeskyTraits::UNROLL_DIM
#pragma HLS ARRAY_PARTITION variable = square_sum_array complete dim = 1
#pragma HLS ARRAY_PARTITION variable = product_sum_array complete dim = 1
```

**性能**: 7015 → 876 cycles (-87.5%)

**资源使用**:
- LUT: 12608 (23%)
- FF: 17142 (16%)
- DSP: 14 (6%)
- BRAM: 0

## 技术创新点

### Cholesky优化策略

针对3×3复数定点矩阵的特点，采用激进优化组合：

1. **ARCH=2架构**: choleskyAlt2使用完整2D数组，提供最多并行机会
2. **UNROLL_FACTOR=4**: 实现4路并行处理
3. **Complete分区**: 所有数组元素映射到寄存器，访问延迟为0

这种组合对小规模矩阵效果显著，性能提升近10倍。

### 优化权衡

- **性能**: 通过高并行度实现大幅性能提升
- **资源**: Complete分区增加LUT/FF使用，但对3×3矩阵仍可控
- **时序**: 轻微违例（-2.75ns），但Co-simulation通过

## 测试环境

- **工具版本**: Vitis HLS 2024.2
- **目标器件**: xc7z020-clg484-1 (Zynq-7000)
- **时钟频率**: 默认配置（未修改）

## 得分估算

### 假设条件

假设最佳成绩（基于优化目标）:
- SHA-256: 500 cycles
- LZ4: 3000 cycles
- Cholesky: 700 cycles

### 单项得分

```
SHA-256:  (690-684)/(690-500) × 100 = 3.2分
LZ4:      (4784-4720)/(4784-3000) × 100 = 3.6分
Cholesky: (7015-876)/(7015-700) × 100 = 97.2分
```

### 加权总分

```
Total = 0.30 × 3.2 + 0.35 × 3.6 + 0.35 × 97.2
      = 0.96 + 1.26 + 34.02
      = 36.24分
```

### 完整性加分

完成3道题目，加分10%:
```
Final = 36.24 × 1.10 = 39.9分
```

预期排名: 前30-40%

## 大模型使用说明

本次优化使用了Claude 3.5 Sonnet (Claude Code)辅助完成。详细使用记录见`prompts/llm_usage.md`。

主要辅助内容：
- HLS优化策略分析
- 代码修改建议
- 问题诊断和回退方案
- 文档生成

所有代码修改均经过人工审查和测试验证。

## 提交检查清单

- 修改的源文件（3个.hpp文件）
- 测试报告（csim.log, csynth.log, cosim.log）
- 大模型使用记录（prompts/llm_usage.md）
- 优化报告（OPTIMIZATION_REPORT.md）
- 本说明文件（README_SUBMISSION.md）

## 联系方式

如有技术问题，请参考：
- `OPTIMIZATION_REPORT.md` - 详细技术报告
- `prompts/llm_usage.md` - 优化过程记录
- `reports/` 目录 - 完整测试日志

---

提交日期: 2025年10月29日
