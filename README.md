<p align="center">
  <img src="https://img.shields.io/badge/bypy-v1.8.9-blue?style=flat-square&logo=python" alt="bypy">
  <img src="https://img.shields.io/badge/aria2c-v1.35.0-green?style=flat-square&logo=download" alt="aria2c">
  <img src="https://img.shields.io/badge/platform-Linux%20x86__64-lightgrey?style=flat-square&logo=linux" alt="platform">
  <img src="https://img.shields.io/badge/license-MIT-yellow?style=flat-square" alt="license">
  <img src="https://img.shields.io/badge/tested-343GB%20%E2%9C%94-success?style=flat-square" alt="tested">
  <img src="https://img.shields.io/badge/sudo-not%20required-brightgreen?style=flat-square" alt="no sudo">
</p>

<h1 align="center">🔁 百度网盘 ↔ 服务器 双向传输</h1>

<p align="center">
  <b>一跳直传 · 无需中转 · 无 sudo 安装 · MD5 验证</b><br>
  <sub>基于 bypy v1.8.9 + aria2c v1.35.0 · 343 GB 实战 32/32 全通过</sub>
</p>

---

## 🧭 为什么需要这个 Skill？

```
❌ 传统方式（两跳，慢）           ✅ 本 Skill（一跳，快）

  百度网盘                          百度网盘
     │                                  │
     │ 下载到本地                        │ bypy 直传
     ▼                                  │
  💻 你的 Windows 电脑                   │ 服务器带宽 40 MB/s+
     │                                  │ 断点续传 · 并行加速
     │ scp/rsync 上传                    ▼
     ▼                             🖥️ Linux 服务器
  🖥️ Linux 服务器
```

| 对比维度 | 传统方式 | 本 Skill |
|----------|:--------:|:--------:|
| 传输跳数 | **两跳**（网盘→本机→服务器） | **一跳**（网盘→服务器） |
| 速度瓶颈 | 家庭宽带 (10-30 MB/s) | 机房带宽 (40+ MB/s) |
| 占用本地 | 需本地磁盘中转 | 零占用 |
| sudo 权限 | 不需要 | 不需要 |
| 断点续传 | 看客户端心情 | ✅ aria2c 原生支持 |
| 并行下载 | 手动 | ✅ bypy 并行 + aria2c 多线程 |
| MD5 校验 | 需手动 | ✅ 内置 md5sum |

---

## 📦 目录

```
baidu-netdisk-transfer/
├── README.md                          ← 你在这里
├── SKILL.md                           ← 🔧 完整技术文档（630+ 行）
├── .gitignore
└── 小白文档/                          ← 📚 零基础三件套
    ├── 百度网盘传输_小白操作手册.docx   # Word · 5 章完整手册
    ├── 百度网盘传输_操作表格.xlsx       # Excel · 8 Sheet 速查
    └── 百度网盘传输_演示文稿.pptx       # PPT · 19 页培训演示
```

---

## ⚡ 快速开始

### ① 安装（仅一次）

```bash
# ── bypy ──
pip install --user bypy==1.8.9

# ── aria2c（静态编译，无 root）──
wget -q https://github.com/aria2/aria2/releases/download/release-1.35.0/aria2-1.35.0-linux-gnu-64bit-build1.tar.bz2
tar xjf aria2-*.tar.bz2 && mkdir -p ~/.local/bin && cp aria2-*/aria2c ~/.local/bin/
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc
```

> 📌 **零系统影响**：全部安装在 `~/.local/`，不修改系统文件，不影响其他软件。

### ② 授权（仅一次）

```bash
bypy info
# → 浏览器打开输出的链接 → 登录百度账号 → 粘贴授权码 → 回车
```

### ③ 开始传输！

```bash
# 下行：网盘 → 服务器
bypy --downloader aria2c download /网盘目录/你的文件.fastq.gz ./

# 上行：服务器 → 网盘（≤8 MiB）
bypy --disable-ssl-check --on-dup overwrite upload ./结果.tar.gz /网盘目录/
```

---

## 📊 性能基准

