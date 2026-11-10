# 双仓流程说明：源码仓 + 编译发布仓（example）

这份文档把 `muse-gadget` / `muse-gadget-build` 这对仓库的做法写下来，供后续项目参考仿照。核心思想只有一句：**源码仓只管代码，编译发布仓只管编译、发布和 OTA，两个仓库各做一件事。**

```
源码仓（可 private）                编译发布仓（必须 public）
┌──────────────────────┐            ┌──────────────────────────────┐
│ 固件源码             │ checkout   │ GitHub Actions 工作流        │
│ 板子配置 / overlay   │ ─────────► │  ├─ build：编译出 .bin     │
│ OTA 开关等           │   (PAT)    │  └─ publish-pages：发到 Pages│
└──────────────────────┘            └──────────────┬───────────────┘
                                                   ▼
                                    GitHub Pages（manifest.json + 固件）
                                                   ├─ 网页烧录器（Web Serial，首次烧录）
                                                   └─ device.ota（后续升级，不用插线）
```

## 两个仓库的分工

| | 源码仓 | 编译发布仓 |
| --- | --- | --- |
| 例子 | `legogogo25e/muse-gadget` | `legogogo25e/muse-gadget-build` |
| 放什么 | 固件源码、各板子的 sdkconfig overlay | 工作流 `.github/workflows/build-esp32.yml`、README、BUILD_LOG、本文件 |
| 可见性 | 可 private（本例已转 private） | **必须 public**，免费版 GitHub Pages 只支持公开仓 |
| 改动频率 | 经常改（功能、修 bug、加板子） | 很少改（工作流定型后基本不动） |

为什么拆成两个：源码仓是上游 SDK 的 fork，保持干净方便以后同步上游；可见性要分开（源码可 private、发布必须 public），一个仓库做不到两种可见性。

如果新项目不是 fork、从零写起，也可以一个仓库搞定（源码 + 工作流 + Pages 全放一起），更简单。拆仓主要是 fork + 源码 private + 发布 public 这三个条件加起来才值得。

## 搭建步骤（新项目照抄）

### 1. 准备源码仓

- fork 上游 SDK，或新建自己的 ESP32 项目仓库。
- 每个板子一个 overlay，例如 `devices/sdkconfig.yd-esp32-s3-classic`，里面写该板子的配置。
- 如需 OTA，在板子的 overlay 里加一行（OTA 默认是关的，不加就没有 `device.ota` 命令）：
  ```
  CONFIG_HOMEHUB_OTA_ENABLED=y
  ```

### 2. 新建编译发布仓（公开）

- 新建一个公开仓库，例如 `<项目>-build`。
- 把本仓的 `.github/workflows/build-esp32.yml` 复制过去，只改三处：
  1. checkout 那步的 `repository:` 改成你的源码仓；
  2. `board` 选项和 `case` 映射：按源码仓的板子列表加行（每行写 `target` + `sdkconfig` overlay）；
  3. 默认 `upstream_ref`（钉死的源码 commit）和 `project_version`（如 `1.0.0`）。
- 工作流有两个 job：`build`（容器 `espressif/idf:v6.0.3`，按板子编译，产物 `firmware-<board>-<version>`，保留 30 天）和 `publish-pages`（生成 Pages 站点并发布）。

### 3. 配一个只读 PAT（源码仓是 private 才需要）

- 在 GitHub 建 fine-grained token：Resource owner 是自己，Repository access 只选源码仓，权限只开 Contents → Read-only。
- 加到编译发布仓的 Secrets，名字固定叫 `UPSTREAM_REPO_TOKEN`。
- 源码仓是公开的可以跳过这步。

### 4. 开 GitHub Pages

- 编译发布仓 → Settings → Pages → Build and deployment → Source 选 **GitHub Actions**。
- 不用选分支，工作流里的 `deploy-pages` 会发布。

### 5. 触发一次编译，验证 manifest

- Actions → Build ESP32 firmware → Run workflow，选板子，跑完变绿。
- 打开 `https://<用户名>.github.io/<编译发布仓>/manifest.json` 核对：
  - 板子、目标芯片、版本号、上游 commit、编译时间（UTC 和 HKT 都有）；
  - `files` 里有 5 个文件，地址、大小、SHA-256 都在；
  - `flash` 参数（mode / freq / size）。
- 再用 `curl` 下一个 `.bin` 对一下 SHA，确认和 manifest 一致。

### 6. 首次烧录必须走 USB

- 空片没有 OTA 代码，第一次必须插线：用网页烧录器（桌面 Chrome / Edge，Web Serial）或 `idf.py flash`。
- 烧完检查两件事：串口有正常启动日志；命令清单里有 `device.ota`（没有就说明 overlay 里 OTA 开关没生效）。

