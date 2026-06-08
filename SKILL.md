---
name: ostt
description: Use when installing, configuring, troubleshooting, or using OSTT, the open-source terminal-native speech-to-text CLI for Linux and macOS. Guides users through installation, provider/model setup, recording, file transcription, local Whisper models, hotkey popup workflows, processing actions, keywords, replacements, history, and debugging.
license: MIT
metadata:
  author: Kristofer Lund
  version: "1.0.0"
  tags: [speech-to-text, transcription, cli, voice-input, local-ai, linux, macos]
  related: [audio, productivity, terminal, whisper]
---

# OSTT Skill

## Overview

OSTT is an open-source, terminal-native speech-to-text tool for Linux and macOS. It records audio, transcribes existing audio files, supports both cloud providers and local Whisper-compatible models, and sends the resulting text to stdout, clipboard, paste, files, shell pipelines, or configurable post-processing actions.

Use this skill to help a user install OSTT, choose a transcription backend, configure hotkeys, transcribe files, build voice-to-text workflows, and troubleshoot issues.

## When to Use

Use this skill when the user wants to:

- Install or upgrade OSTT.
- Configure OSTT authentication, models, local transcription, or provider parameters.
- Record speech from a microphone and turn it into text.
- Transcribe existing audio files such as `.ogg`, `.opus`, `.mp3`, `.m4a`, `.wav`, or meeting recordings.
- Bind OSTT to a global hotkey or popup terminal workflow.
- Use local offline Whisper-compatible models, GPU acceleration, or the local model daemon.
- Improve transcription quality with provider parameters, keywords, or deterministic replace rules.
- Post-process transcriptions with shell commands or AI CLI tools.
- Inspect history, retry a recording with another model, replay saved audio, or debug logs/configuration.

Do not use this skill for general audio editing, diarized meeting-note synthesis, or LLM summarization unless OSTT is specifically the transcription layer.

## Quick Start

```bash
# Install latest OSTT
curl -fsSL https://ostt.ai/install | bash

# Configure credentials and choose a model
ostt auth
ostt model

# Record from microphone, transcribe, print to stdout
ostt

# Record and copy the transcription to clipboard
ostt -c

# Record and paste into the focused app
ostt --paste

# Transcribe an existing audio file
ostt transcribe recording.ogg

# Launch popup recorder for global hotkey use
ostt launch -c
```

Recording controls:

- `Enter` stops recording and transcribes.
- `Space` pauses/resumes recording.
- `Esc`, `q`, or `Ctrl+C` cancels.

## Installation

### Recommended installer

The recommended installation path is the shell installer:

```bash
curl -fsSL https://ostt.ai/install | bash
```

Non-interactive install:

```bash
curl -fsSL https://ostt.ai/install | bash -s -- --yes
```

Useful installer options:

```bash
curl -fsSL https://ostt.ai/install | bash -s -- --yes
curl -fsSL https://ostt.ai/install | bash -s -- --version v0.0.19
curl -fsSL https://ostt.ai/install | bash -s -- --install-dir "$HOME/.local/bin"
curl -fsSL https://ostt.ai/install | bash -s -- --no-deps
curl -fsSL https://ostt.ai/install | bash -s -- --no-gpu
```

The installer:

1. Detects OS: macOS or Linux.
2. Detects CPU architecture: `x86_64`/`amd64` or `aarch64`/`arm64`.
3. Detects Linux distro/session/package manager where applicable.
4. Detects GPU build preference:
   - macOS → Metal build.
   - Linux x86_64 with NVIDIA + CUDA runtime libraries → CUDA build.
   - Linux x86_64 with AMD/Intel GPU + Vulkan runtime → Vulkan build.
   - Otherwise → CPU build.
5. Installs missing runtime dependencies when it can.
6. Downloads release artifacts from `kristoferlund/ostt` GitHub Releases.
7. Verifies the `.sha256` checksum before installing.
8. Installs `ostt` to `~/.local/bin` by default, or uses native `.deb`/`.rpm` packages where supported.
9. Attempts shell completion setup when appropriate.

### Supported platforms

OSTT release artifacts support:

| OS | Architectures | Notes |
| --- | --- | --- |
| macOS | `x86_64`, `aarch64` | Apple Silicon uses `aarch64-apple-darwin`; local GPU acceleration uses Metal. |
| Linux | `x86_64`, `aarch64` | Linux x86_64 may use CPU, CUDA, or Vulkan release artifacts. |

Unsupported architectures should build from source.

### Runtime dependencies

