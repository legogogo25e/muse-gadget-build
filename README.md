# muse-gadget-build

用 GitHub Actions 编译 [facebookincubator/muse-gadget-sdk](https://github.com/facebookincubator/muse-gadget-sdk) 的 ESP32 固件，不用在本地装 ESP-IDF。

- 工具链：`espressif/idf:v6.0.3`（上游 README 要求 ESP-IDF v6.0.3，其他版本不支持）
- 上游默认钉死版本：`463bd2a05380fea4211b0099ad0f70933c108b20`（2026-11-07 的 main），可在 Run workflow 时改 `upstream_ref`
- 默认板子：`devkit` = ESP32-C5 DevKitC-1（上游默认目标）
- 产物：`muse-gadget.bin`（app-only，可用于 OTA）+ `bootloader.bin` + `partition-table.bin` + `flasher_args.json`，见 Actions 产物，保留 30 天
- **固件里不带 SDK token**，烧好后按上游 README 用串口 / BLE 再设置 token

## 怎么跑一次

1. 打开仓库的 **Actions** 标签页，左侧选 **Build ESP32 firmware**。
2. 点 **Run workflow**，选：
   - `board`：默认 `devkit`，其他见下表
   - `upstream_ref`：默认已钉死的 commit，想跟最新 main 就填 `main`
   - `project_version`：默认 `1.0.0`。OTA 有版本门：新版本不比运行中的新会被跳过，所以每次发新版都要涨版本号
3. 等运行变绿，点进那次运行，到底部 **Artifacts** 下载 `firmware-<board>-<version>`。
4. 也可以直接 push 改动到 `.github/workflows/build-esp32.yml`，push 到 main 也会触发一次（只监听这个文件，避免改文档时误触发）。

## 板子对照

| board（Run workflow 时选） | 芯片 | 说明 |
| --- | --- | --- |
| `devkit`（默认） | ESP32-C5 | ESP32-C5 DevKitC-1，状态灯 + 按键 |
| `espressif-s3-devkitc-1` | ESP32-S3 | ESP32-S3-DevKitC-1 |
| `waveshare-c6-lcd-147` | ESP32-C6 | Waveshare ESP32-C6-LCD-1.47 |
| `c6-nopsram` | ESP32-C6 | ESP32-C6 devkit（无 PSRAM） |
| `ideaspark` | ESP32 | ideaspark ESP32 + 1.9" 屏 |
| `muse-waveshare-s3-175c` | ESP32-S3 | Waveshare ESP32-S3-Touch-AMOLED-1.75C（完整 UI） |

想加别的板子：按上游 `.github/workflows/esp32.yml` 的 build matrix，在工作流的 `board` 选项和 `case` 映射里加一行 `target` + `sdkconfig` overlay 即可。

## 拿到 .bin 后怎么烧

首次烧录必须走 USB 串口（空片没有 OTA 代码）：

```bash
# 用上游 web flasher 最省事：Chrome/Edge 打开官方 web flasher，选板子烧录
# 或用 esptool，按产物里的 flasher_args.json 的地址烧 bootloader / partition-table / muse-gadget.bin
```

之后更新可走 Wi-Fi OTA：把 `muse-gadget.bin` 放到 GitHub Pages 上，用 `device.ota` 命令给板子一个 HTTPS URL，板子自己下载、校验、重启。详见 `docs/BUILD_LOG.md` 里第一次运行的记录。

## 首次运行记录

见 [docs/BUILD_LOG.md](docs/BUILD_LOG.md)。

## 常见问题

- **运行失败先看哪一步**：`Build firmware` 是编译本身；`Checkout muse-gadget-sdk` 失败多半是 `upstream_ref` 填错；`Upload firmware artifact` 报错 `if-no-files-found` 说明编译没产出 `.bin`。
- **改了代码想重编**：不用改仓库代码，Run workflow 时改 `upstream_ref` 为新 commit 即可；想长期跟 main，就把工作流默认值改成 `main`。
- **私有仓库**：公开仓库的 Actions 免费无限分钟；私有仓库会消耗免费分钟额度，之后按分钟计费。
