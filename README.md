# Storage Analyzer

macOS / Windows read-only storage analyzer skill. It scans disk usage, asks an agent to classify cleanup candidates, and renders an interactive HTML report with a three-tier cleanup plan.

This repository is an extracted copy of `storage-analyzer` from [KKKKhazix/khazix-skills](https://github.com/KKKKhazix/khazix-skills/tree/main/storage-analyzer). The original project is licensed under MIT; the license is preserved in [LICENSE](./LICENSE).

## What It Does

- Scans common disk hot spots with read-only operations.
- Groups cleanup candidates into:
  - Green: cache/temp files that can usually be regenerated.
  - Yellow: user data or app-managed data that needs manual review.
  - Red: risky items such as app cores or data that should not be deleted directly.
- Generates a browser report with disk overview, Top 5 usage, priority suggestions, cleanup commands, and optional guarded local actions.
- Uses only Python 3 standard library; no `pip install` is required.

## Repository Layout

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

## Install As A Skill

In an agent that supports `SKILL.md`, ask it to install this repository:

```text
Install this skill: https://github.com/markyang2020/storage-analyzer
```

After installation, trigger it with natural language such as:

```text
帮我看看电脑存储
磁盘空间不够，分析一下哪些东西占地方
check my storage usage
C drive is full, help me analyze it
```

## Manual Usage

You can also run the scripts manually from the repository root.

### 1. Scan Disk Usage

macOS / Linux shell:

```bash
python3 scripts/scan.py > /tmp/storage_scan.json
```

Windows PowerShell:

```powershell
python scripts\scan.py > $env:TEMP\storage_scan.json
```

The scanner is read-only. It only reads metadata and directory sizes.

### 2. Create Analysis JSON

`scan.py` only produces raw disk data. An agent should read `/tmp/storage_scan.json`, follow [SKILL.md](./SKILL.md), and produce an analysis JSON with sections such as `top5`, `green`, `yellow`, `red`, and `summary`.

Save that agent-produced file as:

```text
/tmp/storage_analysis.json
```

On Windows, use a normal temporary path such as:

```text
%TEMP%\storage_analysis.json
```

### 3. Open Interactive Report

For the full local web report with guarded buttons:

```bash
python3 scripts/server.py /tmp/storage_analysis.json
```

The server binds to `127.0.0.1`, uses a random port and token, and validates paths against the report allowlist.

### 4. Generate Static HTML

For a shareable report file without local delete/open actions:

```bash
python3 scripts/build_report.py /tmp/storage_analysis.json ~/Desktop/storage-report.html
```

Then open the generated HTML file in a browser.

## Safety Notes

- The scan step is read-only.
- Cleanup actions only appear when the agent provides explicit allowlisted paths.
- Green items may show move-to-trash and direct-delete actions.
- Yellow items may show open-in-file-manager and, only for safe subpaths, move-to-trash actions.
- Red items never expose delete buttons.
- Review every cleanup action before confirming it in the browser.

## Platform Status

- macOS: implemented for common home, Library, application, download, and development-cache locations.
- Windows: implemented with standard-library scanning and recycle-bin support, but should be validated on a real Windows machine before relying on one-click cleanup.

## License

MIT. See [LICENSE](./LICENSE).
