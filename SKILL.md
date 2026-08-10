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

- `ffmpeg` — audio conversion and encoding (required).
- `mpv` — optional; preferred player for `ostt replay`. OSTT falls back to `vlc`, `ffplay`, `paplay` (Linux), or `afplay` (macOS) when `mpv` is absent.

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

AUR — prebuilt binary packages (recommended; no compilation, no Rust toolchain). Pick the one matching the hardware; they conflict with each other and with the source `ostt` package, so only one can be installed:

```bash
yay -S ostt-bin          # CPU build (x86_64, aarch64)
yay -S ostt-cuda-bin     # NVIDIA CUDA build (x86_64)
yay -S ostt-vulkan-bin   # AMD/Intel Vulkan build (x86_64)
```

`paru` works in place of `yay`. To build from source instead, use the `ostt` package:

```bash
yay -S ostt
```

Or build the source package manually:

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

`ostt launch` opens OSTT in a popup terminal. Running the same command again signals the running recorder to finish, transcribe, and output the text. This makes it ideal for global hotkeys. Alias: `ostt l`.

```bash
ostt launch -c
ostt launch --paste
ostt launch -c -p clean
ostt launch -- -c -p translate
```

> **Critical for hotkeys: use the full path to the binary.** Desktop environments usually do **not** include `~/.local/bin` in the PATH that global hotkeys run with, so a bare `ostt launch ...` keybind silently does nothing. Resolve the absolute path once and use it in every keybind:
>
> ```bash
> command -v ostt   # e.g. /home/you/.local/bin/ostt
> ```
>
> Then bind, for example, `/home/you/.local/bin/ostt launch --paste`.

Suggested hotkeys (substitute the full path from `command -v ostt`):

| Hotkey | Command | Result |
| --- | --- | --- |
| `Alt+Space` | `/path/to/ostt launch --paste` | popup recorder, paste into focused app |
| `Alt+Space` | `/path/to/ostt launch -c` | popup recorder, clipboard output |
| `Alt+Ctrl+Space` | `/path/to/ostt launch --paste -p` | popup recorder with processing action picker |

### Per-desktop hotkey setup

#### Hyprland and Omarchy

Two supported Omarchy generations use different Hyprland configuration formats. Omarchy 3.x remains supported and uses Hyprlang `.conf` files. Omarchy 4.x is currently in alpha and uses Lua. Inspect the installed configuration before writing rules or bindings; do not assume either format based on the desktop name alone.

```bash
hyprctl version
test -f "$HOME/.config/hypr/hyprland.lua" && printf 'Lua configuration found\n'
test -f "$HOME/.config/hypr/bindings.lua" && printf 'Lua bindings found\n'
test -f "$HOME/.config/hypr/hyprland.conf" && printf 'Hyprlang configuration found\n'
test -f "$HOME/.config/hypr/bindings.conf" && printf 'Hyprlang bindings found\n'
hyprctl -j binds
```

- **Omarchy 4.x alpha (Lua)** uses `~/.config/hypr/hyprland.lua` as its entry point and `~/.config/hypr/bindings.lua` for personal hotkeys. Add personal window rules after the default imports in `hyprland.lua`. Never modify packaged Omarchy files under `~/.local/share/omarchy` or `/usr/share/omarchy`.
- **Omarchy 3.x and legacy Hyprland (Hyprlang)** use `~/.config/hypr/hyprland.conf` and `~/.config/hypr/bindings.conf`.
- When asked to upgrade an Omarchy 3.x OSTT configuration to Omarchy 4.x, read the active entry point and existing files first. Back up every affected user file, migrate only recognized OSTT binding and window-rule lines, and remove those migrated lines from the old file so the hotkey cannot fire twice. Leave unrelated user configuration untouched.
- If `Alt+Space` is already bound to another user action, tell the user and ask before replacing it. Resolve `ostt` with `command -v ostt` and use the resulting absolute path.

