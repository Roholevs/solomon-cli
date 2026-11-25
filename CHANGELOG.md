# Changelog

All notable changes to Solomon CLI are listed here. Versions follow [Semantic Versioning](https://semver.org/).

## [0.1.0] - unreleased

First public release.

- `solomon chat`: interactive chat in the terminal, with `/reset` and `/exit`
- `solomon complete`: one-shot text completion from `--prompt` or stdin
- `solomon bench`: speed benchmark with JSON output
- `solomon model-info`: checks a GGUF model file
- `--version`, `help` and per-command `--help`
- Supports popular open GGUF chat models (Llama and Qwen families) and split (multi-file) GGUF models
- Uses the Apple GPU automatically
- Fully offline, no telemetry
