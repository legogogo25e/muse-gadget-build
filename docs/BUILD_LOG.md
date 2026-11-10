# 编译运行记录

## 第 1 次运行（2026-11-07 22:33–22:36 HKT，成功 ✅）

- 日期（HKT）：2026-11-07
- 仓库：https://github.com/legogogo25e/muse-gadget-build
- 工作流：Build ESP32 firmware（`.github/workflows/build-esp32.yml`，commit `b9496d9`）
- 运行链接：https://github.com/legogogo25e/muse-gadget-build/actions/runs/42382124920
- 板子：`devkit`（ESP32-C5 DevKitC-1）
- 上游：`facebookincubator/muse-gadget-sdk @ 463bd2a05380fea4211b0099ad0f70933c108b20`
- ESP-IDF：`espressif/idf:v6.0.3`（容器镜像；本次 `build-info.txt` 里 `idf_version=unknown` 是工作流的小 bug：Record build info 那步没先 source `export.sh` 就调 `idf.py --version`，已修，见第 2 次运行）
- 版本：`PROJECT_VER=1.0.0`
- 结果：成功；作业 `Build devkit` 全部 12 步通过
- 耗时：3 分 46 秒（22:33:06–22:36:52 HKT；其中编译 1 分 57 秒）
- 产物：`firmware-devkit-1.0.0`（artifact zip 1,092,352 字节，SHA-256 `2af0543c876787aea9960b2d8fa8285bb48432fd53a8964e2895ca90d67df3e4`，保留 30 天，至 2026-12-07 过期）
- 产物内容（已本地解压核对）：
  - `build-devkit/muse-gadget.bin` — 1,708,032 字节，SHA-256 `79590cacd75c3aab9bad4bac1b8d09264173bc0f2af9862eb75b120ca35da76d`
  - `build-devkit/bootloader/bootloader.bin` — 23,632 字节，SHA-256 `fe36d818ab1bed5ea13ac94afe46a81b24c5e36b640748b79bc469b3ff5f0125`
  - `build-devkit/partition_table/partition-table.bin` — 3,072 字节，SHA-256 `bb6de32cd892ff0e9930d513e561858b46081c48d1e571bdb3b6077851a1e385`
  - `build-devkit/flasher_args.json` — 1,197 字节，SHA-256 `acb9c423ae4b4504b3c9198fe26be5ca50aa864efe2a4b832a096523f594e410`
  - `build-info.txt` — 392 字节

## 第 2 次运行（2026-11-07 22:44–22:48 HKT，成功 ✅，idf_version 修复验证）

- 日期（HKT）：2026-11-07
- 仓库：https://github.com/legogogo25e/muse-gadget-build
- 工作流：Build ESP32 firmware（`.github/workflows/build-esp32.yml`，commit `903c50e`，仅改 1 行：在 Record build info 步首加 source `export.sh`）
- 运行链接：https://github.com/legogogo25e/muse-gadget-build/actions/runs/42383026874
- 板子：`devkit`
- 上游：`facebookincubator/muse-gadget-sdk @ 463bd2a05380fea4211b0099ad0f70933c108b20`
- ESP-IDF：`espressif/idf:v6.0.3`
- 版本：`PROJECT_VER=1.0.0`
- 结果：成功；作业 `Build devkit` 全部步骤通过
- 耗时：4 分 04 秒（22:44:46–22:48:50 HKT；其中编译 2 分 09 秒）
- 产物：`firmware-devkit-1.0.0`（artifact zip 1,092,362 字节，SHA-256 `697cc01e17875c4021bed0a8c27c616a85958c198971111b1d2acefab381cc84`，保留 30 天，至 2026-12-07 过期）
- 产物内容（已本地解压核对）：
  - `build-devkit/muse-gadget.bin` — 1,708,032 字节，SHA-256 `8e1ed8d10a3a70aedb6fb0b4b83e893ac02b1a15314481a7892ff23a591aaea7`
  - `build-devkit/bootloader/bootloader.bin` — 23,632 字节，SHA-256 `236a2fcd2b92007af4f19ff8c8753e2f2a48f3ff7c39981619f52dc16e61be3a`
  - `build-devkit/partition_table/partition-table.bin` — 3,072 字节，SHA-256 `bb6de32cd892ff0e9930d513e561858b46081c48d1e571bdb3b6077851a1e385`（与第 1 次相同）
  - `build-devkit/flasher_args.json` — 1,197 字节，SHA-256 `acb9c423ae4b4504b3c9198fe26be5ca50aa864efe2a4b832a096523f594e410`（与第 1 次相同）
  - `build-info.txt` — 399 字节，`idf_version=ESP-IDF v6.0.3` ✅（修复已验证）
