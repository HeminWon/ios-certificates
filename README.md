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
- 使用 `adhoc` 时通过命令或业务项目 `Fastfile` 覆盖类型

## 文档入口

- [USAGE.md](USAGE.md)：使用规范、业务项目接入、新增或更新 profile、提交前检查、当前覆盖范围
- [fastlane match 官方文档](https://docs.fastlane.tools/actions/match/)：完整参数和行为说明

## 基本原则

- 本仓库只存放 `fastlane match` 加密后的签名资产。
- 日常构建和 CI 固定使用 `--readonly`。
- 不要提交明文证书、私钥或 provisioning profile。
- 不要把 `MATCH_PASSWORD`、Apple ID 密码、GitHub token 写入仓库。

## 当前签名资产

当前仓库已包含：

- development certificate
- distribution certificate
- development provisioning profiles
- ad-hoc provisioning profiles

具体 App Identifier 覆盖范围见 [USAGE.md#当前覆盖范围](USAGE.md#当前覆盖范围)。

## 常用操作

日常拉取 development / ad-hoc、业务项目接入、新增或更新 profile、提交前检查等操作统一维护在 [USAGE.md](USAGE.md)。
