# SceneLoop for Hermes

SceneLoop 是一套面向 AI 漫剧与 AI 短剧生产的智能工作流。用户只需向 Hermes 提供剧本和创作要求，SceneLoop 即可完成剧本视觉化适配、分镜规划、角色与场景资产生成、首帧生成和逐镜视频生成。

本项目由 **西安文鳐网络信息科技有限责任公司** 开发。

[访问 SceneLoop 产品官网](https://www.wenyaotech.com/products?category=aimanju&product=sceneloop)

> 本文档适用于 SceneLoop Hermes Skill 的 macOS 版本。

## 核心能力

- 支持 `.docx`、`.md` 和 `.txt` 剧本。
- 根据用户指定的画面比例、画风、语言、清晰度和目标时长规划项目。
- 自动生成 `storyboard.json`、`characters.json` 和 `locations.json`。
- 自动补齐角色图、场景图、镜头首帧和逐镜视频。
- 支持动画、3D、真人短剧等不同视觉方向。
- 根据相邻镜头关系管理人物、场景和画面连续性。
- 支持按镜头生成、断点续作和失败镜头重试。
- 通过 Hermes 对话、Hermes Desktop 或接入的飞书机器人使用。

## 工作流程

```text
用户上传剧本
  -> 项目初始化
  -> 剧本视觉化适配
  -> 分镜、角色和场景规划
  -> 角色参考图生成
  -> 场景参考图生成
  -> 镜头资产检查
  -> 首帧生成
  -> 逐镜视频生成
  -> 连续性衔接
  -> 视频输出
```

Hermes 负责与用户对话、收集参数和调用 SceneLoop。文本、图片和视频的正式生产必须由 SceneLoop 内部配置的模型完成，Hermes 不会使用自身模型替代生产步骤。

## macOS 安装要求

开始前请准备：

- macOS 14 或更高版本。
- 与 Mac 芯片匹配的 SceneLoop 安装包。
- 已安装并可正常对话的 Hermes Desktop 或 Hermes CLI。
- SceneLoop License Key。
- 文本模型、图片模型和视频模型所需的 API Key。
- 可访问模型服务和 SceneLoop License Server 的网络。

普通用户使用预编译安装包时，不需要安装 Python，也不需要下载 SceneLoop 源码。

### 查看 Mac 芯片架构

打开“终端”，执行：

```bash
uname -m
```

- 返回 `arm64`：使用 `sceneloop-hermes-macos-arm64.zip`。
- 返回 `x86_64`：使用 `sceneloop-hermes-macos-x86_64.zip`。

不同系统和芯片的安装包不能混用。

## 安装 Hermes

推荐先从 [Hermes 官网](https://hermes-agent.nousresearch.com/) 下载 Hermes Desktop，并完成 Hermes Runtime 和 Agent 模型配置。

在安装 SceneLoop 前，请先在 Hermes 中进行一次普通对话，确认 Hermes 能够正常工作。

## 安装 SceneLoop Skill

假设安装包位于 macOS 的“下载”目录，执行：

```bash
mkdir -p "$HOME/.hermes/skills"
unzip "$HOME/Downloads/sceneloop-hermes-macos-arm64.zip" -d "$HOME/.hermes/skills"
chmod +x "$HOME/.hermes/skills/sceneloop/scripts/sceneloop"
chmod +x "$HOME/.hermes/skills/sceneloop/scripts/sceneloop-setup"
```

Intel Mac 请将 ZIP 文件名替换为：

```text
sceneloop-hermes-macos-x86_64.zip
```

检查 Skill：

```bash
hermes skills list
```

安装完成后，请新建一个 Hermes 会话，或在现有会话中执行：

```text
/reset
```

## 首次设置与授权

安装后无需先手动执行一串生产命令。在 Hermes 中上传剧本并提出生成请求时，SceneLoop Skill 会先自动检查运行程序、Redis、License 和模型配置；配置不完整时，Hermes 会自动启动 `sceneloop-setup`。

Setup 将依次完成：

1. 检查 macOS 基础环境。
2. 检查 Redis；本机没有 Redis 时，通过 Homebrew 安装并设置为自动启动。
3. 输入 SceneLoop License Key，完成本机设备绑定。
4. 选择文本模型并输入对应 API Key。
5. 选择图片模型并输入对应 API Key。
6. 选择视频模型并输入对应 API Key。
7. 验证配置并保存到本机。

License Key 和 API Key 只应在本机 Setup 窗口或终端中输入，不要发送到 Hermes 聊天、飞书聊天、群聊或截图中。

### Setup 没有自动打开

可在终端中手动运行：

```bash
"$HOME/.hermes/skills/sceneloop/scripts/sceneloop-setup"
```

检查授权状态：

```bash
"$HOME/.hermes/skills/sceneloop/scripts/sceneloop" license status
```

设备绑定信息会长期保存在本机 Redis 中。运行租约到期后，SceneLoop 会使用已有绑定在线续签，正常情况下不需要再次输入 License Key。

## 第一次生成

在 Hermes 中上传剧本，然后直接说明需求，例如：

```text
请使用 SceneLoop 将这个剧本制作成第一集 AI 漫剧。
```

正式生成前，Hermes 会依次确认：

1. 英文项目 ID 和集数。
2. 画面比例，例如 `16:9` 或 `9:16`。
3. 画风要求；如果用户只说“短剧”，会先确认是否需要真人风格。
4. 台词和旁白语言。
5. 视频清晰度；更高的清晰度通常会产生更高的模型费用。
6. 用户明确提出时，确认目标成片时长。

参数确认完成后，SceneLoop 会自动执行当前项目所需的完整流程。缺少的资产会自动补齐，已经成功生成的资产不会无故重复生成。

## 常用对话示例

### 生成一集竖屏漫剧

```text
请用 SceneLoop 制作这个剧本的第一集。项目 ID 是 city_story，画面比例 9:16，使用高品质 3D 动画风格，台词保持中文，视频清晰度 720p。
```

### 生成真人风格短剧

```text
请用 SceneLoop 制作真人电影质感的竖屏短剧。项目 ID 是 night_case，第一集，比例 9:16，中文台词，视频清晰度 720p。
```

### 继续生成指定镜头

```text
请继续 night_case 第一集第 6 镜，检查已有资产后生成到视频。
```

### 重试失败镜头

```text
请使用 SceneLoop 重试 city_story 第一集生成失败的镜头，不要覆盖已经成功的视频。
```

## 输出文件

SceneLoop 将项目资料和视频分开保存：

```text
references/<project_id>/
  source/                         原始剧本
  scripts_prompts/episode_001/    适配剧本与分镜 JSON
  character_prompts/episode_001/  角色定义与角色参考图
  scene_prompts/episode_001/      场景定义与场景参考图
  shot_prompts/episode_001/       镜头首帧和尾帧
  runtime/episode_001/            运行状态与镜头报告

output/<project_id>/
  episodes/episode_001/
    shot_001.mp4
    shot_002.mp4
```

`references/` 保存剧本、JSON、图片和运行报告，`output/` 只保存视频。

## 常见问题

### `Permission denied`

重新授予程序执行权限：

```bash
chmod +x "$HOME/.hermes/skills/sceneloop/scripts/sceneloop"
chmod +x "$HOME/.hermes/skills/sceneloop/scripts/sceneloop-setup"
```

### macOS 提示程序无法验证或无法打开

确认安装包来自可信的公司交付渠道，然后执行：

```bash
xattr -dr com.apple.quarantine "$HOME/.hermes/skills/sceneloop"
```

### Redis 连接失败

```bash
brew services start redis
redis-cli ping
```

正常结果应为 `PONG`。如果没有安装 Homebrew，请先访问 [Homebrew 官网](https://brew.sh/) 完成安装，然后重新运行 `sceneloop-setup`。

### Hermes 没有调用 SceneLoop

```bash
hermes skills list
```

确认列表中存在 `sceneloop`，然后新建 Hermes 会话或执行 `/reset`。提出任务时明确说明“使用 SceneLoop”。

### License 状态无效

先确认 Redis 正常，再执行：

```bash
"$HOME/.hermes/skills/sceneloop/scripts/sceneloop" license verify
```

如果设备从未激活，重新运行 `sceneloop-setup`。请勿尝试绕过或修改 License 校验。

### 模型返回 401、403 或余额不足

- `401`：通常表示 API Key 无效、过期或配置到了错误的模型服务商。
- `403`：通常表示账号权限、余额或服务开通状态存在问题。
- 修改模型配置后，重新运行 `sceneloop-setup` 完成验证。

### 图片或视频生成超时

模型服务繁忙或网络不稳定时可能发生超时。保留已经成功的资产，让 SceneLoop 重试失败镜头即可，无需从头生成整个项目。

## 安全说明

- 不要在聊天、日志、截图或公开仓库中暴露 License Key 和 API Key。
- 不要修改或绕过 SceneLoop License 校验。
- Hermes 只负责调度，不得使用自身模型替代 SceneLoop 的正式生产模型。
- 安装包不包含任何用户密钥；所有授权与模型配置均在用户本机完成。

## 关于我们

SceneLoop 由 **西安文鳐网络信息科技有限责任公司** 开发并维护。

- 产品官网：[SceneLoop](https://www.wenyaotech.com/products?category=aimanju&product=sceneloop)

Copyright © 西安文鳐网络信息科技有限责任公司. All rights reserved.
