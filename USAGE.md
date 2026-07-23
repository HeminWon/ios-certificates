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
