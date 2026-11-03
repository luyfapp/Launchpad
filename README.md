# 启动台 · Launchpad for macOS 15+

一个独立实现的 macOS 启动台（Launchpad）应用。用 AppKit + SwiftUI 编写，无第三方运行时依赖，最低支持 **macOS 15.0**，同一构建同时支持 macOS 15/26/27。

> **当前版本：2.2（build 45，2026-11-03）**  
> 视觉和交互参考经典 Launchpad（macOS Sequoia / Tahoe 时代）。本版重点：进入退出旋转动画、翻页百叶窗、双指离散翻页、双槽 60fps 分页、无阴影扁平图标、桌面壁纸毛玻璃、文件夹放大/缩小开关动画、隐藏菜单栏图标、帮助与系统手势冲突提示、虚拟机/无触控板时的快捷键提示，以及四指捏合的 7 天试用/卡密激活/付费墙。

![启动台](docs/screenshots/01-launchpad.jpg)

## 功能

| 能力 | 实现 |
| --- | --- |
| 应用发现 | 扫描 `/Applications`、`/System/Applications`、`~/Applications`，包含符号链接应用（例如 Safari）；按原生 `LaunchPadLayout.plist` 的排除规则过滤；FSEvents 监听三个目录，新安装应用自动出现、卸载后自动移除 |
| 全屏体验 | 无边框全屏面板，覆盖菜单栏与 Dock，自动输出到鼠标所在屏幕 |
| 毛玻璃壁纸 | 读取当前桌面壁纸：Aerial 从 wallpaper store 的 assetID 解码视频帧；`com.apple.wallpaper.choice.sonoma` 等内建动态壁纸映射到 `/System/Library/Desktop Pictures` 实际 HEIC。低分辨率 Core Image 模糊后放大，降低内存；每次打开刷新 |
| 图标网格 | 7×5 / 8×6 自适应分页，横向铺满屏幕，图标与标签间距收紧；无阴影的扁平图标/标签，视网膜图标；悬停放大 1.2× |
| 分页 | 双指左右滑动采用 macOS 原生式的**离散翻页**：一次手势翻一页，新手势立即接受（没有 0.5 秒锁）；也支持鼠标拖动空白区、方向键、Page Up/Down、底部页码点。左滑下一页、右滑上一页；静止时只渲染当前页，切换时最多渲染两页 |
| 动画 | 菜单栏“动画”：进入退出可选淡入淡出 / 旋转飞入飞出；翻页可选滑动 / 百叶窗。旋转用 Core Animation 层变换，百叶窗用 12 条纵向 slat 同时收拢 + 静态壁纸页表面，整页按手势方向位移；需授权（与四指捏合同一授权） |
| 搜索 | 中文名、英文名、拼音、拼音首字母、Bundle ID，实时排序与键盘上下选择 |
| 文件夹 | 自动生成“实用工具”文件夹；面板与图标网格同宽，图标用启动台尺寸并铺满面板，标题在面板上方、无背景、1.5× 字号；拖拽两个图标创建文件夹；点击打开、关闭为**放大/缩小**动画；点击标题原地重命名；拖拽排序；拖出移出；右键解散；**只剩 1 个 app 时自动解散**并保留该 app 位置 |
| 编辑模式 | 长按图标抖动；删除角标；快速拖放 = 排序（目标显示细白框）；拖到另一图标上停留 600ms（目标放大 + 高亮）= 创建文件夹；拖到文件夹合并；拖到页面边缘自动翻页；全部操作无触觉反馈 |
| 点击热区 | 只有点击图标 artwork 或标签文字才启动；点击格子的透明留白或页面空白默认退出启动台。编辑态例外：抖动时点空白只退出编辑；文件夹改名时点外部只提交名字；文件夹打开时点面板外只关闭文件夹 |
| 安全删除 | 第三方应用显示删除角标；先弹出玻璃确认卡，确认后移到废纸篓 |
| 右键菜单 | 打开、在访达中显示、从启动台移除、移到废纸篓、重新排列、刷新 |
| 持久化 | 页面顺序、文件夹、隐藏应用、启动频次、菜单栏图标显隐写入 UserDefaults |
| 菜单栏 | 显示/隐藏、重新排列、刷新、动画（进入退出 / 翻页）、四指捏合开关、购买/授权、开机自启动开关（SMAppService）、隐藏/显示菜单栏图标、系统手势冲突 / 环境不支持提示、帮助、输入监控设置、关于、退出 |
| 隐藏菜单栏图标 | 菜单里可隐藏；隐藏后启动台左上角出现齿轮按钮，点它打开同一份菜单；菜单里可以“显示菜单栏图标”恢复；快捷键不受影响 |
| 帮助 | 菜单栏“帮助”或 `launchpad://help` 打开使用说明浮层；`Esc` 或“知道了”关闭 |
| 系统手势冲突 | 检测系统“App（捏拢拇指和其他三指）”和“显示桌面（张开拇指和其他三指）”；开启时在页面底部显示提示条，可直接跳到触控板设置；虚拟机或没有触控板时改为提示“当前环境不支持四指手势操作”，并给出快捷键 |
| 四指捏合 | 读取触控板原始触点，拇指 + 三指：收拢打开，张开退出；针对轻捏降低阈值并支持 4–5 个触点；**需授权**：首次开启可试用 7 天，到期后购买卡密；菜单栏可开关 |
| 授权与试用 | 设备指纹绑定；7 天试用；卡密激活；30 天 ECDSA 签名租约；离线可用；终身使用当前大版本；首发 3 个月内 +1 台设备；付费墙内可购买/激活/解绑 |
| 快捷键 | `⌃⌥Space` 全局显示/隐藏；macOS 26 上额外注册 `F4`；`Esc` 退出；`⌘F` 搜索；`⌘W` 隐藏 |
| 深链接 | `launchpad://show`、`launchpad://search?q=微信`、`launchpad://folder?name=实用工具`、`launchpad://edit`、`launchpad://page?page=2`、`launchpad://help`、`launchpad://login?enable=1` |
| 焦点恢复 | 按 Esc 退出时回到打开启动台之前的应用；从图标启动应用时不会抢回焦点 |

