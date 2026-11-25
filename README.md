# 🌙 合盖不休眠（v1.7.0 纯 Swift 重写版）

<p align="center">
  <img src="Resources/AppIcon.jpg" width="128" height="128" alt="合盖不休眠图标">
</p>

<p align="center">
  <strong>MacBook 合盖后保持运行，自动关闭屏幕省电</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/platform-macOS%20(Universal)-blue" alt="platform">
  <img src="https://img.shields.io/badge/version-v1.7.0--prerelease-green" alt="version">
  <img src="https://img.shields.io/badge/license-MIT-orange" alt="license">
</p>

> 💡 **关于本仓库与上游关系**
>
> 本仓库基于原项目 [1073020318-cell/nosleep-mac](https://github.com/1073020318-cell/nosleep-mac)（MIT 协议）进行重构与增强，**全部代码改动已通过 [PR #1](https://github.com/1073020318-cell/nosleep-mac/pull/1) 提交给上游原作者等待合并**。
>
> 如果你想在上游合并发布之前提前体验 `v1.7.0`（纯 Swift 原生重写、收紧免密权限、低电量保护、重启自动恢复等），可以直接前往本仓库的 **[Releases](https://github.com/H4Ha/nosleep-mac/releases)** 下载预编译的通用版 DMG 安装包。官方正式版请仍以原仓库为准。

---

## 🛠️ 为什么做这个版本（相对原版 v1.6.0 的核心改进）

日常重度使用原版的过程中遇到了一些安全、耗电和稳定性痛点，于是把全部逻辑重新整理并合并为单文件纯 Swift 原生应用（`Sources/menulet.swift`），不再依赖 bash 轮询脚本和 Python：

- **收紧免密安全权限**：旧版免密规则允许本机**任意账户**免密执行 `pmset` 全部命令；现改为按账户隔离（`/etc/sudoers.d/nosleep-<uid>`），仅允许当前账户免密执行 `pmset -a disablesleep 0` / `1`，写入前经 `visudo -cf` 校验，卸载时通过系统授权真正清理干净，且不再往 `/tmp` 写可执行脚本。
- **显著降低后台耗电**：用 IOKit 订阅 `IOPMrootDomain` 合盖事件替代每 0.1 秒启动 5 个子进程的 bash 循环，空闲 CPU 占用从约 18% 降至接近 0。
- **去掉 Python 依赖 & 修复开盖黑屏**：进程内直接调用 DisplayServices 控制背光（未装 Xcode Command Line Tools 的 Mac 合盖也能可靠熄屏）；合盖状态下定时到期时，先恢复亮度再休眠，避免开盖黑屏。
- **不破坏系统原有睡眠设置**：仅切换 `disablesleep` 并持有 `PreventUserIdleSystemSleep` 断言，不再覆盖你自己的显示器/硬盘睡眠时间。
- **系统重启 / 注销 / 升级后自动恢复**：遇到 macOS 自动更新重启、注销或升级替换旧版时，退出前先安全恢复正常休眠，下次登录自启后自动恢复之前的开启状态（开机初期最多重试 3 次，尊重定时过期与低电量保护，失败必发通知；手动关闭/手动退出/外部关闭则清除恢复标记）。
- **低电量保护与完整沙箱测试**：新增电池低电量自动关闭（10% / 20% / 30% 可选），并配套 **36 个场景 / 81 项检查** 的离线沙箱测试套件（`./scripts/test.sh`）。

---

## ✨ 功能特性

- 🌙 **合盖不休眠** — 合上屏幕后电脑继续运行，屏幕自动关闭省电
- ⚡ **开盖秒亮** — 打开屏幕瞬间恢复，无需等待
- 🔄 **重启自动接续** — 系统更新重启或升级替换后自动恢复开启，不再静默失效
- ⏱ **定时关闭** — 支持 30分钟/1h/2h/4h/8h 定时自动关闭，倒计时精确到秒；关闭状态下选定时会直接开启，重启后未到期继续计时
- 🔋 **低电量保护** — 用电池时电量降到 10%/20%/30%（默认 20%）自动关闭，防止合盖放包里把电耗光
- 🚀 **开机自动启动** — 菜单里一键开关（macOS 13+，SMAppService）
- 🔧 **菜单栏常驻** — 不占 Dock 位置，一键开/关
- 🪶 **几乎不耗电** — 合盖监听用系统事件，空闲时 CPU 占用接近 0
- 🗑 **一键卸载** — 关闭合盖不休眠、删除免密配置、删除 App 自己的日志和设置，再把 App 移到废纸篓

## 🎯 解决什么问题？

MacBook 合上屏幕后会自动休眠，导致：

- ❌ 下载任务中断
- ❌ AI 助手 / 本地代理掉线
- ❌ SSH 远程连接断开
- ❌ 后台编译 / 渲染 / 自动化任务停止

**合盖不休眠** 让你合盖走人，电脑继续干活，屏幕关掉省电。

## 📋 安装与提前试用

### 方式一：直接下载本仓库 Release（推荐，抢先体验 v1.7.0）

1. 前往本仓库 **[Releases](https://github.com/H4Ha/nosleep-mac/releases)** 下载最新版 `nosleep-mac_v1.7.0.dmg`（Apple Silicon + Intel 通用）
2. 双击打开 DMG
3. 将「合盖不休眠」拖入「Applications（应用程序）」文件夹
4. 从启动台打开

### 从旧版（v1.6 及以下）升级

1. **先退出旧版**：点菜单栏 🌙 → 「退出」（旧版退出时会恢复正常休眠）
2. 用新版替换「应用程序」里的「合盖不休眠」，再打开
3. 新版会提示「升级免密配置」，点「升级」并输入一次管理员密码

如果打开新版时旧版仍在运行，新版会弹窗提示「改用这个版本」协助平滑接管；若旧版长时间未退出，请先在旧版菜单点「退出」再重开新版。

### 方式二：从源码构建

```bash
# 克隆本仓库
git clone https://github.com/H4Ha/nosleep-mac.git
cd nosleep-mac

# 构建 App（Universal Binary: arm64 + x86_64）
./scripts/build.sh

# 或构建 DMG 安装包
./scripts/build.sh dmg

# 运行沙箱场景测试（模拟合盖、定时、低电量、重启恢复等 36 个场景 / 81 项检查，不改动本机设置）
./scripts/test.sh
```

### ⚠️ 首次打开

App 使用 ad-hoc 本地签名（无苹果付费开发者证书），首次打开会被 macOS Gatekeeper 拦截：
- **macOS 15 及以后**：双击后点「完成」，再到「系统设置 → 隐私与安全性」，在页面底部点「仍要打开」
- **macOS 14 及以前**：右键点击 App → 选择「打开」→ 点击「打开」确认
- 或者在终端执行：`xattr -cr /Applications/合盖不休眠.app`

第一次点「开启」时需要输入一次管理员密码（配置免密权限），之后不再需要。

> 💡 手动点「退出」会同时关闭合盖不休眠且下次重开保持关闭；如果是系统重启 / 注销导致的非手动退出，且已开启「开机自动启动」，下次登录后会自动恢复开启状态（定时已过期或低电量时除外）。

## 🎮 使用方法

打开 App 后，菜单栏会出现 🌙 图标：

| 操作 | 说明 |
|------|------|
| 点击图标 | 弹出菜单，查看状态 |
| 「开启合盖不休眠」 | 开启功能，合盖后电脑继续运行 |
| 「关闭合盖不休眠」 | 关闭功能，恢复正常休眠 |
| 「定时关闭」 | 设置定时自动关闭（关闭状态下选择会同时开启） |
| 「低电量保护」 | 选择电量低于多少时自动关闭，或不启用 |
| 「开机自动启动」 | 登录后自动出现在菜单栏（配合重启自动恢复效果最佳） |
| 「关于 → 查看日志」 | 打开 `~/Library/Logs/合盖不休眠.log` 排查问题 |
| 「卸载并退出」 | 关闭功能、删除免密配置和 App 自己的文件，并把 App 移到废纸篓 |

## 🏗️ 项目结构

```text
nosleep-mac/
├── Sources/
│   └── menulet.swift     # 全部代码（纯 Swift 原生菜单栏应用）
├── Resources/
│   ├── AppIcon.jpg       # 图标源文件
│   ├── Info.plist        # App 配置
│   └── 使用说明.txt       # 用户说明
├── scripts/
│   ├── build.sh          # Universal Binary 构建与 DMG 打包脚本
│   └── test.sh           # 沙箱场景测试入口（36 场景 / 81 项检查）
├── tests/
│   └── sim/              # 系统层假实现与场景测试源码
├── README.md
└── LICENSE
```

## 🔧 技术细节

| 模块 | 技术 | 说明 |
|------|------|------|
| 菜单栏 | Swift + Cocoa | NSStatusItem 菜单栏应用，LSUIElement 模式 |
| 合盖监听 | IOKit | 订阅 IOPMrootDomain 的合盖事件（另有 1 秒一次的进程内兜底检查），不启动任何子进程 |
| 背光控制 | DisplayServices | 进程内直接调用私有框架控制内建屏幕亮度，不需要 Python |
| 休眠控制 | pmset + IOPMAssertion | 仅切换 `disablesleep` 并持有 `PreventUserIdleSystemSleep` 断言，不改你的显示器/硬盘睡眠时间 |
| 低电量保护 | IOKit.ps | 电源变化时系统回调，不轮询 |

### 安全说明

- `/etc/sudoers.d/nosleep-<你的用户ID>` 只允许**你的账户**免密执行两条固定命令：`pmset -a disablesleep 0` 和 `pmset -a disablesleep 1`；每个账户一个文件，互不覆盖
- 写入前用 `visudo -cf` 校验，校验不通过不会替换，避免写坏 sudo 配置
- 从旧版升级时会提示收紧旧的「所有账户可执行任意 pmset」配置
- 无任何网络通信，无数据外传
- 不在 `/tmp/` 放任何脚本；日志在 `~/Library/Logs/合盖不休眠.log`
- 卸载时关闭合盖不休眠、删除免密配置（会请求一次管理员密码）；如果睡眠时间还是旧版留下的「永不」，一并恢复默认值

## 💻 系统要求

| 要求 | 说明 |
|------|------|
| 系统 | **macOS 11.0+** (Big Sur 或更新版本) |
| 芯片 | Apple Silicon（M1/M2/M3/M4）✅ / Intel Mac ✅ |
| 权限 | 首次运行需管理员密码 |

> ✅ 已编译为 Universal Binary，同时支持 Apple Silicon 和 Intel Mac。
>
> ⚠️ **不支持 macOS 10.15 (Catalina) 及更低版本**（使用了 macOS 11+ 的系统特性如 SF Symbols，构建目标为 macOS 11.0）。

## 📜 开源协议

[MIT License](LICENSE) — 自由使用、修改、分发。

## 🙏 致谢

衷心感谢原作者 **[1073020318-cell](https://github.com/1073020318-cell/nosleep-mac)** 构思并开源了 `nosleep-mac`（从最初版本一路迭代到 v1.6.0，解决了无数 Mac 用户的痛点）。

本仓库在原项目基础上完成了纯 Swift 原生重写、权限收紧、低电量保护与重启自动恢复等改进，并已将完整改动通过 **[PR #1](https://github.com/1073020318-cell/nosleep-mac/pull/1)** 回馈给上游仓库。欢迎持续关注原仓库的官方更新！

## ❓ 常见问题与排错

**Q: 提示"无法验证开发者"或"已损坏"？**
> 这是 macOS Gatekeeper 安全机制，因为 App 使用 ad-hoc 签名。解决方法：
> 1. macOS 15 及以后：「系统设置 → 隐私与安全性」底部点「仍要打开」；macOS 14 及以前：右键点击 App →「打开」→「打开」
> 2. 或在终端执行：`xattr -cr /Applications/合盖不休眠.app`

**Q: 需要安装 Python 吗？**
> v1.7.0 起不需要。旧版依赖 Python 3，在没装开发者工具的 Mac 上合盖后屏幕可能不灭，升级即可。

**Q: 遇到问题如何反馈？**
> 1. 打开菜单栏 🌙 →「关于」→「查看日志」（`~/Library/Logs/合盖不休眠.log`）
> 2. 若双击无反应，可在终端运行 `/Applications/合盖不休眠.app/Contents/MacOS/menulet` 查看输出
> 3. 到本仓库 [GitHub Issues](https://github.com/H4Ha/nosleep-mac/issues) 提交问题并附上日志

---

<p align="center">
  如果这个改进版帮到了你，欢迎给原仓库和本仓库都点个 ⭐ Star！
</p>