All platforms:

- `ffmpeg` — audio conversion and encoding.

Linux clipboard support:

- Wayland: `wl-clipboard` (`wl-copy`).
- X11: `xclip`.
- Unknown session: install both if possible.

macOS clipboard support:

- `pbcopy` is built into macOS.

Package-manager examples:

```bash
# macOS / Linux with Homebrew
brew install ffmpeg

# Debian / Ubuntu, Wayland
sudo apt-get update
sudo apt-get install -y ffmpeg wl-clipboard

# Debian / Ubuntu, X11
sudo apt-get update
sudo apt-get install -y ffmpeg xclip

# Fedora
sudo dnf install -y ffmpeg wl-clipboard
sudo dnf install -y ffmpeg xclip

# openSUSE
sudo zypper --non-interactive install ffmpeg wl-clipboard
sudo zypper --non-interactive install ffmpeg xclip

# Arch / Manjaro
sudo pacman -S --needed ffmpeg wl-clipboard
sudo pacman -S --needed ffmpeg xclip
```

### Alternative install methods

Homebrew:

```bash
brew tap kristoferlund/ostt
brew install ostt
```

AUR:

```bash
yay -S ostt
```

Or manually:

```bash
git clone https://aur.archlinux.org/ostt.git
cd ostt
makepkg -si
```

Debian/Ubuntu package:

```bash
curl -sLO https://github.com/kristoferlund/ostt/releases/latest/download/ostt_latest_amd64.deb
sudo apt install ./ostt_latest_amd64.deb
```

Fedora/RHEL package:

```bash
sudo dnf install https://github.com/kristoferlund/ostt/releases/latest/download/ostt-latest.x86_64.rpm
```

openSUSE package:

```bash
sudo zypper install https://github.com/kristoferlund/ostt/releases/latest/download/ostt-latest.x86_64.rpm
```

Build from source:

```bash
git clone https://github.com/kristoferlund/ostt.git
cd ostt
cargo build --profile dist --locked
sudo install -m 0755 target/dist/ostt /usr/local/bin/ostt
```

### Verify installation

```bash
command -v ostt
ostt --version
ostt --help
```

If installed to `~/.local/bin` and `ostt` is not found, add it to `PATH`:

```bash
# bash
printf '\nexport PATH="$HOME/.local/bin:$PATH"\n' >> ~/.bashrc
source ~/.bashrc

# zsh
printf '\nexport PATH="$HOME/.local/bin:$PATH"\n' >> ~/.zshrc
source ~/.zshrc

# fish
fish_add_path ~/.local/bin
```

## Configuration Files and Data Locations

OSTT follows XDG-style locations:

| Purpose | Path |
| --- | --- |
| Main config | `~/.config/ostt/ostt.toml` |
| Credentials | `~/.local/share/ostt/credentials` |
| Recordings | `~/.local/share/ostt/recordings/` |
| History database | `~/.local/share/ostt/transcription_history.db` |
| Logs | `~/.local/state/ostt/ostt.log.*` |
| Local model files | under `~/.local/share/ostt/` |

Useful commands:

```bash
ostt config              # open config in $EDITOR
ostt config path         # print config path
ostt logs                # show recent logs
ostt logs path           # print latest log path
ostt logs follow         # follow latest log
```

Modern provider parameters use top-level provider/model sections, not old `[providers.*]` tables:

```toml
[deepgram.params]
detect_language = true
smart_format = true

[deepgram.nova-3.params]
keyterm = ["OSTT", "OpenCode", "Kubernetes"]

[berget."KBLab/kb-whisper-large".params]
language = "sv"

[whisper.params]
language = "auto"
no_timestamps = true
```

If OSTT errors with `Deprecated config section '[providers]'`, migrate `[providers.deepgram]` to `[deepgram.params]`, `[providers.assemblyai]` to `[assemblyai.params]`, etc. Also remove deprecated `[audio].sample_rate`; use `[audio].output_format` instead.

## Authentication and Model Selection

OSTT is bring-your-own-API-key for cloud providers and can also run local models.

```bash
ostt auth                  # interactive auth flow
ostt auth login deepgram   # add/update one provider credential
ostt auth list             # list authenticated providers
ostt auth status           # summarize auth state
ostt auth logout openai    # remove a provider credential

ostt model                 # interactive model picker
ostt model list            # list available models
ostt model list --format json
ostt model list --provider deepgram
ostt model current
ostt model select deepgram/nova-3
ostt model params deepgram/nova-3
```

