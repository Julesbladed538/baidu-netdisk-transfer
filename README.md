<p align="center">
  <a href="https://github.com/houtianze/bypy"><img src="https://img.shields.io/badge/bypy-v1.8.9-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="bypy"></a>
  <a href="https://github.com/aria2/aria2"><img src="https://img.shields.io/badge/aria2c-v1.35.0-4CBB17?style=for-the-badge&logo=download&logoColor=white" alt="aria2c"></a>
  <br>
  <img src="https://img.shields.io/badge/Platform-Linux%20x86__64-FCC624?style=flat-square&logo=linux&logoColor=black" alt="Platform">
  <img src="https://img.shields.io/badge/License-MIT-CB904D?style=flat-square" alt="License">
  <img src="https://img.shields.io/badge/Verified-343GB_32/32-success?style=flat-square&logo=checkmarx&logoColor=white" alt="Verified">
  <img src="https://img.shields.io/badge/Root-Never_Needed-brightgreen?style=flat-square&logo=superuser&logoColor=white" alt="No Root">
  <img src="https://img.shields.io/badge/Speed-39.6_MB/s-ff69b4?style=flat-square&logo=speedtest&logoColor=white" alt="Speed">
</p>

<br>

<h1 align="center">
  🔁 百度网盘 &harr; Linux 服务器<br>双向大文件传输
</h1>

<p align="center">
  <b>一跳直传 · 无需本地中转 · 零系统依赖 · 断点续传 · MD5 完整性校验</b><br><br>
  <sub>基于 <a href="https://github.com/houtianze/bypy">bypy</a> v1.8.9 百度网盘 CLI + <a href="https://github.com/aria2/aria2">aria2c</a> v1.35.0 多线程下载引擎</sub>
</p>

<br>

---

<br>

## 📌 一句话说明

> 在 Linux 服务器上直接用命令行操作百度网盘，**数据不经过你的 Windows/Mac 电脑**，利用机房带宽（40 MB/s+）极速传输，从测序仪下机到服务器就绪，全程一键。

<br>

## 🧭 传统方式 vs 本方案

<table>
<tr>
<td width="50%" align="center">

### ❌ 传统两跳方式

</td>
<td width="50%" align="center">

### ✅ 本 Skill（一跳直传）

</td>
</tr>
<tr>
<td align="center">

```
┌──────────────┐
│   ☁️ 百度网盘  │
└──────┬───────┘
       │
       │ ① 下载到本地
       │    10~30 MB/s
       ▼
┌──────────────┐
│ 💻 你的电脑   │  ← 需本地磁盘
└──────┬───────┘
       │
       │ ② scp/rsync
       │    宽带上传限速
       ▼
┌──────────────┐
│ 🖥️ Linux 服务器│
└──────────────┘

👤 人需在场
⏱️ 两倍时间
💾 占用本地空间
```

</td>
<td align="center">

```
┌──────────────┐
│   ☁️ 百度网盘  │
└──────┬───────┘
       │
       │ ① bypy 直传
       │    40+ MB/s
       │    断点续传
       │    并行加速
       ▼
┌──────────────┐
│ 🖥️ Linux 服务器│
└──────────────┘

🤖 nohup 后台运行
⚡ 机房带宽直通
💾 本地零占用
✅ MD5 自动校验
```

</td>
</tr>
</table>

| 对比维度 | 传统方式 | 本 Skill |
|:---|:---:|:---:|
| 💰 传输跳数 | **两跳**（网盘 → 本机 → 服务器） | **一跳**（网盘 → 服务器） |
| 🚀 速度瓶颈 | 家庭宽带 10-30 MB/s | 机房带宽 **40+ MB/s** |
| 💾 本地磁盘占用 | 需要同等大小的临时空间 | **零占用** |
| 🔐 sudo 权限 | 不需要 | 不需要 |
| 🔄 断点续传 | 取决于客户端 | ✅ aria2c 原生支持 |
| 🧵 并行下载 | 手动操作 | ✅ bypy 并行 + aria2c 多线程 |
| 🔍 MD5 校验 | 需手动 | ✅ 一行命令搞定 |
| 🤖 无人值守 | 不行 | ✅ nohup 后台运行 |

<br>

---

<br>

## 📦 仓库结构

