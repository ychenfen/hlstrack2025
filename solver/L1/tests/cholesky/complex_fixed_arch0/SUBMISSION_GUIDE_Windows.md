# Cholesky (Complex Fixed-Point ARCH0) L1 测试工程提交与运行指南（Windows 版）

本文面向使用 **Windows 系统**开展 `solver/L1/tests/cholesky/complex_fixed_arch0` 优化实验的本科生，说明本地运行流程、GitHub 提交流程与所需材料。

---

## 1. 本地运行步骤（Windows）

### 1.1 环境准备

1. 已安装 **Xilinx/AMD Vitis HLS 2024.2**（或 Vivado 2024.2 含 Vitis HLS）。
2. 克隆完整的 Vitis_Libraries 仓库：

```cmd
git clone https://gitee.com/Vickyiii/hlstrack2025.git
cd hlstrack2025\solver\L1\tests\cholesky\complex_fixed_arch0
```

---

### 1.2 方法一：命令行流程（推荐用于自动化测试）

#### 步骤 1：打开命令提示符（CMD）

- 按 `Win + R`，输入 `cmd`，回车。

#### 步骤 2：加载 Vitis HLS 环境

```cmd
call C:\Xilinx\Vitis_HLS\2024.2\settings64.bat
```

> **注意**：请根据实际安装路径调整，默认路径通常为 `C:\Xilinx\Vitis_HLS\2024.2\` 或 `C:\Xilinx\Vivado\2024.2\`。

#### 步骤 3：切换到测试目录

```cmd
cd C:\Users\<YourUsername>\Desktop\XUP\Competition\FPGA_national2025\Vitis_Libraries\solver\L1\tests\cholesky\complex_fixed_arch0
```

> 替换 `<YourUsername>` 和路径为你的实际克隆位置。

#### 步骤 4：执行 HLS 脚本

```cmd
vitis_hls -f run_hls.tcl
```

> `run_hls.tcl` 是 Vitis Libraries 提供的 Tcl 脚本，会自动执行 C 仿真、综合等步骤。

#### 步骤 5：查看输出

- HLS 工作目录：`proj_<testcase>\`
- 综合报告：`proj_<testcase>\solution1\syn\report\`
- 日志文件：`vitis_hls.log`

保留这些报告和日志，作为提交材料的一部分（放入 `reports\` 目录）。

---

### 1.3 方法二：图形界面（GUI）流程（推荐用于交互式调试）

#### 步骤 1：启动 Vitis HLS GUI

- 在开始菜单找到 **Vitis HLS 2024.2**，或双击桌面快捷方式。
- 或在 CMD 中执行：

```cmd
call C:\Xilinx\Vitis_HLS\2024.2\settings64.bat
vitis_hls
```

#### 步骤 2：打开或创建项目

**选项 A - 打开现有 Tcl 脚本：**

1. 菜单栏：`File` → `Open Script...`
2. 浏览到 `solver\L1\tests\cholesky\complex_fixed_arch0\run_hls.tcl`
3. 点击 `Open` 执行脚本。

**选项 B - 手动创建项目：**

1. 菜单栏：`File` → `New Project`
2. 设置项目名称：`cholesky_test`
3. **Add Files**：
   - 添加 `../host/test_cholesky.cpp`（测试文件在上级 host/ 目录）
   - 添加 `../kernel/kernel_cholesky_0.cpp`（内核文件在上级 kernel/ 目录）
4. **Top Function**：设置为 `kernel_cholesky_0`
5. **Part Selection**：选择 **xc7z020clg484-1**（Zynq-7020）
6. 点击 `Finish`

#### 步骤 3：运行仿真与综合

在 GUI 工具栏或 `Solution` 菜单中依次执行：

1. **C Simulation**：验证功能正确性
   - `Project` → `Run C Simulation`
2. **C Synthesis**：生成 RTL 和综合报告
   - `Solution` → `Run C Synthesis` → `Active Solution`
3. **Co-simulation**（可选）：RTL 仿真
   - `Solution` → `Run C/RTL Cosimulation`

#### 步骤 4：查看报告

- 综合完成后，双击 `Solution` 面板中的 `syn/report/` 节点查看：
  - **Timing**：时钟周期和时序
  - **Latency**：延迟和吞吐量
  - **Utilization**：资源使用情况（LUT、FF、BRAM、DSP）

#### 参考资料

完整 GUI 操作流程请参考官方教程：
- [Vitis Libraries Getting Started Tutorial](https://github.com/Xilinx/Vitis-Tutorials/tree/2024.2/Getting_Started/Vitis_Libraries)

---

### 1.4 输出文件整理

无论使用命令行还是 GUI，运行结束后请：

1. 在 `solver\L1\tests\cholesky\complex_fixed_arch0\` 下创建 `reports\` 目录
2. 复制以下文件到 `reports\`：
   - `vitis_hls.log` 或 `csim.log` → 重命名为 `csim.log`
   - `proj_*\solution1\syn\report\csynth.rpt` → 重命名为 `csynth.log`
   - `proj_*\solution1\sim\report\cosim.rpt` → 重命名为 `cosim.log`（如有）
3. 保留完整报告供提交审查

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
