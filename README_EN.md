# SceneLoop

[简体中文](README.md) | [English](README_EN.md)

SceneLoop is an intelligent workflow for producing AI comics and AI short dramas. Give a script and creative requirements to an AI agent such as Hermes or OpenClaw, and SceneLoop handles visual adaptation, storyboard planning, character and location assets, first frames, and shot-by-shot video generation.

SceneLoop is developed by **Xi'an Wenyao Network Information Technology Co., Ltd.**

[Visit the SceneLoop product website](https://www.wenyaotech.com/products?category=autodrama&product=sceneloop)

> This guide covers the macOS edition of the SceneLoop Skill.

## Features

- Accepts `.docx`, `.md`, and `.txt` scripts.
- Plans projects from the requested aspect ratio, visual style, language, resolution, and target duration.
- Generates `storyboard.json`, `characters.json`, and `locations.json` automatically.
- Creates missing character images, location images, shot first frames, and videos.
- Supports animation, 3D, live-action, and other visual directions.
- Maintains character, location, and visual continuity between adjacent shots.
- Supports shot-level generation, resumable runs, and retries for failed shots.

## Workflow

```text
Upload script
  -> Initialize project
  -> Adapt script for visual production
  -> Plan storyboard, characters, and locations
  -> Generate character references
  -> Generate location references
  -> Check shot assets
  -> Generate first frames
  -> Generate shot videos
  -> Maintain continuity
  -> Output videos
```

Hermes or OpenClaw handles the user conversation, collects parameters, and invokes SceneLoop. SceneLoop's configured models perform the actual text, image, and video production; the agent must not substitute its own models for these production steps.

## macOS Requirements

Before you begin, prepare:

- macOS 14 or later.
- A prebuilt SceneLoop directory matching your Mac architecture. This repository provides the Apple Silicon (`arm64`) version.
- Git installed and available from Terminal.
- A working Hermes Desktop, Hermes CLI, or OpenClaw installation.
- A SceneLoop License Key.
- API keys required by your selected text, image, and video models.
- Network access to the model services and SceneLoop License Server.

Users of this prebuilt distribution do not need to install Python.

### Check Your Mac Architecture

Open Terminal and run:

```bash
uname -m
```

- If the result is `arm64`, you can use the prebuilt binaries in this repository.
- If the result is `x86_64`, obtain the corresponding Intel build; the current `arm64` binaries are not compatible.

Packages for different operating systems and architectures are not interchangeable.

## Install Hermes

Download Hermes Desktop or Hermes CLI from the [Hermes website](https://hermes-agent.nousresearch.com/), then configure the Hermes Runtime and agent model.

Before installing SceneLoop, start a normal conversation in Hermes to confirm that Hermes works correctly.

## Install OpenClaw

If you use OpenClaw, follow the [OpenClaw getting-started guide](https://docs.openclaw.ai/start/getting-started) to install and configure it. Confirm that this command works:

```bash
openclaw --version
```

You may use either Hermes or OpenClaw; installing both is not required.

## Install the SceneLoop Skill

### Install in Hermes

This GitHub repository already contains the extracted Skill directory. Run:

```bash
mkdir -p "$HOME/.hermes/skills"
git clone https://github.com/WingYouth/sceneloop_mac.git "$HOME/.hermes/skills/sceneloop"
chmod +x "$HOME/.hermes/skills/sceneloop/scripts/sceneloop"
chmod +x "$HOME/.hermes/skills/sceneloop/scripts/sceneloop-setup"
```

Check the Skill:

```bash
hermes skills list
```

After installation, start a new Hermes conversation or run:

```text
/reset
```

### Install in OpenClaw

OpenClaw can install this extracted Skill directly from GitHub:

```bash
openclaw skills install git:WingYouth/sceneloop_mac --as sceneloop --global
openclaw skills list
```

OpenClaw installs a global Skill in its managed Skill directory. Start a new OpenClaw conversation after installation so it loads the updated Skill list. See the [official OpenClaw Skills documentation](https://docs.openclaw.ai/tools/skills) for current commands and directory rules.

## First-Time Setup and Licensing

You do not need to run production commands manually before using the Skill. When you upload a script and request generation in Hermes or OpenClaw, SceneLoop first checks the runtime, Redis, license, and model configuration. If setup is incomplete, the agent launches `sceneloop-setup`.

Setup performs these steps:

1. Checks the macOS environment.
2. Checks Redis; if it is missing, installs it through Homebrew and enables automatic startup.
3. Activates the device with your SceneLoop License Key.
4. Lets you select a text model and enter its API key.
5. Lets you select an image model and enter its API key.
6. Lets you select a video model and enter its API key.
7. Verifies and saves the configuration locally.

Enter License Keys and API keys only in the local Setup window or terminal. Never send them through Hermes, OpenClaw, Feishu, group chats, or screenshots.

### Setup Does Not Open Automatically

Run it manually in Terminal:

```bash
"$HOME/.hermes/skills/sceneloop/scripts/sceneloop-setup"
```

Check the license status:

```bash
"$HOME/.hermes/skills/sceneloop/scripts/sceneloop" license status
```

If you installed the Skill through OpenClaw, use the corresponding `sceneloop/scripts/` path in OpenClaw's managed Skill directory.

Device binding data remains in local Redis. When a runtime lease expires, SceneLoop renews it online using the existing binding, so you normally do not need to enter the License Key again.

## First Generation

Upload a script in Hermes or OpenClaw and describe the request, for example:

```text
Use SceneLoop to turn this script into episode 1 of an AI comic drama.
```

Before production, the agent confirms:

1. An English project ID and episode number.
2. An aspect ratio such as `16:9` or `9:16`.
3. A visual style; if you only request a short drama, it asks whether you want live action.
4. The dialogue and narration language.
5. Video resolution; higher resolutions generally increase model cost.
6. Target finished-video duration when you explicitly request one.

After confirmation, SceneLoop runs the workflow required by the current project. It fills missing assets and does not recreate completed assets without a reason.

## Example Requests

### Vertical AI Comic Drama

```text
Use SceneLoop to make episode 1 of this script. The project ID is city_story, the aspect ratio is 9:16, the style is high-quality 3D animation, keep the dialogue in English, and use 720p video.
```

### Live-Action Short Drama

```text
Use SceneLoop to make a cinematic live-action vertical short drama. The project ID is night_case, episode 1, aspect ratio 9:16, English dialogue, and 720p video.
```

### Continue a Specific Shot

```text
Continue shot 6 of episode 1 in night_case. Check existing assets and render through the video stage.
```

### Retry Failed Shots

```text
Use SceneLoop to retry the failed shots in episode 1 of city_story without overwriting successful videos.
```

## Output Files

SceneLoop stores project references separately from video output:

```text
references/<project_id>/
  source/                         source scripts
  scripts_prompts/episode_001/    adapted script and storyboard JSON
  character_prompts/episode_001/  character definitions and references
  scene_prompts/episode_001/      location definitions and references
  shot_prompts/episode_001/       shot first and last frames
  runtime/episode_001/            run state and shot reports

output/<project_id>/
  episodes/episode_001/
    shot_001.mp4
    shot_002.mp4
```

`references/` contains scripts, JSON, images, and reports. `output/` contains videos only.

## Troubleshooting

### `Permission denied`

Restore executable permissions:

```bash
chmod +x "$HOME/.hermes/skills/sceneloop/scripts/sceneloop"
chmod +x "$HOME/.hermes/skills/sceneloop/scripts/sceneloop-setup"
```

### macOS Cannot Verify or Open the Program

After confirming that the package came from a trusted company delivery channel, run:

```bash
xattr -dr com.apple.quarantine "$HOME/.hermes/skills/sceneloop"
```

### Redis Connection Failure

```bash
brew services start redis
redis-cli ping
```

The expected result is `PONG`. If Homebrew is not installed, install it from the [Homebrew website](https://brew.sh/) and run `sceneloop-setup` again.

### Hermes Does Not Invoke SceneLoop

```bash
hermes skills list
```

Confirm that `sceneloop` appears, then start a new Hermes conversation or run `/reset`. Explicitly ask Hermes to use SceneLoop.

### OpenClaw Does Not Invoke SceneLoop

```bash
openclaw skills list
```

Confirm that `sceneloop` appears, then start a new OpenClaw conversation and explicitly request SceneLoop. If the Skill is absent, repeat the OpenClaw installation commands above.

### Invalid License Status

Confirm that Redis is running, then execute:

```bash
"$HOME/.hermes/skills/sceneloop/scripts/sceneloop" license verify
```

If the device has never been activated, run `sceneloop-setup` again. Do not bypass or modify license validation.

### Model Returns 401, 403, or Insufficient Balance

- `401` usually indicates an invalid or expired API key, or a key configured for the wrong provider.
- `403` usually indicates account permissions, insufficient balance, or a service that has not been enabled.
- After changing model configuration, run `sceneloop-setup` again to verify it.

### Image or Video Generation Times Out

Timeouts may occur when a model service is busy or the network is unstable. Keep completed assets and ask SceneLoop to retry failed shots instead of restarting the entire project.

## Security

- Never expose License Keys or API keys in chats, logs, screenshots, or public repositories.
- Do not modify or bypass SceneLoop license validation.
- Hermes and OpenClaw only coordinate the workflow; they must not replace SceneLoop's production models.
- The package contains no user credentials. Licensing and model configuration take place locally.

## About Us

SceneLoop is developed and maintained by **Xi'an Wenyao Network Information Technology Co., Ltd.**

- Product website: [SceneLoop](https://www.wenyaotech.com/products?category=autodrama&product=sceneloop)

© 2024 - 2026 西安文鳐网络信息科技有限责任公司 版权所有
