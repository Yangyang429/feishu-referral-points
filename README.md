# 飞书内推积分 Skill

拉取技能分支：

```sh
git clone --branch codex/feishu-referral-points-skill --single-branch https://github.com/Yangyang429/feishu-referral-points.git
```

然后在该目录打开你的AI编程助手，告诉它：

> 请读取 SKILL.md，帮我检查依赖、配置我自己的飞书应用和授权，找到招聘机器人会话并启动积分页面。不要让我在聊天中提供App Secret。

AI负责配置步骤与启动；本人完成扫码授权与必要的应用权限审批。只有当前用户可访问的消息用于计算。无需下载之前的工具包。

源码包含 SKILL.md、core.js、server.cjs、index.html。依赖Node.js与官方lark-cli；配置和缓存保持本机，不提交Git。计分规则与限制见SKILL.md。
