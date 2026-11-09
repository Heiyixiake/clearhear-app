# 听得清 ClearHear

> 戴上耳机，把声音听得更清楚。

**ClearHear（`clearhear-app`）** 是一个零依赖、单文件的 PWA 听力辅助工具：用手机麦克风采集声音，通过浏览器 Web Audio API 做分频段补偿，再用普通耳机实时播放。不用装 App、不用买设备、不用构建工具，打开网页就能用；添加到主屏幕后还能离线打开。

- 项目仓库：`Heiyixiake/clearhear-app` · 当前版本：**v1.5.0**
- 技术栈：原生 HTML / CSS / JavaScript + Web Audio API + Service Worker（无框架、无 npm、无 CDN）
- 关键词：PWA、Web Audio、WDRC 宽动态范围压缩、NAL-NL2 验配、适老化 / 长辈模式、离线可用

---

## ✨ 能做什么

| 能力 | 说明 |
| --- | --- |
| 分频段听力补偿 | 6 段带通滤波器组，每段独立 `DynamicsCompressor` 做 WDRC：小声多放大、大声少放大，而不是一刀切调音量 |
| NAL-NL2 简化验配 | 按 6 个频点的听力损失，用 NAL-NL2 简化权重计算各段目标增益 |
| 左右耳独立 | 左右耳各一套 EQ 与增益，按双耳听力图分别补偿 |
| 6 频点测听 | 250 Hz 起的纯音测听，左右耳分别测，按 dB HL 分级（≤25 / 26–40 / 41–60 / 61–80）；也可「跳过听力测试」用模拟数据体验 |
| 麦克风降噪 | 通过 `getUserMedia` 的 `noiseSuppression` 约束实现（界面标为「AI 降噪」），iOS 只使用标准约束 |
| 啸叫保护 | 陷波滤波 + 瞬态限幅，抑制耳机靠近麦克风产生的啸叫（默认关闭，可在设置里开启） |
| 增益 / 延迟调节 | 0–10 档增益滑块与 ± 按钮，实时显示音频延迟毫秒数 |
| 双耳实时频谱 | 输入 / 输出 / 左右耳 `AnalyserNode` 驱动的可视化柱条 |
| 长辈模式 | 三档字号（标准 / 大 / 超大）、高对比次要文字、≥60px 触控、整宽主按钮、白话文案、语音提示 |
| 离线 PWA | `manifest.json` + Service Worker：导航 network-first、静态资源 cache-first，可全屏运行 |

---

## 🚀 快速开始

```bash
git clone https://github.com/Heiyixiake/clearhear-app.git
cd clearhear-app
python3 -m http.server 8765 --bind 127.0.0.1
```

浏览器打开 <http://127.0.0.1:8765>。

> ⚠️ **必须通过 localhost 或 HTTPS 打开。** 直接双击 `index.html`（`file://`）或用局域网 IP 的 `http://` 访问时，浏览器不会授予麦克风权限，Service Worker 也不会注册。真机使用请部署到 HTTPS 地址。

### 使用步骤

1. 插好**有线耳机**（蓝牙耳机延迟高，容易产生回声）
2. 把手机媒体音量调到 **50% 左右**
3. 允许麦克风权限
4. 完成 6 频点听力测试，或点「跳过听力测试」先用模拟数据看效果
5. 点「开始助听」，在设置页用增益滑块从低到高调到舒服为止

> 模拟数据只是用来体验界面，**不是你的真实听力图**，不要据此判断听力状况。

---

## 📱 安装到手机

- **Android / Chrome**：浏览器菜单 →「安装应用」/「添加到主屏幕」
- **iPhone / Safari**：分享 →「添加到主屏幕」（需先部署到 HTTPS）

安装后以 standalone 全屏运行，使用 A 耳廓声波图标；离线时由 Service Worker 回落缓存（`clearhear-v1.5.0`）。页面导航优先走网络，保证改版后能及时看到新版本；静态资源优先走缓存，保证离线可用；更新过程不再弹确认框或自动刷新，避免打断正在进行的助听。

---

## 🧩 怎么做出来的

**整个应用只有一个 HTML 文件。** `index.html` 内联了全部 CSS、DOM 与 JS；`manifest.json`、`sw.js`、`icons/` 是仅有的外部文件。没有 `package.json`、没有构建步骤、没有第三方库或组件库。