- 注意：两次运行同源码同版本，`.bin` 大小同为 1,708,032 字节，但 SHA-256 不同（第 1 次 `79590cac…`，第 2 次 `8e1ed8d1…`），固件内嵌了构建时间戳，属正常现象，不能用 SHA 是否相同判断两次构建是否一致。

## 第 3 次运行（2026-11-09 10:51–10:54 HKT，成功 ✅，首次发布到 GitHub Pages）

- 日期（HKT）：2026-11-09
- 工作流：Build ESP32 firmware（`.github/workflows/build-esp32.yml`，commit `43c3d8b`：新增 `publish-pages` 作业；仓库 Pages 来源事先改为 GitHub Actions）
- 运行链接：https://github.com/legogogo25e/muse-gadget-build/actions/runs/42553177048
- 板子：`devkit`
- 上游：`facebookincubator/muse-gadget-sdk @ 463bd2a05380fea4211b0099ad0f70933c108b20`
- ESP-IDF：`espressif/idf:v6.0.3`
- 版本：`PROJECT_VER=1.0.0`
- 结果：成功；`Build devkit` + `Publish firmware to GitHub Pages` 两个作业都通过
- 耗时：3 分 30 秒（10:51:10–10:54:40 HKT）
- Pages：https://legogogo25e.github.io/muse-gadget-build/manifest.json 已上线并核对：`idf_version=ESP-IDF v6.0.3`，`flash: dio / 80m / keep`，3 个文件（bootloader `0x2000`、partition-table `0x10000`、muse-gadget.bin `0x20000`）；`muse-gadget.bin` 1,708,032 字节，SHA-256 `acdcdfbb50784da23e599d509e674367431172781479305f0b150f502d7734e7`（与 Pages 下载实测一致）
- 更正：第 3 次运行的 `flasher_args.json` 证明 bootloader 地址是 `0x2000`（ESP32-C5 自身偏移），flash 参数是 `dio / keep`，不是之前按 sdkconfig 猜的 0x0 / QIO / 8MB；manifest 以 `flasher_args.json` 为准。另发现产物漏了 `ota_data_initial.bin` 和 `phy_init_data.bin`，见第 4 次运行。

## 第 4 次运行（2026-11-09 11:01–11:06 HKT，成功 ✅，补齐 5 个文件）

- 日期（HKT）：2026-11-09
- 工作流：Build ESP32 firmware（`.github/workflows/build-esp32.yml`，commit `7bf7ddb`：artifact 和 Pages 都加上 `ota_data_initial.bin` + `phy_init_data.bin`）
- 运行链接：https://github.com/legogogo25e/muse-gadget-build/actions/runs/42554090826
- 板子：`devkit`
- 上游：`facebookincubator/muse-gadget-sdk @ 463bd2a05380fea4211b0099ad0f70933c108b20`
- ESP-IDF：`espressif/idf:v6.0.3`
- 版本：`PROJECT_VER=1.0.0`
- 结果：成功；`Build devkit`（3 分 41 秒）+ `Publish firmware to GitHub Pages`（40 秒）都通过
- 耗时：4 分 27 秒（11:01:42–11:06:09 HKT）
- Pages：manifest 已核对，5 个文件齐全：
  - `bootloader.bin` — 0x2000，23,632 字节，SHA-256 `5db4bed99dd23def9b457f6e99bfaf0d6508fdfbddf4ce75628b9a3f320a38d9`
  - `partition-table.bin` — 0x10000，3,072 字节，SHA-256 `bb6de32cd892ff0e9930d513e561858b46081c48d1e571bdb3b6077851a1e385`
  - `ota_data_initial.bin` — 0x17000，8,192 字节，SHA-256 `7d2c7ac4888bfd75cd5f56e8d61f69595121183afc81556c876732fd3782c62f`
  - `phy_init_data.bin` — 0x19000，272 字节，SHA-256 `d2dbf30035dcbe1f7283a25a4f8d27f145e0b0266b407e2e68a298efc6a58042`
  - `muse-gadget.bin` — 0x20000，1,708,032 字节，SHA-256 `1edce269420bb40369d57a797632123d99b84408f98b05cf6b4f043c006d4c5e`
