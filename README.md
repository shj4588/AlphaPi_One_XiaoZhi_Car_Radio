# AlphaPi One 小智语音控车 · V1.0.0

正式发布版（**网页遥控版**，2026-11-10 更新）。在 v62 音频 / 控车基线之上新增**网页手柄遥控**：设备管理页点一下就能用手柄界面控制前进 / 后退 / 转向、用拖动条调速度，并控制 4 颗车灯。包含 **AlphaPi One 主机**、**AlphaPi STEM 体感遥控器 V0.2.1** 的源码与编译好的固件，以及一个**同时支持两个设备**的单文件网页刷机工具。

项目地址：https://github.com/shj4588/AlphaPi_One_XiaoZhi_Car_Radio

> 需要不含网页遥控的旧 V1.0.0（v62 原版）可完整回退：同级目录
> `AlphaPi_One_XiaoZhi_Car_Radio_prev_V1.0.0_before_webpad/` 是改动前的整包备份。

---

## ⚠️ 硬件适配声明

**本固件仅适配下面三张照片里的这一款车**（照片在本目录根下，刷机页里也已内嵌）：

| 照片 | 内容 |
| --- | --- |
| `AlphaPi_Car.jpg` | 小车底盘：半圆形黑色 PCB、两个橡胶轮、中间金手指插槽 |
| `AlphaPi_One.jpg` | AlphaPi One 主机（ESP32-S3），插在小车金手指上 |
| `AlphaPi_Stem（Zero）.jpg` | AlphaPi STEM 扩展板（ESP32-C3），拿在手里当体感遥控器 |

按键、点阵、音频、控车指令全部按「这套底盘 + 板载 N32 协控」调试。
**其他车型 / 其他底盘 / 不同协控固件版本均未适配**，刷入后可能无法开机，或出现按键无响应、电流音 / 无声、控车方向错乱。刷之前请先对照照片确认硬件一致。

**★ 本版只适配「协控款」（第一款车：UART2 控车 + 4 颗 WS2812 车灯）。**
另有一款可插巡线 / 超声波模块的第二款车，是 GPIO 直驱 H 桥（另有 N32 协控之外的一套电机驱动），车控协议完全不通用：刷本固件后 UART2 空闲高可能让轮子自己转，且它没有车灯。第二款车另有单独维护的适配版，不要与本包混用。

---

## 快速开始（推荐：网页刷机）

1. 用 **Chrome / Edge** 打开根目录的 `AlphaPi_One_XiaoZhi_Car_Radio_Flasher_V1.0.0.html`
   （单文件自包含：两套固件 + 三张设备图 + 二维码全部内嵌，约 17 MB）
2. 页面顶部对照照片确认车型 → **① 选择设备** → 点「连接串口」选对应 COM 口
3. 建议先点「**备份固件**」保存原机 Flash，再点「开始烧录」，等日志出现「烧录完成」
4. **一次只接一块板**：One 和 STEM 的 USB VID/PID 相同，同时插会认错串口
5. 波特率固定 **115200**（ESP32-C3 用其他速率会卡在 "Unable to verify flash chip connection"）
6. `esptool-js` 从 unpkg CDN 加载，**首次打开需要联网**；固件数据不依赖网络
7. 两块板**都要刷本发布包**：ESP-NOW 包结构和信道约定是配套调试的，混用旧遥控器可能方向不响应
8. 刷完打开 `http://<设备IP>/`，点「**打开网页手柄**」即可用网页遥控（详见「网页手柄遥控」章节）

### 烧录模式

| 设备 | 模式 | 说明 |
| --- | --- | --- |
| One (S3) | 分文件（5 个） | 默认**保留 NVS 配置**（WiFi / 服务器 / 激活 / 电台 / 增益）；勾选「擦除整片」才清空 |
| STEM (C3) | 分文件（4 个） | 保留配置，推荐升级用 |
| STEM (C3) | 完整固件（4MB 单文件） | 擦除整片后写入全量包，不保留任何配置，适合全新设备 |

