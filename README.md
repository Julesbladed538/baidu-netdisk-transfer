# 百度网盘 ↔ 服务器双向传输 Skill

基于 **bypy v1.8.9 + aria2c v1.35.0** 的百度网盘与服务器双向文件传输助手。

## 特性

- **一跳直传**：服务器 ↔ 百度网盘，不经过本地中转
- **双向传输**：下行（网盘→服务器）+ 上行（服务器→网盘）
- **无 sudo**：全部安装在 ~/.local/，不影响系统其他软件
- **中文文件名**：完美支持，无需转码
- **大文件支持**：下行不限大小（aria2c 加速），上行 ≤8 MiB 直传，>8 MiB 网页上传
- **MD5 校验**：往返传输后自动 MD5 验证
- **实战验证**：343 GB 测序数据 32/32 MD5 通过

## 目录结构

```
baidu-netdisk-transfer/
├── SKILL.md                          # 完整技术文档（628 行）
├── README.md                         # 本文件
├── 小白文档/                         # 零基础操作手册
│   ├── 百度网盘传输_小白操作手册.docx  # Word 详细手册
│   ├── 百度网盘传输_操作表格.xlsx      # Excel 操作表格
│   └── 百度网盘传输_演示文稿.pptx      # PPT 培训演示
└── .gitignore
```

## 快速开始

### 安装（无 sudo，不影响系统）

```bash
# bypy
pip install --user bypy==1.8.9

# aria2c（静态编译到 ~/.local/bin）
wget https://github.com/q3aql/aria2-static-builds/releases/download/v1.35.0/aria2-1.35.0-linux-gnu-64bit-build1.tar.bz2
tar xf aria2-*.tar.bz2
cp aria2-*/aria2c ~/.local/bin/
```

### 授权（仅一次）

```bash
bypy info
# 浏览器打开输出的链接 → 登录百度账号 → 粘贴授权码
```

### 下行（网盘 → 服务器）

```bash
bypy --downloader aria2c download /网盘目录/ ./本地目录/
```

### 上行（服务器 → 网盘）

```bash
# ≤8 MiB：直传
bypy --disable-ssl-check upload ./文件 /网盘目录/

# >8 MiB：网页上传（百度限制）
```

## 小白文档

三份 Office 文档包含：
- **Word**：5 章完整手册，从认识工具到故障排查
- **Excel**：8 个 Sheet，含速查表、检查清单、实战记录
- **PPT**：19 页演示文稿，适合培训讲解

## 相关链接

- [bypy GitHub](https://github.com/houtianze/bypy)
- [aria2 静态构建](https://github.com/q3aql/aria2-static-builds)

## 许可

MIT License
