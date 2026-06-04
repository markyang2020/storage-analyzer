# Storage Analyzer

macOS / Windows 只读磁盘存储分析 Skill。它会扫描电脑里常见的磁盘占用大户，让 Agent 根据扫描结果做清理分级，并生成一份交互式 HTML 报告。

## 功能简介

- 只读扫描常见磁盘热点目录，不主动修改、移动或删除文件。
- 把可清理项分成三类：
  - 绿灯：缓存、临时文件，通常可以重新生成。
  - 黄灯：用户数据或应用托管数据，需要人工判断。
  - 红灯：应用核心数据、系统相关内容，不建议直接删除。
- 生成浏览器报告，包含磁盘总览、占用 Top 5、清理优先级、清理命令和可选的本地操作按钮。
- 只依赖 Python 3 标准库，不需要 `pip install`。

## 目录结构

```text
.
├── SKILL.md
├── assets/
│   └── report_template.html
├── references/
│   ├── macos.md
│   └── windows.md
└── scripts/
    ├── build_report.py
    ├── scan.py
    └── server.py
```

## 作为 Skill 安装

在支持 `SKILL.md` 的 Agent 里，直接让它安装这个仓库：

```text
安装这个 skill：https://github.com/markyang2020/storage-analyzer
```

安装后可以用自然语言触发，例如：

```text
帮我看看电脑存储
磁盘空间不够，分析一下哪些东西占地方
C 盘满了，帮我看看怎么清理
check my storage usage
```

## 手动运行

也可以在仓库根目录手动运行脚本。

### 1. 扫描磁盘占用

macOS：

```bash
python3 scripts/scan.py > /tmp/storage_scan.json
```

Windows PowerShell：

```powershell
python scripts\scan.py > $env:TEMP\storage_scan.json
```

扫描过程是只读的，只读取目录大小和基础元信息。

### 2. 生成分析 JSON

`scan.py` 只输出原始扫描数据。Agent 需要读取 `/tmp/storage_scan.json`，按照 [SKILL.md](./SKILL.md) 的规则分析，并生成包含 `top5`、`green`、`yellow`、`red`、`summary` 等字段的分析 JSON。

macOS 可以保存为：

```text
/tmp/storage_analysis.json
```

Windows 可以保存为：

```text
%TEMP%\storage_analysis.json
```

### 3. 打开交互式报告

如果需要带本地操作按钮的完整网页报告：

```bash
python3 scripts/server.py /tmp/storage_analysis.json
```

服务会绑定到 `127.0.0.1`，使用随机端口和随机 token，并根据分析 JSON 里的白名单校验可操作路径。

### 4. 生成静态 HTML

如果只需要一份可分享、可留存的静态报告：

```bash
python3 scripts/build_report.py /tmp/storage_analysis.json ~/Desktop/storage-report.html
```

静态 HTML 不包含本地删除、移废纸篓、打开文件夹等操作能力。

## 安全说明

- 扫描阶段只读，不会删除文件。
- 网页操作按钮只会在 Agent 明确提供白名单路径时出现。
- 绿灯项可以出现「移到废纸篓」和「直接删除」操作。
- 黄灯项最多出现「在文件管理器打开」，只有确认安全的子路径才会出现「移到废纸篓」。
- 红灯项不会出现删除按钮。
- 浏览器里每次执行清理动作前都需要二次确认。

## 平台状态

- macOS：已实现 home、Library、Applications、Downloads、开发缓存等常见位置扫描。
- Windows：已实现标准库扫描和回收站能力，但建议在真实 Windows 环境验证后再使用一键清理。

## 许可证

MIT。见 [LICENSE](./LICENSE)。
