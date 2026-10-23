# NYIST Connect · Android

南阳理工学院校园网认证助手的 **安卓客户端**，由 [GUYU2233/AuthingNYIST](https://github.com/GUYU2233/AuthingNYIST)（复刻自 [Palvef/AuthingNYIST](https://github.com/Palvef/AuthingNYIST)，协议受 [z4yx/GoAuthing](https://github.com/z4yx/GoAuthing) 启发）移植而来。

在校园 Wi-Fi（**nyist-tec / nyist-stu**）下一键完成 srun（深澜）Web 认证：无需账号后缀拼接，填学校提供的完整账号即可；支持后台保活自动重连、连上 Wi-Fi 自动认证、开机自启。

> **遵循 GPL-3.0 许可证**（见 LICENSE）。移植过程保留了上游的协议实现与错误码表； redistribution 时请保留原版权与来源说明。

---

## 功能

| 功能 | 说明 |
| --- | --- |
| nyist-tec / nyist-stu 选择 | 与桌面版一致（仅作配置记忆，Android 不会自动切换 Wi-Fi） |
| 账号密码保存 | `EncryptedSharedPreferences`（Android Keystore 加密），对应桌面版 DPAPI |
| 认证 / 状态检查 / 确认注销 | 与桌面版同一套流程：先查在线 → 防误断他人会话 → challenge → login/logout |
| 后台保活 | 前台服务每 30 秒检查一次，掉线自动重连 |
| 自动连接 | 打开应用或连上校园 Wi-Fi 时自动认证 |
| 开机自启 | 开机后启动保活（需同时开启自动连接） |
| 错误提示 | 移植上游 ecode 中文映射（密码错误、已经在线、IP 绑定错误等） |

### 与桌面版的安全边界差异（有意为之）

- 认证被拒 / 检测到其他账号在线 → 立即停止重试，不自动重试，避免锁号（与上游一致）。
- 暂时性失败连续 5 次（2/4/8/16 秒退避）后停止（与上游一致）。
- **上游差异**：手机离开校园网后没有网络时，保活不计失败、等待网络恢复；桌面版假设网络常开，连续失败即退出。
- 程序**不会**帮你连接/切换 Wi-Fi（Android 系统限制，且需要位置权限）；请先在系统设置里手动连上校园 Wi-Fi，本程序只负责认证。
- 默认服务器 `https://auth.nyist.edu.cn`，`ac_id=1`；默认 HTTPS。

---

## 获取 APK（推荐：云端构建，无需安装任何工具）

1. 在 GitHub 上 **Fork 本仓库**；
2. 打开 fork 仓库的 **Actions** 页，选择 **Android CI** → **Run workflow**；
3. 构建完成后在本次运行页面下载 **NYIST-Connect-debug-apk**；
4. 推送 `v*` 标签（如 `v1.1.0`）时会自动把 APK 挂到 Release。

> debug APK 安装时系统可能提示来源不受信任，属正常现象（未做应用签名公证）。

## 本地构建

**方式一：Android Studio**（Hedgehog 2023.1+，含 JDK 17）

File → Open 打开本项目根目录，等待 Gradle Sync 完成后 Run。若提示缺少 wrapper，执行 File → Sync 或命令行 `gradle wrapper --gradle-version 8.7` 一次即可。

**方式二：命令行**（需 JDK 17 + Gradle 8.7+）

```bash
gradle assembleDebug
# 产物：app/build/outputs/apk/debug/app-debug.apk
```

---

## 使用说明

1. 在系统设置中**手动连上**校园 Wi-Fi（nyist-tec 或 nyist-stu）；
2. 打开应用，选择网络，填写**学校提供的完整账号**（SSID 不等于账号后缀，按学校说明填写，如 `2024xxxxxx@nyist-stu`）和密码；
3. 点击「连接」；需要常驻时打开「后台保活」（首次会请求通知权限）；
4. 「断开认证」有确认对话框，会向服务器发送注销请求；
5. 常用开关：
   - **后台保活**：连接成功后常驻通知栏，每 30 秒检查，掉线自动重连；
   - **自动连接**：下次打开应用或连上校园 Wi-Fi 自动认证；
   - **开机自启**：开机后自动启动保活（需同时开启自动连接）。

**常见问题**

- 提示「无法连接认证服务器」：先确认手机确实连着校园 Wi-Fi 并能打开认证页；
- 提示「其他账号在线」：路由器/校园网里有别的设备用同一账号在线，先断开那台设备；
- 密码错误提示为「密码错误」（E2553）等中文说明，来自上游错误码表；
- 勾选「后台保活」后建议在系统设置里关闭本应用「电池优化」，厂商 ROM 杀后台会导致保活中断。

## 项目结构

```
app/src/main/java/cn/nyist/connect/
├── SrunClient.kt      # srun 协议：get_challenge / srun_portal / rad_user_info
│                      #   + QuirkBase64、XEncode(XXTEA)、HMAC-MD5、SHA1
│                      #   （已通过上游 coding_test.go 全部 15 个测试向量验证）
├── PortalErrors.kt    # 认证错误码中文映射（移植 libauth/messages.go）
├── ConfigStore.kt     # EncryptedSharedPreferences 加密配置存储
├── MainActivity.kt    # 界面：网络选择 / 账号 / 开关 / 状态
├── KeepAliveService.kt# 前台服务保活循环（对应 portal.go Run）
├── WifiJobService.kt  # 后台 Wi-Fi 触发的一次性认证（JobScheduler 加急任务）
├── BootReceiver.kt    # 开机自启
└── App.kt             # Wi-Fi 网络回调：连上校园网自动认证
```

## 来源、致谢与许可证

- 认证协议与错误码表移植自 **GUYU2233/AuthingNYIST**（GPL-3.0），上游为 **Palvef/AuthingNYIST**；
- 协议思路源自 **z4yx/GoAuthing**；
- 本项目以 **GPL-3.0** 分发，详见 [LICENSE](LICENSE)。再分发与修改时请保留原有版权、许可和来源说明。

本项目为非官方工具，与学校或认证服务提供方不存在隶属关系；请仅使用本人或获授权的账号，并遵守校园网管理规定。
