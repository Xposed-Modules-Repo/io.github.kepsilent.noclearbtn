# NoClearBtn · 移除最近任务一键清除按钮

把 ColorOS 最近任务面板的**「一键清除」按钮**直接干掉。 从 [fentia/RecentsNoClear](https://github.com/fentia/RecentsNoClear)（GPL-3.0）重写而来的独立 LSPosed 模块。

**机制级兼容** · 无版本白名单 · **3.6KB** · 欢迎实测机型反馈

## 特性

- **隐藏「一键清除」按钮**：hook `OplusClearAllPanelView#inflateIfNeeded`（回退 `onFinishInflate`），按字段类型定位 `PressFeedbackButton` 并以 `setAlpha(0)` 隐藏，不破坏面板其余布局

- **只 hook 一个类**：不碰点击回调、不调系统接口；最近任务面板的滑动/长按/多任务操作保持原生路径

- **启用即生效**：无界面、无设置、无桌面图标，静态作用域预加载；模块启用后无需任何配置

- **3.6KB**：dex 3.0KB + 最小清单 + 三件套；零死重（无 Kotlin/Compose/androidx）

## 安装与使用

1. 手机已装 **LSPosed 2.x**（libxposed API 102）或其它兼容框架

2. 安装 APK，在 LSPosed 中启用本模块

3. **作用域勾选系统桌面**：`com.android.launcher`、`com.oppo.launcher`，重启生效

4. 打开最近任务，清除按钮应已隐藏

遇到问题？提交 [Issue](https://github.com/Kepsilent/noclearbtn-module/issues)

## 兼容性

- 机制与**系统版本无关**（代码无版本白名单）：只要系统仍保留 `OplusClearAllPanelView` 类名与 `PressFeedbackButton` 字段结构即生效

- 逻辑基线：基于 ColorOS 最近任务面板原生结构，反编译核对

- 若后续系统版本改动类名或字段结构，hook 失效仅记日志，不影响系统

- 改机有风险，请先完整备份

## 构建

需要 JDK 17+、Android SDK（platform 34）、libxposed API 102。

```
export ANDROID_HOME=/path/to/android-sdk
./build.sh
# 产物：release/NoClearBtn-v1.0.apk
```

构建链：`javac --release 8 → R8 → aapt2 link → zip 重组 → 自签 v2`（脚本见仓库根目录 `build.sh`，目标 4KB 级体积）。

## 验证

```
apksigner verify --verbose release/NoClearBtn-v1.0.apk
# 期望：Verifies · Verified using v2 scheme: true
```

## 原理

ColorOS 最近任务面板由 `OplusClearAllPanelView` 承载清除按钮。本模块在面板视图装载完成前：

1. hook `inflateIfNeeded`（回退 `onFinishInflate`），在装载后按字段类型定位 `com.android.launcher.views.PressFeedbackButton`；

2. 对其 `setAlpha(0)`，按钮透明隐藏，不参与点击。

`setAlpha` 为按钮自身方法，字段类型匹配规避混淆差异，逻辑与原版一致，仅去掉热重载输送与冗余日志。

## 目录结构

```
app/src/main/
├── AndroidManifest.xml      # 仅 label（无 Activity/Provider/图标）
├── java/io.github.kepsilent.noclearbtn/
│   └── NoClearBtnHook.java  # 唯一的 hook（极简单文件）
└── res/xposed/              # java_init.list / module.prop / scope.list
build.sh                     # 4KB 级一键构建
```

## 体积演进

| 版本 | 体积 | 说明 |
|---|---|---|
| fentia/RecentsNoClear v1.0 | 16.8 KB | 原版，含热重载输送与冗余 fallback |
| **NoClearBtn v1.0** | **3.6 KB** | 单点 hook 重写 + R8 收缩 + 紧凑 v2 签名（−76%） |

## 许可

- Hook 逻辑与实现源自 [fentia/RecentsNoClear](https://github.com/fentia/RecentsNoClear)，**GPL-3.0**

- Banner 视觉遵循 [EGD（Elegant Geek Design）](https://github.com/Kepsilent/Elegant-Geek-Design) 设计规范