Supported cloud providers and common models include:

| Provider | Example models |
| --- | --- |
| OpenAI | `gpt-4o-transcribe`, `gpt-4o-mini-transcribe`, `gpt-4o-transcribe-diarize`, `whisper-1` |
| Deepgram | `nova-3`, `nova-2` |
| Groq | `whisper-large-v3`, `whisper-large-v3-turbo` |
| DeepInfra | Whisper and Voxtral-hosted models, including `openai/whisper-large-v3` |
| AssemblyAI | `universal-3-pro` |
| Berget | `KBLab/kb-whisper-large`, `NbAiLab/nb-whisper-large`, `openai/whisper-large-v3` |
| ElevenLabs | `scribe_v2`, `scribe_v1` |
| Mistral | `voxtral-mini-latest`, `voxtral-mini-2602` |
| Whisper/local | `tiny`, `base`, `small`, `medium`, `large-v3`, `turbo`, KB/NB Whisper variants |

Per-run model override:

```bash
ostt -m deepgram/nova-3 -c
ostt transcribe audio.ogg -m openai/gpt-4o-transcribe
ostt retry 2 -m berget/KBLab/kb-whisper-large -c
```

Per-run transcription parameter override:

```bash
ostt transcribe audio.ogg --param detect_language=true
ostt -m deepgram/nova-3 --param smart_format=true
```

## Core Commands

### Record from microphone

```bash
ostt                         # default: record, transcribe, print to stdout
ostt record                  # explicit equivalent
ostt -c                      # copy transcription to clipboard
ostt --paste                 # paste into focused app
ostt -o notes.txt            # write transcription to file
ostt -p clean -c             # process with action and copy result
ostt -m deepgram/nova-3 -c   # use a specific model for this run
```

Aliases: `ostt r` for `ostt record`.

### Transcribe existing files

```bash
ostt transcribe recording.ogg
ostt transcribe voice-memo.mp3 -c
ostt transcribe meeting.wav -o transcript.txt
ostt transcribe audio.ogg | grep keyword
```

Aliases: `ostt t` for `ostt transcribe`.

Input can be any audio format `ffmpeg` can read: `.ogg`, `.opus`, `.mp3`, `.m4a`, `.wav`, `.flac`, `.aac`, and many more.

For large recordings, convert first to a compact speech-friendly file:

```bash
ffprobe -v error -show_entries format=duration,size -of default=noprint_wrappers=1 recording.wav
ffmpeg -y -v error -i recording.wav -ac 1 -ar 16000 -b:a 64k /tmp/recording.mp3
ostt transcribe /tmp/recording.mp3 -o transcript.txt
```

### Retry and replay saved recordings

OSTT stores recordings locally so the same audio can be retried with a different provider/model.

```bash
ostt retry                 # retry most recent recording
ostt retry 2 -c            # retry second-most-recent, copy result
ostt retry 3 -m openai/gpt-4o-transcribe
ostt replay                # play most recent recording
ostt replay 2              # play second-most-recent recording
```

Aliases: `ostt rp` for `ostt replay`.

### History

```bash
ostt history               # interactive browser; Enter copies selected transcript
ostt history list
ostt history list --limit 20
ostt history list --format json
ostt history show 3
ostt history copy 3
```

Alias: `ostt h`.

### Device and config inspection

```bash
ostt config list-devices
ostt config path
ostt config
```

Use `ostt config list-devices` to find microphone names or indexes, then set `[audio].device` in `~/.config/ostt/ostt.toml`.

## Popup and Global Hotkeys

`ostt launch` opens OSTT in a popup terminal. Running the same command again signals the running recorder to finish, transcribe, and output the text. This makes it ideal for global hotkeys.

```bash
ostt launch -c
ostt launch --paste
ostt launch -c -p clean
ostt launch -- -c -p translate
```

Suggested hotkeys:

| Hotkey | Command | Result |
| --- | --- | --- |
| `Alt+Space` | `ostt launch -c` | popup recorder, clipboard output |
| `Alt+Ctrl+Space` | `ostt launch -c -p` | popup recorder with processing action picker |
| `Alt+Space` | `ostt launch --paste` | popup recorder, paste into focused app |

Configure popup terminal/window behavior in `~/.config/ostt/ostt.toml` under `[popup]`:

```toml
[popup]
terminal = "ghostty" # or kitty, alacritty, foot, konsole, gnome-terminal, xfce4-terminal
x = 630
y = 790
width = 90
height = 15
font_size = 6
borderless = true
```