## 截图

| 搜索 | 文件夹 |
| --- | --- |
| ![搜索](docs/screenshots/02-search.jpg) | ![文件夹](docs/screenshots/03-folder.jpg) |

| 编辑模式 | 帮助 |
| --- | --- |
| ![编辑模式](docs/screenshots/04-edit.jpg) | ![帮助](docs/screenshots/05-help.jpg) |

![系统手势冲突提示](docs/screenshots/06-gesture.jpg)

![购买 / 授权](docs/screenshots/07-paywall.jpg)

## 当前版本与更新记录

当前：**2.2 build 45**（2026-11-03，macOS 15+）。build 44 维护版为上一版；此前旧 45–56 迭代已废弃，本 45 为全屏自适应重做版。

| 构建 | 主要变化 |
| --- | --- |
| build 45 | 全屏幕类型自适应：按屏幕点数与横竖屏自动选择 6–10 列、4–8 行；1440p/4K/超宽屏填满垂直空间并放大图标；翻页触发阈值 10、spring 0.30；macOS 15+。SHA-256 `5c4e6760…`，CDHash `dd47d52e…` |
| build 44 维护版 | 当前发布线：macOS 15.0 最低系统；动态壁纸 `com.apple.wallpaper.choice.sonoma` 正确映射到系统 HEIC；壁纸模糊源进一步降到 0.10× 并加大模糊半径；隐藏启动台后销毁全屏 hosting view，闲置约 70–90MB；统一应用启动/前台激活（DeepSeek、极空间、已运行应用）；恢复原菜单与原界面。SHA-256 `09e04d7a…`，CDHash `2f38c216…`；1440p 网格改为填满垂直空间、图标 128pt，翻页触发阈值降低 |
| build 41 | 启用 CDHash 白名单 + 一次性 nonce attestation：所有 build 的试用/激活/续租必须先向 `/v1/attest/challenge` 取 nonce；Worker 新增 `build_attestations`、`attest_nonces` 与管理员注册接口；编译脚本增加 `__RESTRICT`/Hardened Runtime/entitlement 强制校验，后续构建自动检查 |
| build 40 | 试用客户端信息采集：设备名、App 版本、build、macOS 版本、架构、语言、机型、CDHash；新增 `/v1/trial/heartbeat` 与管理员 `/v1/admin/trials`；锁定菜单项真正灰色禁用；服务端 lease 改为 24 小时 |
| build 39 | 安全加固：`__RESTRICT` 段、强化注入检查、CDHash/attestation 字段预留、原生 P-256 lease/trial 校验、Ed25519 更新清单签名 + DMG SHA-256 校验、24 小时在线校验 / 7 天离线宽限 |
| build 38 | 应用内“去购买”改为可配置购买渠道：爱发电商品上架前按钮禁用并提示“购买渠道正在上架”，不再打开 mbd.pub 占位页；下载页加回 GitHub |
| build 37 | 内存优化：图标缓存 256→192px、上限 128；壁纸模糊尺寸 0.62→0.55、缓存 8→2；翻页动画全屏壁纸改为按需加载，默认滑动/淡入淡出不加载，隐藏时释放 |
| build 36 | 进入退出和翻页子菜单第二位新增“随机动画”：每次显示/隐藏/翻页随机选择一个具体动画，避免连续重复；默认仍为淡入淡出/滑动 |
| build 35 | 蜂巢六边形改为相互重叠，进入结束时保留全屏 mask，不再移除带缝隙的 mask；彻底消除蜂巢进入最后一帧的闪现 |
| build 34 | 修复楔入遮罩方向（起始全屏、结束为空），不再先消失再出现；翻页楔入同步修正 |
| build 33 | 新增 24 种进入退出/翻页动画与“动画 Pro / 四指捏合 Pro”绿色标识；统一参数化过渡引擎，原有滑动、百叶窗、四指捏合逻辑不变 |
| build 32 | 百叶窗不再移动整页（避免边缘黑条），改为 12 条纵向 slat 同时向手势方向收拢 |
| build 31 | 修正双指横向滚动方向：恢复自然滚动下的符号归一化，双指左滑 = 下一页（页面向左），右滑 = 上一页（页面向右） |
| build 30 | 百叶窗改为**12 条纵向 slat 同时收拢**，不再逐条从左到右；整页按手势方向做 5.5% 位移 |
| build 29 | 百叶窗按 PowerPoint 方向改为**纵向 12 条 slat**，去掉上一版的横向黑色边界线；源页不透明壁纸表面 + 纵向遮罩，仍只渲染两页、不重建网格 |
| build 28 | 新增付费动画：菜单栏“动画 → 进入退出”可选淡入淡出 / 旋转飞入飞出，“翻页”可选滑动 / 百叶窗；旋转使用 Core Animation 层变换（不触发布局） |
| build 27 | 虚拟机与无触控板环境不再显示“系统手势冲突：去关闭…”（那台机器上根本没有“触控板”设置面板），改为提示“当前环境不支持四指手势操作”，并列出 `⌃⌥Space` / `Esc` / `⌘F` 快捷键；提示条按钮改为“查看快捷键”，菜单栏入口同步 |
| build 26 | 新增 FSEvents 监听 `/Applications`、`/System/Applications`、`~/Applications`：新安装应用自动出现，卸载后自动移除，不再需要手动刷新 |
| build 25 | 修复点击“检查更新”时启动台闪一下（已显示时不再调用 `showLaunchpad()` 重置页面）；购买/授权移到隐藏菜单栏图标下面，成为功能区最后一项 |
| build 24 | 修复“检查更新”卡死：更新结果改为启动台内部非模态浮层，不再使用 `NSAlert.runModal()`；菜单顺序调整（检查更新放到关于上方，隐藏菜单栏图标移入上方功能列表） |
| build 23 | 新增半自动更新：菜单“检查更新…”读取 `update.json`，提示新版本/更新说明，打开 DMG 下载链接；每 24 小时自动检查一次 |
| build 22 | 试用改为服务器签名租约：本地改 `trialEndsAt` 不再能延长试用；客户端校验签名、设备指纹和到期时间 |
| build 21 | 安全硬化：Hardened Runtime、拒绝调试器、注入/可疑 dylib 检测、签名校验、符号剥离；校验失败锁定付费功能 |
| build 20 | 运行中每小时检查试用到期和租约续期，避免长期不重启导致试用/租约过期后仍可用 |
| build 19 | 授权缓存改为本地文件（避免 ad-hoc 重建后 Keychain 弹密码框），不落盘明文卡密；解绑需重新输入卡密 |
| build 18 | 客户端授权：7 天试用、卡密激活、ECDSA 租约验签、设备指纹、付费墙、购买/授权菜单 |
| build 17 | 文件夹打开/关闭改为放大/缩小动画（0.24s 放大、0.18s 缩小），去掉隐式淡入淡出和关闭后自动聚焦搜索，避免两段动画叠加 |
| build 16 | 图标抖动或文件夹改名时，点图标以外的区域只退出编辑/改名，不再回桌面；增加重复点击保护，关文件夹不会连带退出启动台 |
| build 15 | 新增“隐藏菜单栏图标”；隐藏后启动台左上角齿轮作为菜单入口，菜单里可恢复图标 |
| build 14 | 移除全部触觉反馈（翻页、拖拽、建文件夹、删除、搜索选择、捏合等） |
| build 13 | 快速拖放 = 排序；停留 600ms 才建文件夹；排序目标细白框；文件夹只剩 1 个 app 自动解散 |
| build 12 | 帮助浮层；系统四指手势冲突检测与底部提示条；菜单栏“帮助”和“系统手势冲突：去关闭…” |
| build 11 | 双指翻页改为一次手势一页的离散翻页，去掉 0.56 秒时间锁；新手势可立即翻页 |
| build 10 及以前 | 双槽分页架构、去阴影、壁纸模糊、文件夹面板样式、1.2 悬停、原生 Launchpad 应用过滤、DMG 打包等 |

