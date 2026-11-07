# 编译运行记录

## 第 1 次运行（待填：等浏览器建库、推送、首次运行完成后补）

- 日期（HKT）：
- 仓库：https://github.com/legogogo25e/muse-gadget-build
- 工作流：Build ESP32 firmware（`.github/workflows/build-esp32.yml`）
- 运行链接：
- 板子：`devkit`（ESP32-C5 DevKitC-1）
- 上游：`facebookincubator/muse-gadget-sdk @ 463bd2a05380fea4211b0099ad0f70933c108b20`
- ESP-IDF：`espressif/idf:v6.0.3`
- 版本：`PROJECT_VER=1.0.0`
- 结果：
- 耗时：
- 产物：`firmware-devkit-1.0.0`（`muse-gadget.bin` + `bootloader.bin` + `partition-table.bin` + `flasher_args.json` + `build-info.txt`，保留 30 天）
- `muse-gadget.bin` 大小 / SHA-256：

## 以后每次运行怎么记

复制上面一段，填：日期、运行链接、板子、上游 commit、版本、结果、耗时、产物名、`.bin` 大小和 SHA-256。只改版本号不改上游时，OTA 会因版本门跳过安装，记得涨 `project_version`。
