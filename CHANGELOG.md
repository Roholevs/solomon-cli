# Changelog

## 0.1.3
- Much shorter wait for the first token on long prompts with large mixture-of-experts models on Apple GPUs (up to ~4x in our tests).
- Faster first reply after a long prompt on those models.
- Loading a model fully into memory no longer keeps a second copy, so it uses less memory and pushes other apps into swap less.

## 0.1.2
- Faster: the CLI now picks the best speed settings for each model automatically (up to ~2.5x faster on Qwen3.5 models on Apple GPUs, much shorter wait for the first token). Replies are unchanged.
- Automatic memory check before loading: models that fit are loaded fully into memory; when memory is tight a short note suggests closing other apps.
- `--threads auto` (new default) uses the number of performance cores.

## 0.1.1
- Chat hides the model's reasoning (`<think>`) by default; `--thinking show` displays it.
- No blank lines before replies.
- `--version` shows platform and GPU backend.

All notable changes to Solomon CLI are listed here. Versions follow [Semantic Versioning](https://semver.org/).

## 0.1.0

First public release.

- `solomon chat`: interactive chat in the terminal, with `/reset` and `/exit`
- `solomon complete`: one-shot text completion from `--prompt` or stdin
- `solomon bench`: speed benchmark with JSON output
- `solomon model-info`: checks a GGUF model file
- `--version`, `help` and per-command `--help`
- Supports popular open GGUF chat models (Llama and Qwen families) and split (multi-file) GGUF models
- Uses the Apple GPU automatically
- Fully offline, no telemetry
