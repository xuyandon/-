# 北宋报时 · 晨钟暮鼓 · 五更梆子

> 以乐司时，按时而眠 —— 按当地日出日落，自动排定钟鼓梆声的安卓应用

[![Android](https://img.shields.io/badge/Android-8.0%2B-3DDC84?logo=android)](https://developer.android.com/)
[![Kotlin](https://img.shields.io/badge/Kotlin-1.9.24-7F52FF?logo=kotlin)](https://kotlinlang.org/)
[![minSdk](https://img.shields.io/badge/minSdk-26-brightgreen)](https://developer.android.com/studio/releases/platforms)
[![targetSdk](https://img.shields.io/badge/targetSdk-34-blue)](https://developer.android.com/studio/releases/platforms)
[![License](https://img.shields.io/badge/License-MIT-yellow)](#license)

## 为什么做这个

> 古之「樂」與「藥」本為一字之變——繁體字里，樂字添上一叢草，便是藥。古人早已明白：**音樂是治心的藥**。
> 鐘鼓司時、五更報更，晨鐘催醒、暮鼓安神，人循著天地節律起居作息，身心自然安頓。
>
> 這個應用想做的，就是把被現代人遺忘的這套「以樂司時、按時而眠」的古代睡眠制度，重新裝進手機：
> **太陽在哪，聲音就在哪**——日出晨鐘喚醒，日落暮鼓入夜，夜裡五更梆聲報更，讓一天與日月同步。
> 現代人的失眠與紊亂，往往始於與天時的脫節；願鐘鼓之聲，能如古時良藥，助你找回安穩的睡眠，調養身心。

夜自天黑起更，至天亮五更止；日出敲钟、日中敲钟、日落击鼓；交五更时诸寺院行者打木鱼铁牌，循门报晓 —— 本应用把《东京梦华录》里记载的北宋京城报时制度，装进你的手机。

---

## ✨ 设计理念

### 1. 天文驱动，不靠固定时刻表
内置 **SunCalc 天文算法**（Kotlin 逐行移植），根据**经纬度 + 日期**精确计算当地每天的
天亮、日出、正午、日落、天黑，再据此排定当日全部报时：
晨钟在日出、暮鼓在日落、五更在天黑→天亮之间均分五段、报晓在天亮前 40 分钟。
换地点、换季节，报时时刻自动跟随太阳走。

### 2. 报时制度，忠实还原
- **晨钟暮鼓**：日出敲钟、日落击鼓（不设开关，报时之本）
- **五更五点**：天黑起更，天亮止更；更首鼓/梆一声，更内点钟四记（共 5 更 × 5 点）
- **交五更报晓**：天亮前 40 分钟，木鱼铁牌循门报晓
- **自定义报时**：固定时刻，或基于天亮/日出/正午/日落/天黑 ± 偏移

### 3. 零后台常驻，到点高优先级响铃
这是本应用最核心的工程理念 —— **不做一个"流氓常驻"的闹钟**：

```
平时：应用零进程、零耗电
        └── 打开一次应用（或每日换日点自动触发）→ 把当天全部报时时刻
            交给系统闹钟（AlarmManager.setAlarmClock）保管
            闹钟注册在系统进程里，应用被杀 1000 次都不影响

到点：系统唤醒应用 → 前台服务 → 闹钟流（STREAM_ALARM）高优先级播放
        └── 音频焦点 + 唤醒锁 → 播完自动退出，恢复零进程
```

- `setAlarmClock`：系统闹钟语义，Doze 免打扰、用户可见、**安卓 8–15 全程可用**
- 精确闹钟权限：声明 `USE_EXACT_ALARM`（安卓 13+ 安装即授予）+ `SCHEDULE_EXACT_ALARM`，
  并内置降级兜底（无权限时改用 `setAndAllowWhileIdle`，**任何情况不崩溃**）
- **开机自动续排**：`BOOT_COMPLETED` 接收器在重启后自动重排当天闹钟，无需再打开应用
- **每日自动换日**：次日天亮后 5 分钟的系统闹钟触发重排，每天自动更新，永不漏报
- 设置页内置「精确闹钟权限」「电池优化白名单」一键引导，对抗各厂商省电策略

### 4. 声音，响度归一
8 段内嵌音频（鸡鸣、晨钟、暮鼓、梆子、锣、木鱼、三清铃、铜磬）经 **EBU R128 两遍线性归一化至 -16 LUFS**，
各声响起伏一致；长尾钟鼓截短，单次报时干净利落。播放走**闹钟流**（独立音量通道），
音量 = 总音量 × 单声音音量，可分别调节。

---

## 📱 功能

| 页面 | 内容 |
|---|---|
| **现在时间** | 大时钟、时辰（子丑寅卯…）、五更状态、昼夜/黎明/黄昏判定、下次报时倒计时、当日日出入、报时记录 |
| **报时安排** | 自动报时开关、总音量、当日全部钟鼓梆声清单（白昼/夜五更分组，可展开报点）、逐条试听、立即重排 |
| **试听** | 8 种声音音板，独立试听 |
| **设置** | 见下 |

**设置页（全部为 App 内页面）**

- **地点与日出入 · 可手动设置**：8 城预设（汴京/西京/临安/长安/燕京/南京/广州/成都）+ 手动经纬度 + 定位；
  选城市即生效，并**实时显示该地点的天亮/日出/正午/日落/天黑**计算结果
- **报时选项**：自动报时、报点、更声（鼓/梆子）、点声（钟/锣）、报晓/开市/报午开关
- **声音管理**：每声独立音量、替换音频（文件选择器）、恢复默认、新增/删除自定义声音
- **报时序列编辑**：任意报时的"声音 × 击数 × 间隔"序列编辑，恢复默认
- **自定义报时**：固定时刻或太阳时刻 ± 偏移，增删改
- **后台与保活**：精确闹钟权限、电池优化白名单、通知权限、开机自启引导
- **数据备份**：配置 + 替换音频导出/导入（JSON，与网页版格式兼容）

---

## 🖼 截图

| 现在时间 | 报时安排 | 试听 | 设置 |
|---|---|---|---|
| ![](screenshots/clock.png) | ![](screenshots/schedule.png) | ![](screenshots/preview.png) | ![](screenshots/settings.png) |

> 截图暂缺：安装 `北宋报时-v1.3.apk` 后自行截图，放入 `screenshots/` 目录即可。

---

## 🛠 技术栈

- **语言**：Kotlin 1.9.24（纯 JVM 核心逻辑，可单元测试）
- **UI**：AndroidX + Material Components（自绘古风配色：宣纸底、朱砂红、黛金）
- **调度**：`AlarmManager.setAlarmClock` 精确闹钟 + 前台服务（`specialUse` 类型）+ 广播接收器
- **天文**：SunCalc 算法（MIT）移植，含极昼/极夜兜底
- **音频**：assets 内嵌 MP3，`MediaPlayer` 播放，`AudioManager` 音频焦点管理
- **兼容**：minSdk 26（安卓 8.0）/ targetSdk 34，单 Activity + Fragment 架构

## 📁 项目结构

```
app/src/main/java/com/beisong/baoshi/
├── SunCalc.kt            # 日出日落天文算法（纯 JVM）
├── ScheduleEngine.kt     # 报时排程引擎：晨钟暮鼓/五更五点/自定义（纯 JVM，单测覆盖）
├── Model.kt              # 配置与数据模型
├── Prefs.kt              # SharedPreferences + JSON 持久化
├── ScheduleManager.kt    # 系统闹钟调度（含精确闹钟降级兜底）
├── AlarmReceiver.kt      # 报时闹钟接收器 → 启动前台服务
├── BootReceiver.kt       # 开机/时间变更自动重排
├── StopChimeReceiver.kt  # 通知栏停止按钮
├── ChimeService.kt       # 报时前台服务（闹钟流播放/音频焦点/唤醒锁）
├── AudioPlayer.kt        # 音频播放封装
├── MainActivity.kt       # 单 Activity：三标签页
└── ui/                   # ClockFragment / ScheduleFragment / PreviewFragment / SettingsActivity
app/src/main/assets/sounds/   # 8 段归一化音频
app/src/test/.../ScheduleEngineTest.kt  # 排程引擎单元测试（7 项）
```

## 🔨 构建

```bash
# 环境：JDK 17 + Android SDK 34 + Gradle 8.7
gradle assembleRelease
# 产物：app/build/outputs/apk/release/app-release.apk
# 单元测试：
gradle testDebugUnitTest
```

> 项目路径含非 ASCII 字符时需在 `gradle.properties` 保留 `android.overridePathCheck=true`。

## 🧪 测试

排程引擎 7 项单元测试全部通过，包括**真实世界校验**：
北京 2024-06-21 计算日出应为 **04:46**，与天文台数据吻合。

## 📲 安装与权限

- 安卓 **8.0（API 26）及以上**，targetSdk 34
- 首次打开即自动排定当日闹钟；此后每天换日点自动续排
- 建议到「设置 → 后台与保活」完成：① 精确闹钟权限（安卓 12+）② 电池优化白名单
- 小米/华为/OPPO/vivo/三星等 ROM 建议在系统设置中允许自启动

## 🏛 报时制度考据

> 北宋承古制置钟鼓司时——夜自天黑起更，至天亮五更止；每更分五段：更首以鼓（或梆）为节，
> 更内点钟四记，闻声知更知点。日出敲钟、日中敲钟、日落击鼓；交五更时，诸寺院行者打铁牌子
> 或木鱼循门报晓（《东京梦华录·天晓诸人入市》）。

## 🙏 致谢

- **SunCalc**（MIT）—— 日出日落天文算法
- 网页版原型（本仓库 `index.html`）—— 本项目由其改造而来，保留全部计时逻辑与内嵌音频

## 📄 License

[MIT](LICENSE)

---

*晨钟暮鼓，五更梆子 —— 让手机替你守更。*
