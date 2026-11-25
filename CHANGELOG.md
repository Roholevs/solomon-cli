# Changelog

All notable changes to Solomon CLI are listed here. Versions follow [Semantic Versioning](https://semver.org/).

## [0.2.0]

- New `solomon optimize --file MODEL.gguf`: makes a large model smaller and faster on this computer, with answer quality close to the original. Large models that did not fit in memory can now run (for example a 35B model: 20.6 GB to 11.7 GB). Shows progress with the time left, uses little memory, and checks free disk space first.
- Optimized models that fit in memory are read in fully before the first answer, with a progress line; an optimized 35B model generates about 11 tokens/s on a 16 GB Mac, where the original model does not run.
- Optimized models also answer faster than the original on Apple GPUs (a 7B model: about 30 tokens/s instead of 21 on an M4 Mac).
- `chat`, `complete` and `bench` run optimized files directly, on the Apple GPU and on the CPU (Linux, Windows).
- `solomon chat` gives a tip when a large model would benefit from optimizing or when an optimized copy already exists.

## [0.1.3]

- Much shorter wait for the first token on long prompts with large mixture-of-experts models on Apple GPUs (up to ~4x in our tests).
- Faster first reply after a long prompt on those models.
- Loading a model fully into memory no longer keeps a second copy, so it uses less memory.

## [0.1.2]

- Faster: the CLI now picks the best speed settings for each model automatically (up to ~2.5x faster on Qwen3.5 models on Apple GPUs, much shorter wait for the first token). Replies are unchanged.
- Automatic memory check before loading: models that fit are loaded fully into memory; when memory is tight a short note suggests closing other apps.
- `--threads auto` (new default) uses the number of performance cores.

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
