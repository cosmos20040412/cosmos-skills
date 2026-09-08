# cosmos 的 skill 库

个人可复用的 Codex 技能库。

## 腾讯云浏览器部署

[`tencent-lighthouse-browser-deploy`](skills/tencent-lighthouse-browser-deploy/SKILL.md) 将一次实际完成的腾讯云网页游戏部署整理成可重复使用的流程：

- 连接用户已登录的浏览器，识别轻量应用服务器和现有网站。
- 准备适合服务器环境的发布包，检查子路径、数据库和多人联机功能。
- 使用腾讯云文件管理上传，通过自动化助手执行部署。
- 配置独立服务、开机自启和 Nginx HTTPS 入口，保留已有服务。
- 校验公网访问和业务功能，处理更新、失败回滚及维护记录。

详细步骤位于技能目录的 `references/` 中。示例不包含真实账号、服务器地址、登录信息或游戏房间数据。

## 使用

把 `skills/tencent-lighthouse-browser-deploy` 文件夹放到 Codex 用户技能目录中（通常为 `~/.codex/skills/`），在新任务中调用：

```text
使用 $tencent-lighthouse-browser-deploy，通过我已登录的腾讯云浏览器部署这个游戏。
```

技能需要当前环境提供可用的浏览器控制能力，并需要用户对目标服务器的部署授权。浏览器与控制台界面以实际页面为准。
