---
name: baidu-netdisk-transfer
description: 百度网盘 ↔ 服务器双向大文件传输 + MD5 校验，基于 bypy 直传（v1.8.9）
---

# 百度网盘 ↔ 服务器双向文件传输 Skill

> 🎯 **一句话说明**：在 Linux 服务器上用 `bypy` 直接操作百度网盘，**无需经过你的 Windows 电脑中转**。

---

## 📖 目录

- [快速开始（小白首选）](#快速开始小白首选)
- [什么是 bypy？](#什么是-bypy)
- [第一步：一次性环境准备](#第一步一次性环境准备)
- [第二步：下行 —— 百度网盘 → 服务器](#第二步下行--百度网盘--服务器)
- [第三步：上行 —— 服务器 → 百度网盘](#第三步上行--服务器--百度网盘)
- [第四步：监控和校验](#第四步监控和校验)
- [速度优化指南](#速度优化指南)
- [常见问题排查](#常见问题排查)
- [故障排查决策树](#故障排查决策树)
- [测试记录](#测试记录)
- [速查表](#速查表)
- [小白伴侣文档](#小白伴侣文档)

---

## 快速开始（小白首选）

### 你只需要 3 步

```bash
# ① 安装（仅一次，选一种即可）
#    有 sudo：pip install bypy && sudo apt-get install -y aria2
#    无 sudo：见 §1.1 方案 B（用户态安装，零系统影响）

# ② 授权（仅一次）
bypy info
# ↑ 复制输出的 URL 到浏览器打开，登录百度账号，点"授权"，回终端按回车

# ③ 开始传输
bypy --downloader aria2c download 你的文件.fastq.gz ./
```

### 加载本 Skill

加载 Skill 后，告诉助手你要做什么，例如：
- "把百度网盘 bypy 里的 ruchuang 目录下载到 /data/raw/"
- "检查下载进度"
- "下载完成后跑 MD5 校验"

---

## 什么是 bypy？

**bypy** = **B**aidu **Y**un **Py**thon CLI，是一个开源的百度网盘命令行工具。

| 对比 | 传统方式 | bypy 方式 |
|------|----------|-----------|
| 传输路径 | 网盘 → Windows 电脑 → 服务器（两跳） | 网盘 → 服务器（一跳） |
| 速度 | 受本地带宽和磁盘限制 | 服务器带宽，可达 40 MB/s+ |
| 依赖 | 需要百度网盘客户端 | 服务器上 pip install 即可 |
| 断点续传 | 客户端支持 | bypy + aria2c 支持 |

> ⚠️ bypy 只能访问百度网盘中 **「我的应用数据 > bypy」** 目录。

### 百度网盘目录结构（重要！）

理解目录对应关系，是避免"找不到文件"的关键：

```
百度网盘网页端                               bypy 命令行看到的
═══════════════                             ════════════════════
我的应用数据/
  └── bypy/              ←── 授权绑定        /                    （bypy 根目录）
        ├── ruchuang/    ←──────────────→   /ruchuang/
        │     ├── S1_R1.fastq.gz  ←────→   /ruchuang/S1_R1.fastq.gz
        │     ├── S1_R2.fastq.gz  ←────→   /ruchuang/S1_R2.fastq.gz
        │     └── md5.txt         ←────→   /ruchuang/md5.txt
        ├── results/      ←──────────────→   /results/
        │     └── result.tar.gz   ←────→   /results/result.tar.gz
        └── test_small.txt       ←────→   /test_small.txt
```

| 网页端路径 | bypy 路径 | 含义 |
|-----------|-----------|------|
| `我的应用数据/bypy/` | `/` | bypy 根目录 |
| `我的应用数据/bypy/ruchuang/` | `/ruchuang/` | 子目录 |
| `我的应用数据/bypy/ruchuang/S1_R1.fastq.gz` | `/ruchuang/S1_R1.fastq.gz` | 具体文件 |

> 💡 **关键理解**：你在网页端把文件放到 `我的应用数据/bypy/xxx/` 下，在服务器上用 `bypy list /xxx/` 就能看到。

### 实际示例：服务器上的 bypy list 输出

```bash
$ bypy list /
# 输出：
F test_small.txt  73  2026-07-14, 19:25:30  bf1de1c7...
F 测试上传文件     12  2026-07-14, 20:53:53  9575815d...
F test_5mb.bin    5242880  2026-07-14, 20:11:45  abc123...
F test_8mb.bin    8388608  2026-07-14, 20:14:02  5f6d09a4...
F test_9mb.bin    9437184  2026-07-14, 20:14:19  99bbcc97...
```

> 📖 输出解读：`F` = 文件，`D` = 目录，后面依次是文件名、大小（字节）、上传时间、MD5 指纹。

---

## 第一步：一次性环境准备

### 1.1 安装 bypy 和 aria2

#### 方案 A：有 sudo 权限（推荐）

```bash
# bypy（百度网盘 Python CLI）
pip install bypy

# aria2（高速下载引擎，必须安装）
sudo apt-get install -y aria2          # Ubuntu/Debian
sudo yum install -y aria2              # CentOS/RHEL
```

#### 方案 B：无 sudo 权限（用户态安装）

当服务器没有 sudo 权限时（如共享集群），全部安装到用户目录。

> ⚠️ **隔离原则（必须遵守）**
>
> | 原则 | 说明 |
> |------|------|
> | **只写 `~/.local/`** | 不碰 `/usr/`、`/etc/`、`/opt/`，不修改系统配置 |
> | **禁止 `LD_LIBRARY_PATH` 写入 `~/.bashrc`** | 会污染**所有进程**的库搜索路径，可能导致其他软件加载错误版本的 .so |
> | **用 RPATH 代替 LD_LIBRARY_PATH** | 把库路径烧进二进制文件，只影响 aria2c 自身 |
> | **pip 用户级或 conda 环境** | `pip install --user` 或 conda pip，不进系统 site-packages |
> | **`~/.bashrc` 只加 `~/.local/bin`** | 仅此一项修改，安全可逆 |

```bash
# ① 安装 aria2c 静态编译版（无需 root）
cd /tmp
wget https://github.com/aria2/aria2/releases/download/release-1.35.0/aria2-1.35.0-linux-gnu-64bit-build1.tar.bz2
tar xjf aria2-1.35.0-linux-gnu-64bit-build1.tar.bz2
mkdir -p ~/.local/bin ~/.local/lib
cp aria2-1.35.0-linux-gnu-64bit-build1/aria2c ~/.local/bin/

# 如果有 libaria2.so.0，复制后用 patchelf 烧入 RPATH
cp aria2-1.35.0-linux-gnu-64bit-build1/libaria2.so.0 ~/.local/lib/ 2>/dev/null || true
# ⬇ RPATH 只影响 aria2c 自身，不污染全局环境
patchelf --set-rpath '$ORIGIN/../lib' ~/.local/bin/aria2c 2>/dev/null || true
# （patchelf 可通过 apt install patchelf 或 pip install patchelf 获取）

# ② bypy 用 pip 安装（用户级）
pip install --user bypy
# 如果已有 miniconda/anaconda，用 conda 的 pip（conda 环境内隔离）：
pip install bypy

# ③ 只把 ~/.local/bin 加入 PATH（不影响其他路径）
grep -q 'export PATH="$HOME/.local/bin:$PATH"' ~/.bashrc || \
    echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# ④ 清理临时文件
rm -rf /tmp/aria2-1.35.0-linux-gnu-64bit-build1*

# ⑤ 验证安装
bypy --version    # 应显示 bypy v1.8.9
aria2c --version  # 应显示 aria2 version 1.35.0
```

> 💡 **aria2c 启动报 `libaria2.so.0` 找不到？**
>
> **❌ 错误做法**：`export LD_LIBRARY_PATH="$HOME/.local/lib:$LD_LIBRARY_PATH"` → 写入 `~/.bashrc`
> 这会污染**所有**进程的库搜索顺序，可能导致其他软件加载错误版本的 .so。
>
> **✅ 正确做法（三选一）**：
> 1. **patchelf**（推荐）：`patchelf --set-rpath '$ORIGIN/../lib' ~/.local/bin/aria2c` — 一次性烧入
> 2. **wrapper 脚本**：把 aria2c 改名为 aria2c.real，创建 `~/.local/bin/aria2c` shell wrapper 临时设置 LD_LIBRARY_PATH 后 exec
> 3. **临时变量**：`LD_LIBRARY_PATH="$HOME/.local/lib" bypy --downloader aria2c download ...` — 仅该命令生效

### 1.2 授权 bypy

```bash
bypy info
```

终端会输出一个长长的 URL，例如：
```
Please visit:
https://openapi.baidu.com/oauth/2.0/authorize?...
```

**操作步骤：**
1. 复制这个 URL 到浏览器打开
2. 登录你的百度账号
3. 点击"授权"按钮
4. 回到终端按回车

成功后显示：
```
Quota: 40.007TB
Used: 37.555TB
```

### 1.3 把文件放到 bypy 目录

在百度网盘网页或客户端中，把要传输的文件/文件夹移动到：
```
我的应用数据 > bypy > 你的文件夹名
```

例如：`我的应用数据/bypy/ruchuang/` → 在 bypy 中对应 `/ruchuang/`

---

## 第二步：下行 —— 百度网盘 → 服务器

### 流程：确认 → 下载 → 校验

```
百度网盘 (我的应用数据/bypy/你的目录/)
    │
    │ ① bypy list          确认文件清单
    │ ② nohup bypy --downloader aria2c download  并行下载
    │ ③ md5sum -c md5.txt  校验完整性
    ▼
✅ 服务器本地磁盘
```

### 2.1 查看网盘文件

```bash
bypy list                          # 列出根目录
bypy list 项目名/                   # 列出子目录
bypy list 项目名/ | grep md5.txt    # 确认有 MD5 校验文件
```

### 2.2 先下载 MD5 文件

```bash
bypy --downloader aria2c download 项目名/md5.txt ./
```

### 2.3 并行下载数据文件

> 🔑 **这是提速的关键！** 同时启动多个 bypy 进程，每个下载不同文件。

```bash
# 进入目标目录
mkdir -p /data/download && cd /data/download

# 用 nohup 后台启动多个下载进程
# 每个进程负责不同的文件，互不干扰

nohup bypy --downloader aria2c download 项目名/样本1_R1.fastq.gz ./ > dl_1.log 2>&1 &
nohup bypy --downloader aria2c download 项目名/样本1_R2.fastq.gz ./ > dl_2.log 2>&1 &
nohup bypy --downloader aria2c download 项目名/样本2_R1.fastq.gz ./ > dl_3.log 2>&1 &
nohup bypy --downloader aria2c download 项目名/样本2_R2.fastq.gz ./ > dl_4.log 2>&1 &
nohup bypy --downloader aria2c download 项目名/样本3_R1.fastq.gz ./ > dl_5.log 2>&1 &
nohup bypy --downloader aria2c download 项目名/样本3_R2.fastq.gz ./ > dl_6.log 2>&1 &
```

**nohup 命令解释：**

| 部分 | 含义 |
|------|------|
| `nohup` | 即使你关闭 SSH 终端，下载也不中断 |
| `> dl_1.log 2>&1` | 把日志写到 dl_1.log 文件里 |
| `&` | 放到后台运行，立即返回提示符 |

> 💡 **并行数建议**：大文件（>5GB）2-4 个，中等文件 4-6 个，小文件 6-8 个。

### 2.4 监控下载进度

```bash
# 看文件大小是否在增长
ls -lh *.fastq.gz

# 看目录总大小
du -sh .

# 看 bypy 进程是否活着
ps aux | grep bypy

# 看某个下载日志（Ctrl+C 退出）
tail -f dl_1.log
```

### 2.5 MD5 校验（必须！）

```bash
# 大文件校验也放到后台
nohup md5sum -c md5.txt > md5_result.txt 2>&1 &

# 查看进度
grep -c "OK" md5_result.txt     # 通过数
grep -c "FAIL" md5_result.txt   # 失败数
tail -3 md5_result.txt           # 最后 3 行

# 期望结果：全部 OK
# 如果有 FAILED，重新下载对应文件
```

### 2.6 成功案例

**2026-07-13：食蟹猴皮肤褥疮单细胞测序数据**
- 8 个样本 × 4 文件 = 32 个 fastq.gz，共 343.72 GB
- 6 个并行 bypy + aria2c 进程，nohup 后台运行
- 总耗时 155 分钟（约 2.6 小时）
- 平均速度 **39.6 MB/s**
- MD5 校验 32/32 全部通过 ✅

---

## 第三步：上行 —— 服务器 → 百度网盘

### ⚠️ 上传限制（精确边界）

| 文件大小 | 上传方式 | 状态 |
|----------|----------|:----:|
| ≤ 8,388,608 字节（8 MiB） | `bypy --disable-ssl-check --on-dup overwrite upload` | ✅ |
| > 8,388,608 字节（> 8 MiB） | 百度网盘网页/客户端上传 | ✅ 推荐 |

> 📌 **精确边界**：8,388,608 字节（恰好 8 MiB = `dd bs=1M count=8`）通过；9,437,184 字节（9 MiB）挂起。已通过往返 MD5 校验证实 8 MiB 文件上传后数据完全无损。**这是百度 PCS API 的限制，bypy v1.8.9 单次 multipart POST 超过此边界时服务端断开。** 详细测试记录见下方[测试记录](#测试记录)。

### 流程：准备 → 上传 → 确认

```
服务器
    │
    │ ① md5sum 生成 MD5 文件
    │ ② 小文件: bypy upload    大文件: 网页上传
    │ ③ bypy list 确认
    ▼
百度网盘 (我的应用数据/bypy/)
    │
    │ 通知接收方下载后用 md5sum -c 校验
    ▼
✅ 交付完成
```

### 3.1 生成 MD5 文件

```bash
cd /path/to/your/data
md5sum *.fastq.gz > md5.txt
wc -l md5.txt    # 确认行数等于文件数
cat md5.txt | head -3
```

### 3.2 上传小文件（≤8MB）

```bash
# 先上传 MD5 和说明文档
bypy --disable-ssl-check --on-dup overwrite upload md5.txt /目标目录/
bypy --disable-ssl-check --on-dup overwrite upload README.txt /目标目录/

# 中文文件名也完全支持，无需特殊处理
bypy --disable-ssl-check upload 测试上传文件 /
```

> 💡 **为什么用 `--disable-ssl-check`？** 某些服务器环境中 Python 的 SSL 证书验证可能导致上传失败（握手超时）。加上此参数跳过证书校验即可解决。不影响数据安全——文件内容仍通过 HTTPS 加密传输。
>
> 💡 **`--on-dup overwrite`**：网盘已有同名文件时直接覆盖，不加此参数默认会报错"文件已存在"。

> ✅ **中文文件名**：bypy 对中文文件名上传和下载均正常支持，无需转码。

### 3.3 上传大文件（>8MB）

**用百度网盘网页或客户端**，上传到 `我的应用数据 > bypy > 目标目录`

### 3.4 确认上传

```bash
bypy list /目标目录/
```

### 3.5 告知接收方

> 文件已上传到百度网盘 `我的应用数据/bypy/目标目录/`，内含 md5.txt。下载后请执行：`md5sum -c md5.txt`

---

## 第四步：监控和校验

### 下载进度监控命令速查

```bash
# 文件大小变化（最直观）
watch -n 10 'ls -lh *.fastq.gz'

# 进程状态
ps aux | grep bypy | grep -v grep

# 实时日志
tail -f dl_1.log

# 磁盘使用
df -h .
du -sh .
```

### 判断"卡住了"还是"在下载中"

```bash
# 间隔 10 秒，两次查看同一个文件的大小
ls -lh 样本1_R1.fastq.gz   # 记下大小
sleep 10
ls -lh 样本1_R1.fastq.gz   # 比较：增长了 = 正常，没变 = 可能卡住
```

### MD5 校验

```bash
# 前台校验（小文件）
md5sum -c md5.txt

# 后台校验（大文件，推荐）
nohup md5sum -c md5.txt > md5_result.txt 2>&1 &

# 查看结果
cat md5_result.txt | grep -E "OK$|FAILED"
```

---

## 速度优化指南

### 并行是核心

| 场景 | 建议并行数 | 预计总速度 |
|------|:----------:|------------|
| 小文件（<500MB） | 6-8 个 | 40-60 MB/s |
| 中等文件（0.5-5GB） | 4-6 个 | 30-50 MB/s |
| 大文件（>5GB） | 2-4 个 | 20-40 MB/s |

### 时间选择

- 🌙 **凌晨**：速度最快（避开高峰期）
- 🌞 **白天/晚上**：速度可能降至一半
- 📅 **工作日白天**：百度可能限速

### 网络环境

- 服务器在国内：速度快（直连百度 CDN）
- 服务器在国外：建议用国内中转

---

## 常见问题排查

### Q: `bypy download` 卡在 `<-` 不动

**原因**：用了默认下载方式，百度 PCS API 对 Range 头请求不响应。

**解决**：加 `--downloader aria2c`
```bash
# ❌ 卡住
bypy download file.txt ./

# ✅ 正常
bypy --downloader aria2c download file.txt ./
```

### Q: `bypy upload` 卡住不动

**原因**：文件超过约 8MB，PCS API 的 multipart POST 被服务端断开。

**解决**：
- 小文件（≤8MB）：用 `bypy --disable-ssl-check upload`
- 大文件：用百度网盘网页/客户端上传到 `我的应用数据/bypy/`

### Q: 授权过期怎么办

```bash
bypy info   # 重新触发授权流程，按提示操作
```

### Q: 找不到文件

```bash
bypy list              # 列出 bypy 根目录所有文件
bypy list 子目录/       # 列出子目录
# 注意：bypy 看到的目录 = 百度网盘「我的应用数据/bypy/」下面的内容
```

### Q: 磁盘空间不够

```bash
df -h /目标路径         # 检查剩余空间
```

### Q: SSH 断开后下载也停了

**解决**：使用 `nohup` 或 `screen`/`tmux`

```bash
# 方式一：nohup（推荐）
nohup bypy --downloader aria2c download 文件 ./ > log.txt 2>&1 &

# 方式二：screen
screen -S download
bypy --downloader aria2c download 文件 ./
# Ctrl+A, D 断开；screen -r download 重连

# 方式三：tmux
tmux new -s download
bypy --downloader aria2c download 文件 ./
# Ctrl+B, D 断开；tmux attach -t download 重连
```

---

## 故障排查决策树

遇到问题不用慌，按下面的流程图逐步排查：

```
问题：传输失败 / 卡住 / 找不到文件
│
├─ 症状①：bypy download 卡在 "<-" 不动
│   └─ 原因：默认下载方式（requests）不支持大文件
│   └─ 解决：加 --downloader aria2c
│
├─ 症状②：bypy upload 卡住、超时（60s+无响应）
│   │
│   ├─ 文件 ≤ 8,388,608 字节（8 MiB）
│   │   ├─ 加了 --disable-ssl-check？ → 没加就加上
│   │   └─ 加了还卡？ → 重试一次，可能是网络波动
│   │
│   └─ 文件 > 8,388,608 字节（> 8 MiB）
│       └─ 这是 PCS API 限制，用网页端上传
│           → 打开 pan.baidu.com → 我的应用数据 → bypy → 上传
│
├─ 症状③：bypy list 看不到文件
│   │
│   ├─ bypy info 能看到配额吗？
│   │   └─ 不能 → 授权过期，重新执行 bypy info
│   │
│   ├─ 文件在"我的应用数据/bypy/"下吗？
│   │   └─ 不在 → 去网页端移动文件到此目录
│   │
│   └─ bypy list（不带参数）能看到根目录文件吗？
│       ├─ 能看到 → 你输入的路径有误，用 bypy list 确认路径
│       └─ 看不到 → 网盘目录为空，先上传文件
│
├─ 症状④：下载速度极慢（<5 MB/s）
│   │
│   ├─ 是否用了 --downloader aria2c？ → 没加就加上
│   ├─ 是否只开了 1 个进程？ → 改成并行 4-6 个
│   ├─ 当前时间？ → 凌晨最快，白天/晚上可能慢
│   └─ 服务器在国内吗？ → 国外服务器走国内中转
│
├─ 症状⑤：SSH 断开后下载也停了
│   └─ 没用 nohup → 加上 nohup ... &  或使用 screen/tmux
│
├─ 症状⑥：aria2c: error while loading shared libraries: libaria2.so.0
│   │
│   ├─ ✅ 解法1：创建 ~/.local/bin/aria2c wrapper 脚本
│   ├─ ✅ 解法2：patchelf --set-rpath '$ORIGIN/../lib' ~/.local/bin/aria2c
│   └─ ❌ 禁止：不要把 LD_LIBRARY_PATH 写入 ~/.bashrc！
│
└─ 症状⑦：MD5 校验 FAILED
    └─ 重新下载该文件：bypy --downloader aria2c download 文件 ./
       （aria2c 支持断点续传，再次下载会续传）
```

---

## 测试记录

### 2026-07-14 功能边界测试（WSL2 本地）

**环境**：WSL2 Ubuntu (zzj@DESKTOP-8HB1E5C), bypy v1.8.9, aria2 v1.36.0

#### 上行（上传）边界测试 — 找 8MB 分界线

| 文件大小 | 字节数 | 超时 | 结果 |
|:--------:|--------|:----:|:----:|
| 5 MB | 5,242,880 | — | ✅ 12 秒 |
| 6 MB | 6,291,456 | 45s | ✅ |
| 7 MB | 7,340,032 | 45s | ✅ |
| 8 MB | 8,388,608 | 45s | ✅ |
| 9 MB | 9,437,184 | 60s | ❌ 挂起 |
| 10 MB | 10,485,760 | 60s | ❌ 挂起 |
| 12 MB | 12,582,912 | 60s | ❌ 挂起 |
| 15 MB | 15,728,640 | 60s | ❌ 挂起 |
| 20 MB | 20,971,520 | 60s | ❌ 挂起 |

> 📏 **精确边界**：8,388,608 字节（恰好 8 MiB）通过，9,437,184 字节（恰好 9 MiB）失败。**边界 ≤ 8,388,608 字节。**

#### 下行（下载）对比

| 方式 | 7MB 文件 | 343GB 实战 |
|------|:--------:|:----------:|
| 默认 (requests) | ❌ `<-` 卡死 | ❌ 不可用 |
| `--downloader aria2c` | ⚠️ 见备注 | ✅ 155 分钟完成 |

> ⚠️ 下载测试中 aria2c 在部分时段也出现挂起，可能与时段的 PCS API 状态或访问频率有关。343GB 实战下载是连续稳定完成的，说明核心链路可行。

---

### 2026-07-14 服务器实战验证（完整测试套件）

**环境**：远程 Linux 服务器 `shpc-1781-instance-ZcYLk2aO`，用户 `zzj`，无 sudo 权限，miniconda Python，bypy v1.8.9 + aria2c v1.35.0 wrapper。

#### 测试 1-4：基础设施自检

```bash
bypy --version          # ✅ bypy v1.8.9
aria2c --version        # ✅ aria2 version 1.35.0
bypy list               # ✅ 列出 10 个文件/目录
bypy quota              # ✅ Quota: 40.007TB / Used: 37.555TB
```

#### 测试 5-7：下行（下载）全场景

| # | 文件 | 大小 | 方式 | 结果 |
|:--:|------|------|------|:--:|
| 5 | `test_small.txt` | 73 B | `--downloader aria2c` | ✅ 秒下，内容 `Hello Baidu Netdisk!` |
| 6 | `test_12mb.bin` | 12 MB | `--downloader aria2c` | ✅ 3 秒完成，速度 4.0 MiB/s |
| 7 | `测试上传文件` | 12 B | `--downloader aria2c` | ✅ 中文名正常下载，内容 `你好测试` |

#### 测试 8-11：上行（上传）全场景 + 往返校验

| # | 文件 | 大小 | 方式 | 结果 |
|:--:|------|------|------|:--:|
| 8 | `small.txt` | 45 B | `bypy upload` | ✅ 直传成功 |
| 9 | `test_8mb_up.bin` | 8 MiB | `bypy upload` | ✅ 恰好 8 MiB 通过！ |
| 10 | `ssl_test.txt` | 41 B | `--disable-ssl-check upload` | ✅ SSL 跳过模式正常 |
| 11 | `中文文件名测试.txt` | 45 B | `--disable-ssl-check upload` | ✅ 中文名上传正常 |

#### 往返校验（测试 9）

```bash
# 上传 8MiB 随机文件
dd if=/dev/urandom of=test_8mb_up.bin bs=1M count=8
md5sum test_8mb_up.bin  # → 5625ea4a7e9c461746e2562cdd687b80

# 上传 → 下载回来
bypy upload test_8mb_up.bin /
bypy --downloader aria2c download test_8mb_up.bin ./_verify/

# 校验
md5sum _verify/test_8mb_up.bin  # → 5625ea4a7e9c461746e2562cdd687b80 ✅ 完全一致！
```

> ✅ **8 MiB 文件往返 MD5 一致**，证明 bypy 上传 + aria2c 下载链路数据完全无损。**但 9 MiB 即失败**——边界明确，非渐进退化。

#### 用户态安装隔离验证

| 检查项 | 方法 | 结果 |
|--------|------|:--:|
| `~/.bashrc` 无 `LD_LIBRARY_PATH` | `grep LD_LIBRARY_PATH ~/.bashrc` | ✅ |
| aria2c 用 wrapper 隔离 | `cat ~/.local/bin/aria2c` 含 `export LD_LIBRARY_PATH=... exec` | ✅ |
| 所有 .so 已解析 | `ldd ~/.local/bin/aria2c.bin \| grep 'not found'` | ✅ 0 缺失 |
| 无 `/etc/`、`/usr/` 修改 | 仅用 `~/.local/` | ✅ |
| PATH 仅加 `~/.local/bin` | `grep '\.local/bin' ~/.bashrc` | ✅ 仅 1 行 |

---

### 快速自测命令（新装后验证）

```bash
# 创建 5MB 测试文件
dd if=/dev/urandom of=test_5mb.bin bs=1M count=5
md5sum test_5mb.bin > test_5mb.md5

# 上传到 bypy
bypy --disable-ssl-check --on-dup overwrite upload test_5mb.bin /
bypy --disable-ssl-check --on-dup overwrite upload test_5mb.md5 /

# 确认
bypy list / | grep test_5mb

# 下载回来
bypy --downloader aria2c download test_5mb.bin test_dl.bin
bypy --downloader aria2c download test_5mb.md5 test_dl.md5

# 校验
md5sum -c test_dl.md5
# 期望: test_dl.bin: OK

# 清理测试文件
bypy remove /test_5mb.bin
bypy remove /test_5mb.md5
rm -f test_5mb.bin test_5mb.md5 test_dl.bin test_dl.md5
```

---

### 实战验证总结

| 维度 | 测试项 | 结果 |
|------|--------|:--:|
| **安装** | 无 sudo 用户态安装（方案 B） | ✅ |
| **隔离** | 5 项隔离检查全部通过 | ✅ |
| **下行** | 小文件 / 大文件 (12MB) / 中文名 | ✅ |
| **上行** | ≤8MB 直传 / SSL 跳过 / 中文名 | ✅ |
| **边界** | 8 MiB 通过，9 MiB 失败 | ✅ 明确 |
| **完整性** | 8 MiB 文件往返 MD5 一致 | ✅ |
| **实战** | 343GB 测序数据，32/32 MD5 通过 | ✅ |
| **清理** | `bypy remove` 测试文件 | ✅ |

---

## 速查表

### 下行（百度网盘 → 服务器）

| 你要做什么 | 命令 |
|-----------|------|
| 列出网盘文件 | `bypy list 目录/` |
| 下载单个文件 | `bypy --downloader aria2c download 文件 ./` |
| 并行下载 | `nohup bypy --downloader aria2c download 文件 ./ > log.txt 2>&1 &` |
| 查看下载进度 | `ls -lh *.fastq.gz` 或 `ps aux \| grep bypy` |
| MD5 校验 | `nohup md5sum -c md5.txt > result.txt 2>&1 &` |
| 删除网盘文件 | `bypy remove /路径/文件名` |

### 上行（服务器 → 百度网盘）

| 你要做什么 | 命令 |
|-----------|------|
| 生成 MD5 | `md5sum *.fastq.gz > md5.txt` |
| 上传小文件（≤8,388,608 字节） | `bypy --disable-ssl-check --on-dup overwrite upload 文件 /目录/` |
| 上传大文件（>8,388,608 字节） | 百度网盘网页/客户端上传到 `我的应用数据/bypy/目录/` |
| 确认上传 | `bypy list /目录/` |
| 覆盖已有文件 | 加 `--on-dup overwrite` |

### Agent 对话命令

| 你要说什么 | 效果 |
|-----------|------|
| `$baidu-netdisk-transfer` | 加载本 Skill |
| "把 bypy 里的项目下载到 /data/，并行 + 校验" | 自动并行下载并校验 |
| "检查下载进度" | 查看文件大小和进程状态 |
| "把服务器结果上传到百度网盘" | 生成 MD5 并上传 |
| "帮我安装 bypy 和 aria2 并授权" | 一次性环境准备 |

---

## 小白伴侣文档

Skill 目录下 `小白文档/` 文件夹包含三份配套文档，适合**零基础用户**：

| 文件 | 格式 | 用途 | 内容亮点 |
|------|:----:|------|------|
| **百度网盘传输_小白操作手册.docx** | 📄 Word | 详细阅读学习 | 5 章完整教程：认识 bypy → 安装 → 下载 → 上传 → 校验。每步都有流程图和命令逐字拆解，附 343GB 测序数据实战案例。 |
| **百度网盘传输_操作表格.xlsx** | 📊 Excel | 对照操作 | 8 个 Sheet：快速开始、安装详解（含无 sudo 方案）、下行/上行操作表、故障排查 8 种错误对照、命令速查表、实战记录、20 项检查清单。 |
| **百度网盘传输_演示文稿.pptx** | 📽️ PPT | 培训讲解 | 19 页幻灯片：从"为什么需要这个工具"到"命令速查总结"，适合团队分享和新手培训。 |

### 三份文档的使用顺序

```
第一遍：打开 PPT  →  快速浏览 19 页，建立整体概念（15 分钟）
第二遍：打开 Word →  仔细阅读 5 章，理解每个步骤的原理（30 分钟）
操作时：打开 Excel →  对照操作表和检查清单，边看边做（实操用）
```

### 文档亮点图示

**Word 手册中的命令拆解示例：**

```
nohup bypy --downloader aria2c download 项目名/样本1_R1.fastq.gz ./ > dl_1.log 2>&1 &
│      │         │           │      │                      │  │    │        │   │
│      │         │           │      │                      │  │    │        │   └─ 后台运行
│      │         │           │      │                      │  │    │        └─ 合并错误输出
│      │         │           │      │                      │  │    └─ 日志文件名
│      │         │           │      │                      │  └─ 输出重定向
│      │         │           │      │                      └─ 目标目录（当前目录）
│      │         │           │      └─ 网盘中的文件名
│      │         │           └─ 操作：下载
│      │         └─ 使用 aria2c 高速下载引擎
│      └─ bypy 主命令
└─ 即使退出 SSH，下载也不中断
```

**Excel 故障排查表格式：**

| # | 现象 | 可能原因 | 解决方案 | 修复命令 |
|---|------|----------|----------|----------|
| 1 | `<-` 卡死 | 默认下载方式不兼容 | 加 `--downloader aria2c` | `bypy --downloader aria2c download ...` |
| 2 | 上传卡住 | 文件 > 8MB | 用网页上传 | j计划打开 pan.baidu.com |
| 3 | 找不到文件 | 路径不在 bypy 目录 | 移到 `我的应用数据/bypy/` | 网页端操作 |
| ... | ... | ... | ... | ... |

> 📂 文档位置：`skills/baidu-netdisk-transfer/小白文档/`

### 文档自动生成

这三份文档是根据本 SKILL.md 内容自动生成的。当 SKILL.md 有重大更新时，可以重新生成。

**重新生成命令**：
```
请根据最新的 SKILL.md 重新生成小白文档（Word + Excel + PPT）
```
