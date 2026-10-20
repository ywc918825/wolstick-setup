# 手机扫码配网页(Web Bluetooth)

远程开机棒的**免装 App 配网入口**。手机扫机身二维码 → 浏览器打开 → 填几项 →
直接把配置写进设备。**全程不用装任何 App、不用开路由器后台、不用碰目标台式机。**

## 用法

1. 扫机身二维码(或直接打开下面的地址):
   ```
   https://ywc918825.github.io/wolstick-setup/?w=WOL1:<设备MAC>:<配网令牌>
   ```
   地址里的 `?w=` 会被网页自动解析,**配网令牌自动预填**,用户不用记。
2. 点「🔗 连接蓝牙设备」→ 在系统弹窗里选名字以 `WOL-` 开头的那台。
3. 填 4 项:Wi-Fi 名称、Wi-Fi 密码、目标电脑的 MAC、目标电脑的 IPv4。
   - 目标电脑的 MAC/IP 在**路由器后台的「已连接设备 / DHCP 客户端列表」**里能直接看到,
     不用打开那台电脑。
   - 云端私钥/主题**默认折叠、默认不填** —— 设备里已经配好的话,不动它。
4. 点「写入配置到开机棒」,页面会一步步显示:校验令牌 → 写入 → 读回校验 → 等待落盘。
5. 配完后手机上装**巴法云官方 App**,用同一个账号登录,添加设备(主题名填
   `wolnddz99006`,类型选「开关」),之后点「开」就是远程开机。

## 平台支持

| 平台 | 能否用本网页 |
|---|---|
| Android + Chrome / Edge | ✅ 可以 |
| 桌面 Chrome / Edge | ✅ 可以(Windows 自带蓝牙即可) |
| **iPhone + Safari** | ❌ **不支持 Web Bluetooth**(Apple 至今未实现) |

iOS 用户请改用电脑端脚本(在任意一台 Windows 电脑上跑一次即可):

```powershell
cd tools
python -m pip install bleak
python ble_provision.py provision --token <令牌> --ssid <Wi-Fi> --pass <密码> --mac <电脑MAC> --ip <电脑IP>
```

## 技术说明

- 单文件、零依赖、零构建。改完直接推到 GitHub Pages,所有用户立刻生效。
- 协议与固件 `sketches/phase4_cloud/ble_prov.h` 一一对应:

  | 特征 | UUID 尾号 | 属性 | 说明 |
  |---|---|---|---|
  | service | `...ad01` | — | `7c4fad01-0000-1000-8000-00805f9b34fb` |
  | ssid | `...ad02` | R/W | 家里 Wi-Fi 名称 |
  | pass | `...ad03` | W | Wi-Fi 密码(永不回显) |
  | mac | `...ad04` | R/W | 目标电脑有线网卡 MAC |
  | ip | `...ad05` | R/W | 目标电脑 IPv4 |
  | uid | `...ad06` | R/W | 巴法云私钥 |
  | topic | `...ad07` | R/W | 巴法云主题(须以 006 结尾) |
  | status | `...ad08` | R/Notify | 状态 JSON(落盘确认靠它) |
  | ctrl | `...ad09` | W | `wol` / `status` 等控制命令 |
  | auth | `...ad0a` | W | 配网令牌(**必须先写它**) |

- 写入顺序固定:`auth`(令牌)→ 其余特征。固件在每次写入后会把特征值拉回**真实配置**
  (`syncChars()`),所以「读回校验」读到的是固件真正存下的值,不是蓝牙栈里的残留。
- 网页要求 **HTTPS**(安全上下文)。GitHub Pages 自带 HTTPS,`http://` 或 `file://`
  打不开蓝牙。
- 落地时的判定链:写完 → 逐项读回比对 → 等设备推 `{"e":"cfg","v":"saved"}` → 才报成功。

## 改了固件/协议怎么办

- 改**配网网址**:改 `sketches/phase4_cloud/secrets.h` 里的 `WOL_SETUP_URL`,
  重新烧录,再用串口 `qr` 命令拿到新的二维码内容。
- 改**网页**:直接改 `index.html`,`git commit && git push` 即可,无需重新部署任何东西。
- 改**特征 UUID**:网页里的 `CH{}` 表和固件 `ble_prov.cpp` 里的 `CHR_*` 要同时改。

## 仓库

- 网页源码:https://github.com/ywc918825/wolstick-setup
- 线上地址:https://ywc918825.github.io/wolstick-setup/