| 场景 | 数据量 | 文件数 | 耗时 | 速度 | MD5 |
|------|:------:|:------:|:----:|:----:|:---:|
| 🐵 食蟹猴单细胞测序 | 343.72 GB | 32 | 155 min | **39.6 MB/s** | 32/32 ✅ |
| 🧪 边界测试 (8 MiB) | 8 MB | 1 | < 5 sec | 2.3 MB/s | ✅ |

---

## 🔧 命令速查

### 下行（网盘 → 服务器）

| 操作 | 命令 |
|------|------|
| 查看网盘文件 | `bypy list` |
| 下载单文件 | `bypy --downloader aria2c download /路径/文件 ./` |
| 并行下载（后台） | `nohup bypy --downloader aria2c download /路径/文件 ./ > dl.log 2>&1 &` |
| 监控进度 | `watch -n 10 'ls -lh *.fastq.gz'` |
| MD5 校验 | `nohup md5sum -c md5.txt > result.txt 2>&1 &` |
| 查看配额 | `bypy quota` |

### 上行（服务器 → 网盘）

| 操作 | 命令 |
|------|------|
| 生成 MD5 | `md5sum *.fastq.gz > md5.txt` |
| 上传 ≤8 MiB | `bypy --disable-ssl-check --on-dup overwrite upload ./文件 /路径/` |
| 上传 >8 MiB | 网页/客户端上传到 `我的应用数据/bypy/` |
| 确认上传 | `bypy list /路径/` |
| 删除网盘文件 | `bypy remove /路径/文件` |

---

## 🛡️ 隔离原则（5 条铁律）

| # | 原则 | 说明 |
|:--:|------|------|
| 1 | 只写 `~/.local/` | 不碰 `/usr/`、`/etc/`、`/opt/` |
| 2 | **禁止** `LD_LIBRARY_PATH` 入 `.bashrc` | 会污染所有进程的库搜索 |
| 3 | 用 patchelf RPATH 或 wrapper | 只影响 aria2c 自身 |
| 4 | pip `--user` 或 conda 环境 | 不进系统 site-packages |
| 5 | `.bashrc` 只加 `~/.local/bin` | 唯一的一行 PATH 修改 |

---

## 🩺 故障排查

| 症状 | 可能原因 | 解决 |
|------|----------|------|
| `bypy: command not found` | `~/.local/bin` 不在 PATH | `source ~/.bashrc` |
| `aria2c: libaria2.so.0` | 共享库未找到 | 用 patchelf 设 RPATH 或 wrapper 脚本 |
| 上传 >8 MiB 挂起 | 百度 PCS API 限制 | 改用网页/客户端上传 |
| SSL 错误 | 证书验证问题 | 加 `--disable-ssl-check` |
| 下载特别慢 | 单线程 | 启用并行下载 `--downloader aria2c` |
| `bypy info` 无输出 | 未授权 | 重新运行 `bypy info` 授权 |
| 找不到文件 | 路径不对 | 确认文件在 `我的应用数据/bypy/` 下 |

---

## 📚 小白文档

三份 Office 文档，专为零基础用户设计：

| 文档 | 格式 | 内容 |
|------|:----:|------|
| 📄 **小白操作手册** | Word | 5 章：认识工具 → 安装 → 下载 → 上传 → 故障排查 |
| 📊 **操作表格** | Excel | 8 Sheet：安装详解 · 下行操作 · 上行操作 · 故障排查 · 速查表 · 实战记录 · 检查清单 · 命令拆解 |
| 📽️ **演示文稿** | PPT | 19 页幻灯片：适合课题组培训讲解 |

> 📥 文档在 `小白文档/` 目录下，可直接下载使用。

---

## 🔗 相关链接

- [bypy GitHub](https://github.com/houtianze/bypy) — 百度网盘 Python CLI
- [aria2 静态构建](https://github.com/aria2/aria2/releases) — 多线程下载引擎
- [WispTerm](https://wispterm.com) — 本 Skill 运行平台

---

<p align="center">
  <sub>Made with ❤️ for bioinformatics & data science · MIT License</sub>
</p>