For current window-rule syntax, always fetch the official [Hyprland Window Rules documentation](https://wiki.hypr.land/Configuring/Basics/Window-Rules/) before editing. The format changes frequently.

##### Omarchy 4.x Alpha Lua

Add the hotkey to `~/.config/hypr/bindings.lua`:

```lua
o.bind("ALT + SPACE", "OSTT speech-to-text", "/home/you/.local/bin/ostt launch --paste")
```

Add the popup rule after the default imports in `~/.config/hypr/hyprland.lua`:

```lua
-- Keep only the OSTT popup floating, pinned, and centered near the bottom edge.
o.window({ class = "ostt-popup" }, {
  float = true,
  move = { "((monitor_w*0.5)-(window_w*0.5))", "(monitor_h*0.85)" },
  pin = true,
})
```

##### Omarchy 3.x And Legacy Hyprlang

Add the hotkey to `~/.config/hypr/bindings.conf`:

```text
bindd = ALT, SPACE, ostt, exec, /home/you/.local/bin/ostt launch --paste
```

Add these rules to `~/.config/hypr/hyprland.conf` after broader matching rules:

```text
# OSTT window overrides
windowrule = float on, match:class ostt-popup
windowrule = move ((monitor_w*0.5)-(window_w*0.5)) (monitor_h*0.85), match:class ostt-popup
windowrule = pin on, match:class ostt-popup
```

After any Hyprland change, run and inspect both commands:

```bash
hyprctl reload
hyprctl configerrors
```

Fix reported errors before considering the change complete. Confirm the expected binding with `hyprctl -j binds`, then test `ostt launch -c`. If the popup does not match the `ostt-popup` class rule, inspect `hyprctl clients` and use its actual class rather than matching a terminal class broadly. Omarchy's `SUPER+V` sends `shift+insert`, so set `[output.paste].paste_key = "shift+insert"`.

- **GNOME** — Settings → Keyboard → Custom Shortcuts; command `/path/to/ostt launch --paste`. The Wayland compositor controls window placement (`[popup].x`/`y` are ignored; `width`/`height` still apply).
- **KDE Plasma** — System Settings → Shortcuts → Custom Shortcuts → New → Global Shortcut → Command/URL; action `/path/to/ostt launch --paste`.
- **macOS** — create a Shortcut (Shortcuts.app) with a *Run Shell Script* action calling `/path/to/ostt launch --paste`, then assign a key. macOS may prompt for **Accessibility permission** for the terminal app the first time paste is used; grant it or paste will silently fail. Use Ghostty, kitty, or Alacritty (Terminal.app lacks truecolor).

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

Paste mode works by copying the text to the clipboard, sending a paste shortcut to the focused app, and then optionally restoring the previous clipboard. Configure it under `[output.paste]`:

```toml
[output.paste]
paste_key = "ctrl+v"        # macOS default cmd+v; Omarchy default shift+insert
restore_clipboard = true    # restore previous clipboard after pasting
restore_delay_ms = 750      # wait before restoring (let the app read the clipboard)
post_popup_delay_ms = 1000  # delay after the popup closes (for `ostt launch --paste`)
```

`paste_key` defaults differ by environment: macOS `cmd+v`, Omarchy `shift+insert`, other Linux `ctrl+v`. If paste lands in the wrong app or not at all, increase `post_popup_delay_ms` (focus hasn't returned yet) or switch `paste_key` to what the target app expects (`shift+insert` works in many terminals/X11 apps).

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

### GPU build issues (local models)

The installer auto-selects a GPU build (CUDA on NVIDIA, Vulkan on AMD/Intel, Metal on macOS) and falls back to CPU. Two common failures:

**1. `ostt` won't start at all after install / "library not found" on launch.**
A GPU build was installed but its runtime libraries are missing. CUDA needs `libcuda.so` + `libcublas.so`; Vulkan needs `libvulkan.so.1` (usually from Mesa). Either install the GPU runtime, or reinstall the CPU build:

```bash
curl -fsSL https://ostt.ai/install | bash -s -- --no-gpu
```

**2. Local transcription is slow — is the GPU actually being used?**
Confirm via the logs; OSTT logs which backend it activated:

```bash
ostt logs | grep -i "GPU acceleration"
# Expect one of:
#   local transcription: CUDA GPU acceleration enabled
#   local transcription: Vulkan GPU acceleration enabled
#   local transcription: Metal GPU acceleration enabled
```

If none appears, OSTT is running on CPU. Reinstall with the appropriate GPU runtime present, or accept CPU and use a smaller model / the daemon. Note: GPU acceleration only affects local `whisper/*` models — cloud providers are unaffected.

### Popup does not appear (`ostt launch`)

First confirm OSTT itself works and a supported terminal is installed:

```bash
ostt launch -c   # run directly in a terminal, not via the hotkey
command -v ghostty kitty alacritty foot konsole gnome-terminal xfce4-terminal
```

- If `ostt launch -c` works directly but the **hotkey** does nothing, the keybind is almost certainly using a bare `ostt` instead of the full path — see the full-path note under "Popup and Global Hotkeys".
- If no terminal is found, install one (Ghostty, kitty, or Alacritty) and/or set it explicitly:

  ```toml
  [popup]
  terminal = "ghostty"
  ```

- **GNOME Wayland**: the compositor controls window placement, so `[popup].x`/`y` are ignored (`width`/`height` still apply).
- **macOS**: Terminal.app lacks truecolor — use Ghostty, kitty, or Alacritty.

### Processing action fails

```bash
ostt process list                 # confirm the action ID exists
RUST_LOG=debug ostt process clean # see the underlying error
```

AI-type actions shell out to an external CLI tool that must be installed **and** authenticated:

```bash
opencode --version   # requires 1.4.3+
claude --version
gemini --version
codex --version
```

Check that `[process].default_tool` / the action's `tool` matches an installed binary (`opencode`→`opencode`, `claude-code`→`claude`, `gemini-cli`→`gemini`, `codex-cli`→`codex`), that `default_model`/`model` is valid for that tool, and that the tool is logged in. AI actions time out after 300 seconds. Bash-type actions receive the transcript on stdin and must print the result to stdout.

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
