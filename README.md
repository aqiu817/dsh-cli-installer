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

## 🔍 它会做什么（行为清单）

一行式命令确实不透明。这里是 `install.ps1` 实际会执行的全部动作，逐条对应源码，
方便你判断是否符合预期。默认选项全程**用户级、免管理员**，不写注册表系统区（`HKLM`）、
不创建服务、不装 EXE。

### 写入的文件

| 位置 | 内容 |
|------|------|
| `%LOCALAPPDATA%\DeepSeekHarness\` | `launch-dsh.cmd`（启动器）、`dsh.ico`（图标）、`dsh-tray.ps1`（托盘控制）、`stop-dsh.cmd` |
| `%LOCALAPPDATA%\DeepSeekHarness\dsh-web.log` | dsh 服务的 stdout 日志。**这是启动器能自动打开界面的关键**：dsh 每次启动都会打印一个带一次性 token 的 URL，启动器从这份日志里读出它再交给 Edge 打开。启动服务时会被清空重写，停止服务后不会自动删除 |
| `%LOCALAPPDATA%\DeepSeekHarness\EdgeProfile\` | Edge 应用模式的独立用户数据目录（不污染你的日常浏览器配置） |
| `%LOCALAPPDATA%\Programs\nodejs\` | Node.js（仅当选择"用户级安装"） |
| `%APPDATA%\npm\` | npm 全局包，即 `@deepseek-ai/dsh`（仅当未选 `-SkipNpm`） |
| `%TEMP%\` | 下载的中转文件，安装完即删除 |

### 创建的快捷方式

- 桌面 `DeepSeek Harness.lnk`
- 开始菜单 `DeepSeek Harness.lnk`
- `%LOCALAPPDATA%\DeepSeekHarness\DeepSeekHarnessEdge.lnk`（供托盘程序使用）
- 若已存在旧版桌面 `停止 DSH.lnk`，会被删除（功能已并入托盘）

### 修改的环境变量

| 变量 | 范围 | 说明 |
|------|------|------|
| `Path`（用户级） | `HKCU\Environment` | 仅在缺失时追加 `%LOCALAPPDATA%\Programs\nodejs` 与 `%APPDATA%\npm` |
| `npm config set registry` | `%APPDATA%\npmrc` | 改成你在菜单里选的源。这会影响之后**所有**全局 npm 安装，介意的话手动改回 `https://registry.npmjs.org` |

不会修改系统级 `Path`、不会写 `HKLM`、不会创建 Windows 服务。
（可选的"系统级安装 Node"会走 MSI 并弹出 UAC 提权窗口；不选则完全免管理员。）

### 网络请求

- npm registry：默认淘宝 npmmirror，也可选官方源 / 腾讯云 / 华为云 / 自定义
- Node 二进制：`cdn.npmmirror.com/binaries/node`（国内源）或 `nodejs.org/dist`（官方源）
- 运行时监听 `http://127.0.0.1:3080`，仅本机回环地址

### 后台进程

启动后跑两个进程，均隐藏窗口：`dsh web --no-open`（服务）与托盘程序。
关闭 Edge 应用窗口或从托盘右键"停止服务"，托盘会 `taskkill` 掉 3080 端口上的进程，不留残留。

### 认证 URL

dsh web 给界面加了一道认证：裸的 `http://127.0.0.1:3080/` 会返回 401，页面提示
*authentication required; reopen the URL printed by dsh web*。真实可访问的 URL 形如
`http://127.0.0.1:3080/?token=...`，由 dsh 在**每次启动时生成并打印到 stdout**，
不落盘、不持久化。启动器因此把 dsh 的 stdout 重定向到 `dsh-web.log`，再从中解析出
带 token 的 URL 写入快捷方式 —— 这就是启动器与托盘都能直接打开界面的原因。

若解析失败（日志缺失、服务在我们启动前就已运行），会退回裸 URL 并明确提示。

## ✅ 校验下载文件

命令是明文，下载过程也走 TLS，但仍建议校验哈希。`%TEMP%\dsh-install.ps1` 就是脚本本身：

```powershell
Get-FileHash "$env:TEMP\dsh-install.ps1" -Algorithm SHA256
```

> **当前 `src/install.ps1`（main 分支）：**
>
> ```
> 8035519D16397C0BE3AA6F9CCD1988E7D65D007CA77FC922664045533CC51461
> ```
>
> 每次更新安装脚本都会重新发布此值。用 PowerShell 的 `Get-FileHash` 和 Linux 的
> `sha256sum` 对同一文件算出的是**同一个值**（只是一个大写一个小写），任选其一比对即可。
>
> 下载脚本与 git 仓库内容一致，是确认「我拿到的就是仓库里那一份」的最简单方式 ——
> 比检查命令本身更能证明内容没被中间环节改动。

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

**页面提示 `dsh web authentication required; reopen the URL printed by dsh web`**

说明你打开的是**不带 token 的裸 URL**。dsh 每次启动都生成一次性 token 并打印到 stdout
（不落盘），而旧版本安装器把快捷方式写死成了裸 `http://127.0.0.1:3080`，所以会 401。
这是安装器的问题，不是你 dsh 配置的问题，也不是需要关掉的校验。

重新跑一次安装器即可 —— 它会重写桌面/开始菜单快捷方式，改为使用 dsh 实际打印的带 token URL：

```powershell
$tmp = Join-Path $env:TEMP "dsh-install.ps1"; (New-Object Net.WebClient).DownloadFile("https://cdn.jsdelivr.net/gh/aqiu817/dsh-cli-installer@main/src/install.ps1", $tmp); Invoke-Expression (Get-Content $tmp -Raw -Encoding UTF8)
```

应急办法：手动运行 `dsh web`（去掉 `--no-open`），复制终端里打印出的完整 URL 直接打开。

## 📄 协议

[MIT License](LICENSE)