> One 没有单独提供 8MB 全量镜像（其中大半是空白 0xFF），需要清配置时用「擦除整片」+ 分文件写入，效果等价。
> 从旧 V1.0.0 升级只需覆盖 `0x20000` 的 `xiaozhi.bin`，bootloader / 分区表 / 资源分区都不用重刷。

---

## 固件清单

### One 主机（ESP32-S3，8MB Flash，DIO / 80MHz）— v62 基线 + 网页手柄遥控

| 文件 | 地址 | 大小 | md5 |
| --- | --- | --- | --- |
| `bootloader.bin` | `0x000000` | 16,672 | `5f943e3ec00dbea0341ae9dc46760760` |
| `partition-table.bin` | `0x008000` | 3,072 | `948bf2c00af6f84675136371d34bb0f6` |
| `ota_data_initial.bin` | `0x018000` | 8,192 | `84d04c9d6cc8ef35bf825d51a5277699` |
| `xiaozhi.bin` | `0x020000` | 3,330,608 | `285d65b609331b090faf3488adf7364c` |
| `generated_assets.bin` | `0x700000` | 561,001 | `baea5eb459158f8b50e3200e3bfaddc5` |

分区：`nvs` 60KB @0x9000、`otadata` @0x18000、`app` @0x20000、`assets` @0x700000。
app 分区 0x370000，本版占用后剩 0x42DD0（8%）。
只升级主程序时可仅写 `0x8000 / 0x18000 / 0x20000` 三个（板子上已有 bootloader 与 assets）；**全新设备请写全 5 个**。
`bootloader.bin` 相对旧 V1.0.0 有 47 字节差异，仅是本次重新构建写入的编译时间戳，功能一致；旧 bootloader 可直接启动本 app（实机验证时用的就是旧 bootloader）。
`xiaozhi.bin` 已烧录实机验证：COM6 / 115200 / 只写 `0x20000`，`Hash of data verified`。

### STEM 遥控器（ESP32-C3，4MB Flash，huge_app）— V0.2.1

| 文件 | 地址 | 大小 | md5 |
| --- | --- | --- | --- |
| `bootloader.bin` | `0x000000` | 13,200 | `7a15dc571f53efe9b125f3d334517243` |
| `partitions.bin` | `0x008000` | 3,072 | `70007348574201233bc0cb17155e9d12` |
| `boot_app0.bin` | `0x00E000` | 8,192 | `e6327541e2dc394ca2c3b3280ac0f39f` |
| `AlphaPi_Stem_Remote_V0.2.1_app.bin` | `0x010000` | 731,008 | `ad9b6c0e9094c9698e4912656852aad9` |
| `AlphaPi_Stem_Remote_V0.2.1_full_4MB.bin` | `0x000000` | 4,194,304 | `916e7771d9a25bdf718b7074a6cbf906` |

全量包 = 上表前 4 项按偏移拼进 4MB、空白填 `0xFF`，刷它等于「擦除整片 + 分文件写入」。
本版**未改动** STEM 遥控器源码与固件，仍是 V0.2.1。

---

## 目录结构

```
AlphaPi_One_XiaoZhi_Car_Radio/
├── README.md                                         本文件
├── AlphaPi_One_XiaoZhi_Car_Radio_Flasher_V1.0.0.html  ← 双设备网页刷机工具（交付入口）
├── AlphaPi_One_XiaoZhi_Car_Radio_Flasher_V1.0.0.7z    ← 上面这个 HTML 的压缩包
├── AlphaPi_Car.jpg / AlphaPi_One.jpg / AlphaPi_Stem（Zero）.jpg
├── one/
│   ├── firmware/           5 个 bin（见上表；xiaozhi.bin = 网页遥控版 app）
│   ├── src/                V1.0.0 全量源码快照（含 sdkconfig.s3 / CMakeLists.txt）
│   │   ├── main/web_admin/gamepad_page.h   内嵌手柄页（由下方脚本生成）
│   │   └── idf_patches/    ESP-IDF v6.0.2 的三个必需补丁
│   ├── webpad_page/        手柄原型页 + 设备版预览 + make_page.py
│   └── build.info          本版完整构建 / 烧录 / 实机验证记录
├── stem_remote/
│   ├── AlphaPi_Stem_Remote.ino   遥控器源码（单文件 sketch）
│   ├── firmware/           4 个分文件 bin + 4MB 全量包
│   └── build.info          本次编译信息
└── tools/
    ├── flasher_template.html      刷机页模板（占位符版）
    ├── generate_flasher.py        由固件 + 图片重新生成单文件刷机页
    ├── donate_qr.png / group_qr.jpg
```

