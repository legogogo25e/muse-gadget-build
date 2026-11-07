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

## 以后每次运行怎么记

复制上面一段，填：日期、运行链接、板子、上游 commit、版本、结果、耗时、产物名、`.bin` 大小和 SHA-256。只改版本号不改上游时，OTA 会因版本门跳过安装，记得涨 `project_version`。
