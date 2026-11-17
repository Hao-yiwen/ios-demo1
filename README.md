# ios-demo1

![Platform](https://img.shields.io/badge/platform-iOS%2017%2B-black)
![Language](https://img.shields.io/badge/language-Objective--C-blue)
![Xcode](https://img.shields.io/badge/Xcode-15.1-1575F9)
![Fastlane](https://img.shields.io/badge/fastlane-2.221.1-00F200)

用 **Fastlane** 和 **GitHub Actions** 练习 iOS 手动签名、打包与上传的学习仓库。应用本身是 Xcode 生成的空白 UIKit 工程，打开后是一块系统背景色的空页面，没有业务功能、第三方库或测试目标。

源文件头标注创建于 2024-06-21，工程由 Xcode 15.1 生成（`CreatedOnToolsVersion = 15.1`，`LastUpgradeCheck = 1510`）。

## 仓库里有什么

| 部分 | 现状 |
| --- | --- |
| iOS App | 单 target `ios-demo1`，Objective-C + Storyboard，仅 `UIApplication` 启动链路 |
| 签名 | 手动签名。Debug 使用 `match Development` 描述文件，Release 使用 `Apple Distribution` 与 `match AppStore` 描述文件 |
| Fastlane | 三条 lane：`build_release`、`build_debug`、`increment_version` |
| CI | 推送 `v*` tag 后，在 `macos-latest` 上打包、上传 TestFlight，并创建 GitHub Release |
| 测试 / 依赖管理 | 没有单元测试、UI 测试、CocoaPods 或 Swift Package |

`fastlane/README.md` 是 Fastlane 自动生成的 lane 列表，与 `Fastfile` 一致。

## 技术栈

| 项 | 版本或取值 |
| --- | --- |
| 语言 | Objective-C（ARC、modules），C 标准 `gnu17` |
| UI | UIKit，Scene 生命周期，`Main.storyboard` / `LaunchScreen.storyboard` |
| 最低系统 | iOS 17.0 |
| 设备 | 仅 iPhone（`TARGETED_DEVICE_FAMILY = 1`），构建平台包含 `iphoneos` 与 `iphonesimulator` |
| Bundle ID | `com.yiwen.ios-demo1` |
| 版本 | Marketing `1.0.0`，Build `30`（`Info.plist` 与工程设置一致） |
| 签名团队 | Developer Portal Team ID `84F8R9TAQN` |
| Xcode 工程 | `objectVersion` 56，兼容版本 Xcode 14.0 |
| Ruby 依赖 | `Gemfile` 声明 `fastlane`、`pry`；锁文件为 **fastlane 2.221.1**、**Bundler 2.5.13** |
| CI Ruby | `3.3.3`（`.github/workflows/build_release_and_deploy_testflight.yml`） |

`Gemfile` 没有锁死 fastlane 的版本号，以 `Gemfile.lock` 为准。`pry` 写在 Gemfile 里，lane 中没有调用。

Apple ID、App Store Connect Team ID（`126682121`）写在 `fastlane/Appfile`。证书仓库地址写在 `fastlane/Matchfile`：`git@github.com:Hao-yiwen/ios-demo1-fastlane-match.git`（`storage_mode: git`）。

## 目录结构

```text
.
├── Gemfile / Gemfile.lock          # Bundler 依赖
├── README.md
├── .github/workflows/
│   └── build_release_and_deploy_testflight.yml
├── fastlane/
│   ├── Appfile                     # Bundle ID、Apple ID、Team ID
│   ├── Fastfile                    # 三条 iOS lane
│   ├── Matchfile                   # match 证书仓库与 API Key 路径
│   ├── README.md                   # Fastlane 自动生成的 lane 说明
│   └── report.xml                  # 一次 increment_build_number 的历史报告
├── ios-demo1/
│   ├── main.m                      # UIApplicationMain
│   ├── AppDelegate.h / .m          # 启动与 Scene 配置
│   ├── SceneDelegate.h / .m        # 窗口场景回调（无自定义逻辑）
│   ├── ViewController.h / .m       # 空的 viewDidLoad
│   ├── Info.plist                  # Scene manifest 与版本号；其余键由 GENERATE_INFOPLIST_FILE 生成
│   ├── Base.lproj/
│   │   ├── Main.storyboard         # 空白 ViewController，系统背景色
│   │   └── LaunchScreen.storyboard # 空白启动页
│   └── Assets.xcassets/
│       ├── AppIcon.appiconset/     # 一张 1024×1024 通用图标
│       └── AccentColor.colorset/   # 未指定具体颜色
└── ios-demo1.xcodeproj/            # 工程。含本机 xcuserdata，没有提交共享 .xcscheme
```

仓库里没有应用运行截图。图标文件在 `ios-demo1/Assets.xcassets/AppIcon.appiconset/`，它是资源目录里的 App Icon，不是界面截图。

`.gitignore` 忽略 `ios-demo1.ipa`、`ios-demo1.app.dSYM.zip`、`ios-study-api-key.json` 和 `build/`。

## 环境要求

在 Mac 上编译、签名和跑模拟器或真机：

- macOS，以及能打开该工程的 Xcode（工程按 Xcode 15.1 生成，部署目标 iOS 17.0）
- 已加入团队 `84F8R9TAQN` 的 Apple Developer 账号
- `match` 证书仓库的读取权限，以及 `MATCH_PASSWORD`
- 仓库根目录的 App Store Connect API Key 文件 `ios-study-api-key.json`（不入库；CI 用 secret 解码生成）
- Ruby 与 Bundler。CI 使用 Ruby 3.3.3；锁文件要求 Bundler 2.5.13。本仓库在 Ubuntu 24.04 的 Ruby 3.2.3 上也可以安装依赖并列出 lane

当前这台 Linux 环境没有 `xcodebuild`，无法编译或启动 App，也无法执行会调用 `match` / `build_app` 的 lane。

## 安装依赖

在仓库根目录：

```bash
gem install bundler -v 2.5.13
bundle install
bundle exec fastlane lanes
```

`bundle exec fastlane lanes` 应列出下面三条 lane：

- `ios build_release` — Push a new beta build to TestFlight
- `ios increment_version` — increment build version
- `ios build_debug` — Build and sign a Debug package

已在 Ubuntu 24.04、Ruby 3.2.3、Bundler 2.5.13 上执行过 `bundle install`（安装锁文件中的 92 个 gem，fastlane 2.221.1）和 `bundle exec fastlane lanes`，输出与上表一致。

## 在 Xcode 里打开

```bash
open ios-demo1.xcodeproj
```

Fastlane 使用的 scheme 名是 `ios-demo1`，与 target 同名。仓库没有提交 `xcshareddata/xcschemes/ios-demo1.xcscheme`。用 Xcode 打开仅有一个 target 的工程时，Xcode 会在本机生成同名 scheme。若命令行提示找不到 scheme，在 Xcode 的 Manage Schemes 里勾选 Shared，再把生成的 scheme 文件纳入版本库。

签名是手动的：

| 配置 | 身份 / 描述文件 |
| --- | --- |
| Debug | `match Development com.yiwen.ios-demo1` |
| Release | `Apple Distribution`，`match AppStore com.yiwen.ios-demo1` |

本机没有这些描述文件和证书时，签名打包会失败。描述文件来自 `Matchfile` 指向的 git 仓库，lane 里以 `match(..., readonly: true)` 只读同步，不会在 lane 中新建证书。

界面就是 Storyboard 里的空控制器，`ViewController` 没有添加控件或文案。

## Fastlane

先准备：

1. 能访问 `git@github.com:Hao-yiwen/ios-demo1-fastlane-match.git` 的 SSH 密钥。
2. 仓库根目录的 `ios-study-api-key.json`。`Matchfile` 与 `build_release` 都写死了这个路径。
3. 环境变量 `MATCH_PASSWORD`。上传相关账号信息见 workflow 中的 `FASTLANE_USER` 与 `FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD`。

```bash
bundle exec fastlane ios increment_version   # 只把 Build 号 +1，会改工程文件
bundle exec fastlane ios build_debug         # Development 签名，输出 Debug IPA
bundle exec fastlane ios build_release       # Build 号 +1，打 App Store 包并上传 TestFlight
```

`build_debug` 的实际参数：

- scheme `ios-demo1`，configuration `Debug`，`export_method: development`
- 描述文件映射：`com.yiwen.ios-demo1` → `match Development com.yiwen.ios-demo1`
- 产物：`./build/ios-demo1-debug.ipa`，并生成 dSYM

`build_release` 的实际参数：

- `setup_ci` 仅在环境变量 `CI` 存在时执行
- `match(type: "appstore", readonly: true)`
- `increment_build_number` 修改 `ios-demo1.xcodeproj` 的 Build 号
- `build_app`：`export_method: app-store`，`xcargs: -allowProvisioningUpdates`
- `upload_to_testflight(api_key_path: "./ios-study-api-key.json")`

`increment_version` 与 `build_release` 都会改写工程里的 Build 号。没有证书、描述文件和 API Key 时不要执行这两条打包 lane。上述打包命令依赖 macOS 与 Xcode，本次没有在当前环境执行。

## GitHub Actions

工作流文件：`.github/workflows/build_release_and_deploy_testflight.yml`。

触发条件：向仓库推送匹配 `v*` 的 tag，例如 `v1.0.0`。

Runner 为 `macos-latest`。主要步骤：

1. checkout（`actions/checkout@v2`），并用 `SSH_PRIVATE_KEY` 配置 SSH。
2. 安装 Ruby 3.3.3，执行 `bundle install`，再执行一次未锁版本的 `gem install fastlane`。后续步骤调用的是 `fastlane`，不是 `bundle exec fastlane`。
3. 把 secret `BASE64_API_KEY_CONTENT` 解码成 `ios-study-api-key.json`。
4. 依次运行 `fastlane build_release` 与 `fastlane build_debug`。
5. 用 `qrencode` 生成两个二维码。内容是对应 IPA 在本次 Release 上的下载地址，扫码打开的是 GitHub 资源链接。
6. 用 `PAT_TOKEN` 创建 Release，并上传 IPA、dSYM 与二维码。

需要的 Actions secrets：

| Secret | 用途 |
| --- | --- |
| `SSH_PRIVATE_KEY` | 拉取 match 证书仓库 |
| `BASE64_API_KEY_CONTENT` | App Store Connect API Key JSON 的 Base64 |
| `MATCH_PASSWORD` | 解密 match 仓库 |
| `FASTLANE_USER` | Apple ID |
| `FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD` | Apple 专用密码 |
| `KEY_ID` / `ISSUER_ID` | 注入为环境变量。当前 `Fastfile` 上传时读取的是 JSON 文件，没有引用这两个变量 |
| `PAT_TOKEN` | 创建 Release 并上传附件 |

工作流期望的产物路径：

| 文件 | 路径 |
| --- | --- |
| App Store IPA | 仓库根目录 `ios-demo1.ipa` |
| App Store dSYM | 仓库根目录 `ios-demo1.app.dSYM.zip`（Release 资源名为 `ios-demo1.dSYM.zip`） |
| Debug IPA | `build/ios-demo1-debug.ipa` |
| Debug dSYM | `build/ios-demo1-debug.app.dSYM.zip`（Release 资源名为 `ios-demo1-debug.dSYM.zip`） |
| 二维码 | `ios-demo1-qr.png`、`ios-demo1-debug-qr.png` |

本次没有推送 tag，也没有在 macOS runner 上跑过这条工作流。

## 备注

- 应用没有网络请求、本地存储或设置页。阅读打包流程时，从 `fastlane/Fastfile` 和上面的 workflow 开始即可。
- `Info.plist` 里同时写了 `CFBundleShortVersionString` 与 `CFBundleVersion`；Xcode 还打开了 `GENERATE_INFOPLIST_FILE`，Marketing / Build 号以工程构建设置为准，当前两边都是 `1.0.0` / `30`。
- 工程的 `xcuserdata` 是本机 Xcode 界面状态，不参与编译。
- API Key、IPA 与 `build/` 不入库。不要把 `ios-study-api-key.json` 或 `MATCH_PASSWORD` 提交进仓库。
