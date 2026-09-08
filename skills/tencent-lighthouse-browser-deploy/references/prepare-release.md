# 准备部署包

## 只读清点

从已有项目说明识别运行时、构建命令、环境变量、持久化需求和测试入口。检查现有源码变更，避免覆盖用户修改。服务器检查按需要使用下列命令类别，而不是照单执行所有命令：

```sh
cat /etc/os-release
df -h /
command -v node npm python3 nginx systemctl
node --version
python3 --version
ss -lntp
ls -l /etc/nginx/sites-enabled
```

随后只读与目标域名有关的配置。不要读取证书私钥；配置中的证书路径足以确认接入方式。先检查现有首页响应，作为部署后比较依据。

Nginx 配置文件的名字可能与真正的业务不一致。根据 `server_name` 和代理目标确认正确文件，尤其注意 HTTPS 和 HTTP 重定向通常位于不同 server 块。

## 平台适配

静态网站可由 Nginx 直接提供文件；需要联机状态的游戏还需要服务器后端和数据库。选择当前项目支持的最小改动方案。

曾成功使用的组合为 React/Vite 客户端、Node.js HTTP 后端、Python 标准库 SQLite 工作进程。它解决的是原项目使用 Cloudflare D1 而目标机器没有 D1 的特定问题，不是所有项目的默认架构。若项目本来支持 Node SQLite、Postgres 或其他数据库，优先保持原方案。

需要替换 D1 一类平台适配层时，保持迁移、参数绑定、原子写入和并发版本检查语义。SQLite 的持久化进程应具有 ready 状态、请求 ID、错误传递和关闭处理；工作进程失败要让主服务退出并由服务管理器恢复，避免主进程活着却永久无法访问数据库。不要用拼接用户输入的 SQL 替代原参数化查询。

## 子路径部署

例如最终地址使用 `https://example.com/game/`，必须同时处理：

- 打包工具的静态资源 base、应用路由、品牌首页链接。
- API 请求地址 `/game/api/...`，不能残留根路径 `/api/...`。
- 邀请链接保留 `/game/` 和房间查询参数。
- HttpOnly 会话 Cookie Path 匹配 API 路径；HTTPS 时正确使用 Secure。
- Origin 应是 `https://example.com`，不包含 `/game`。CSRF 校验不能为了修复部署而关闭。
- 反向代理保留还是移除前缀，必须与后端路由匹配。
- 静态文件路径规范化和目录穿越防护。

最终 origin 应来自明确的服务配置或经过验证的主机白名单。不要无条件信任公网传入的 Host/X-Forwarded-* 来生成安全判断或邀请地址。

## 构建与测试

用与目标运行时兼容的构建 target；确认发布包不依赖本地开发服务器或本机绝对路径。运行改动相关的业务和接口测试。对子路径、会话恢复、数据持久化及平台数据库适配进行有意义的验证。

联机游戏要验证两个以上独立会话，并覆盖设计人数上限、并发操作和私有状态隔离。重启恢复测试最好用临时数据库和单独端口，不碰现有线上对局。

## 打包边界

在专用发布目录中放入编译后客户端、服务器入口、必要运行依赖、数据库迁移和启动所需文件。明确排除 `.env`、Git 目录、账号数据、SSH 文件、本地 SQLite 数据及测试日志。不要把源码根目录整个打包。

使用系统 tar 或可靠打包工具；先列出包内成员，拒绝绝对路径、`..` 越界和非预期链接。计算 SHA256，并在远端解包前做相同校验。Windows 可使用：

```powershell
tar -czf $releaseArchive -C $releaseDirectory .
tar -tzf $releaseArchive
Get-FileHash -Algorithm SHA256 -LiteralPath $releaseArchive
```

这些变量应指向本次已经检查的绝对路径。运行程序与数据目录分离，使更新程序不覆盖房间数据库。
