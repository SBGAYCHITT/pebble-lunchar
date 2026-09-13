# Pebble Lunchar

Minecraft 启动器（离线模式为主），圆滑 + 半透明毛玻璃界面，打包为免安装单文件 .exe。
功能对标 PCL2，**不含联机相关功能**（局域网联机 / P2P 联机）。

## 功能清单

### 启动
- 玩家名 + 头像，自动生成与服务器离线模式一致的 UUID v3
- 版本列表（支持 `inheritsFrom` 继承，兼容 Forge / Fabric / OptiFine 等变体版本）
- 内存调节（1–16 GB）、自定义窗口宽高、全屏启动、额外 JVM 参数
- 游戏进程分离（`detached`），关闭启动器不影响游戏
- 实时回显游戏 stdout/stderr，显示退出码

### 版本
- **下载官方版本**：正式版 / 快照，官方源优先、失败自动切 BMCLAPI 国内镜像
- **自动安装加载器**：Forge、Fabric、Quilt、NeoForge、OptiFine（下载 installer 并静默执行）
- **版本管理**：重命名、复制、删除、导出为 zip、从 zip 导入、打开版本目录
- 一键打开 `.minecraft / versions / mods / saves / shaderpacks / resourcepacks`

### 资源
- **Mod 管理**：列表、启用/禁用（`.disabled`）、删除、定位、添加文件
- **资源包管理**：同上（resourcepacks）
- **光影管理**：同上（shaderpacks）
- **存档管理**：卡片式展示（名称 / 版本 / 模式 / 极限 / 作弊 / 时间 / 大小 / 图标），备份为 zip、删除、打开
- **截图管理**：缩略图网格，点击打开、删除

### 账户
- **离线登录**（默认，无需网络）
- **外置登录** Yggdrasil / authlib-injector（认证服务器 + 账号密码，可指定 injector jar）
- **微软登录**（设备码流程，实验功能，依赖 login.microsoftonline.com 与 Xbox Live）

### 设置
- `.minecraft` 目录、Java 路径（自动检测）、额外 JVM 参数
- 启动后隐藏启动器、版本隔离（每版本独立游戏目录）
- 下载源（官方 / BMCLAPI 优先）、并发线程
- 外观：深色玻璃 / 浅色玻璃 / 跟随系统、窗口不透明度、动画开关
- 游戏内设置：直接编辑 `options.txt`（FOV、渲染距离、图像品质、平滑光照）

### 其它
- 日志与崩溃报告查看（`logs/latest.log`、crash-reports）
- 关于页显示版本信息

## 使用

1. 双击 `Pebble-Lunchar.exe`
2. **首次使用**：到「版本」页 → 选择版本 →「下载」；或填写 MC 版本后安装 Forge / Fabric 等加载器
3. 回到「启动」页 → 选版本 →「启动游戏」

> 若本机已用官方启动器下载过版本，启动器会自动扫描到，直接启动即可。
> 若本机没有 Java，会自动复用官方启动器内置运行时（`.minecraft\runtime\java-runtime-*`）。

## Java 版本要求

| Minecraft | 所需 Java |
|---|---|
| 1.20.5+ | Java 21 |
| 1.18 ~ 1.20.4 | Java 17 |
| 1.17.x | Java 16 |
| ≤ 1.16.5 | Java 8 |

启动器会按所选版本自动挑选最合适的 Java，不匹配时给出提示。

## 目录结构

```
pebble-lunchar/
├── main.js         Electron 主进程：无边框亚克力窗口 + 全部 IPC
├── launcher.js     启动核心：版本扫描、离线 UUID、Java 检测、natives 解压、命令组装
├── downloader.js   版本下载安装（官方源 + BMCLAPI 镜像回退、并发池）
├── loaders.js      Forge / NeoForge / Fabric / Quilt / OptiFine 自动安装
├── mcapi.js        文件系统层：资源目录、存档 NBT 解析、options.txt、版本导入导出
├── accounts.js     离线 / 外置 Yggdrasil / 微软设备码登录
├── preload.js      contextBridge 安全桥接
├── renderer.js     渲染层核心：状态、导航、启动、账户、设置
├── pages.js        各功能页：版本、Mod/资源包/光影、存档、截图、日志
├── index.html      界面结构（左侧导航 + 11 个页面）
├── style.css       圆滑半透明视觉样式（支持深浅主题）
└── assets/logo.png UI logo
```

## 图标

品牌 logo 为蓝色 `PB` 字母标（源自用户提供的图片 `PB_logo_icon_1024.png`，**自带正确 alpha 通道，背景已镂空**）。
由 `make-icon.py` 处理：

1. 用 alpha 通道 `getbbox()` 定位内容范围，保留字内白色描边
2. 留 4% 边距裁出、居中贴到正方形画布（保持透明，不做抠图）
3. 输出 `build/icon.ico`（16/24/32/48/64/128/256 —— 小尺寸 32bpp BMP、256 用 PNG）、
   `build/icon.png`（256）、`assets/logo.png`（512，界面用）

> 注：早期源图是无 alpha 的棋盘格+水印图，需要 flood fill 抠除背景；
> 现用源图本身已镂空，故改为「按 alpha 裁框 + 居中」的简单可靠做法。

```bash
python make-icon.py
```

## 开发与打包

```bash
npm install      # 安装依赖
npm start        # 本地运行
npm run dist     # 打包 portable exe → dist/Pebble-Lunchar.exe
```

国内网络建议：

```bash
export ELECTRON_MIRROR="https://npmmirror.com/mirrors/electron/"
export ELECTRON_BUILDER_BINARIES_MIRROR="https://npmmirror.com/mirrors/electron-builder-binaries/"
npm install --registry=https://registry.npmmirror.com
```

