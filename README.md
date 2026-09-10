# dsh-cli-installer

DeepSeek Harness（`@deepseek-ai/dsh`）一键安装器 —— 纯命令行，单文件，无需管理员权限，无需 EXE。

## ✨ 一键安装

在 **PowerShell** 中复制粘贴执行（无需下载文件）：

```powershell
$tmp = Join-Path $env:TEMP "dsh-install.ps1"; (New-Object Net.WebClient).DownloadFile("https://cdn.jsdelivr.net/gh/aqiu817/dsh-cli-installer@main/src/install.ps1", $tmp); Invoke-Expression (Get-Content $tmp -Raw -Encoding UTF8)
```

国内网络也可使用 GitHub 原始源（较慢，或需代理）：

```powershell
$tmp = Join-Path $env:TEMP "dsh-install.ps1"; (New-Object Net.WebClient).DownloadFile("https://raw.githubusercontent.com/aqiu817/dsh-cli-installer/main/src/install.ps1", $tmp); Invoke-Expression (Get-Content $tmp -Raw -Encoding UTF8)
```

> 全程交互式菜单，直接回车即可使用默认推荐选项（自动安装 Node.js + 淘宝镜像源）。
>
> ⚠️ 不要用 `irm ... | iex` 的写法。`install.ps1` 是 UTF-8 with BOM（PowerShell 5.1
> `-File` 执行所必需），而 `irm` 按 `charset=utf-8` 解码时会把 BOM 转成 `U+FEFF` 字符并保留在
> 字符串开头，`iex` 解析器在首个 token 遇到该字符会解析失败，报
> `一元运算符"--"后面缺少表达式`、`意外的标记"param"` 等错误。上面的写法先下载为文件、
> 再以 `-Encoding UTF8` 读取（PowerShell 会自动剥离 BOM），因此同时兼容 PS 5.1 与 PS 7。
> PowerShell 7 (`pwsh`) 用户若已安装，也可直接 `irm ... | iex`。
>
> 💡 想保留安装脚本以便重复执行或排查：把 `$tmp` 换成你想保存的路径即可。

## 🚀 功能

- **Node.js 环境切换**：自动安装（用户级，免管理员）/ 系统级 / 使用已有 / 跳过
- **npm 数据源切换**：官方 / 淘宝 npmmirror / 腾讯云 / 华为云 / 自定义
- **完整安装 dsh**：自动识别 Node.js 环境并安装 `@deepseek-ai/dsh`，并对镜像同步延迟自动重试
- **一键启动**：创建桌面 + 开始菜单快捷方式，Edge 应用模式打开界面
- **系统托盘控制**：托盘鲸鱼图标，右键可打开界面 / 启动服务 / 停止服务
- **自动清理**：关闭浏览器窗口后自动停止后台服务，不留残留进程

## 🖥️ 使用

安装完成后，双击桌面「DeepSeek Harness」快捷方式即可启动。

手动启动：

```powershell
dsh web
```

手动停止（或使用托盘右键菜单）：

```powershell
netstat -ano | findstr ":3080" | findstr "LISTENING"
# 找到 PID 后
taskkill /f /pid <PID>
```

## 📦 命令行参数

| 参数 | 说明 |
|------|------|
| `-Silent` | 静默模式，使用默认选项 |
| `-SkipNpm` | 跳过 npm 安装（仅部署启动器） |
| `-DryRun` | 演练模式，只演示不实际执行 |

## ❓ 常见问题

**`npm error code ETARGET` / `No matching version found for @deepseek-ai/...`**

dsh 由 60+ 个 `@deepseek-ai/*` 子包组成。上游发版后，各镜像（淘宝 npmmirror / 腾讯云 / 华为云）
的包元数据是异步同步的，短时间内可能出现「版本列表已更新、但部分子包的版本条目尚未可解析」的
不一致状态，npm 就会报 ETARGET。**这是镜像层的瞬时故障，不是你的环境有问题，也不取决于你本地。**

安装器已内置自动重试（最多 4 次，间隔 5 / 10 / 20 秒，指数退避）。若仍失败，按顺序尝试：

1. 稍后再跑一次（通常几分钟到几小时内镜像同步完成）
2. 换数据源 —— 菜单里选「自定义地址」填官方源 `https://registry.npmjs.org`
3. 手动执行：`npm install -g @deepseek-ai/dsh --registry=https://registry.npmmirror.com`

> 说明：安装器只对「可重试」的错误自动重试（ETARGET、E404、E401/E403、E5xx、各类网络与证书错误）。
> 对重试无意义的错误（如 `EACCES` 权限不足、`ENOENT` 命令缺失）会立即失败并提示，避免无效等待。

## 📄 协议

[MIT License](LICENSE)