> 维护约定：每次功能、交互、构建方式或版本号变化，都要同步更新本 README 的“功能 / 使用 / 更新记录 / 当前版本”，并同步 `HANDOFF.md`。迁移到 macOS 15.8 见 `MIGRATION-MACOS15.md`。

## 运行

### 直接运行成品

- 推荐：打开 `dist-xcode27/Launchpad-2.2-build51.dmg`，把 Launchpad 拖到 Applications；
- 已安装：`open /Applications/Launchpad.app`；
- 旧构建：`dist/Launchpad.app`（Xcode 16.4 构建，停留在 2.2 build 26）或 `dist-xcode27/Launchpad.app`（Xcode 27.2 / SDK 27.2 构建，最低 26，当前 2.2 build 51）；
- 想在 macOS 15 上预览同一套代码：`dist/Launchpad-Preview.app`。

首次从网络下载后如被 Gatekeeper 拦截，请右键应用 →“打开”。应用为本地 ad-hoc 签名，**未做 Apple 公证**。

### 从源码构建

要求：macOS 15+ 开发机、Xcode 26+（已在 macOS 27.2 + Xcode 27.1 / SDK 27.0 上验证）、[XcodeGen](https://github.com/yonaskolb/XcodeGen)（工程文件已包含，缺失时可手动打开 `Launchpad.xcodeproj`）。

```bash
./scripts/build.sh
```

脚本会：

1. 生成/更新 Xcode 工程；
2. 构建 `Release`，最低部署版本 **macOS 15.0**，Universal（arm64 + x86_64）；
3. 预览构建同样使用 macOS 15.0；
4. 进行 ad-hoc 签名并输出：
   - `dist/Launchpad.app`
   - `dist/Launchpad-Preview.app`
   - `dist/Launchpad-macOS26.zip` / `Launchpad-Preview-macOS15.zip`
   - `dist/Launchpad-2.2-buildN.dmg`（拖拽安装包，内含 `Applications` 链接）

用 Xcode 27 SDK 构建、保持 26 部署目标并输出到 `dist-xcode27/`：

```bash
RELEASE_TARGET=26.0 PREVIEW_TARGET=26.0 OUTPUT_DIR="$PWD/dist-xcode27" ./scripts/build.sh
# 同时生成 dist-xcode27/Launchpad-2.2-buildN.dmg
```

也可以直接在 Xcode 中打开 `Launchpad.xcodeproj` 运行。若 Xcode 版本较老，出现“deployment target 26.0 高于 SDK 支持范围”的警告是预期现象；二进制的最低系统版本由 `LC_BUILD_VERSION` 和 `LSMinimumSystemVersion` 共同保证为 15.0。

### 单独打 DMG

```bash
./scripts/package-dmg.sh dist-xcode27/Launchpad.app
# 输出 dist-xcode27/Launchpad-2.2-buildN.dmg，并自动挂载校验
```

## macOS 27 迁移

macOS 27 是独立系统，但当前工程不依赖 26 的绝对路径或私有 SDK；现有 26 包已实测可直接运行。2026-11-01 已用 Xcode 27.1 / SDK 27.0、保持 26 部署目标重新构建并通过冒烟测试（源码零修改，产物在 `dist-xcode27/`）。详细迁移、双目标构建、私有触控板框架风险与会话保留方式见：

- [docs/MACOS27_MIGRATION.md](docs/MACOS27_MIGRATION.md)
- [HANDOFF.md](HANDOFF.md) — 当前开发上下文、架构决策与下一步
- [dist/Launchpad-source-v2.2.zip](dist/Launchpad-source-v2.2.zip) — 可复制的完整源码包（不含构建缓存）

在 macOS 27 + Xcode 27 上构建 27-only 包：

```bash
./scripts/build-macos27.sh 27.0 26.0
# 输出 dist-macos27/
```

当前发布使用 `RELEASE_TARGET=15.0`，同时覆盖 macOS 15/26/27。

## 使用

- **打开**：菜单栏 3×3 图标、`⌃⌥Space`、macOS 26 的 `F4`；已授权/试用中可用拇指 + 其他三指在触控板上收拢（四指捏合）。
- **菜单栏图标被隐藏时**：打开启动台，点左上角齿轮按钮，即可打开完整菜单；菜单里的“显示菜单栏图标”可恢复。
- **搜索**：直接输入。支持中文、拼音、首字母和 Bundle ID，↑/↓ 选择，Enter 打开。
- **分页**：双指左滑下一页、右滑上一页；一次手势翻一页，快速连续滑动会立即响应。也可拖动空白区域、方向键、Page Up/Down 或页码点。
- **点击**：只有图标和图标标签是启动热区；点击页面空白、图标之间的空隙默认退出/收起启动台。
  - 图标抖动时点空白：只退出编辑，留在启动台；
  - 文件夹改名时点外部：提交名字，文件夹保持打开；
  - 文件夹打开时点面板外：只关闭文件夹，留在启动台。
- **编辑**：按住任意图标约 0.6 秒；或右键空白处 →“编辑启动台”。
- **排列**：在编辑模式拖动图标，快速松手落在目标格子就是排序（目标显示细白框）；在另一图标上停留约 600ms（目标放大 + 高亮）才会创建文件夹；拖到文件夹上会合并；拖到屏幕左右边缘会自动翻页。
- **文件夹**：点开/关闭有放大/缩小动画；点开后点标题可原地重命名；拖动图标可排序；拖出网格会移回主网格；只剩 1 个 app 时自动解散；右键可解散文件夹。
- **四指捏合（付费）**：首次开启会自动开始 7 天试用，收拢打开、张开退出；试用到期后需在“购买 / 授权…”里输入卡密。若没有响应，在菜单栏菜单选择“输入监控设置…”授权。状态菜单可关闭该手势。
- **购买 / 激活**：菜单栏“购买 / 授权…”打开付费墙，可查看试用剩余天数、购买档位、输入卡密激活；已授权可解绑本机（30 天 1 次，解绑需重新输入卡密）。卡密不落盘，租约缓存在 `~/Library/Application Support/Launchpad/license.json`。
- **系统手势冲突**：如果底部出现“系统四指捏合手势已开启”，到 系统设置 → 触控板 → 更多手势，关闭“App（捏拢）”和“显示桌面（张开）”；点提示条上的“去关闭”可直接打开触控板设置。
- **虚拟机 / 没有触控板**：底部提示条会变成“当前环境不支持四指手势操作”，并直接列出快捷键（`⌃⌥Space` 显示/隐藏、`Esc` 退出、`⌘F` 搜索）；点“查看快捷键”打开帮助浮层。这类环境里系统设置没有“触控板”项，所以不再引导过去。
- **退出**：`Esc`、`⌘W`、点击图标外区域，或再次按全局快捷键。

## 深链接

在终端、快捷指令或脚本中：

```bash
open 'launchpad://show'
open 'launchpad://search?q=weixin'
open 'launchpad://folder?name=实用工具'
open 'launchpad://edit'
open 'launchpad://page?page=2'
open 'launchpad://help'
open 'launchpad://close-help'
open 'launchpad://login?enable=1'
```

`launchpad://preview?search=...&folder=...&edit=1&page=2` 可以组合多个状态，方便自动化截图或演示。

调试用：`launchpad://gesture-check` 强制显示系统手势冲突提示；`launchpad://gesture-unsupported-check` 强制显示“当前环境不支持四指手势操作”提示。

## 工程结构

```
Sources/
  AppDelegate.swift              应用生命周期、菜单栏、深链接、全局快捷键
  main.swift                     NSApplication 入口
  Models/
    AppEntry.swift               应用元数据、扫描、拼音索引、默认排序
    LayoutItem.swift             应用/文件夹布局模型
    LaunchpadModel.swift         状态机、搜索、分页、拖拽、持久化
  Services/
    LayoutStore.swift            UserDefaults 编解码
    IconCache.swift              图标缓存
    WallpaperRenderer.swift      壁纸模糊渲染
    GlobalHotKey.swift           Carbon 全局热键
    TrackpadGestureMonitor.swift 四指捏合识别（MultitouchSupport 触点桥接）
    SystemGestureConflict.swift  系统四指手势冲突检测
    LicenseManager.swift         试用/卡密激活/设备指纹/ECDSA 租约/本地缓存
    SecurityGuard.swift          反调试/注入检测/签名校验
    UpdateChecker.swift          半自动更新检查（update.json）
  Support/
    GridMetrics.swift            网格几何与命中测试
    L10n.swift                   中英文本地化
  Views/
    LaunchpadRootView.swift      全屏根视图、分页与手势
    LaunchpadPageView.swift      网格页面、拖拽浮层、删除确认、Toast
    AppIconCell.swift            图标/文件夹单元、按压、长按、抖动、排序高亮
    FolderOverlayView.swift      文件夹浮层与内部拖拽
    HelpOverlayView.swift        帮助浮层、系统手势提示条
    PaywallView.swift            购买/试用/激活/解绑付费墙
    TopSearchBar.swift           搜索框
    PageDotsView.swift           页码点
    Glass.swift                  Liquid Glass 风格表面与按钮
    LaunchpadBackground.swift    壁纸、暗角、噪点
    LaunchpadWindowController.swift
Resources/                       Info.plist、Assets.xcassets、App 图标、本地化
scripts/make_icon.py             生成原创火箭图标
scripts/build.sh                 一键构建与打包
scripts/package-dmg.sh           生成拖拽安装 DMG
scripts/package-dmg-encrypted.sh 生成 AES-256 私有测试 DMG（仅防转发）
```

## 实现说明

- **触控板手势**：四指捏合通过动态加载系统 `MultitouchSupport.framework` 读取原始触点（4 个 contact），用触点包围盒的收缩/扩张比例识别，不依赖 Apple 未公开的 Dock 手势。该接口是系统私有框架，仅适合本机/独立分发，不适合 Mac App Store；应用会在状态菜单显示手势可用性并提供输入监控设置入口。
- **双指翻页**：采用离散翻页而不是跟手拖拽。一次手势只翻一页；`scrollGestureHandled` 防止一次手势连续翻多页；`phase == .began` 或间隔超过 0.30 秒即视为新手势，立即接受，因此快速左右滑动不会被时间锁吞掉。
- **其余实现**：AppKit / SwiftUI / Core Image / Carbon 公开 API；玻璃效果由 `NSVisualEffectView`、渐变描边、内阴影和高光组合实现。
- **壁纸**：通过壁纸 store 读取当前桌面/动态壁纸，在后台队列用 Core Image 做重模糊，再叠加静态玻璃高光、暗角和噪点；避免每帧实时模糊，每次打开时重新生成。全屏实时 `NSVisualEffectView` 已移除，以免翻页时多一次全屏 GPU 合成。
- **图标**：`NSWorkspace.icon(forFile:)` + `NSCache`；一次性预栅格化为 256px 位图，按需加载；无 SwiftUI shadow。
- **拖拽**：自定义 `DragGesture` + 画布坐标 + 网格命中测试，支持跨页自动切换、文件夹候选、排序目标高亮；拖到图标上需停留 600ms 才进入建文件夹候选，快速松手只排序。
- **文件夹**：只剩 1 个 app 时在拖出、删除/隐藏、启动加载三个路径统一归一化为普通图标；关闭文件夹时加 0.3 秒重复点击保护，避免同一次点击又退出启动台。
- **编辑态点击**：`backgroundTapped()` 优先处理抖动/改名，再处理文件夹和普通退出；普通状态点空白仍然退出启动台。
- **性能**：静止时只渲染当前页（35/48 个图标），翻页时最多同时渲染源页与目标页两页；窗口内容在打开/关闭之间复用；壁纸模糊只算一次，图标在首次显示后由后台队列预热；打开/关闭由窗口 alpha + 整体轻微缩放完成。
- **动画**：页面切换使用单一的 `interactiveSpring`（临界阻尼，无末端回弹），鼠标拖动支持两页实时跟手；触控板离散翻页、页码点、键盘、搜索翻页共用同一套两页动画容器。
- **触觉反馈**：已全部移除。

## 半自动更新

- 菜单栏 **检查更新…** 会读取（启动台已显示时只叠加更新浮层，不重置页面）：

```
https://download.luyfapp.cn/update.json
```

- 启动后每 24 小时静默检查一次；发现新版本时菜单显示“发现新版本 x.y.z…”，并在启动台弹一条提示。
- 点“检查更新…”会在启动台内显示**非模态浮层**（不是系统模态窗口，不会阻塞其他 App），可关闭/稍后/下载；下载后打开 DMG，把 Launchpad 拖进 Applications 覆盖安装即可。
- 授权租约、试用期、布局都在 `~/Library/Application Support/Launchpad/`，覆盖安装不会丢；不需要重新输入卡密。
- 这是“半自动”：App 只负责检查和打开下载链接，安装由用户手动完成；后续有收入再换 Sparkle 一键更新。

`update.json` 示例见 [docs/update.example.json](docs/update.example.json)：

```json
{
  "version": "2.2.24",
  "build": 24,
  "url": "https://download.luyfapp.cn/Launchpad-2.2.24.dmg",
  "notes": "1. 新增...\n2. 修复...",
  "minimumSystemVersion": "26.0"
}
```

发布时：

1. 把新 DMG 上传到 R2 的 `download.luyfapp.cn`；
2. 用上面的格式更新 `update.json`（`build` 必须大于 `CFBundleVersion`）；
3. 上传 `update.json`；
4. 用旧版本点“检查更新…”验证能打开新 DMG 链接。

未做 Developer ID 公证时，新 DMG 首次打开仍可能需要右键 → 打开。

## 安全与反逆向

- **构建硬化**：`ENABLE_HARDENED_RUNTIME = YES`；Release 构建开启符号剥离、`DEAD_CODE_STRIPPING`、`ENABLE_TESTABILITY = NO`；`codesign --options runtime`。
- **运行时检测**：`ptrace(PT_DENY_ATTACH)` 拒绝调试器附加；`P_TRACED` 检测已附加调试器；检测 `DYLD_INSERT_LIBRARIES`；扫描已加载 dylib 中的 Frida/Cycript/Reveal 等注入特征；`SecCodeCheckValidity` 校验签名完整性。任何一项失败只锁定付费功能，基础功能仍可用。
- **服务端约束**：设备指纹、1/3/5 台上限、30 天租约、解绑冷却、吊销；卡密不落盘。试用期也由服务器签名租约，改本地 `trialEndsAt` 不会延长试用。真正的授权上限在服务器，不在客户端。
- **能做到什么**：挡住 LLDB/Frida/Reveal 类动态注入、阻止未签名的改写、提高 Hopper/IDA 静态分析成本。
- **做不到什么**：本机校验一定能被补丁绕过；DMG 加密只防安装包转发，安装后仍可分析；没有 App Store FairPlay 那种平台级加密（本项目使用私有 API，不能上架）。
- **付费产品建议**：用 Developer ID 签名 + 公证（Gatekeeper 信任、更新稳定）；当前 ad-hoc + Hardened Runtime 只是过渡。
- **商业加固**：VMProtect 等 macOS 代码虚拟化可以再提高门槛，但可能与公证、Swift 运行时、私有框架冲突，需单独分支验证；不建议在没验证前直接上架。
- **私有测试加密包**：`scripts/package-dmg-encrypted.sh` 可生成 AES-256 加密 DMG，但密码必须单独发，客户安装后仍可被分析，不能替代上述措施。

## 已知边界

- 四指捏合依赖系统 `MultitouchSupport` 私有框架；macOS 大版本变化时可能失效。若系统拒绝原始触点或输入监控权限，手势不会生效，状态菜单会显示“不可用/关”，可先打开“输入监控设置…”授权。
- 系统“App / 显示桌面”四指手势会与启动台冲突；检测到开启时只提示，不会替用户修改系统设置。
- 虚拟机（Parallels 等）或没有触控板的 Mac 上，`MultitouchSupport` 拿不到设备，四指手势无法启动；应用改为提示“当前环境不支持四指手势操作”并给出快捷键，不再引导到这类机器上并不存在的“触控板”设置面板。
- 隐藏菜单栏图标后，应用仍可通过 `⌃⌥Space` / `F4`、Applications 里的图标或启动台左上角齿轮访问；不会出现“完全找不到入口”。
- 因为使用私有触控板接口，该项目适合本地构建和独立分发，不建议提交 Mac App Store。
- 删除应用会真实移入废纸篓（有确认步骤）。从启动台移除只是隐藏，可通过“重新排列图标”恢复全部应用。
- 仅在授权相关操作时访问 `https://license.luyfapp.cn`（试用开始、激活、续租、解绑）；基础功能不联网。
- 四指捏合需要授权；未授权/试用到期时只关闭手势，基础功能仍可用。
- 应用为 ad-hoc 签名、未公证；授权缓存放在本地文件而不是 Keychain，避免每次 ad-hoc 重建后弹钥匙串密码框。
- 移入废纸篓、全局热键、输入监控等行为受系统权限与安全策略影响；如系统阻止，会以 Toast 或状态菜单提示。

## 还原范围

在不修改系统文件的前提下，用户可见的核心 Launchpad 行为已覆盖：原生应用列表过滤、全屏毛玻璃壁纸、双指翻页、四指捏合、搜索、文件夹、抖动编辑、拖拽/合并、点击热区、启动、删除、右键菜单、键盘、多屏与状态持久化。它不是 Apple 原版组件的二进制补丁，而是一款独立应用；系统动画曲线、触觉强度与 Apple 私有手势的逐帧表现无法保证完全一致。

## 隐私

- 只有授权/试用相关请求会联网：试用开始、试用心跳、激活、启动后续租、解绑；未启用四指捏合时不会上传应用数据。
- 授权/试用请求只发送：设备指纹哈希（`SHA256(IOPlatformUUID + 序列号 + 固定盐)`，不上传原始序列号）、设备名称、App 版本、build、macOS 版本、架构、语言、机型、CDHash、卡密（仅激活/解绑时）；服务端还会记录 Cloudflare 提供的国家代码，不保存原始 IP。
- 试用客户端信息只用于试用状态、设备数限制、安全审计和售后支持；不包含应用列表、搜索内容、文件路径或触控轨迹。
- 不上传应用列表、搜索内容、图标或触控轨迹。
- 本地授权缓存 `~/Library/Application Support/Launchpad/license.json` 含签名租约、试用到期时间、档位/设备数；**不含明文卡密**。
- 只读取本机 `/Applications` 等目录中的 `.app` 元数据和图标，用于展示。
- 四指手势只读取触控板接触点的数量、标识和归一化坐标；不做内容识别、不保存轨迹。
- 删除操作需要用户二次确认。
- 需要重置布局或测试偏好时：`defaults delete com.luyf.launchpad`。
- 如需开机自动驻留菜单栏，可在“系统设置 → 通用 → 登录项”中加入本应用，或使用菜单栏的“开机自启动”。

## License

MIT。App 图标与自定义玻璃界面由项目自行绘制，SF Symbols 由系统提供；不使用 Apple 的 Launchpad 图标资源。应用列表中显示的是本机已安装应用的系统图标。

Apple、macOS、Launchpad 是 Apple Inc. 的商标。本项目与 Apple 无关联。