- 说明：`ota_data_initial.bin` 会把 OTA 槽位选择重置回 ota_0，没有它，之前从 ota_1 启动过的板子会继续跑旧固件。

## 第 26 次运行（2026-11-10 15:40–15:44 HKT，成功 ✅，重新发布烧录页）

- 日期（HKT）：2026-11-10
- 工作流：Build ESP32 firmware（`.github/workflows/build-esp32.yml`，手动触发）
- 运行链接：https://github.com/legogogo25e/muse-gadget-build/actions/runs/42776942717
- 板子：`yd-esp32-s3-classic`
- 上游：`legogogo25e/muse-gadget @ b08385006431eb6d133ef43b6129c1fd5c0a8ca0`
- ESP-IDF：`espressif/idf:v6.0.3`
- 版本：`PROJECT_VER=1.0.0`
- 结果：成功；耗时 4 分 02 秒（15:40:35–15:44:37 HKT）
- 目的：重新发布 Pages，让更新过的 `flasher.html` 上线（说明 OTA 是日常更新路径、YD USB 烧录已实机验证，commit `6eebb3c`）。固件代码没变，只是重编一次；manifest 仍只含 YD，`built_at_hkt` 2026-11-10T15:44:04+08:00；线上烧录页已核对，三处新文案都在，旧的 “not yet verified” 已消失。
- 注意：第 5 至 25 次运行（私有化验证、HKT 时间戳、serial.read 等）未逐一记入本文件，见主聊天记录；下次起恢复逐次记录。

## 第 27 次运行（2026-11-10 16:01–16:07 HKT，成功 ✅，device.reboot 新命令）

- 日期（HKT）：2026-11-10
- 工作流：Build ESP32 firmware（`.github/workflows/build-esp32.yml`，手动触发）
- 运行链接：https://github.com/legogogo25e/muse-gadget-build/actions/runs/42779491603
- 板子：`yd-esp32-s3-classic`
- 上游：`legogogo25e/muse-gadget @ 0e50bfca7416b4618ba125376d16abcb38095487`（新增 `device.reboot` 命令：noise_control.cpp 注册、app.c 用 `restart_task` 先回包、500ms 后 `esp_restart()`；host test 8/8 通过）
- ESP-IDF：`espressif/idf:v6.0.3`
- 版本：`PROJECT_VER=1.0.0`
- 结果：成功；耗时 5 分 37 秒（16:01:59–16:07:36 HKT）
- Pages：manifest 已核对，仍只含 YD，`built_at_hkt` 2026-11-10T16:06:35+08:00；`muse-gadget.bin` 1,380,352 字节，SHA-256 `aed1f809…`。
- OTA 上板（2026-11-10 16:09:36 HKT，用户批准后发 `device.ota {url, force: true}`）：回 `applied`，约 110 秒后回线，`device.health` ok（uptime 112 秒，新启动），`device.describe` 已含 `device.reboot`（6 个命令，证明跑的是新固件，不是回滚）。
- `device.reboot` 实测（同日 16:12）：发命令回 `{"ok": true}`，60 秒后回线，`device.health` ok（uptime 64 秒），`serial.read` 见全新启动日志（连 LSCMGUEST、IP 192.168.252.53、Noise 握手、`OTA image validated`，0 丢行），闭环完成。

## 以后每次运行怎么记

复制上面一段，填：日期、运行链接、板子、上游 commit、版本、结果、耗时、产物名、`.bin` 大小和 SHA-256。只改版本号不改上游时，OTA 会因版本门跳过安装，记得涨 `project_version`。
