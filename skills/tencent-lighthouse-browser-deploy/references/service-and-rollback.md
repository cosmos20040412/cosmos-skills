# 安装与回滚

以下路径和配置都是示例，不能不经检查直接复制到服务器。按当前应用填写服务名、运行时、启动入口、路径和健康检查地址。

## 首次安装

在 `/opt/应用名/releases/版本号` 创建独立版本目录，以 `/opt/应用名/current` 指向当前版本；持久化数据放在 `/var/lib/应用名`。若这些目标已存在，先检查归属，进入更新流程，不覆盖安装。

服务使用独立非交互系统用户，不创建供远程登录使用的新凭据。程序目录保持只读，数据目录只允许服务用户写入。使用现有兼容运行时；需要新增依赖时按用户授权和当前工具规则处理。

创建独立 systemd 服务，配置 Restart 和开机自启。示例：

```ini
[Unit]
Description=Example web game
After=network.target

[Service]
Type=simple
User=example-game
Group=example-game
WorkingDirectory=/opt/example-game/current
ExecStart=/usr/bin/node /opt/example-game/current/server.mjs
Environment=NODE_ENV=production
Restart=on-failure
RestartSec=3
UMask=0077
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/var/lib/example-game

[Install]
WantedBy=multi-user.target
```

应用还需明确配置监听地址 `127.0.0.1`、空闲端口、公开 origin 和数据库路径；环境变量名以实际项目为准。资源限制应按应用负载确定，不能把某个游戏的内存上限作为固定值。若项目无法在这些文件系统限制下运行，检查实际需求后做最小调整。

执行 `systemctl daemon-reload` 和该服务的 `enable --now`，检查本机健康端点及最小业务操作，失败先读该服务日志。数据库工作进程也必须正常就绪，不能只验证监听端口。

## Nginx 路由

先备份实际配置文件的内容、权限和所属关系，记录备份位置。只修改与目标域名和协议匹配的 server 块。不要采用跨所有文件的全局字符串替换；脚本修改前断言目标块唯一。

若后端本身识别 `/game/`，相应示例为：

```nginx
location = /game {
    return 308 /game/$is_args$args;
}
location ^~ /game/ {
    proxy_pass http://127.0.0.1:3210;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    client_max_body_size 16k;
    proxy_read_timeout 35s;
}
```

这里的 `proxy_pass` 没有末尾 `/`，用于保留 `/game/`。端口、请求体上限和超时仅适用于对应应用的需求。WebSocket 应根据项目需要增加已验证的 Upgrade/Connection 配置；轮询 API 不需要假装使用 WebSocket。

先运行 `nginx -t`，通过后 reload；不要为一条新路由重启其他应用或整机。测试失败恢复备份，再验证并 reload。只有实际恢复成功才能称为已回滚。

## 更新已有版本

- 读取当前版本、服务配置和数据迁移版本；保留旧目录及当前链接目标。
- 新版本放入全新目录。尽可能先在隔离端口和临时数据上验证，不让两个进程误改同一个数据库。
- 对生产数据库做一致性备份，检查迁移是否向后兼容。SQLite 在线备份使用其 backup API，不只复制主 `.sqlite3` 文件而遗漏 WAL。
- 在不会误伤正在进行的游戏时切换版本并重启该应用。存在实际服务中断时，在当前任务授权和用户使用状况下安排操作。
- 失败恢复原程序链接、相关配置和服务状态。若迁移不兼容，必须采用预先确定的数据恢复方案；不要盲目运行旧代码，也不要覆盖更新后已经产生的用户数据。
- 保留可追溯版本，不在同次部署中顺手清理历史数据或无关目录。

## 验证和留档

记录本机健康、公网 HTTPS、实际业务测试、原网站响应、服务 active/ enabled。健康返回 200 或 enable 成功各自只能证明对应检查。

最终维护记录至少包括 URL、服务名、版本目录、数据路径、配置备份位置、查看日志命令，以及本次没有完成的验证。运行中服务的自动重启与数据备份是两回事；服务器本地持久化也不等于异地备份。

本技能创建或发布到代码仓库时仅保留通用说明和示例，不夹带此前用户的实例 ID、真实账号、域名、Cookie、数据库、上传包或服务器配置全文。