重新生成刷机页：

```bash
python tools/generate_flasher.py V1.0.0
```

改手柄样式：编辑 `one/webpad_page/gamepad_preview.html` → 运行
`python one/webpad_page/make_page.py` → 把生成的 `gamepad_page.h` 复制到
`one/src/main/web_admin/gamepad_page.h` → 重新编译固件 → 重跑上面的刷机页生成脚本。

---

## 源码与构建

### One 主机：ESP-IDF v6.0.2

`one/src/` 是 V1.0.0（v62 基线 + 网页手柄遥控）的完整源码快照（基于 xiaozhi-esp32，upstream `527604d`），不是 git 提交。

```bash
. ~/esp/esp-idf-v6.0.2/export.sh
idf.py -B build_s3 -D SDKCONFIG=sdkconfig.s3 build
# 新增源文件后先 idf.py reconfigure
```

- **换 IDF 版本必须重打** `one/src/idf_patches/` 里三个补丁：`systimer_hal.c` 有界等 VALUE_VALID、`port_systick.c` / `esp_timer_impl_systimer.c` 的 `can_stall=false`。
- v62 相对 v61 唯一逻辑改动：`main/boards/alphapi-one/alphapi_one_audio_codec.cc`，把增益路径的硬 clamp 换成**固定 512 项 LUT 软限幅**（T=26213, R=6554，逐样本、无状态、纯整数），不会忽大忽小也不会削顶。
- v58 起的音频判据按 **N32 协控真实状态**决定要不要重发 44B WAV 头，不要改回时间阈值。
- Web「偏好设置」里软件增益默认开 180%，建议范围 130~250%。

**V1.0.0 网页遥控版相对 v62 的 4 处改动**（均已编译、烧录、实机验证）：

| 文件 | 改动 |
| --- | --- |
| `main/web_admin/web_admin.h` | 新增两个注册式 hook 类型与 `RegisterWebDriveHook` / `RegisterWebLightsHook`；Web 侧依旧不依赖任何板级头 |
| `main/web_admin/web_admin.cc` | 管理页新增「小车遥控」卡片与 `/gamepad` 入口；`GET /gamepad` 按 1KB 分块流式输出手柄页；新增 `POST /api/drive`、`POST /api/lights`，都过访问密码校验，参数 0~100 越界返回 400；`max_uri_handlers` 10 → 14 |
| `main/web_admin/gamepad_page.h` | **新增**，约 19KB 自包含手柄页（由 `make_page.py` 生成，`R"PADHTML(...)"` 包住），无外部依赖 |
| `main/boards/alphapi-one/car_controller.cc` | `ExpandWebDir()` 把 9 种方向映射成左右轮速（0~100% → 0~1000，差速内侧 40%）；注册两个 hook；独立任务 100ms 巡检、**1 秒无指令自动停车** |

持续运动走 `DriveSpeed()` 绕开带时长的 `Submit` 队列，UART2 写有互斥锁；日志只在轮速变化时打印，不会被 120ms 心跳刷屏。灯光 hook 复用既有 `CarLights` 能力（`SoloAt` / `Turn` / `Blink` / `ReverseLight` / `Identify`，以及自动转向灯 / 倒车灯开关）。

### STEM 遥控器：Arduino / arduino-cli

```bash
arduino-cli --config-file <arduino-cli.yaml> compile \
  --fqbn "esp32:esp32:esp32c3:FlashSize=4M,PartitionScheme=huge_app" \
  --build-path <输出目录> stem_remote/
```

