# GitHub Actions 使用教程

本教程介绍如何使用 **GitHub Actions** 运行 `douyin-auto-fire`。

使用这种方式不需要自己准备服务器，也不需要电脑每天开机。配置完成后，GitHub Actions 会按照设定时间自动运行任务。

> 建议第一次只配置 **1 个抖音账号 + 1 个好友 + 1 条文字消息**。确认正常运行后，再添加其他好友、原生表情、随机消息或多账号。

---

## 1. Fork 项目

打开项目仓库：

**<https://github.com/unmev/douyin-auto-fire>**

点击右上角 **Fork**，将项目 Fork 到自己的 GitHub 账号。

![Fork 项目](https://img.908988.xyz/file/教程/douyin-auto-fire/DKPd0GVi.webp)

Fork 完成后，后面的所有操作都在 **你自己 Fork 出来的仓库** 中进行。

---

## 2. 启用 GitHub Actions

进入自己 Fork 后的仓库，点击顶部的 **Actions**。

如果 GitHub 提示 Fork 仓库的 Workflow 被禁用，点击启用工作流。

启用以后应该可以看到：

```text
Send Douyin Messages
```

这就是项目每天自动运行使用的工作流。

---

## 3. 获取抖音 Cookie

程序需要 Cookie 才能保持抖音登录状态。

### 3.1 登录抖音网页版

使用电脑浏览器打开：

**<https://www.douyin.com/>**

登录自己的抖音账号，并确认能够正常进入私信页面。

### 3.2 安装 Cookie-Editor

推荐使用浏览器扩展 **Cookie-Editor**：

**<https://chromewebstore.google.com/detail/hlkenndednhfkekhgcdicdfddnkalmdm>**

安装完成后，回到已经登录抖音的页面并打开 Cookie-Editor。

![打开 Cookie-Editor](https://img.908988.xyz/file/教程/douyin-auto-fire/STZqIxDn.webp)

### 3.3 导出 Cookie

点击 Cookie-Editor 的导出功能，导出格式选择 **JSON**。

![导出 Cookie](https://img.908988.xyz/file/教程/douyin-auto-fire/1rilVYmK.webp)

然后复制完整的 JSON 内容。

![复制 Cookie JSON](https://img.908988.xyz/file/教程/douyin-auto-fire/QKQHfndn.webp)

正确格式大致如下：

```json
[
  {
    "name": "xxx",
    "value": "xxx",
    "domain": ".douyin.com",
    "path": "/"
  }
]
```

请注意：

- 必须复制完整的 `[ ... ]` JSON 数组。
- 不要使用 `name=value; name=value;` 形式。
- 不要删除 Cookie 中的字段。
- 不要把 Cookie 提交到 GitHub 仓库。

> ⚠️ Cookie 相当于账号登录凭证，请不要发送给其他人，也不要公开到 Issue、日志或截图中。

---

## 4. 生成发送配置

除了 Cookie，程序还需要知道给谁发送、发送什么内容以及消息发送间隔。

如果不想自己写 JSON，可以直接使用配置生成器：

**<https://douyin-config.pages.dev/>**

生成完成后复制网站生成的完整 JSON。

一个最简单的配置例如：

```json
{
  "friends": ["好友昵称"],
  "messages": [
    {"type": "text", "value": "续火花 ✨"}
  ],
  "send_interval_seconds": {
    "min": 3,
    "max": 8
  },
  "prevent_duplicates": false
}
```

第一次使用建议先只配置：

```text
1 个好友 + 1 条文字消息
```

先把最基础的流程跑通，再增加其他功能。

---

## 5. 添加 GitHub Secrets

进入自己 Fork 的仓库，依次打开：

```text
Settings
↓
Secrets and variables
↓
Actions
↓
New repository secret
```

![进入 Secrets](https://img.908988.xyz/file/教程/douyin-auto-fire/aiPBHuxJ.webp)

![创建 Secret](https://img.908988.xyz/file/教程/douyin-auto-fire/BKtXckyQ.webp)

第一次使用至少需要添加下面两个 Secret：

| Secret          | 内容                              | 必须 |
| --------------- | ------------------------------- | -- |
| `DOUYIN_COOKIE` | Cookie-Editor 导出的完整 Cookie JSON | ✅  |
| `DOUYIN_CONFIG` | 配置生成器生成的完整配置 JSON               | ✅  |

### 5.1 添加 `DOUYIN_COOKIE`

点击 **New repository secret**。

Name 填：

```text
DOUYIN_COOKIE
```

Secret 粘贴刚刚导出的完整 Cookie JSON，然后保存。

### 5.2 添加 `DOUYIN_CONFIG`

再次点击 **New repository secret**。

Name 填：

```text
DOUYIN_CONFIG
```

Secret 粘贴刚刚生成的完整配置 JSON，然后保存。

配置完成后至少应该存在：

```text
DOUYIN_COOKIE
DOUYIN_CONFIG
```

GitHub 保存 Secret 后不会再次显示具体内容，这是正常现象。

---

## 6. 第一次运行：Dry Run

配置完成后，不建议第一次就直接真实发送。

项目提供了 **Dry Run** 模式，用来检查：

- Cookie 是否有效；
- 是否能够正常登录抖音；
- 是否能够找到目标好友；
- 配置是否正确。

Dry Run **不会真正发送消息**。

进入：

```text
Actions
↓
Send Douyin Messages
↓
Run workflow
```

第一次运行时，将 `dry_run` 开启（即 `true`），然后点击 **Run workflow**。

![运行 GitHub Actions](https://img.908988.xyz/file/教程/douyin-auto-fire/NLFF8g94.webp)

如果最后显示绿色的 `✓`，说明本次运行成功。

如果失败，点击本次 Workflow Run，进入：

```text
send
↓
Run
```

查看具体错误日志。不要只看最下面的 `Process completed with exit code 1`，真正的报错通常在它前面。

---

## 7. 测试真实发送

Dry Run 成功后，再手动运行一次工作流。

这一次关闭 `dry_run`，也就是：

```text
dry_run = false
```

然后运行。

这一次程序会真正向好友发送消息。

第一次真实发送仍建议只保留 **1 个测试好友**，确认好友、消息和发送结果都正确以后，再增加其他好友。

---

## 8. 每天自动运行

项目已经自带 GitHub Actions 定时任务。

工作流文件位于：

```text
.github/workflows/send.yml
```

当前默认配置：

```yaml
schedule:
  - cron: "0 0 * * *"
    timezone: "Asia/Shanghai"
```

表示每天北京时间 **00:00** 自动运行一次。

定时触发会直接进行真实发送，不会自动进入 Dry Run。

### 修改运行时间

例如每天北京时间 **08:30**：

```yaml
schedule:
  - cron: "30 8 * * *"
    timezone: "Asia/Shanghai"
```

每天北京时间 **20:00**：

```yaml
schedule:
  - cron: "0 20 * * *"
    timezone: "Asia/Shanghai"
```

格式为：

```text
分钟 小时 * * *
```

---

## 9. Cookie 失效怎么办？

Cookie 并不是永久有效。

如果 Actions 日志提示登录失效、需要重新登录、安全验证或 Cookie 无效：

1. 使用浏览器重新登录抖音网页版；
2. 用 Cookie-Editor 重新导出 Cookie JSON；
3. 打开仓库 `Settings`；
4. 进入 `Secrets and variables` → `Actions`；
5. 更新 `DOUYIN_COOKIE`；
6. 保存后手动执行一次 `dry_run = true`。

Dry Run 成功后即可继续正常使用。

---

## 10. 钉钉通知（可选）

如果希望通过钉钉接收任务结果，可以额外添加：

| Secret             | 内容            |
| ------------------ | ------------- |
| `DINGTALK_WEBHOOK` | 钉钉机器人 Webhook |
| `DINGTALK_SECRET`  | 钉钉机器人 Secret  |

这两个 Secret 必须同时配置。

如果不需要钉钉通知，两个都不要添加即可，不影响项目正常运行。

---

## 10.1 运行结果发到 QQ 邮箱（可选）

除了钉钉，也可以让工作流在**每次运行结束后**（无论成功还是失败）把结果摘要和运行日志发送到你的 QQ 邮箱。

该功能已内置在 `.github/workflows/send.yml` 中，不需要额外安装任何第三方 Action，使用 Python 标准库通过 QQ 邮箱 SMTP 发送。

### 准备 QQ 邮箱授权码

QQ 邮箱的 SMTP 登录密码**不是你的 QQ 密码**，而是专门的「授权码」。获取步骤：

1. 电脑浏览器打开 **<https://mail.qq.com>** 并登录；
2. 点击右上角 **设置** → **账户**（新版界面叫「账号与安全」）；
3. 找到 **POP3/IMAP/SMTP/Exchange/CardDAV/CalDAV 服务**；
4. 开启 **IMAP/SMTP 服务**（或 POP3/SMTP 服务）；
5. 按提示用绑定手机发送短信验证；
6. 验证后会显示一串 **16 位授权码**（形如 `abcd efgh ijkl mnop`），复制保存。

> 如果界面没有该入口，说明账号未开启 SMTP；Foxmail 邮箱（`@foxmail.com`）同样在 mail.qq.com 按上述步骤开启，入口一致。

### 添加 Secrets

在仓库 `Settings` → `Secrets and variables` → `Actions` 中添加下面两个 Secret（Foxmail 邮箱同样适用）：

| Secret             | 内容                                                  |
| ------------------ | --------------------------------------------------- |
| `QQ_SMTP_USER`     | 你的 QQ 邮箱地址，如 `123456789@qq.com` 或 `xxx@foxmail.com` |
| `QQ_SMTP_AUTHCODE` | 上面拿到的授权码（**去掉空格**）                                  |

这两个 Secret 必须同时配置，工作流才会发送邮件。

如果不需要邮件通知，两个都不要添加即可，不影响项目正常运行。

### 邮件内容

发送条件：工作流结束即触发（`if: always()`），成功和失败都会收到。

邮件包含：

- 触发方式（`workflow_dispatch` 或 `schedule`）；
- 运行状态（success / failure）；
- 提交号与运行链接；
- 最近 **8000 字** 的运行日志正文。

### 调整发送行为

- **只在成功时发**：把 `send.yml` 中 `Send result to QQ Mail` 步骤的 `if: always()` 改成 `if: success()`。
- **发给其他人**：保持 `QQ_SMTP_USER` 为发件人（SMTP 鉴权要求），把脚本里的 `msg["To"] = user` 改成对方邮箱地址即可。

---

## 11. 多账号（可选）

项目当前最多支持 **5 个抖音账号**。

第一次使用不建议直接配置多账号。先确保单账号模式下的：

```text
DOUYIN_COOKIE
DOUYIN_CONFIG
```

能够正常运行。

之后可以按照账号添加：

```text
DOUYIN_COOKIE_ACCOUNT1
DOUYIN_CONFIG_ACCOUNT1

DOUYIN_COOKIE_ACCOUNT2
DOUYIN_CONFIG_ACCOUNT2

DOUYIN_COOKIE_ACCOUNT3
DOUYIN_CONFIG_ACCOUNT3
```

以此类推，最多到 `ACCOUNT5`。

每个账号的 Cookie 和 Config 必须成对配置，不能只添加其中一个。

### 老用户增加第二个账号

如果以前一直使用：

```text
DOUYIN_COOKIE
DOUYIN_CONFIG
```

不需要删除原来的配置。

可以直接增加：

```text
DOUYIN_COOKIE_ACCOUNT2
DOUYIN_CONFIG_ACCOUNT2
```

原来的 `DOUYIN_COOKIE` / `DOUYIN_CONFIG` 会继续作为第一个账号使用。

---

## 12. 运行失败后的诊断文件

如果 GitHub Actions 运行失败，项目会自动上传诊断文件，可能包括：

```text
run.log
result.json
screenshots/
traces/
```

进入失败的 Workflow 页面，在页面底部找到 **Artifacts** 即可下载。

失败诊断 Artifact 默认保留 **3 天**。

这些文件可以帮助判断：

- Cookie 是否失效；
- 是否出现安全验证；
- 好友是否没有找到；
- 页面结构是否变化；
- Playwright 在哪一步失败。

> ⚠️ 截图和日志可能包含聊天内容或账号相关信息，请不要直接公开上传。

---

## 第一次使用推荐流程

```text
Fork 项目
    ↓
启用 Actions
    ↓
登录抖音
    ↓
导出 Cookie
    ↓
生成发送配置
    ↓
添加 DOUYIN_COOKIE
    ↓
添加 DOUYIN_CONFIG
    ↓
开启 Dry Run
    ↓
确认运行成功
    ↓
关闭 Dry Run
    ↓
测试真实发送
    ↓
确认成功
    ↓
等待每天自动运行
```

第一次不要同时配置多账号、多个好友、原生表情、随机消息和钉钉通知。

先把最基础的流程跑通，这样即使出现问题，也更容易判断是哪一步出了问题。

---

## 返回项目主页

👉 [返回 douyin-auto-fire](../README.md)
