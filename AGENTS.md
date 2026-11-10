# AGENTS.md — muse-gadget / muse-gadget-build 操作手册

这份文件是给之后接手这个项目的 agent 看的：怎么编译、发布、烧录、OTA、调试这块 ESP32 板子，以及哪些坑不要再踩。背景和搭建步骤见 [docs/EXAMPLE-FLOW.md](docs/EXAMPLE-FLOW.md)；这份只写**日常操作**。

## 系统现状（2026-11-10）

- **板子**：YD-ESP32-S3-Classic，ESP32-S3，`yd-esp32-s3-classic`
  - MAC `f0:f5:bd:4b:3e:c4`，BLE 名 `MuseGadget-4B3EC4`，node_id `homelink-4b3ec4`
  - 无咪、无喇叭、无电池：不能语音、不能报电量。语音板是 Seeed reSpeaker Lite，别搞混。
  - 板上跑的是 run #16 的固件（带 serial.read 的版本），代码与最新 manifest 一致，只差编译时间戳，不用 OTA。
- **源码仓**：`legogogo25e/muse-gadget`（private，上游 fork 的独立仓），当前 main head `b0838500`
- **编译发布仓**：`legogogo25e/muse-gadget-build`（public，本仓）
- **工具链**：ESP-IDF v6.0.3（上游要求，其他版本不行）
- **固件发布**：GitHub Pages，`https://legogogo25e.github.io/muse-gadget-build/`
  - 根 manifest：`manifest.json`；板子 manifest：`firmware/<board>/manifest.json`
  - 烧录页：`flasher.html`（本仓根目录，发布时由工作流拷进 Pages）
- **控制通道**：板子经 Wi-Fi 连用户自己的 Muse VM，加密 Noise 会话（`wss://<noise_host>/v1/noise`）。Agent 经 VM 的认证通道直接对板子发命令，不用蓝牙、不用用户贴 log。
- **Geek 板**：用户已明言先不编。它的 overlay 缺 OTA 开关，编之前要先加 `CONFIG_HOMEHUB_OTA_ENABLED=y`。不要主动开编。

## 先查状态，再动手

做任何事之前先读这三处，不要凭记忆猜：

1. **manifest**：`curl -s https://legogogo25e.github.io/muse-gadget-build/manifest.json`
   核对：板子名、版本号、上游 commit、编译时间（UTC 和 HKT 都有）、5 个文件（名字、地址、大小、SHA-256）、flash 参数。
2. **板子在线状态**：`device.health`（只验在线）、`device.describe`（看命令清单里有没有 `device.ota`、`serial.read`）。
3. **板子日志**：直接用 `serial.read` 读板上的 link 日志，**不要再问用户贴 log**。注意：它只返回 diagnostic ring 里 `link.*` 的行，不是整条 console stream，旧 log 会滚动覆盖，单次返回要分页。

## 编译一次（改了源码后）

1. 触发工作流：Actions → Build ESP32 firmware → Run workflow。
   - 没有 API 能触发 dispatch（GitHub MCP 里没有这个工具），**走浏览器**，用用户自己的 GitHub 登录。
   - 输入：`board`（如 `yd-esp32-s3-classic`）、`upstream_ref`（源码 commit，默认是旧的钉死值，手动填当前 head）、`project_version`（如 `1.0.0`）。
   - push 改动到 `.github/workflows/build-esp32.yml` 也会触发一次（默认 devkit），但那会把 manifest 短暂切成 devkit，之后要马上再触发一次 YD 编译把 manifest 恢复回来。
2. 等运行变绿（约 4 分钟），再读 manifest 核对（见上节）。不要只看 Actions 变绿就报完成。
3. 版本门：OTA 时新版本号不比运行中的新会被跳过；同版本想重装要加 `force`。涨版本时改 `project_version` 输入。

## 发布一次新固件到板子（OTA）

1. 确认 manifest 已更新到这次编译，`muse-gadget.bin` 的 URL 是 Pages 地址：
   `https://legogogo25e.github.io/muse-gadget-build/firmware/<board>/muse-gadget.bin`
2. 经 VM 通道发 `device.ota {url, force}`。板子自己经 Wi-Fi 下载、校验、重启。
3. 等板子回线（约 100 秒），查 `device.health` 是 `ok`，再查 `device.describe` 确认 `device.ota` 还在清单里——在清单里才证明跑的是新固件，不是回滚的旧版。
4. 安全网：新固件启动后 300 秒内连不上控制通道会自动回滚。
5. 多块板子：每块单独发一次命令，先升一块、验好再升其他的，不要同时全升。
6. **OTA 之前必须确认 manifest 里还是这块板的固件**。发布是整站重写、一次只留一块板；如果中间编了别的板，Pages 上这块板的固件就没了，这时不要对它发指着 Pages URL 的 `device.ota`。

## 首次烧录 / 救砖（USB）

