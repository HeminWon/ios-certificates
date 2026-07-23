# iOS 证书与描述文件仓库

本仓库使用 [fastlane match](https://docs.fastlane.tools/actions/match/) 统一管理多个 iOS 项目的证书和 provisioning profiles。

所有证书和描述文件都经过 OpenSSL 加密，解密密码通过 `MATCH_PASSWORD` 提供。

> 重要：本仓库必须保持 private，只允许需要签名权限的成员访问。

## 仓库配置

当前 `Matchfile` 使用：

```ruby
git_url("https://github.com/HeminWon/ios-certificates.git")
storage_mode("git")
git_branch("main")
platform("ios")
type("development")
```

说明：

- 证书仓库地址：`https://github.com/HeminWon/ios-certificates.git`
- 分支：`main`
- 平台：`ios`
- 默认类型：`development`
- 使用 `adhoc` 时通过命令或项目 `Fastfile` 覆盖类型

## 安装 fastlane

推荐在每个 iOS 项目中使用 Bundler 管理 fastlane：

```bash
bundle init
bundle add fastlane
```

也可以使用 Homebrew：

```bash
brew install fastlane
```

确保已安装 Xcode Command Line Tools：

```bash
xcode-select --install
```

## 本机使用方式

### 1. 设置 match 解密密码

在当前 Shell 中设置：

```bash
export MATCH_PASSWORD="你的-match-密码"
```

不要把 `MATCH_PASSWORD` 写入 Git 仓库。

### 2. 在任意 iOS 项目中拉取 development 证书

```bash
fastlane match development \
  --readonly \
  --git_url https://github.com/HeminWon/ios-certificates.git \
  --git_branch main \
  --app_identifier com.example.app
```

### 3. 在任意 iOS 项目中拉取 ad-hoc 证书

```bash
fastlane match adhoc \
  --readonly \
  --git_url https://github.com/HeminWon/ios-certificates.git \
  --git_branch main \
  --app_identifier com.example.app
```

`--readonly` 表示只从证书仓库读取，不会登录 Apple Developer Portal 创建或修改证书/Profile。CI 和日常构建都建议使用 `--readonly`。

## 多项目推荐接入方式

每个业务项目维护自己的 `fastlane/Fastfile`，只复用这个证书仓库。

示例：

```ruby
def match_options(app_identifier)
  {
    git_url: "https://github.com/HeminWon/ios-certificates.git",
    git_branch: "main",
    readonly: true,
    app_identifier: app_identifier
  }
end

platform :ios do
  desc "同步 development 签名资产"
  lane :sync_development_signing do
    match(type: "development", **match_options("com.example.app"))
  end

  desc "同步 ad-hoc 签名资产"
  lane :sync_adhoc_signing do
    match(type: "adhoc", **match_options("com.example.app"))
  end
end
```

然后在项目目录执行：

```bash
bundle exec fastlane sync_development_signing
bundle exec fastlane sync_adhoc_signing
```

如果项目没有使用 Bundler，也可以执行：

```bash
fastlane sync_development_signing
fastlane sync_adhoc_signing
```

## 多个 App Identifier

如果一个项目包含多个 bundle id，可以传数组：

```ruby
match(
  type: "development",
  git_url: "https://github.com/HeminWon/ios-certificates.git",
  git_branch: "main",
  readonly: true,
  app_identifier: [
    "com.example.app",
    "com.example.app.extension"
  ]
)
```

也可以在命令行中用逗号分隔：

```bash
fastlane match development \
  --readonly \
  --git_url https://github.com/HeminWon/ios-certificates.git \
  --git_branch main \
  --app_identifier com.example.app,com.example.app.extension
```

## CI 使用方式

CI 中需要配置以下环境变量或 secret：

```bash
MATCH_PASSWORD=你的-match-密码
```

如果证书仓库是 private，CI 还需要有读取该 GitHub 仓库的权限，例如：

- GitHub token
- deploy key
- 已授权的 CI 账号

CI 中推荐固定使用 `readonly`：

```bash
bundle exec fastlane match development \
  --readonly \
  --git_url https://github.com/HeminWon/ios-certificates.git \
  --git_branch main \
  --app_identifier com.example.app
```

ad-hoc 构建：

```bash
bundle exec fastlane match adhoc \
  --readonly \
  --git_url https://github.com/HeminWon/ios-certificates.git \
  --git_branch main \
  --app_identifier com.example.app
```

## 补充或创建新的 ad-hoc profile

如果某个 App 还没有 ad-hoc provisioning profile，不能使用 `--readonly`，需要登录 Apple Developer Portal 让 fastlane 创建或更新：

```bash
MATCH_PASSWORD=你的-match-密码 fastlane match adhoc \
  --git_url https://github.com/HeminWon/ios-certificates.git \
  --git_branch main \
  --app_identifier com.example.app
```

如果需要指定 Apple Developer Team：

```bash
MATCH_PASSWORD=你的-match-密码 fastlane match adhoc \
  --git_url https://github.com/HeminWon/ios-certificates.git \
  --git_branch main \
  --app_identifier com.example.app \
  --username your-apple-id@example.com \
  --team_id TEAM_ID
```

设备列表变更后，可以使用：

```bash
MATCH_PASSWORD=你的-match-密码 fastlane match adhoc \
  --git_url https://github.com/HeminWon/ios-certificates.git \
  --git_branch main \
  --app_identifier com.example.app \
  --force_for_new_devices
```

创建完成后，日常构建和 CI 再恢复使用 `--readonly`。

## 当前已导入的签名资产

当前仓库已包含：

- development certificate
- distribution certificate
- development provisioning profiles
- ad-hoc provisioning profiles

当前覆盖的 App Identifier：

| 类型 | App Identifier | Profile 文件 |
| --- | --- | --- |
| development | `*` | `profiles/development/Development_*.mobileprovision` |
| development | `com.cloud.ai.AICloud` | `profiles/development/Development_com.cloud.ai.AICloud.mobileprovision` |
| development | `com.verge.AIBrowser` | `profiles/development/Development_com.verge.AIBrowser.mobileprovision` |
| development | `com.rockship.zzs` | `profiles/development/Development_com.rockship.zzs.mobileprovision` |
| adhoc | `com.cloud.ai.AICloud` | `profiles/adhoc/AdHoc_com.cloud.ai.AICloud.mobileprovision` |
| adhoc | `com.verge.AIBrowser` | `profiles/adhoc/AdHoc_com.verge.AIBrowser.mobileprovision` |
| adhoc | `com.rockship.zzs` | `profiles/adhoc/AdHoc_com.rockship.zzs.mobileprovision` |

注意：`fastlane match` 对 provisioning profile 的保存路径按 `type + bundle id` 命名。同一个类型下，同一个 bundle id 只能保留一个 profile。

## 常用命令

拉取 development：

```bash
MATCH_PASSWORD=你的-match-密码 fastlane match development \
  --readonly \
  --git_url https://github.com/HeminWon/ios-certificates.git \
  --git_branch main \
  --app_identifier com.example.app
```

拉取 ad-hoc：

```bash
MATCH_PASSWORD=你的-match-密码 fastlane match adhoc \
  --readonly \
  --git_url https://github.com/HeminWon/ios-certificates.git \
  --git_branch main \
  --app_identifier com.example.app
```

查看 fastlane match 帮助：

```bash
fastlane match --help
```

## 安全注意事项

- 不要提交 `MATCH_PASSWORD`
- 不要提交 Apple ID 密码、App Store Connect API Key、GitHub token
- 不要公开本仓库
- CI 只能授予必要的最小权限
- 普通构建流程使用 `--readonly`，避免意外修改证书仓库或 Apple Developer Portal