```
baidu-netdisk-transfer/
│
├── README.md                           ← ⭐ 你在这里
├── SKILL.md                            ← 📖 完整技术文档 (630+ 行)
├── LICENSE                             ← 📜 MIT License
├── .gitignore
│
└── 小白文档/                            ← 📚 零基础三件套
    ├── 百度网盘传输_小白操作手册.docx     # 📄 Word  · 5 章完整手册
    ├── 百度网盘传输_操作表格.xlsx         # 📊 Excel · 8 Sheet 速查
    └── 百度网盘传输_演示文稿.pptx         # 📽️ PPT   · 19 页培训演示
```

<br>

---

<br>

## ⚡ 三步上手

<p align="center">
  <img src="https://img.shields.io/badge/Step-1-blue?style=for-the-badge" alt="Step 1">
  &nbsp;
  <img src="https://img.shields.io/badge/Step-2-blue?style=for-the-badge" alt="Step 2">
  &nbsp;
  <img src="https://img.shields.io/badge/Step-3-blue?style=for-the-badge" alt="Step 3">
</p>

### ① 安装（仅一次，约 30 秒）

```bash
# ── bypy ──
pip install --user bypy==1.8.9

# ── aria2c 静态编译（无需 root）──
wget -q https://github.com/aria2/aria2/releases/download/release-1.35.0/aria2-1.35.0-linux-gnu-64bit-build1.tar.bz2
tar xjf aria2-*.tar.bz2
mkdir -p ~/.local/bin
cp aria2-*/aria2c ~/.local/bin/
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

> 💡 **零系统影响**：全部安装在 `~/.local/` 下，不修改系统文件，不影响服务器上其他软件。

### ② 授权（仅一次，约 1 分钟）

```bash
bypy info
```

浏览器打开输出的链接 → 登录百度账号 → 粘贴授权码 → 回车。

### ③ 开始传输！

```bash
# ⬇️ 下行：网盘 → 服务器
bypy --downloader aria2c download /网盘目录/你的文件.fastq.gz ./

# ⬆️ 上行：服务器 → 网盘（≤ 8 MiB）
bypy --disable-ssl-check --on-dup overwrite upload ./结果.tar.gz /网盘目录/
```

<br>

---

<br>

## 📊 实战性能

| 实战案例 | 数据量 | 文件数 | 耗时 | 速度 | MD5 通过率 |
|:---|:---:|:---:|:---:|:---:|:---:|
| 🐵 食蟹猴皮肤褥疮单细胞测序 | **343.72 GB** | 32 | 155 min | **39.6 MB/s** | ✅ 32/32 |
| 🔬 边界上传测试 (8 MiB) | 8 MB | 1 | < 5 sec | 2.3 MB/s | ✅ 往返一致 |
| 📄 小文本下载 | 73 B | 1 | < 1 sec | — | ✅ |

> 🔬 **上传边界精确值**：百度 PCS API 允许直传最大 **8,388,608 字节（8 MiB）**。大于此值的文件请用百度客户端或网页上传到 `我的应用数据/bypy/` 目录。

<br>

---

<br>

## 🔧 命令速查表

### ⬇️ 下行（网盘 → 服务器）

| 操作 | 命令 | 说明 |
|:---|:---|:---|
| 🔍 查看网盘文件 | `bypy list` | 列出 `/apps/bypy/` 下所有文件 |
| 📥 下载单文件 | `bypy --downloader aria2c download /路径/文件 ./` | aria2c 多线程加速 |
| 🤖 后台并行下载 | `nohup bypy --downloader aria2c download /路径/文件 ./ > dl.log 2>&1 &` | 断线也不中断 |
| 👀 监控进度 | `watch -n 10 'ls -lh *.fastq.gz'` | 每 10 秒刷新文件大小 |
| ✅ MD5 校验 | `nohup md5sum -c md5.txt > result.txt 2>&1 &` | 下载完成后验证 |
| 📊 查看配额 | `bypy quota` | 剩余空间 + 已用空间 |

### ⬆️ 上行（服务器 → 网盘）

| 操作 | 命令 | 说明 |
|:---|:---|:---|
| 🔑 生成 MD5 | `md5sum *.fastq.gz > md5.txt` | 上传前先生成校验文件 |
| 📤 上传 ≤ 8 MiB | `bypy --disable-ssl-check --on-dup overwrite upload ./文件 /路径/` | SSL 跳过 + 自动覆盖 |
| ☁️ 上传 > 8 MiB | 用百度客户端/网页上传到 `我的应用数据/bypy/` | 受 PCS API 限制 |
| ✅ 确认上传 | `bypy list /路径/` | 验证文件已在网盘 |
| 🗑️ 删除网盘文件 | `bypy remove /路径/文件` | 清理不需要的文件 |

<br>

---

<br>

## 🛡️ 隔离原则 — 不影响任何其他软件

<p align="center">
  <b>这是本 Skill 最核心的设计纪律</b>
</p>

| # | 原则 | 具体做法 |
|:---:|:---|:---|
| 1 | 📂 只写 `~/.local/` | 不碰 `/usr/` `/etc/` `/opt/` `/lib/` |
| 2 | 🚫 **禁止** `LD_LIBRARY_PATH` 入 `.bashrc` | 会污染所有进程的库搜索路径 |
| 3 | 🔗 用 patchelf RPATH 或 wrapper | 共享库隔离，只影响 aria2c 自身 |
| 4 | 🐍 pip `--user` 或 conda 环境 | 不进系统 site-packages |
| 5 | 📝 `.bashrc` 只加 `~/.local/bin` | 唯一的一行 PATH 修改 |

```bash
# ❌ 绝对不要做
echo 'export LD_LIBRARY_PATH=...' >> ~/.bashrc