本次编译结果：Sketch uses 714,182 bytes (22%)，Global variables 37,724 bytes (11%)。
遥控器与 One 通过 ESP-NOW 通信：姿态包 `[0xA5 ...]` 10 字节广播，One 回传 `[0x5A dir boost chk]` 4 字节方向包在点阵上画箭头；上电自动扫描并跟随 One 所在的 WiFi 信道，无需配对 MAC。

---

## 命令行烧录（网页工具之外的备用方式）

```bash
# One（ESP32-S3），约 190 秒
esptool.exe --chip esp32s3 -p COM6 -b 115200 write_flash \
  --flash_mode dio --flash_size 8MB \
  0x0      one/firmware/bootloader.bin \
  0x8000   one/firmware/partition-table.bin \
  0x18000  one/firmware/ota_data_initial.bin \
  0x20000  one/firmware/xiaozhi.bin \
  0x700000 one/firmware/generated_assets.bin

# STEM（ESP32-C3）
esptool.exe --chip esp32c3 -p COM3 -b 115200 write_flash \
  --flash_mode dio --flash_size 4MB \
  0x0      stem_remote/firmware/bootloader.bin \
  0x8000   stem_remote/firmware/partitions.bin \
  0xe000   stem_remote/firmware/boot_app0.bin \
  0x10000  stem_remote/firmware/AlphaPi_Stem_Remote_V0.2.1_app.bin
```

- **不要用 `esptool --no-stub run` 复位**，复位用 `--after hard_reset flash_id`。
- 烧录窗口停在下载模式是正常的；连不上就重新插拔 USB，不用按 BOOT。
- 烧 STEM 前先关掉占用 COM3 的串口抓包脚本。
- **不要用全量 `esptool read_flash` 校验 app**：它会把设备拉进下载模式并长时间占串口，看起来像死机。`Hash of data verified` 已经足够；万一卡住，用 `--before default_reset --after hard_reset flash_id` 硬复位即可。

---

## 使用说明

**One 主机**
- 按键：A 单击「下一台 / 说话」，A 长按「收音机暂停」；B 单击「上一台 / 静音」，B 长按 0.9s「切换模式」；L / R 单击音量 ±。
- 配网：**按住 One 上的 A 键再通电、保持按住**，即可重新进入配网模式；手机连设备热点后访问 `192.168.4.1` 配置 WiFi。换路由器 / 改密码 / 小智连不上服务器时用这一招。
- Web 后台：`http://<设备IP>/` 查看状态、改偏好设置，并进入**网页手柄遥控**；IP 由 DHCP 分配**会变**，请以串口日志或实际扫描为准，不要写死。
- 软件增益只改样本值、不改数据量和缓冲，**不是播放卡顿的原因**。

**STEM 遥控器**
- A = 急停，按住 B = 加速档（点阵四角点亮），屏幕箭头显示的是**车实际在做什么**（One 回传）。
- 与 One 连同一个路由器即可；SC7A20 姿态数据做过三次取中值，避免随机跳变。

---

## 网页手柄遥控（V1.0.0 新增）

1. 浏览器打开 `http://<设备IP>/`，在顶部「**小车遥控**」卡片点「**打开网页手柄**」（即 `/gamepad`）。
2. 页面由设备直接提供、打开即自动连上；若管理页设置过访问密码，页面会展开密码框，输入后点「记住」，后续指令自动带 `X-Auth` 头。
3. **左侧十字键**：上前进、下后退、左右原地转向；上+左 / 上+右 可组合成弧线转向，共 9 种方向。右侧 ABXY 是灯光快捷键，中间红色按钮急停。
4. **速度拖动条 0~100%** 实时生效，映射到轮速 0~1000；键盘 `WASD` / 方向键同样可用，空格急停，手机支持多指触摸。
5. 下方灯光面板：关灯 / 前灯 / 尾灯 / 左转 / 右转 / 双闪 / 倒车灯 / 逐颗点名，另有亮度条与「转向自动闪灯 / 倒车自动亮灯」开关。
6. **安全**：按住期间每 120ms 发一次心跳；松手、切走页面、关标签页都立即停车；固件侧另有独立任务，**1 秒收不到指令自动停车**，页面崩了也不会让车一直跑。

