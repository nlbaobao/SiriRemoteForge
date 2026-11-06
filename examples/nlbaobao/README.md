# nlbaobao 的遥控器配置

从 2026-11-07 的本机配置整理，配套 SiriRemoteForge / HyperVibe。
使用设备为 USB-C 第三代 Siri Remote（A2854），配置来源版本为 v0.2.0-beta.9。

## 两份完整配置

- [general.jsonc](general.jsonc)：通用配置，所有应用使用同一套映射。
- [interview-aibox.jsonc](interview-aibox.jsonc)：包含完整通用配置，并在 Interview AiBox 位于前台时自动启用专属映射；切换到其他应用时恢复通用配置。日常同时使用两套映射时选择这一份即可。

两份文件均可直接作为 `~/.config/siriremote/config.jsonc` 使用，不需要将片段手动合并。
它们不包含密码、API Key、遥控器地址或序列号。

## 通用映射

- 电源键：打开已安装的 ChatGPT。
- 返回键：退格删除；长按连续删除。
- Siri 侧键：按住触发微信输入法语音，松开结束；映射目标是 `Fn + Space`。
- 电视键：打开 macOS 的 Apps 应用中心。
- 音量加／减：上一页／下一页（Page Up / Page Down）；长按连续翻页。
- 触控移动、中央点击和外圈滚动沿用 HyperVibe 的行为。

音量键这里用于翻页，不用于调整音量。Page Up / Page Down 是整页滚动，实际响应取决于当前获得焦点的控件。

## Interview AiBox 专属映射

应用标识为 `com.interview.aibox`，模式为 `interview_aibox`，继承 `global`：

- 中央确定键：`Command + Enter`。
- 圆环上／下：上一页／下一页；长按连续翻页。
- 圆环左／右：左／右方向键，用于移动输入光标；长按连续移动。
- 播放／暂停键：鼠标左键，补充中央键改为确认后的点击能力。
- 返回键、Siri 键、电源键、电视键和音量键继续使用通用映射。

这些是发送给前台应用的快捷键，不代表所有 Interview AiBox 页面都采用相同的确认或翻页行为。

## 安装或切换

在仓库根目录执行。先备份已有配置，再任选一份复制：

```sh
mkdir -p "$HOME/.config/siriremote"
if [ -f "$HOME/.config/siriremote/config.jsonc" ]; then
  cp "$HOME/.config/siriremote/config.jsonc" \
    "$HOME/.config/siriremote/config.jsonc.backup-$(date +%Y%m%d-%H%M%S)"
fi

# 通用 + Interview AiBox 自动切换
cp examples/nlbaobao/interview-aibox.jsonc "$HOME/.config/siriremote/config.jsonc"

# 如果只需要通用配置，改用这一条：
# cp examples/nlbaobao/general.jsonc "$HOME/.config/siriremote/config.jsonc"
```

HyperVibe 会热加载配置。保持它运行，并启用辅助功能、输入监控。
如果音量键同时触发翻页和系统音量，先正常退出并重新打开 HyperVibe，
确认只有一个运行实例；这曾在本机恢复媒体按键拦截，但不保证能解决所有同类问题。

## 微信输入法语音

微信输入法中需要启用“启动语音输入”，快捷键为 `Fn + Space`，
并启用“快捷键全局生效”。配置使用 `pushToTalk`：按住约 0.2 秒后发送开启快捷键，
松开再发送一次以结束输入。

HyperVibe 自带的云端语音功能保持关闭，无需向本配置填写 API Key。
本机设置使用 **MacBook Pro 麦克风**收音；这两份配置不会安装遥控器麦克风组件，
也不会将 Siri Remote 自动变成系统麦克风。微信输入法的麦克风选择与权限需在其设置中管理。

## 电源键和系统版本

本配置的应用中心路径是 `/System/Applications/Apps.app`，面向 macOS 26 或更高版本；
旧系统需要将该项改成当地可用的应用启动器。

电源键映射需要按上游说明关闭 macOS 的短按电源键睡眠行为，
否则可能在打开 ChatGPT 的同时触发睡眠／锁屏：

```sh
sudo defaults write /Library/Preferences/com.apple.loginwindow PowerButtonSleepsSystem -bool false
```

这是系统级设置，**同时影响 Mac 机身电源键的短按睡眠行为**，不能只理解成遥控器设置。
先记录原值；需要撤销时恢复原值。如果原先没有这一项，可删除该项恢复系统默认：

```sh
sudo defaults delete /Library/Preferences/com.apple.loginwindow PowerButtonSleepsSystem
```

该系统设置不会由复制 JSON 文件自动应用。配置也不会安装软件、开启登录启动或自动授予权限。

## 验证边界

导出时检查两份文件的 JSON、上游 ConfigLoader 解析、模式继承与按应用切换规则。
此前在本机确认了遥控器配对、指针移动、中央点击、外圈滚动，以及重启 HyperVibe 后音量键仅翻页。
微信语音的完整按住／松开流程和 Interview AiBox 每个页面的快捷键效果仍需按实际应用版本验证，
不将配置解析通过等同于全部硬件操作已验证。

上游项目与许可说明见仓库根目录；本目录只提供配置，不改应用源码。