# ✅ 正确做法之一：wrapper 脚本
cat > ~/.local/bin/aria2c << 'EOF'
#!/bin/bash
export LD_LIBRARY_PATH="$HOME/.local/lib${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}"
exec ~/.local/libexec/aria2c "$@"
EOF
chmod +x ~/.local/bin/aria2c
```

<br>

---

<br>

## 🩺 故障速查

| 症状 | 原因 | 解决 |
|:---|:---|:---|
| `bypy: command not found` | `~/.local/bin` 不在 PATH | `source ~/.bashrc` |
| `aria2c: libaria2.so.0` | 共享库未找到 | 用 patchelf 设 RPATH 或 wrapper 脚本 |
| 上传 > 8 MiB 挂起 | 百度 PCS API 限制 | 改用网页/客户端上传 |
| SSL 证书错误 | 证书验证问题 | 加 `--disable-ssl-check` |
| 下载特别慢 | 未启用 aria2c | 务必加 `--downloader aria2c` |
| `bypy info` 无输出 | 未授权或 token 过期 | 重新运行 `bypy info` 授权 |
| 找不到文件 | 路径不对 | 先 `bypy list` 确认，网盘文件需在 `我的应用数据/bypy/` |

需要更系统的排查流程？详见 [SKILL.md § 故障排查决策树](SKILL.md#故障排查决策树)。

<br>

---

<br>

## 📚 小白文档三件套

专为零基础用户设计的 Office 文档，从"什么是百度网盘"讲到"343 GB 数据实战"：

<table>
<tr>
<td align="center" width="33%">
  <h3>📄</h3>
  <b>小白操作手册</b><br>
  <sub>Word · 5 章</sub>
</td>
<td align="center" width="33%">
  <h3>📊</h3>
  <b>操作表格</b><br>
  <sub>Excel · 8 Sheet</sub>
</td>
<td align="center" width="33%">
  <h3>📽️</h3>
  <b>演示文稿</b><br>
  <sub>PPT · 19 页</sub>
</td>
</tr>
<tr>
<td valign="top">

1. 认识工具
2. 环境安装
3. 下载数据
4. 上传数据
5. 故障排查

</td>
<td valign="top">

- 安装详解
- 下行操作
- 上行操作
- 故障排查
- 命令速查表
- 实战记录
- 检查清单
- 命令拆解

</td>
<td valign="top">

- 为什么需要这个工具
- 传统方式 vs bypy
- 安装步骤图解
- 下载操作流程
- 上传操作流程
- 常见问题
- 课堂练习题

</td>
</tr>
</table>

> 📥 所有文档在 `小白文档/` 目录下，可直接下载使用。适合课题组培训、新人 onboarding。

<br>

---

<br>

## 🔗 参考链接

| 资源 | 链接 |
|:---|:---|
| bypy 项目 | [github.com/houtianze/bypy](https://github.com/houtianze/bypy) |
| aria2 静态构建 | [github.com/aria2/aria2/releases](https://github.com/aria2/aria2/releases) |
| 完整技术文档 | [SKILL.md](SKILL.md) |

<br>

---

<p align="center">
  <sub>
    🔬 Made for bioinformatics & data science<br>
    ⭐ If this helps you, please star this repo!<br>
    📜 MIT License · Copyright © 2026
  </sub>
</p>
