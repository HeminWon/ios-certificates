# 使用规范

## 基本原则

- 本仓库只用于存放 `fastlane match` 加密后的 iOS 签名资产。
- 不要提交明文证书、私钥或 provisioning profile。
- 不要把 `MATCH_PASSWORD`、Apple ID 密码、GitHub token 写入仓库。
- 日常构建只使用 `--readonly`。

## 业务项目接入

在业务项目中拉取 development：

```bash
MATCH_PASSWORD="你的-match-密码" fastlane match development \
  --readonly \
  --git_url https://github.com/HeminWon/ios-certificates.git \
  --git_branch main \
  --app_identifier com.example.app
```

拉取 ad-hoc：

```bash
MATCH_PASSWORD="你的-match-密码" fastlane match adhoc \
  --readonly \
  --git_url https://github.com/HeminWon/ios-certificates.git \
  --git_branch main \
  --app_identifier com.example.app
```

## 仅使用 xcodebuild 的项目接入

业务项目不需要使用 fastlane 作为打包工具，也可以复用本仓库。

关键点：`fastlane match` 只负责从本仓库拉取、解密并安装签名资产；真正打包仍然可以继续使用 `xcodebuild`。

### 1. 同步签名资产

在执行 `xcodebuild` 之前先运行一次：

```bash
MATCH_PASSWORD="你的-match-密码" fastlane match adhoc \
  --readonly \
  --git_url https://github.com/HeminWon/ios-certificates.git \
  --git_branch main \
  --app_identifier com.example.app
```

如果是 development 构建，把 `adhoc` 改为 `development`。

这一步会把：

- certificate / private key 安装到本机或 CI 的 Keychain
- provisioning profile 安装到 `~/Library/MobileDevice/Provisioning Profiles/`

### 2. 使用 xcodebuild 打包

建议业务项目使用 Manual Signing，并显式指定 team 和 profile：

```bash
xcodebuild \
  -workspace YourApp.xcworkspace \
  -scheme YourApp \
  -configuration Release \
  -destination generic/platform=iOS \
  -archivePath build/YourApp.xcarchive \
  DEVELOPMENT_TEAM=YOUR_TEAM_ID \
  CODE_SIGN_STYLE=Manual \
  PROVISIONING_PROFILE_SPECIFIER="match AdHoc com.example.app" \
  archive
```

导出 ipa：

```bash
xcodebuild \
  -exportArchive \
  -archivePath build/YourApp.xcarchive \
  -exportPath build/export \
  -exportOptionsPlist ExportOptions.plist
```

`ExportOptions.plist` 示例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>method</key>
  <string>ad-hoc</string>
  <key>signingStyle</key>
  <string>manual</string>
  <key>teamID</key>
  <string>YOUR_TEAM_ID</string>
  <key>provisioningProfiles</key>
  <dict>
    <key>com.example.app</key>
    <string>match AdHoc com.example.app</string>
  </dict>
</dict>
</plist>
```

### 3. 注意事项

- `xcodebuild` 本身不能直接使用本仓库里的加密文件，必须先解密并安装到 Keychain / Provisioning Profiles 目录。
- 推荐仍然用 `fastlane match --readonly` 做“签名资产同步”，但不要求业务项目使用 fastlane lanes。
- 不建议在 CI 中使用 `-allowProvisioningUpdates`，否则可能绕过本仓库直接修改 Apple Developer Portal。
- wildcard profile 的名称通常是 `match AdHoc *` 或 `match Development *`；明确 bundle id 的 profile 名称通常是 `match AdHoc com.example.app` 或 `match Development com.example.app`。

## 新增或更新 Profile

优先在临时目录操作，不要在本仓库工作区直接运行 `fastlane match import`。

推荐流程：

1. clone match 仓库到临时目录
2. 解密仓库
3. 复制新增 `.mobileprovision`
4. 加密新增文件
5. 使用规范 commit message 提交
6. push 到 `main`

提交信息示例：

```text
chore: import adhoc provisioning profile
```

## 禁止操作

不要直接提交以下明文状态的文件：

```text
certs/**/*.cer
certs/**/*.p12
profiles/**/*.mobileprovision
profiles/**/*.provisionprofile
```

如果 `file` 命令显示类似下面结果，说明可能是明文，不能提交：

```text
Certificate, Version=3
data
```

加密后的 match 文件通常是文本形式。

## 提交前检查

每次提交前执行：

```bash
git status --short
git diff --cached --stat
```

如果看到大量 `certs/` 或 `profiles/` 修改，先不要提交。

清理本地解密残留：

```bash
git restore --staged certs profiles
git restore certs profiles
```

## 当前覆盖范围

development：

- `*`
- `com.cloud.ai.AICloud`
- `com.verge.AIBrowser`
- `com.rockship.zzs`

ad-hoc：

- `*`
- `com.cloud.ai.AICloud`
- `com.verge.AIBrowser`
- `com.rockship.zzs`
