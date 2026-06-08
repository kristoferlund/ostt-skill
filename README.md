<p align="center">
  <img src="./ostt.png" width="160" alt="OSTT logo" />
</p>

<p align="center">
  <strong>OSTT agent skill — let your AI coding agent install and run OSTT</strong>
</p>

<p align="center">
  <a href="https://skills.sh/kristoferlund/ostt-skill"><img src="https://skills.sh/b/kristoferlund/ostt-skill" alt="skills.sh"></a>
  <a href="https://github.com/kristoferlund/ostt-skill/stargazers"><img src="https://img.shields.io/github/stars/kristoferlund/ostt-skill?style=flat&color=yellow" alt="Stars"></a>
  <a href="https://github.com/kristoferlund/ostt-skill/commits/main"><img src="https://img.shields.io/github/last-commit/kristoferlund/ostt-skill?style=flat" alt="Last Commit"></a>
  <a href="LICENSE"><img src="https://img.shields.io/github/license/kristoferlund/ostt-skill?style=flat" alt="License"></a>
</p>

<p align="center">
  <a href="#install">Install</a> •
  <a href="#what-it-covers">What it covers</a> •
  <a href="#usage">Usage</a> •
  <a href="https://ostt.ai/guide/ai-skill">Docs</a> •
  <a href="https://github.com/kristoferlund/ostt">OSTT</a>
</p>

---

This is the official [agent skill](https://skills.sh) for **[OSTT](https://github.com/kristoferlund/ostt)**, the open source terminal-native speech-to-text tool for Linux and macOS. A skill is a portable instruction file that teaches AI coding agents — such as [Claude Code](https://claude.com/claude-code) and other skill-aware tools — how to do something well. Add this one, and your agent can install, configure, and troubleshoot OSTT for you instead of you following the docs by hand.

> [!TIP]
> There is nothing to install first — `npx` runs the `skills` CLI on demand:
> ```bash
> npx skills add kristoferlund/ostt-skill
> ```

## Install

```bash
npx skills add kristoferlund/ostt-skill
```

This installs the skill into the skills directory of every skill-aware agent `npx skills` detects on your machine. Re-run the same command to update to the latest version.

## What it covers

The skill gives your agent accurate, up-to-date knowledge of OSTT, including:

- **Installation** — the shell installer and all its flags, Homebrew, AUR (`ostt-bin` / `ostt-cuda-bin` / `ostt-vulkan-bin` and source), `.deb`/`.rpm`, and building from source.
- **Providers and models** — authenticating cloud providers, choosing models, and per-run overrides.
- **Recording and files** — recording from the mic, transcribing audio files, retrying, and replaying.
- **Hotkeys and popup** — `ostt launch`, global hotkeys, and per-desktop setup for Hyprland, GNOME, KDE, and macOS.
- **Local models** — downloading Whisper-compatible models, GPU acceleration, and the local model daemon.
- **Keywords, replacements, and processing actions** — improving accuracy and post-processing with bash or AI tools.
- **Troubleshooting** — PATH issues, clipboard/paste, GPU build problems, popup failures, processing-action errors, and config debugging.

## Usage

Once the skill is added, just ask your agent in plain language. For example:

- "Install OSTT and set up Deepgram as the provider."
- "Bind OSTT to `Alt+Space` on Hyprland with paste output."
- "OSTT says `command not found` after install — fix my PATH."
- "Download a local Whisper model and run it with GPU acceleration."
- "Add a processing action that cleans up filler words."

## About OSTT

OSTT records from a hotkey, transcribes with local Whisper-compatible models or your chosen cloud provider, then sends the result to your clipboard, a file, stdout, an AI prompt, or any shell command.

- Website and docs: **https://ostt.ai**
- Installing via an agent: **https://ostt.ai/guide/ai-skill**
- Source: **https://github.com/kristoferlund/ostt**

## License

[MIT](LICENSE) © Kristofer Lund
