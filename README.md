# tg机器人服务器要什么配置：2 核 2G 够不够？放国内还是洛杉矶？一年多少钱？（附 SixtyNet 全套餐对比）

跑一个 Telegram Bot，2 核 2G、40G 系统盘就能起步。EmbyTGBot 的部署文档里直接写着“一台 Debian / Ubuntu 服务器（2C2G40G 完全足够）”，tg-watchbot 这类双向对话 Bot 说到底就是一个 Python 进程加一个 SQLite 文件。真正会卡住你的是另一件事：这台机器能不能稳定连上 Telegram。国内机房得常驻代理，海外机房要看线路和 IP。下面按“要多大机器 → 放哪 → 多少钱”的顺序讲，最后把 SixtyNet 现在在售的套餐列全，给出对应的选择。

> 多数人的情况：只想让 Bot 长期在线、能收发消息、偶尔开个网页面板看状态，选 **LAX-STARTER**（2 核 / 2G / 40G NVMe，5TB 流量，**$200/年**，原价 $240）就够。这个配置和上面那些项目文档里的建议完全对得上。如果这台机器还要兼职跑社媒账号、多开登录这类看重 IP 属性的活，再看 **Residential-STARTER**（同配置，**$260/年**，住宅 IP，80/443 不开放）。👉 [查看 SixtyNet 中文价格页与全部套餐](https://sixtynet.net/vps-hosting?language=chinese&aff=39)

## SixtyNet 全套餐对比（洛杉矶机房）

| 套餐 | CPU / 内存 / NVMe | 网络与流量 | 年付价（USD） | 入口 |
| --- | --- | --- | --- | --- |
| Residential-STARTER | 2 核 / 2G / 40G，12500 IOPS | 2G 带宽，5TB，住宅 IP，**80/443 不开放** | **$260**（原价 $300，省 13%） | [查看该档](https://sixtynet.net/vps-hosting?aff=39) |
| LAX-STARTER | 2 核 / 2G / 40G，12500 IOPS | 2G 带宽，5TB，带 DDoS 防护 | **$200**（原价 $240，省 17%） | [查看该档](https://sixtynet.net/vps-hosting?aff=39) |
| LAX-MICRO | 4 核 / 4G / 80G，12500 IOPS | 4G 带宽，10TB，带 DDoS 防护 | **$400**（原价 $480） | [查看该档](https://sixtynet.net/vps-hosting?aff=39) |
| LAX-LARGE | 8 核 / 16G / 320G，25000 IOPS | 10G 带宽，40TB，带 DDoS 防护 | **$1,600**（原价 $1,920） | [查看该档](https://sixtynet.net/vps-hosting?aff=39) |
| Los Angeles China Unicom-Premium | 页面未标注 CPU / 内存，主打 100M 基础带宽、3 倍突发、95 计费、免费 BGP | 联通精品 AS9929 | **$2,500**（原价 $3,000） | [查看该档](https://sixtynet.net/vps-hosting?aff=39) |

价格页上还有几档没有单独列进上表：住宅 IP 系列另有 4 核/4G/80G、8 核/8G/160G、16 核/16G/320G 三个更高配置，以及一组标注“AS9929 Optimize for China”的住宅套餐（2 核/2G/40G 和 4 核/4G/80G 两档）；此外有一档 4 核/4G、300G + 200G 免费加成 RAID10、10TB 流量、带 DDoS 防护的配置。跑 TG Bot 用不到这个量级，需要时在同一个页面里往下翻即可。

那组 AS9929 优化的住宅套餐有两条需要提前知道的限制，页面上写得很直白：默认拒绝一切外部探测和扫描，除非你自己配置过白名单或端口转发，否则连不进去；退款只退到网站余额，不原路退回。这两条对 Bot 本身影响不大（原因见下一节），但会影响你怎么管理这台机器。

## 你的 Bot 到底吃多少资源

一个纯对话型 Bot 的负载低到有点无聊。tg-watchbot 用 aiogram 收消息、FastAPI + Uvicorn 起面板、APScheduler 跑定时任务、httpx 抓网页，数据全部落在 Python 标准库的 `sqlite3` 里，存消息、用户、去重记录和监控状态。它的面板默认监听 `127.0.0.1:8765`，整个服务的内存占用以百 MB 计，CPU 长期接近空闲。

如果你想自建 Telegram Bot API Server（脱离官方 api.telegram.org 的限制），`tgserver/tgserver` 镜像只有 14 MB，暴露 8080 端口，健康检查打 `/healthcheck`，需要 `TELEGRAM_API_ID`、`TELEGRAM_API_HASH`、`TELEGRAM_API_SECRET` 三项，可选 `TELEGRAM_WORK_DIR` 指定文件目录、`EXTRA_ARGS=--local` 追加启动参数。这个镜像在 Docker Hub 上的最近更新时间是 12 个月前（截至 2026 年 11 月），属于可用但陈旧，别指望它跟着官方更新走。

功能更重的 tg-post-bot 镜像 97.4 MB，WebUI 跑在 9624 端口，需要持久化 `/app/config`、`/app/logs`、`/app/backups` 三个目录。它首次启动会在 `/app/config/install_id` 生成本机安装 ID 作为授权设备标识，删掉容器重建，这个 ID 可能就变了，所以 `/app/config` 一定不能丢。这类项目的实际占用仍然在 2G 内存范围内，40G 系统盘对纯 Bot 是富余的。👉 [按 2 核 2G 起步配置看当前报价](https://sixtynet.net/vps-hosting?aff=39)

## 国内机还是洛杉矶机：出网才是硬约束

tg-post-bot 的“系统要求”第四条写的是：可访问 Telegram、TMDB、HDHive 等服务的网络环境；国内网络通常需要代理。它的 FAQ 第一条“Bot 无响应”的排查项同样是网络能否访问 Telegram，并支持 `HTTP_PROXY` / `HTTPS_PROXY` / `NO_PROXY` 环境变量，WebUI 设置页里也能直接填 `http://127.0.0.1:7890` 这类地址。tg-watchbot 在 2026-05-28 的“频道媒体转发”更新里写明支持 SOCKS5/HTTP 代理，适合国内服务器。

这两条合起来就是决策规则：机器放国内，代理是必配项，不是可选项，而且要接受代理进程本身也是一处可能出故障的地方。SixtyNet 是洛杉矶机房（套餐名 LAX 和页面上的 Los Angeles 都指向这里，第三方评测也提到目前只有洛杉矶一个机房），从洛杉矶出网连 Telegram 不需要额外套一层代理，少一个要维护的组件。

住宅 IP 那组套餐不开放 80/443，这条对 TG Bot 基本无影响：Bot 是主动出网去连 Telegram 的接口，不需要入站端口；面板本来就建议只监听 `127.0.0.1`，你自己通过 SSH 隧道或 Cloudflare Tunnel 访问。真正会被这条限制卡住的场景，是你想在同一台机器上挂一个对外的网站。

AS9929 中国优化线路要多花一笔钱，它影响的是你从国内 SSH 登录、打开面板的延迟。Bot 连 Telegram 走的是洛杉矶本地出网，跟这条线路优不优化没关系。为 Bot 买 AS9929 属于买错了地方。

## 限流：加 CPU 不会让监控更快

tg-post-bot 的频道监控页面给的建议是轮询间隔 3 到 10 分钟，过短可能触发 Telegram 限流。tg-watchbot 内置了限流和去重参数：`bot.rate_limit.window_seconds: 10` 配 `max_messages: 3`，群监听有 `ai_min_interval_seconds`（同一个群的最小推送间隔）和 `ai_dedupe_window_seconds`（相同内容去重窗口，示例值 300 秒），对普通用户也有防刷屏的简单限流。

这些数字说明瓶颈在 Telegram 侧的频率限制和出口 IP 质量，不在你的 CPU。把轮询压到 3 分钟以下既不会让消息来得更早，还可能把账号送进限流名单。买 4 核还是 8 核，对这类监控任务没有区别。

群监听如果用 `user_session` 模式（Telethon 用户账号静默监听，Bot 不必在群里），还要记住一条硬约束：同一个 session 不能并发跑两份。telegram-mcp 项目为此专门加了排他进程锁 `/tmp/telegram-mcp.lock`，原因是 Telegram 会在重复连接时永久吊销 session key（`AuthKeyDuplicatedError`）。多开容器分担负载这条路在这里是走不通的。

## 部署方式对系统盘和权限的要求

Docker 和 systemd 两条路，对机器的要求略有差别。

Docker 路线拉的是 GHCR 上的 `ghcr.io/gongyichuren/tg-watchbot:latest`，同时提供 `linux/amd64` 和 `linux/arm64` 双架构，不用自己 build。要注意 `.env` 会被容器挂载并写入（比如自动生成的 session secret），不能设成只读的 `:ro`。

systemd 路线把服务装在 `/opt/tg-watchbot`，用独立的系统用户 `tg-watchbot` 跑，`.env` 权限 `chmod 600`，单元文件是仓库里的 `systemd/tg-watchbot.service`。这条路线要求你有一台能装 Python 3.11+ 的机器和 root 权限，好处是不用维护 Docker。

两种方式都只占几 GB 系统盘。40G NVMe 真正会紧张的时候，是你开启了 tg-watchbot 的媒体转发下载（内置下载到服务器、断点续传、并发下载），或者跑 tg-post-bot 那种带网盘转存的项目。这时候 5TB 流量和磁盘余量才是你要算的东西。

## 面板不要直接暴露到公网

tg-watchbot 的 README 写得很明确：不建议直接把 8765 裸露到公网；如需公网访问，优先用 Cloudflare Tunnel、Nginx/Caddy 反代鉴权或 SSH 隧道。它甚至给了 Cloudflare Tunnel + Zero Trust Access 的具体做法：Public Hostname 指向 `http://127.0.0.1:8765`，在 Access 里加一条邮箱登录策略；临时调试用 `ssh -L 8765:127.0.0.1:8765`。

tg-post-bot 那边风险点不同。它的 WebUI 默认监听 `0.0.0.0:9624`，还提供了一个“重启系统”按钮，需要额外挂载 `/var/run/docker.sock`。官方警告写得很清楚：Docker Socket 具备宿主机 Docker 控制权限，公网部署不建议挂载，只建议在可信内网环境使用。

这条和选机器是连着的。走隧道方案的话，机器不需要开放任何入站端口，LAX 系列的 DDoS 防护对你就没那么关键；如果你打算直接把面板端口暴露出去，带防护的 LAX 系列比住宅 IP 系列更合适，毕竟住宅那组连 80/443 都不开。

## 哪种项目该买大一点

按项目类型分，结论差别不小。

纯双向对话 Bot、关键词推送、RSS 监控这类，2 核 2G 起步，40G 系统盘够用，LAX-STARTER 就是天花板之上的配置。

群监听加 AI 摘要，内存需求没有明显上涨，但要保证机器能出网访问你配置的 AI 接口地址。tg-watchbot 的 `summary_mode: ai` 需要填 `ai_base_url`、`ai_api_key`、`ai_model`（示例 `gpt-4o-mini`）、`ai_interface`（`responses` 对应 `/v1/responses`，`chat` 对应 `/v1/chat/completions`）、`ai_temperature`、`ai_timeout_seconds`（示例 30 秒）；AI 调用失败会回退到模板摘要，不会把消息丢掉。

频道媒体转发、网盘转存、多容器并存这类，才开始吃磁盘和流量。4 核 4G、80G 盘的 LAX-MICRO（$400/年）属于有余量的选择；真的要做媒体库归档，再往上考虑 LAX-LARGE 的 320G 和 40TB。

## 按场景选，以及下单前要问清的几件事

- 一个 Bot 长期在线，面板自己看 → **LAX-STARTER，$200/年**
- 同一个 IP 还要跑社媒账号、多开登录 → **Residential-STARTER，$260/年**，接受 80/443 不开放
- 媒体转发、网盘转存、几个容器一起跑 → **LAX-MICRO，$400/年**
- 大流量、大存储、多项目混跑 → **LAX-LARGE，$1,600/年**

下单前建议确认四件事。年付价是折后价（$300 折到 $260、$240 折到 $200），续费按哪个价走页面上没写明，问一句。5TB、10TB 这些流量额度按什么周期计算，页面也没标清。退款政策按套餐不同，AS9929 那组住宅套餐明确只退到网站余额。最后确认商家对长期常驻进程、Docker 安装、root 权限的态度，Bot 这类用途 CPU 占用极低但网络连接是 7×24 开着的。

👉 [打开 SixtyNet 套餐页，对照上面这份清单确认后下单](https://sixtynet.net/vps-hosting?aff=39)
