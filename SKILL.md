---
name: feishu-referral-points
description: 从当前授权用户的飞书招聘机器人会话读取完整历史，按人名、第一岗位和阶段计算内推积分，生成本机统计页面或可分发的Windows工具。用于内推积分统计、日期趋势和该工具的配置、打包。
---

# 飞书内推积分

使用当前用户明确指定的招聘会话或已核验的机器人会话。会话ID不是访问凭据；每位使用者需要自己的应用配置和用户授权。不要共享应用密钥、账户授权、真实消息或缓存。

## 计分口径

- 按人名清洗合并，去除姓名空白；同名视为同一个人，结果不等同于候选人ID去重。
- 按完整消息历史时间排序，每人最早出现的岗位作为第一流程。投递及简历筛选消息也参与锁定；同一时间按原始顺序判定。该代理口径不能证明招聘系统的实际第一志愿。
- 锁定第一流程后，岗位名含“实习”的不计分；第二流程不递补。
- 第一流程分别累计：笔试50，首次面试150，offer沟通800。每类只计一次，最高1000；没有出现的阶段不补计。
- “新动态”“新进展”只看实际阶段。HR调整到阶段也计入；二面、HR面等合并为面试。
- 终止、淘汰、撤回和完成面试不新增积分，终止不扣此前积分；不能从“终止阶段：笔试”补推笔试积分。
- 用消息发送日期作为积分日期，先全历史锁定流程、去重，再按用户日期段汇总。展示总分、来源和每日新增／区间累计趋势，无需人员明细。
- offer沟通解析仅有合成样本验证；新用户应核验真实通知格式，不能把零匹配宣称为没有offer。

## 读取与授权

先检查目标机器的 `lark-cli` 帮助及登录态。若相关 lark skill 可用，按其授权规范执行；否则查看 `lark-cli auth login --help`、`auth status --help`、`config init --help`。

读取已知会话的命令：

```powershell
lark-cli im +chat-messages-list --chat-id <实际会话ID> --order asc --page-size 50 --page-all --page-limit 100 --no-reactions --as user --format json
```

身份与会话必须匹配，不要失败后改用bot绕过。使用profile时所有配置、授权和读取都显式沿用该profile。结果必须 `ok=true`、`meta.pagination.complete=true`、`data.has_more=false`；任何失败或未完整分页不得覆盖成功结果。工具默认最多5000条，超过时应增加上限并重新核验，不能给出完整统计结论。

授权链接与device code只用于本次授权，不作为技能或分发配置。需要用户扫码时先展示原始链接和PNG二维码，让用户完成授权后再继续。使用App Secret时通过隐藏输入和stdin传递，不写进代码、命令参数、日志或Git。

## 让AI完成配置

本仓库是可直接读取的skill，用户拉取后可直接说“请读取SKILL.md，帮我配置并启动内推积分”。不要要求用户安装之前的工具包。

1. 检查 node、lark-cli 和当前账户授权；缺依赖时使用官方Node.js与官方 npm 包 @larksuite/cli。不要复制他人的配置或凭据。Windows 可用 npm.cmd 安装 CLI。
2. 使用独立 integral-tool profile。先检查当前CLI help。已具备用户自己的应用配置时复用；否则发起CLI官方应用配置流程，或让用户在本机隐藏输入自己的App Secret，经stdin传给 config init。不要让用户在聊天中粘贴密钥。
3. 发起最小必要用户授权。参考 auth login --scope "search:message im:message:readonly im:chat:read" --no-wait --json --profile integral-tool；按实际接口缺失权限调整。展示原始授权链接和二维码，用户确认后完成device flow。
4. 从指定会话ID定位，或用当前用户的消息搜索核验“飞书招聘”机器人会话，让用户确认目标。不猜会话ID。只有本人可访问的真实会话才用于计算。
5. 在源码根目录创建被.gitignore排除的 settings.json，包含 chatId 和 cliPath。cliPath 是官方原生可执行程序的绝对路径，Windows全局npm安装通常位于 npm root -g 输出的 @larksuite/cli/bin/lark-cli.exe。不要填 .cmd/.ps1。不写App Secret或token。
6. Windows 下运行 node server.cjs，可使用 Start-Process -WindowStyle Hidden 在后台启动；只监听127.0.0.1:18767。打开 http://127.0.0.1:18767，点击同步并验证读取完整分页、来源总分及日期趋势。用户改配置或账户后重启服务。

core.js 是确定性计分逻辑，index.html 是简洁界面，server.cjs 调用当前用户CLI。维护计分逻辑时同时同步core.js和页面内副本。运行环境、settings.json、local-data、真实样本、二维码及凭据不能提交Git。

验证覆盖三阶段累计、多面去重、第二岗位排除、实习第一岗位不递补、终止不补计、跨日期不重复加分与完整分页门禁。当前计分及本机读取已验证；他人的应用device-flow兼容性、可用范围及权限需本人实测。界面只展示来源汇总和趋势，不展示人员清单。