- 空片没有 OTA 代码，第一次必须插线：用 Pages 上的 `flasher.html`（桌面 Chrome / Edge，Web Serial），或 `idf.py flash`。
- 烧录页打开自动读 manifest，下载后逐个验 SHA 才烧；烧完自动重启并开串口监视器（115200）。
- iPhone、Safari、微信打不开串口，页面里有提示，不要让人去试。
- 烧录不碰 NVS，配对和 Wi-Fi 凭证保留（整片擦除除外）。

## 改烧录页（flasher.html）

- 文件在编译发布仓根目录，**可以用 API 推**（不是 workflow 文件）。更新已有文件要先拿 blob SHA。
- 推完不会自动上线：要再触发一次工作流重新发布 Pages（publish job 里有一步 `cp flasher.html pages/flasher.html`），跑完再到线上核对改动。
- 显示用 `built_at_hkt`（没有才退回 UTC），标签写 HKT；但检测、新固件判断、发给服务器的字段和“已烧录记录”那行保持 UTC，不要改。
- Muse 里那个 Space「ESP32 固件烧录」是另一份拷贝，改 repo 的 flasher.html 不会自动同步它。

## 改工作流（.github/workflows/build-esp32.yml）

- **只能经浏览器网页改**。GitHub App 没有 `workflows` 权限，任何 API 推送碰到这个目录都会报 403。不要叫用户重装 App，重装解决不了权限问题。
- 改之前先在本地用 `python3 -c "import yaml; yaml.safe_load(...)"` 验 YAML，用 `node --check` 验内嵌 JS（publish job 里的 Python 也要能编译通过）。
- 改完核对 diff 再提交；提交本身会触发一次默认 devkit 编译，之后马上再触发一次 YD 编译把 manifest 恢复回来。

## 铁律（踩过坑的，不要再犯）

- **烧录地址、flash 参数一律以编译产物 `flasher_args.json` / manifest 为准，不要 hardcode IDF 默认值。**实测：ESP32-C5 bootloader `0x2000`（不是 0x0），ESP32-S3 bootloader `0x0`；分区表 `0x10000`（不是 IDF 默认的 0x8000）；app（ota_0）`0x20000`；flash 以 manifest 为准（当前 `dio / 80m / keep`）。
- **5 个文件都要烧**：bootloader、partition-table、ota_data_initial、phy_init_data、app。缺 `ota_data_initial.bin` 会让之前从 ota_1 启动过的板子继续跑旧固件。
- **固件里不带 SDK token**，烧好后再经串口 / BLE 设置。token 值永远不要写进记忆、文件或聊天记录。
- **PAT 不要贴在聊天里。**只读 PAT（`UPSTREAM_REPO_TOKEN`，限源码仓）由用户亲手建、亲手加到编译发布仓的 Secrets；PAT 授权范围要改也是用户亲手做。
- **每次编译的 `.bin` SHA 都不同是正常的**，固件内嵌了编译时间戳；大小一样、SHA 不同不代表代码变了。
- **OTA 永远是板子按 URL 自己拉**，没有“服务器把二进制推给板子”这条路。编译放哪里只改变 `.bin` 的来源，不改变 OTA 机制。
- `serial.read` 只收 link 日志，`serial.write` 只写去 UART、**不会执行 console 命令**（`>sdk_token=` 要在本地串口直接打才有用）。
- 报告 / 回传 Worker 一直没部署（卡在 Cloudflare 登录、GitHub PAT、确认目标仓三件事上），不要假设它存在；验收结果目前是手动复制报告。
- 用户叫停的线不要再提：毅行训练和 Gordon（2026-10-28 已解决）、按揭 / 转按 / 赎楼（2026-10-30 叫停）。

## 出错先查什么

- 编译失败：先看 `Build firmware` 那步是不是源码问题；`Checkout` 失败多半是 `upstream_ref` 填错，或 PAT 没覆盖源码仓（私有化之后 checkout 失败先查 `UPSTREAM_REPO_TOKEN` 是否就位）。
- manifest 还是旧的：先确认运行真的成功了，再看是不是读到了 CDN 缓存（直接 curl 再读一次）。
- OTA 后板子没回来：等够 300 秒，看是不是回滚了；用 `serial.read` 读启动日志，不要叫用户插线重烧。
- 烧录页监视器在硬复位后没输出：先确认是 monitor 重连问题还是板子问题——用 `serial.read` 或 BLE 扫描独立判断，不要一上来就叫用户重复插线。
- Pages 上某块板子的固件突然 404：八成是刚编了别的板，整站被重写了；重新编这块板恢复。

## 关键链接

- 源码仓：https://github.com/legogogo25e/muse-gadget
- 编译发布仓：https://github.com/legogogo25e/muse-gadget-build
- Pages：https://legogogo25e.github.io/muse-gadget-build/
- manifest：https://legogogo25e.github.io/muse-gadget-build/manifest.json
- 烧录页：https://legogogo25e.github.io/muse-gadget-build/flasher.html
- 运行记录：本仓 [docs/BUILD_LOG.md](docs/BUILD_LOG.md)
- 双仓流程说明：[docs/EXAMPLE-FLOW.md](docs/EXAMPLE-FLOW.md)