音频链路（简化）：

```text
getUserMedia → AudioContext(44.1 kHz, latencyHint≈0)
  → 高通 / 陷波 / 低通 + 瞬态限幅
  → 6 段带通滤波器组，每段 DynamicsCompressor 做 WDRC
       增益 = NAL-NL2 简化权重 × 该频点听力损失
  → 左右耳独立 EQ → ChannelMerger(2)
  → 输出限幅保护 → 耳机
  ↳ AnalyserNode × N → 双耳实时频谱
```

真机兼容上做了几件不那么「教科书」的事：

- iOS Safari 只认标准 `noiseSuppression`，Chrome 才会用 `googNoiseSuppression` 等私有约束，代码里按平台分支处理；
- 21.5 kHz 正弦振荡器作为后台保活信号，配合启动版本号与锁屏恢复逻辑，避免锁屏后音频链路被系统挂起或重复启动；
- 麦克风去重、启动竞态、按钮状态即时响应、PWA 缓存版本 bump 等都做过真机向的修复（详见原仓库提交历史）。

**UI** 是手写仿 iOS HIG：`-apple-system / PingFang SC` 字体栈、`#007AFF` 系统蓝、20px 卡片圆角、0.5px 分隔线、`cubic-bezier(0.32, 0.72, 0, 1)` 缓动、`env(safe-area-inset-bottom)` 安全区。右下角那个「扇形」按钮其实是一个 430×430 的圆，把圆心钉在容器右下角再用 `overflow:hidden` 裁出 1/4 扇区；增益滑块是原生 `<input type="range">` 换皮，蓝色填充由 JS 写 `linear-gradient` 实现。

**长辈模式**不是另一套页面，而是同一套 iOS 视觉语言的大字号版。开启时不刷新、不中断助听：`WeakMap` 存下每个文本节点的原文，`MutationObserver` + `requestAnimationFrame` 增量捕获动态文案并替换成白话说法，退出时逐节点原样还原；同时补齐 `aria-live`、`focus-visible`、`role="dialog"`、`prefers-reduced-motion` 与语音播报（`speechSynthesis`）。

---

## 📁 目录结构

```text
clearhear-app/
├── index.html       # 应用全部代码（CSS + DOM + JS）
├── manifest.json    # PWA 清单、名称与图标声明
├── sw.js            # Service Worker：离线缓存与更新策略
├── icons/           # 192/512/maskable/apple-touch/favicon 及 SVG 源稿
├── .gitignore
└── README.md
```

---

## ❓ 常见问题

| 现象 | 原因 / 处理 |
| --- | --- |
| 点「开始助听」没反应 | 检查是否用 localhost / HTTPS 打开、麦克风权限是否允许 |
| 声音发尖、有啸叫 | 戴好耳机、调低增益，或在设置里开启啸叫保护 |
| 感觉延迟明显 | 换有线耳机；设置页可查看实时延迟数值 |
| 锁屏后声音断了 | 重新进入页面，锁屏恢复逻辑会重建音频链路 |
| 图标或名字还是旧的 | Service Worker 缓存未刷新；版本已 bump 到 `clearhear-v1.5.0`，重开一次或清除站点数据 |
| 双击或双指会把界面放大缩小 | 已禁用误触缩放；需要更大字请用长辈模式的三档字号，系统级辅助功能放大依然可用 |

---

## ⚠️ 免责声明

本项目是**个人实验性工具，不是医疗器械**，不提供听力诊断，也不能替代专业验配的助听器。如有听力下降、耳鸣、耳痛，请及时就医。使用时请从低增益开始，避免音量过大造成不适或听力损伤；请勿在驾驶、过马路等需要感知环境音的场合依赖本工具。

---

## 🔗 版本与来源

基于 [Heiyixiake/heardit-app](https://github.com/Heiyixiake/heardit-app) v1.4.56 的品牌与适老化改版：更名为「听得清 ClearHear」、换用 A 耳廓声波图标、新增长辈模式、补齐 PWA 图标与缓存策略，当前 **v1.5.0**。原仓库保留改版前的代码与完整提交历史，本仓库是可独立部署的改版版本。
