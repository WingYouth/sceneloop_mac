# SceneLoop

[简体中文](README.md) | [English](README_EN.md)

SceneLoop is a packaged binary workflow for producing AI comics, AI short dramas, single-image animation, and AI product ads. Submit scripts, images, product materials, and creative requirements through an AI agent such as Hermes or OpenClaw; the native programs in this distribution perform the actual text, image, and video production.

SceneLoop is developed by **Xi'an Wenyao Network Information Technology Co., Ltd.**

[Visit the SceneLoop product website](https://www.wenyaotech.com/products?category=autodrama&product=sceneloop)

> This guide covers the macOS edition of the SceneLoop Skill.

## Package Contents

```text
sceneloop_mac/
├── README.md
├── README_EN.md
├── SKILL.md
└── scripts/
    ├── sceneloop   drama and single-image animation
    ├── ai-ads     AI product advertising
    └── setup      licensing and model configuration
```

All three programs are prebuilt Apple Silicon (`arm64`) binaries. Users do not need Python, a compiler, or the project source code, and should not move an individual executable out of this Skill directory.

## Features

- Accepts `.docx`, `.md`, and `.txt` scripts.
- Plans projects from the requested aspect ratio, visual style, language, resolution, and target duration.
- Generates `storyboard.json`, `characters.json`, and `locations.json` automatically.
- Creates missing character images, location images, shot first frames, and videos.
- Supports animation, 3D, live-action, and other visual directions.
- Maintains character, location, and visual continuity between adjacent shots.
- Supports shot-level generation, resumable runs, and retries for failed shots.
- Supports **Animate one image**: one uploaded image and a motion or camera description produce a video directly.
- Supports the MiniMax H3 v5 multi-reference video model for `9:16` and `16:9` image animations and drama shots.
- Automatically preserves the uploaded image's subjects, composition, colors, lighting, materials, and original visual style while applying only the motion, local effects, or camera movement explicitly requested by the user.
- Creates one 15-second Fast UGC product ad from one to five product images or commerce-page screenshots.
- Runs AI Ads through the separate `ai-ads` binary while preserving authorization, factual constraints, resumable state, and successful assets.

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

Image animation uses a separate lightweight workflow:

```text
Uploaded image + motion description
  -> Create a lightweight project and permanently archive the source image
  -> Use the image as the opening frame and visual-style reference
  -> Generate one animated video directly
  -> Save the source image, video, and generation settings
```

This workflow does not create a script, storyboard, characters, locations, or a separate shot first-frame project.

AI product ads use a separate workflow:

```text
Product images or commerce screenshot + short brief
  -> Identify grounded product facts and plan the creative
  -> Generate five image assets
  -> Generate five three-second video clips
  -> Assemble one 15-second Fast UGC preview
```

Drama, image animation, and AI Ads share local licensing and model configuration, but their project data remains separate.

## macOS Requirements

Before you begin, prepare:

- macOS 14 or later.
- A prebuilt SceneLoop directory matching your Mac architecture. This repository provides the Apple Silicon (`arm64`) version.
- Git installed and available from Terminal.
- A working Hermes Desktop, Hermes CLI, or OpenClaw installation.
- A SceneLoop License Key.
- API keys required by your selected text, image, and video models.
- Network access to the model services and SceneLoop License Server.
- `ffmpeg` and `ffprobe` available in `PATH` when assembling AI Ads videos.

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
chmod +x "$HOME/.hermes/skills/sceneloop/scripts/ai-ads"
chmod +x "$HOME/.hermes/skills/sceneloop/scripts/setup"
```

Check the Skill:

```bash
hermes skills list
```

After installation, start a new Hermes conversation or run:

```text
/reset
```

### Update an Existing Hermes Installation

If SceneLoop is already installed, run:

```bash
cd "$HOME/.hermes/skills/sceneloop"
git pull --ff-only origin main
chmod +x scripts/sceneloop scripts/ai-ads scripts/setup
```

After updating, start a new Hermes conversation or run `/reset` in the current conversation.

### Install in OpenClaw

OpenClaw can install this extracted Skill directly from GitHub:

```bash
openclaw skills install git:WingYouth/sceneloop_mac --as sceneloop --global
openclaw skills list
```

OpenClaw installs a global Skill in its managed Skill directory. Start a new OpenClaw conversation after installation so it loads the updated Skill list. See the [official OpenClaw Skills documentation](https://docs.openclaw.ai/tools/skills) for current commands and directory rules.

## First-Time Setup and Licensing

You do not need to run production commands manually before using the Skill. When you request generation in Hermes or OpenClaw, SceneLoop first checks the runtime, Redis, license, and model configuration. If setup is incomplete, the agent launches `setup`.

Setup performs these steps:

1. Checks the macOS environment.
2. Checks Redis; if it is missing, installs it through Homebrew and enables automatic startup.
3. Activates the device with your SceneLoop License Key.
4. Lets you select Vision, Text, Image, and Video models.
5. Collects their API keys through the local interactive interface.
6. Verifies and saves the configuration locally.

To use MiniMax H3 v5, select `minimax-h3-lightx2v-v5` from the video-model list and enter its `minimax_h3_v5` only in the local Setup prompt. Setup saves and checks the configuration but does not submit a paid video-generation task just to test this credential.

Enter License Keys and API keys only in the local Setup window or terminal. Never send them through Hermes, OpenClaw, Feishu, group chats, or screenshots.

### Setup Does Not Open Automatically

Run it manually in Terminal:

```bash
"$HOME/.hermes/skills/sceneloop/scripts/setup"
```

Check the license status:

```bash
"$HOME/.hermes/skills/sceneloop/scripts/sceneloop" license status
```

If you installed the Skill through OpenClaw, use the corresponding `sceneloop/scripts/` path in OpenClaw's managed Skill directory.

Device binding data remains in local Redis. When a runtime lease expires, SceneLoop renews it online using the existing binding, so you normally do not need to enter the License Key again.

You normally run `setup` only once when installing SceneLoop on a new computer. Daily generation does not require Setup again. Rerun it only after moving to another computer, losing the License or Redis data, losing the `.env` configuration, or when changing models or API keys.

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
5. A video model and one of that model's supported resolutions; higher resolutions generally increase model cost.
6. Whether to use the recommended character-and-location review flow or, after a risk warning, generate the full episode directly.
7. Target finished-video duration when you explicitly request one.

The recommended default generates character and location references first and presents them for approval before first-frame and video generation. SceneLoop fills missing assets and does not recreate completed assets without a reason.

## Animate One Image

Upload one JPEG, PNG, or WebP image in Hermes or OpenClaw and describe the desired subject motion, local effect, or camera movement. For example:

```text
Use SceneLoop's image-animation workflow. Make the person blink gently and smile, let the hair move slightly in the wind, and slowly push the camera forward. Generate a five-second video.
```

The agent selects **Animate one image** directly and does not ask for a script, project ID, episode number, characters, locations, or visual style. SceneLoop automatically:

1. Infers portrait or landscape orientation from the image; for a square image, it asks the user to choose when the model does not support `1:1`.
2. Uses the uploaded image as the authoritative visual reference and exact opening frame.
3. Preserves subject identity and appearance, object design, environment, composition, colors, lighting, materials, and the overall visual style.
4. Applies only the requested motion, local effects, and camera movement while avoiding unrelated additions, deformation, flicker, identity drift, and unintended restyling.
5. Creates a lightweight project that permanently stores the source image, expanded model prompt, generation settings, and final video.

For `minimax-h3-lightx2v-v5`, the available resolutions are:

- Portrait: `480p竖`, `768p竖`, and `1080p竖`
- Landscape: `480p横`, `768p横`, and `1080p横`

It supports whole-second durations from 1 through 10 seconds. Queueing and generation may wait for up to 30 minutes in total, with status polled once per second.

## Create an AI Product Ad

Upload one to five product images or commerce-page screenshots in Hermes or OpenClaw and ask for a product ad. For example:

```text
Use SceneLoop to create a natural 15-second vertical UGC ad from this product screenshot. Generate it directly.
```

The current Fast UGC workflow produces five three-second shots and assembles one 15-second preview. It supports `9:16` and model-supported `16:9`. One clear product screenshot is sufficient; users do not need to transcribe the visible product name, price, or specifications.

When the user authorizes direct generation, the agent states and confirms the complete cost scope once. The `ai-ads` runtime then performs product understanding, planning, image generation, video generation, and assembly. If execution is interrupted, resume the same project; successful assets and provider tasks are not submitted again.

Check AI Ads readiness from Terminal:

```bash
"$HOME/.hermes/skills/sceneloop/scripts/ai-ads" readiness
"$HOME/.hermes/skills/sceneloop/scripts/ai-ads" models
```

The current release is not intended for arbitrary ad durations, bulk variants, automatic publishing, or complete post-production.

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

### Animate an Uploaded Image

```text
Use SceneLoop to animate this image: make the person blink naturally, add a light breeze to the clothes and hair, and slowly push the camera forward while preserving the original visual style.
```

### Create a 15-Second Product Ad

```text
Use SceneLoop to turn these product images into a natural 15-second 9:16 UGC ad with native English speech. Run it directly.
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

Image animations use a separate lightweight project directory:

```text
image_animation_projects/<project_id>/
  source/original.<jpg|jpeg|png|webp>  archived uploaded image
  output/animated_<request_hash>.mp4   generated animation video
  project.json                         original motion text, expanded model prompt, and generation state
```

Repeated requests with the same image, prompt, model, resolution, duration, and seed reuse the existing video. A change to any generation setting or prompt-policy version creates a new request result.

AI Ads projects are stored in the Skill's ads workspace by default:

```text
workspace/ads/ads_projects/<project_id>/
  manifest.json                    project manifest and resumable state
  ...                              evidence, plans, images, videos, and final preview
```

Do not delete the project directory after an interrupted run. Resume the same project to reuse completed results.

## Troubleshooting

### `Permission denied`

Restore executable permissions:

```bash
chmod +x "$HOME/.hermes/skills/sceneloop/scripts/sceneloop"
chmod +x "$HOME/.hermes/skills/sceneloop/scripts/ai-ads"
chmod +x "$HOME/.hermes/skills/sceneloop/scripts/setup"
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

The expected result is `PONG`. If Homebrew is not installed, install it from the [Homebrew website](https://brew.sh/) and run `setup` again.

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

If the device has never been activated, run `setup` again. Do not bypass or modify license validation.

### Model Returns 401, 403, or Insufficient Balance

- `401` usually indicates an invalid or expired API key, or a key configured for the wrong provider.
- `403` usually indicates account permissions, insufficient balance, or a service that has not been enabled.
- After changing model configuration, run `setup` again to verify it.

### Image or Video Generation Times Out

Timeouts may occur when a model service is busy or the network is unstable. Keep completed assets and ask SceneLoop to retry failed shots instead of restarting the entire project.

MiniMax H3 v5 image-animation and multi-reference video tasks wait for up to 30 minutes, including both queueing and generation. If the task is still incomplete after 30 minutes, SceneLoop stops waiting; whether the provider continues processing depends on the model service. Check the lightweight project for an existing output before retrying to avoid unnecessary duplicate generation.

## Security

- Never expose License Keys or API keys in chats, logs, screenshots, or public repositories.
- Do not modify or bypass SceneLoop license validation.
- Hermes and OpenClaw only coordinate the workflow; they must not replace SceneLoop's production models.
- The package contains no user credentials. Licensing and model configuration take place locally.

## About Us

SceneLoop is developed and maintained by **Xi'an Wenyao Network Information Technology Co., Ltd.**

- Product website: [SceneLoop](https://www.wenyaotech.com/products?category=autodrama&product=sceneloop)

© 2024 - 2026 西安文鳐网络信息科技有限责任公司 版权所有
