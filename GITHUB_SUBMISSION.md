# GitHub 提交与推送指南

## 当前状态

您的代码已经准备就绪，可以推送到GitHub。以下是详细的步骤：

## 步骤1: 创建GitHub仓库

### 方法1: 通过GitHub网页界面

1. 访问 [GitHub](https://github.com)
2. 点击右上角的 "+" 号
3. 选择 "New repository"
4. 填写仓库信息:
   - **Repository name**: `hlstrack2025`
   - **Description**: `Vitis HLS Algorithm Optimization Competition 2025 - 3 algorithms optimized for FPGA acceleration`
   - **Visibility**: Private (推荐) 或 Public
   - **不要**勾选 "Add a README file" (已有)
   - **不要**勾选 "Add .gitignore" (已有)
   - **不要**勾选 "Choose a license" (已有)
5. 点击 "Create repository"

### 方法2: 通过GitHub CLI (如果已安装)

```bash
# 如果未安装，可通过以下命令安装
# brew install gh (macOS)
# 或访问 https://cli.github.com/

gh repo create hlstrack2025 --private --description="Vitis HLS Algorithm Optimization Competition 2025 - 3 algorithms optimized for FPGA acceleration"
```

## 步骤2: 推送代码到GitHub

### 添加远程仓库

```bash
# 替换 YOUR_USERNAME 为您的GitHub用户名
git remote add origin https://github.com/YOUR_USERNAME/hlstrack2025.git
```

### 推送代码

```bash
# 推送主分支
git branch -M main
git push -u origin main
```

### 验证推送

1. 访问您的GitHub仓库: `https://github.com/YOUR_USERNAME/hlstrack2025`
2. 确认所有文件已上传

## 步骤3: 提交报告

### 报告文件清单

以下文件已准备好提交：

#### 核心优化文件
- [x] `security/L1/include/xf_security/sha224_256.hpp` - SHA-256优化
- [x] `data_compression/L1/include/hw/lz4_compress.hpp` - LZ4压缩优化
- [x] `solver/L1/include/hw/cholesky.hpp` - Cholesky分解优化

#### 文档文件
- [x] `PERFORMANCE_REPORT.md` - 详细性能报告
- [x] `SUBMISSION_GUIDE.md` - 提交指南
- [x] `OPTIMIZATION_SUMMARY.md` - 优化总结
- [x] `README.md` - 项目说明（已有）

#### Git管理
- [x] `.gitignore` - Git忽略规则

## 步骤4: 添加提交标签（可选）

如果您想要标记这个提交为竞赛提交：

```bash
# 创建标签
git tag -a v1.0-competition -m "Competition submission: 3 algorithms optimized for FPGA acceleration"

# 推送标签
git push origin v1.0-competition
```

## 步骤5: 验证提交内容

提交完成后，您的仓库应包含：

```
hlstrack2025/
├── .gitignore
├── README.md                          # 项目说明
├── OPTIMIZATION_SUMMARY.md            # 优化总结
├── PERFORMANCE_REPORT.md              # 性能报告 ⭐
├── SUBMISSION_GUIDE.md                # 提交指南 ⭐
├── GITHUB_SUBMISSION.md               # 本文件
│
├── security/
│   └── L1/
│       └── include/
│           └── xf_security/
│               └── sha224_256.hpp     # ⭐ 已优化
│
├── data_compression/
│   └── L1/
│       └── include/
│           └── hw/
│               └── lz4_compress.hpp   # ⭐ 已优化
│
└── solver/
    └── L1/
        └── include/
            └── hw/
                └── cholesky.hpp       # ⭐ 已优化
```

## 自动化脚本

如果您想要自动化推送过程，可以使用以下脚本：

```bash
#!/bin/bash
# push_to_github.sh

echo "=== Vitis HLS优化竞赛 - GitHub提交脚本 ==="

# 检查远程仓库是否已设置
if ! git remote get-url origin > /dev/null 2>&1; then
    echo "错误: 尚未设置远程��库"
    echo "请运行: git remote add origin https://github.com/YOUR_USERNAME/hlstrack2025.git"
    exit 1
fi

echo "1. 推送代码到GitHub..."
git push -u origin main

echo "2. 创建竞赛标签..."
git tag -a v1.0-competition -m "Competition submission: 3 algorithms optimized for FPGA acceleration"
git push origin v1.0-competition

echo "3. 完成！"
echo "请访问您的仓库验证提交："
git remote get-url origin | sed 's|\.git$||'
```

使用方式：
```bash
chmod +x push_to_github.sh
./push_to_github.sh
```

## 竞赛提交清单

提交前请确认：

- [x] 所有三个算法的头文件已优化
- [x] 优化总结文档已创建
- [x] 性能报告已创建
- [x] 提交指南已创建
- [x] Git仓库已初始化
- [x] 文件已提交到本地仓库
- [ ] 远程仓库已创建
- [ ] 代码已推送到GitHub

## 提交信息格式

我们的提交信息遵循以下格式：

```
feat: 优化三个Vitis HLS算法以减少延迟

- SHA-256: 添加循环展开(factor=2)，增加流深度(128)，预期改进30-35%
- LZ4 Compress: 增加循环展开(factor=4)，增加流深度，预期改进25-35%
- Cholesky: 增加展开因子(8)，优化依赖，预期改进10-15%

技术要点:
- 循环展开增加并行性
- 流水线优化保持II=1
- 数组分区减少访问冲突
- 流深度优化减少阻塞

目标平台: xc7z020-clg484-1
工具版本: Vitis HLS 2024.2

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

## 常见问题

### Q: 如果推送时提示认证错误？

A: 请使用以下方法之一：

1. **使用Personal Access Token**:
   - 在GitHub生成token: Settings → Developer settings → Personal access tokens
   - 推送时使用token而非密码

2. **使用SSH** (推荐):
   ```bash
   # 生成SSH密钥
   ssh-keygen -t ed25519 -C "your_email@example.com"

   # 添加到ssh-agent
   ssh-add ~/.ssh/id_ed25519

   # 在GitHub添加SSH密钥
   # Settings → SSH and GPG keys → New SSH key

   # 切换到SSH远程
   git remote set-url origin git@github.com:YOUR_USERNAME/hlstrack2025.git
   ```

### Q: 仓库已经是public，可以改为private吗？

A: 可以：
1. 进入仓库 Settings
2. 滚动到 "Danger Zone"
3. 点击 "Change visibility" → "Change to private"

### Q: 如何撤销上一次的提交？

A: 使用以下命令：
```bash
# 撤销上一次提交但保留更改
git reset --soft HEAD~1

# 完全删除上一次提交
git reset --hard HEAD~1
```

## 联系信息

如果您在提交过程中遇到问题：

1. 查看 [GitHub官方文档](https://docs.github.com/en)
2. 检查GitHub仓库Settings中的Notifications设置
3. 确认您有推送权限

---

**提交时间**: 2025-10-31
**版本**: v1.0-competition
**优化者**: Claude Code (Anthropic)

---

🎉 **恭喜！您的优化代码已准备好提交到GitHub！**
