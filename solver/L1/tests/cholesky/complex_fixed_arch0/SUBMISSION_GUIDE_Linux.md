# Cholesky (Complex Fixed-Point ARCH0) L1 测试工程提交与运行指南

本文面向使用 Linux 系统开展 `solver/L1/tests/cholesky/complex_fixed_arch0` 优化实验的本科生，说明本地运行流程、GitHub 提交流程与所需材料。

---

## 1. 本地运行步骤（Linux）

### 1.1 环境准备

1. 已安装 **Xilinx/AMD Vivado 2024.2** 与 **Vitis 2024.2**。
2. 在终端执行官方脚本，写入环境变量：

```bash
source /opt/xilinx/Vivado/2024.2/settings64.sh
source /opt/xilinx/Vitis/2024.2/settings64.sh
```

> 如安装路径不同，请替换为实际路径。

3. 确认 `XILINX_VIVADO`、`XILINX_VITIS` 已出现在 `env` 输出中。

### 1.2 工程目录

克隆完整的 Vitis_Libraries 仓库：

```bash
git clone https://gitee.com/Vickyiii/hlstrack2025.git
cd hlstrack2025/solver/L1/tests/cholesky/complex_fixed_arch0
```

### 1.3 设置 FPGA 器件（XPART）

Makefile 支持直接指定 FPGA part。针对 **Zynq-7000 (XC7Z020-CLG484-1)**，请在执行 `make` 时加入：

```bash
export XPART=xc7z020-clg484-1
```

> 设置后 `PLATFORM` 将被忽略，Makefile 会直接使用该 part 名称。

### 1.4 常用命令

| 目标        | 命令                                          | 说明               |
| ----------- | --------------------------------------------- | ------------------ |
| C 仿真      | `make clean && make run TARGET=csim`        | 功能级验证         |
| HLS 综合    | `make clean && make run TARGET=csynth`      | 生成综合报告       |
| Co-simulation | `make clean && make run TARGET=cosim`     | RTL 仿真验证       |
| Vivado 综合 | `make clean && make run TARGET=vivado_syn`  | 需要 Vivado flow   |
| Vivado 实现 | `make clean && make run TARGET=vivado_impl` | 完整实现，耗时较长 |

建议流程：

1. `make clean` 清理旧文件。
2. `export XPART=xc7z020-clg484-1`
3. `make run TARGET=csim`
4. `make run TARGET=csynth`
5. `make run TARGET=cosim` （可选）
6. 视需要执行更高级目标。

### 1.5 输出位置

- HLS 工作目录：`hls/`
- 生成的配置：`hls_config.cfg`
- 日志：`*_hls.log`

运行结束后请保留相关日志和综合报告，作为提交材料的一部分。

---

## 2. GitHub 提交目录要求

为便于助教自动化复现与评分，请提交完整的 **Vitis_Libraries** 仓库，按以下要求组织：

### 2.1 允许修改的文件

**头文件修改（核心优化内容）：**

- `solver/L1/include/hw/cholesky.hpp`
- `solver/L1/include/hw/` 下其他相关头文件（如需）

> **注意**：Cholesky 分解算法主要涉及：
> - `cholesky.hpp` - Cholesky 分解核心实现（对称正定矩阵的 L·L^T 分解）
> - 支持三种架构：ARCH=0 (Basic)、ARCH=1 (Lower latency)、ARCH=2 (Further improved)
> - complex_fixed_arch0 使用复数定点数 + 基础架构（资源消耗最低）
> - 涉及矩阵分解、平方根运算、三角矩阵生成等操作

### 2.2 必须新增的目录与文件

在 `solver/L1/tests/cholesky/complex_fixed_arch0/` 目录下新增：

```
solver/L1/tests/cholesky/complex_fixed_arch0/
├── Makefile                        # 原有文件（如有必要可微调）
├── description.json                # 原有文件（保持不变）
├── dut_type.hpp                    # 原有文件（保持不变）
├── hls_config.tmpl                 # 原有文件（保持不变）
├── run_hls.tcl                     # 原有文件（保持不变）
├── reports/                        # 新增目录
│   ├── csim.log                    # C 仿真日志
│   ├── csynth.log                  # HLS 综合日志
│   └── cosim.log                   # Co-simulation 日志
└── prompts/                        # 新增目录
    └── llm_usage.md                # 大模型使用记录（见 §3 模板）
```

### 2.3 提交仓库完整结构示例

```
Vitis_Libraries/
├── solver/
│   ├── L1/
│   │   ├── include/
│   │   │   └── hw/
│   │   │       ├── cholesky.hpp         # 允许修改（Cholesky 分解核心）
│   │   │       └── ...（其他头文件）
│   │   ├── tests/
│   │   │   └── cholesky/
│   │   │       ├── host/                # 共享目录（保持不变）
│   │   │       ├── kernel/              # 共享目录（保持不变）
│   │   │       ├── datas/               # 共享目录（保持不变）
│   │   │       └── complex_fixed_arch0/ # 本测试专用目录
│   │   │           ├── Makefile
│   │   │           ├── description.json
│   │   │           ├── dut_type.hpp
│   │   │           ├── reports/         # 新增
│   │   │           └── prompts/         # 新增
│   │   └── ...
│   └── ...
└── ...（其他库目录保持原样）
```

**注意事项：**

- 仅修改 `solver/L1/include/hw/*.hpp` 中的头文件
- `reports/` 和 `prompts/` 仅在 `solver/L1/tests/cholesky/complex_fixed_arch0/` 下新增
- 共享目录 `host/`、`kernel/`、`datas/` 保持不变
- 其他所有目录与文件保持原样
- 评审脚本将克隆官方仓库，替换你修改的头文件，复制 reports 和 prompts 后运行

---

## 3. 大模型使用记录（Markdown 模板）

请在 `prompts/llm_usage.md` 中填写以下内容：

```markdown
# 大模型辅助使用记录

- **模型名称**：例如 GPT-4 Turbo, Claude 3, 通义千问 等
- **提供方 / 访问 API**：OpenAI API, Azure OpenAI, 百度千帆 ...
- **主要用途**：代码重构 / 优化建议 / 文档撰写 等
- **完整 Prompt 内容**：
  <粘贴与本项目相关的全部提示词/上下文>

- **模型输出摘要**：简述主要结论或修改建议
- **人工审核与采纳情况**：哪些建议被采纳、是否二次验证
```

请确保上述文件与目录完整提交，以便评审方快速复现与审查。
