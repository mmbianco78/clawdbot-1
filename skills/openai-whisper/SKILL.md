---
name: openai-whisper
description: Local speech-to-text with Whisper (prefers whisper.cpp on Apple Silicon).
homepage: https://openai.com/research/whisper
metadata: {"clawdbot":{"emoji":"🎙️","requires":{"bins":["whisper-cli"]},"install":[{"id":"brew-cpp","kind":"brew","formula":"whisper-cpp","bins":["whisper-cli"],"label":"Install whisper.cpp (recommended for Apple Silicon)"},{"id":"brew-py","kind":"brew","formula":"openai-whisper","bins":["whisper"],"label":"Install Python Whisper (slower)"}]}}
---

# Whisper (Local Transcription)

Fast local speech-to-text. **On Apple Silicon, uses whisper.cpp (4-10x faster).**

## Quick Start

```bash
# Using whisper.cpp (fastest on M1/M2/M3/M4)
whisper-cli -m ~/.clawdbot/soundcore/models/ggml-large-v3-turbo.bin -f audio.ogg -otxt

# Using Python whisper (fallback)
whisper audio.mp3 --model turbo --output_format txt
```

## Models

### whisper.cpp (GGML format)
Download to `~/.clawdbot/soundcore/models/`:
- `ggml-tiny.en.bin` - 75MB, fastest, English only
- `ggml-base.bin` - 142MB, good balance
- `ggml-large-v3-turbo.bin` - 1.5GB, best quality/speed (recommended)

Download: `curl -L -o ~/.clawdbot/soundcore/models/ggml-large-v3-turbo.bin https://huggingface.co/ggerganov/whisper.cpp/resolve/main/ggml-large-v3-turbo.bin`

### Python whisper
Models auto-download to `~/.cache/whisper/` on first run.

## Performance (Apple Silicon)

| Method | M4 24GB | Notes |
|--------|---------|-------|
| whisper.cpp + Metal | ~10x realtime | GPU accelerated |
| Python whisper + MPS | ~3x realtime | PyTorch Metal |
| Python whisper CPU | ~1x realtime | Slow |

## Options

whisper.cpp: `whisper-cli -m <model> -f <audio> [-l <lang>] [-otxt|-osrt|-ovtt]`
Python: `whisper <audio> --model <size> --output_format <txt|srt|vtt>`
