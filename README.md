# dsh-cli-installer

DeepSeek Harness（`@deepseek-ai/dsh`）一键安装器 —— 纯命令行，单文件，无需管理员权限，无需 EXE。

## ✨ 一键安装

在 **PowerShell** 中复制粘贴执行（无需下载文件）：

```powershell
irm https://raw.githubusercontent.com/aqiu817/dsh-cli-installer/main/src/install.ps1 | iex
```

国内网络推荐使用 jsDelivr CDN（更快）：

```powershell
irm https://cdn.jsdelivr.net/gh/aqiu817/dsh-cli-installer@main/src/install.ps1 | iex
```

> 全程交互式菜单，直接回车即可使用默认推荐选项（自动安装 Node.js + 淘宝镜像源）。

## 🚀 功能

- **Node.js 环境切换**：自动安装（用户级，免管理员）/ 系统级 / 使用已有 / 跳过
- **npm 数据源切换**：官方 / 淘宝 npmmirror / 腾讯云 / 华为云 / 自定义
- **完整安装 dsh**：自动识别 Node.js 环境并安装 `@deepseek-ai/dsh`
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

## 📄 协议

[MIT License](LICENSE)