macOS popup mode requires a supported terminal emulator such as Ghostty, kitty, or Alacritty.

## Local Whisper Models

OSTT can run local Whisper-compatible models for offline transcription. This avoids cloud uploads and subscriptions, at the cost of local CPU/GPU usage.

```bash
ostt model                  # interactive local/cloud model picker
ostt model list --provider whisper
ostt model list --installed
ostt model local download turbo
ostt model local download kb-whisper-large
ostt model select whisper/turbo
ostt model local remove turbo
```

Use local models when:

- Privacy/offline operation matters.
- API keys are unavailable.
- You want predictable local cost.
- The machine has sufficient CPU/GPU resources.

Use cloud models when:

- Lowest latency matters on weak hardware.
- You need best current accuracy without managing model files.
- Provider-specific features such as smart formatting or language handling matter.

### Local model daemon

The local model daemon keeps the active local model loaded in memory, reducing startup latency.

```bash
ostt daemon start
ostt daemon status
ostt daemon restart
ostt daemon stop
ostt daemon install      # auto-start on login
ostt daemon uninstall
```

The daemon serves the currently active local model selected with `ostt model`.

## Keywords and Text Replacements

### Keywords

Keywords help the transcription model recognize names, product terms, acronyms, and domain vocabulary.

```bash
ostt keyword
ostt keyword list
ostt keyword add OSTT OpenCode Kubernetes "Kristofer Lund"
ostt keyword remove Kubernetes
```

Alias: `ostt k`.

Use keywords for terms the model should hear correctly. Keep the list focused; too many unrelated terms can reduce value.

### Deterministic replacements

Replacements fix final text after transcription. Use them for casing, acronyms, product names, and common misrecognitions.

```bash
ostt replace
```

Config example:

```toml
[text.replace]
"ostt" = "OSTT"
"open code" = "OpenCode"
"github" = "GitHub"
"api" = "API"
```

Use replacements for deterministic final-text cleanup, not semantic rewriting. For rewriting, use processing actions.

## Processing Actions

Processing actions transform transcripts after recording or from history. They can run bash commands or AI CLI tools.

Common usage:

```bash
ostt -p                         # record, transcribe, show action picker
ostt -p clean -c                # record, clean, copy
ostt transcribe audio.ogg -p summarize -o summary.md
ostt process                    # process most recent history item, show picker
ostt process clean              # process most recent with action `clean`
ostt process 5                  # process history item 5, show picker
ostt process 5 clean -c         # process history item 5, clean, copy
ostt process list
ostt process list --format json
```

Alias: `ostt p`.

Processing config lives in `~/.config/ostt/ostt.toml`. A bash action receives the transcription on stdin:

```toml
[process.actions.clean]
name = "Clean transcript"
type = "bash"
command = "python3 ~/.config/ostt/scripts/clean_transcript.py"
```

AI actions can send structured inputs to an AI CLI tool:

```toml
[process]
default_tool = "opencode"
default_model = "openai/gpt-4.1-mini"

[process.actions.summarize]
name = "Summarize"
type = "ai"
inputs = [
  { role = "system", content = "Summarize the transcript clearly and concisely." },
  { role = "user", source = "transcription" }
]
```

Supported AI tool identifiers include `opencode`, `claude-code`, `gemini-cli`, and `codex-cli`. Only configure tools that are actually installed and authenticated on the user's machine.

## Shell Completions

```bash
ostt completions bash > ostt.bash
ostt completions zsh > _ostt
ostt completions fish > ostt.fish
ostt completions powershell > ostt.ps1
ostt completions install bash
ostt completions install zsh
ostt completions install fish
```

When the installer skips completions due to missing TTY or sudo access, install them manually later.

## Recommended Workflows

### Global voice input

1. Install OSTT.
2. Run `ostt auth` or choose a local model with `ostt model`.
3. Test `ostt -c` in a terminal.
4. Bind a global hotkey to `ostt launch -c`.
5. Optionally bind another hotkey to `ostt launch -c -p` for AI processing.

### Fast file transcription

```bash
ostt transcribe "$AUDIO" -o transcript.txt
```

If the file is huge, compress first with `ffmpeg` as shown above.

### Try another model without re-recording

```bash
ostt retry 1 -m deepgram/nova-3
ostt retry 1 -m openai/gpt-4o-transcribe
ostt retry 1 -m whisper/turbo
```

### Swedish or Nordic-language transcription

Good first choices:

```bash
ostt model select deepgram/nova-3
ostt model select berget/KBLab/kb-whisper-large
ostt model select whisper/kb-whisper-large
ostt model select whisper/nb-whisper-large
```

Use Berget or KB/NB Whisper variants when Swedish/Norwegian accuracy matters more than provider-neutral defaults.

### Privacy-first offline mode

```bash
ostt model local download turbo
ostt model select whisper/turbo
ostt daemon start
ostt -c
```

## Troubleshooting

### `ostt: command not found`

Check where it was installed:

```bash
find "$HOME/.local/bin" /usr/local/bin /usr/bin -name ostt -type f 2>/dev/null
```

Add `~/.local/bin` to `PATH` if needed.

### Deprecated config error

If you see:

```text
Deprecated config section '[providers]' is no longer supported.
```

Then update old provider tables:

```toml
# old
[providers.deepgram]
punctuate = true

# new
[deepgram.params]
punctuate = true
```

Also remove deprecated `[audio].sample_rate` and control encoding/sample rate through:

```toml
[audio]
output_format = "mp3 -ab 16k -ar 12000"
```

### Clipboard or paste does not work

Linux Wayland:

```bash
command -v wl-copy || sudo apt-get install -y wl-clipboard
```

Linux X11:

```bash
command -v xclip || sudo apt-get install -y xclip
```

macOS:

```bash
command -v pbcopy
```

For paste mode, adjust `[output.paste].paste_key`; some Linux desktops/apps prefer `shift+insert` instead of `ctrl+v`.

### No microphone or wrong microphone

```bash
ostt config list-devices
ostt config
```

Set `[audio].device` to an index or device name.

### Transcription fails

```bash
ostt logs
ostt logs path
RUST_LOG=debug ostt transcribe audio.ogg
ostt auth status
ostt model current
```

Check:

- API key exists for the selected cloud provider.
- The selected model belongs to the selected provider.
- The audio file exists and `ffmpeg` can read it.
- Network access is available for cloud models.
- Local model files are downloaded for `whisper/*` models.

### Local model is slow to start

Use the daemon:

```bash
ostt daemon start
ostt daemon status
```

Or choose a smaller model such as `whisper/base`, `whisper/small`, or `whisper/turbo`.

### Installer dependency failure

If the installer cannot install dependencies, install manually:

```bash
# Debian/Ubuntu
sudo apt-get update
sudo apt-get install -y ffmpeg wl-clipboard xclip

# Fedora
sudo dnf install -y ffmpeg wl-clipboard xclip

# Arch
sudo pacman -S --needed ffmpeg wl-clipboard xclip

# macOS
brew install ffmpeg
```

Then rerun:

```bash
curl -fsSL https://ostt.ai/install | bash -s -- --yes --no-deps
```

## Verification Checklist

After installing or changing OSTT, verify:

- [ ] `ostt --version` prints the expected version.
- [ ] `ostt --help` shows the command list.
- [ ] `ostt auth status` works without config errors.
- [ ] `ostt model current` shows the intended provider/model.
- [ ] `ostt transcribe <audio-file>` works on a known audio file.
- [ ] `ostt -c` can record and copy text, if clipboard workflow is desired.
- [ ] `ostt launch -c` opens a popup, if hotkey workflow is desired.
- [ ] `ostt logs` contains no relevant errors.

## Command Reference

```bash
ostt [record options]        # default record command
ostt record                  # record audio
ostt transcribe <FILE>       # transcribe existing audio
ostt retry [N]               # retry saved recording
ostt replay [N]              # replay saved recording
ostt auth                    # manage cloud credentials
ostt model                   # choose/manage models
ostt history                 # browse/list/show/copy history
ostt keyword                 # manage transcription keywords
ostt replace                 # manage deterministic replace rules
ostt config                  # edit/show config and list devices
ostt logs                    # inspect logs
ostt process                 # run processing actions
ostt launch                  # popup terminal workflow
ostt completions             # generate/install shell completions
ostt daemon                  # manage local model daemon
```

Global output flags used by record/transcribe/retry/process:

```bash
-c, --clipboard              # copy result to clipboard
--paste                      # paste result into focused app
-o, --output <FILE>          # write result to file
-p, --process [ACTION]       # run processing action or show picker
-m, --model <PROVIDER/MODEL> # override model for this run
--param KEY=VALUE            # override transcription param for this run
```

## Links

- Website/docs: https://ostt.ai
- Installer: https://ostt.ai/install
- GitHub: https://github.com/kristoferlund/ostt
- Releases: https://github.com/kristoferlund/ostt/releases