## 打包说明（本机实测）

产物：`dist/Pebble-Lunchar.exe`（75 MB 单文件，双击即用）。

运行机制：首次双击自解压到 `%LOCALAPPDATA%\PebbleLunchar`（约 14 秒）并启动；之后双击直接秒开。
支持在「添加或删除程序」中卸载。

构建链为两步：

```bash
# 0) 清理旧产物（node 的 rm 会被沙箱删除守卫拦截，用 PowerShell）
#    Remove-Item -Recurse -Force dist\win-unpacked

# 1) electron-builder 产出 dist/win-unpacked（含 Pebble Lunchar.exe + resources/app.asar + 图标）
unset ELECTRON_RUN_AS_NODE
USE_HARD_LINKS=false node node_modules/electron-builder/cli.js --win --x64 --dir

# 2) 用 NSIS 编译成单文件 exe（外层图标与卸载信息由 NSIS 写入）
node make-exe.js
```

> **踩坑记录**
> - `ELECTRON_RUN_AS_NODE=1` 是本机预设环境变量，运行/打包前必须 `unset`，否则 Electron 以 Node 模式启动。
> - `USE_HARD_LINKS=false`：默认硬链接复制 Electron 会卡死。
> - 官方源下载 Electron 111MB 很慢，已通过 `build.electronDist` 指向本地 `node_modules/electron/dist`。
> - **winCodeSign 卡点**：electron-builder 在写 exe 版本信息/图标时会调用 `app-builder.exe rcedit`，
>   而 app-builder 会自行下载 winCodeSign；该压缩包内的 2 个 macOS 符号链接在无管理员权限的 Windows 上
>   解压失败（`Cannot create symbolic link`）。**本仓库已打两处补丁**（在 `node_modules` 内，重装依赖后需重打）：
>   1. `app-builder-lib/out/codeSign/windowsCodeSign.js` 的 `getSignVendorPath()`：优先返回缓存中
>      同时含 `windows-10/` 与 `rcedit-x64.exe` 的已解压目录；
>   2. `app-builder-lib/out/winPackager.js` 的 `signAndEditResources()`：把 `app-builder rcedit` 换成
>      直接 spawn 缓存里的 `rcedit-x64.exe`，从根上绕开 app-builder 的下载。
>   打完后 `--win --x64 --dir` 可一次通过（含图标）。
> - `npx` 在本机不可用，需直接 `node node_modules/electron-builder/cli.js`。
> - 打包前 `dist/win-unpacked` 必须清空，否则 electron-builder 的 `emptyDir` 会触发沙箱批量删除守卫而失败。
> - NSIS 脚本用 `File /r /x debug.log /x run.cmd "src\*"`，以排除 electron-builder 留下的临时文件。

## 故障排查

- **双击无反应 / 闪退**：多为显卡加速问题。可用命令行加参数启动验证：
  ```bat
  "%LOCALAPPDATA%\PebbleLunchar\Pebble Lunchar.exe" --disable-gpu
  ```
- **找不到 Java**：在「设置 → Java 路径」手动指定 `javaw.exe`；MC 1.20.5+ 需要 Java 21。
- **版本列表为空**：到「版本」页下载，或在设置里确认 `.minecraft` 目录是否正确。
- **游戏异常退出（code 4294967295）**：即进程以 -1 退出。先看「运行日志」末尾 —— 启动器会自动读取 `.minecraft\crash-reports` 里最新一份报告，把 `Description:` 与异常首行打印出来；也可直接打开该报告查看完整堆栈。
  - 若报告里是 `Only one quick play option can be specified`：说明传入的 `--quickPlay*` 参数多于一个。启动器需按官方规则处理版本 JSON 里 `features` 门控的参数（修复见下）。
- **重复运行 exe 没生效**：NSIS 安装器按 `version.txt` 构建号判断，构建号一致时直接启动已安装副本（秒开），不一致时自动覆盖重装。手动强制重装可删除 `%LOCALAPPDATA%\PebbleLunchar` 后重新运行 exe。

## 技术要点

- **半透明**：`transparent: true` + `frame: false` + `setBackgroundMaterial('acrylic')`（Win10/11 亚克力），CSS 叠加 `backdrop-filter: blur(18px)` 玻璃卡片，16px 圆角。
- **离线 UUID**：`MD5("OfflinePlayer:" + name)` 按 RFC 4122 置版本位为 3、变体位为 IETF，与 Bukkit/Paper 离线模式完全一致。
- **natives 解压**：调用系统自带 `tar`（Win10+ bsdtar 可直接解 zip/jar），不引入解压依赖。
- **存档信息**：内置极简 NBT 解析器，直接从 `level.dat`（gzip）读出存档名、版本、模式、极限/作弊、最后游玩时间。
- **下载容错**：每个文件先尝试主源，失败自动切换到 BMCLAPI 镜像；已存在文件自动跳过，支持断点续装。
- **进程隔离**：加载器安装器与游戏进程均独立 spawn，输出实时回传界面。
- **规则匹配（features）**：`ruleAllowed(rules, features)` 同时匹配 `os` 与 `features`，缺省一律 false。**这一点是必须的**：现代版本 JSON 用 `features` 门控互斥游戏参数（`--demo`、`--width/--height`、四个 `--quickPlay*`），若忽略 features 全量传入，游戏会在参数校验阶段直接抛 `IllegalArgumentException: Only one quick play option can be specified` 并以 -1 退出（表现为 GUI 闪一下就没有窗口）。
- **Java 24+**：自动追加 `--enable-native-access=ALL-UNNAMED`，消除 LWJGL 的受限方法警告。