接口契约（供二次开发）：

```
POST /api/drive   {"dir":"forward|back|spinleft|spinright|left|right|...|stop","speed":0..100}
POST /api/lights  {"mode":"off|head|tail|left|right|hazard|reverse|identify",
                   "brightness":0..100,"auto_turn":bool,"auto_reverse":bool}
GET  /gamepad     手柄页（1KB 分块流式输出，Cache-Control: no-store）
```

---

## 小智语音指令速查

直接对小智说话即可，不用记工具名。刷机页里也内嵌了同样的速查卡片。

### 启动 / 退出遥控模式（`self.car.motion`）

| 对小智说 | 效果 |
| --- | --- |
| 「启动遥控模式」「用遥控器控车」「体感遥控」 | 打开 ESP-NOW 体感遥控，开始等 STEM 遥控器信号 |
| 「关闭遥控」「退出遥控模式」 | 退出遥控并立即停车 |
| 「校准遥控器」「重新校准零点」 | 以当前平放姿态为中位（约 1 秒，期间保持不动） |
| 「遥控灵敏一点」「阈值调低」 | 调倾斜阈值 / 轴 / 反转 / 轮速（`self.car.motion_tune`） |

前提：**先给 STEM 遥控器上电**，两块板连同一个路由器（信道自动跟随，无需配对 MAC）。
小智若回「遥控接收还没就绪」= WiFi 没连上，先配网。
启动后保持遥控器平放约 1 秒自动校准零点，之后前倾前进、后倾后退、左右倾斜转向、放平就停；
**A 键急停、按住 B 键加速、C 键单击立即校准**；点阵箭头显示车实际在做什么；丢包 0.4 秒自动停车。

### 直接让小智控车 / 控灯

| 对小智说 | 对应工具 / 效果 |
| --- | --- |
| 「前进两秒」「后退」「左转 90 度」「右转」「停车」 | `self.car.move`，时间到自动停；转向约 45°/秒 |
| 「打开自动灯光」「自动打灯」 | `self.car.auto_light`：转弯自动闪转向灯、后退亮倒车灯、停车熄灭 |
| 「车灯亮红色」「车灯闪三下」「关灯」「逐颗点名认灯」 | `self.car.light`：四角 WS2812 独立设色 / 闪烁 / 呼吸 |
| 「开左转向灯」「开倒车灯」 | `self.car.turn_light` / `self.car.reverse_light` |
| 「小车现在什么状态」 | `self.car.status`：动作 / 速度 / 补偿 / 车灯 / 遥控信号 |
| 「走直线有点偏，调一下」 | `self.car.trim`：左右轮补偿 |

直行 `speed=720≈5cm/s`、`1000≈7cm/s`（底盘极速已饱和），走 1 米约需 14 秒。
开过「自动灯光」后，转弯 / 倒车不用再单独喊开灯。

---

## 已知限制

- 协控 N32 恒定按「1 字节 = 1 样本 @24000 B/s」处理 PCM，**8bit 是唯一可用格式**；剩余底噪是 8bit 量化噪声，不改协控固件消不掉。
- 收音机「暂停」按设计是**断连重连**，不是真暂停。
- 播放偶有顿挫多来自源流网络抖动（看串口 `net rx` / `stall`），与软件增益、软限幅无关。
- 深睡时协控 UART 断电，按键无法唤醒；One 已整体去掉休眠，只保留自动息屏。
- 设备 IP 会变；Web 通但按键/音频链路断时先试 `POST /api/reboot`，别急着拔插。
- **网页遥控别与语音 / 体感同时用**：网页持续写电机，语音带时长动作到期时会互相覆盖、短暂停一下。
- 网页遥控的「双闪」是闪 20 次约 8 秒后自动熄灭，不是持续双闪；自动转向灯沿用既有逻辑，只在两轮反向（原地转向）时触发，弧线转向不闪灯。
- 本版**不适配第二款 GPIO 直驱车**（协议不同、无车灯），详见开头硬件适配声明。

---

本项目完全免费 · 自愿捐赠 · 捐赠与否不影响任何功能