### 7. 以后升级走 OTA，不用插线

- 发 `device.ota` 命令，带 Pages 上 `muse-gadget.bin` 的 URL，板子自己经 Wi-Fi 下载、校验、重启。
- 版本门：新版本号不比运行中的新会被跳过；同版本想重装要加 `force`。
- 安全网：新固件启动后 300 秒内连不上控制通道会自动回滚到旧固件。
- 多块板子：每块板子单独发一次命令，先升一块、验好再升其他的，分批最稳妥。

## 关键约定（都是踩过坑的）

- **工作流文件只能经浏览器网页改**。GitHub App 没有 `workflows` 权限，任何 API 推送碰到 `.github/workflows/` 都会报 403；README、BUILD_LOG 这类普通文件可以用 API 推。
- **烧录地址一律以编译产物的 `flasher_args.json` / manifest 为准，不要 hardcode IDF 默认值**。实测：ESP32-C5 bootloader 是 `0x2000`（不是 0x0），分区表是 `0x10000`（不是 IDF 默认的 0x8000），app（ota_0）是 `0x20000`；flash 参数以 manifest 为准（本例是 `dio / 80m / keep`，不是 sdkconfig 里猜的 QIO / 8MB）。
- **5 个文件都要烧**：bootloader、partition-table、ota_data_initial、phy_init_data、app。缺 `ota_data_initial.bin` 的话，之前从 ota_1 启动过的板子会继续跑旧固件。
- **manifest 一次只留一块板**：每次发布会把整个 Pages 站点重写，只留刚编的那块板。多块板子共用时要改成累积发布（保留已发布的各板子固件，manifest 并列多块板），否则编别的板会冲掉这块板的 OTA 链接。
- **固件里不带 SDK token**，烧好后再经串口 / BLE 设置 token。
- **每次编译的 `.bin` SHA 都不同是正常的**，固件内嵌了编译时间戳；大小一样、SHA 不同不代表代码变了。
- **OTA 永远是板子按 URL 自己拉**，没有“服务器把二进制推给板子”这条路。编译放哪里（GitHub Actions 还是自己的 VM）只改变 `.bin` 的来源，不改变 OTA 机制；想换来源只要换 `device.ota` 里的 URL。

## manifest 格式

根 manifest（`manifest.json`）和每块板子的 manifest（`firmware/<board>/manifest.json`）结构：

```json
{
  "repo": "<编译发布仓>",
  "updated_at_utc": "2026-11-10T05:19:32Z",
  "updated_at_hkt": "2026-11-10T13:19:32+08:00",
  "boards": {
    "<board>": {
      "board": "<board>",
      "target": "esp32s3",
      "project_version": "1.0.0",
      "upstream_ref": "<源码 commit>",
      "upstream_sha": "<源码 commit>",
      "idf_version": "ESP-IDF v6.0.3",
      "built_at_utc": "...",
      "built_at_hkt": "...",
      "github_run": "<Actions 运行链接>",
      "flash": { "mode": "dio", "freq": "80m", "size": "keep" },
      "files": [
        { "name": "muse-gadget.bin", "url": "firmware/<board>/muse-gadget.bin",
          "address": 131072, "size": 1380352, "sha256": "..." }
      ]
    }
  }
}
```

网页烧录器和 OTA 都读这份 manifest：烧录器按 `files` 里的地址烧录、按 SHA 验文件；OTA 命令里的 URL 就是 `muse-gadget.bin` 那一项的地址。

## 可选：网页烧录器

本例另有一个 Space「ESP32 固件烧录」：打开自动从 Pages 拉最新 manifest，核对 SHA 后用 esptool-js + Web Serial 烧录，烧完自动切串口监视器看启动日志。只在桌面 Chrome / Edge 可用（Web Serial），iPhone / 微信打不开串口。新项目仿照时，烧录器本身不用重写，改个 manifest 地址就能用。

## 新项目检查清单

- [ ] 源码仓建好，板子 overlay 齐全，OTA 开关已加（如需 OTA）
- [ ] 编译发布仓建好（公开），工作流三处已改对（checkout 仓库、板子映射、默认版本）
- [ ] PAT 已建（源码仓 private 才需要），已加为 `UPSTREAM_REPO_TOKEN`
- [ ] Pages 来源已设为 GitHub Actions
- [ ] 触发一次编译成功，manifest 已核对（5 文件、地址、SHA）
- [ ] 首次 USB 烧录成功，串口日志正常，`device.ota` 在命令清单里
- [ ] OTA 实测一次：发 `device.ota`，板子回线、health 正常、`device.ota` 还在清单里
