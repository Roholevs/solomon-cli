# Solomon CLI

**Bigger AI models on smaller computers. One tiny file. No install. Nothing leaves your machine.**

- **Run models that don't fit:** `solomon optimize` makes a large model smaller and faster on your computer. A 35B model that llama.cpp could not run on a 16 GB Mac answers at ~20 tokens/s.
- **One tiny file:** a single binary of under 1 MB (about 0.3–0.4 MB to download). Nothing to install, no Python, no dependencies.
- **Nothing leaves your machine:** the program has zero network code. No account, no cloud, no telemetry.
- **Fast without tuning:** speed settings are picked automatically for each model, and a memory check runs before a model loads.
- **Mac, Linux and Windows:** the Apple GPU on Macs, the CPU on Linux and Windows.

*Українською: [README.uk.md](README.uk.md)*

> Solomon CLI is **free to use** and **closed source**. This repository has the documentation, release downloads and issue tracker only. See [LICENSE](LICENSE).

## Features

- Chat with a local AI model in your terminal
- One-shot text completion that works in shell scripts and pipes
- A built-in speed benchmark
- `solomon optimize`: makes a large model smaller and faster on your computer, so models that would not fit in memory can run well
- Uses the Apple GPU automatically on Macs; runs on the CPU on Linux and Windows
- **100% local**: your prompts and answers never leave your computer

## Speed compared with llama.cpp

Only the cases where Solomon is faster, or where llama.cpp cannot run the model, are listed. Median of 3 runs after a warm-up run.

| Model | Test | llama.cpp | Solomon CLI 0.2.1 |
|---|---|---|---|
| Qwen3.6-35B-A3B UD-Q4_K_M (20.6 GB) | reply speed on a 16 GB Mac | **does not run** (stops with "Insufficient Memory") | **20 tokens/s** with the optimized copy (11.7 GB), also in a chat with a ~860-token first message |
| Qwen2.5-Coder-7B Instruct Q4_K_M | summary of a ~340-token text, 256 tokens | 21.8 tokens/s | **27.5 tokens/s** (same file) |
| Qwen3.5-2B Q4_K_M | reply speed, 128 tokens | 63.8 tokens/s | **65.8 tokens/s** |

Measured on 25 November 2026 (35B row: 26 November 2026, Solomon CLI 0.2.1) on a MacBook Air with Apple M4 and 16 GB of memory, macOS 26.6, on AC power, with no other heavy apps running and all models on the internal disk. llama.cpp build 12359 (Homebrew). *The optimized copy is a smaller file made by `solomon optimize`; its answers stay close to the original but can differ in wording.

How to reproduce:

```sh
# llama.cpp
llama-bench -m Qwen3.5-2B-Q4_K_M.gguf -p 512 -n 128 -r 1
llama-bench -m Qwen2.5-Coder-7B-Instruct-Q4_K_M.gguf -p 512 -n 128 -r 1
llama-completion -m Qwen2.5-Coder-7B-Instruct-Q4_K_M.gguf -f summary.txt -n 256 --temp 0 -no-cnv
llama-bench -m Qwen3.6-35B-A3B-UD-Q4_K_M.gguf -p 0 -n 32 -r 1

# Solomon CLI
solomon bench --file Qwen3.5-2B-Q4_K_M.gguf --count 128
solomon bench --file Qwen2.5-Coder-7B-Instruct-Q4_K_M.gguf --prompt-file summary.txt --count 256
solomon optimize --file Qwen2.5-Coder-7B-Instruct-Q4_K_M.gguf
solomon bench --file Qwen2.5-Coder-7B-Instruct-Q4_K_M.solomon --count 128
solomon bench --file Qwen2.5-Coder-7B-Instruct-Q4_K_M.solomon --prompt-file summary.txt --count 256
solomon optimize --file Qwen3.6-35B-A3B-UD-Q4_K_M.gguf
solomon bench --file Qwen3.6-35B-A3B-UD-Q4_K_M.solomon --prompt "Write a 300-word story about a lighthouse keeper who finds a message in a bottle." --count 512
```

`summary.txt` is "Summarize the following text in 5 bullet points:" followed by the first paragraphs of the Wikipedia article "Industrial Revolution" (about 340 tokens in all).

## Requirements

| | |
|---|---|
| macOS | Apple Silicon (M1, M2, M3, M4 or later), macOS 14 Sonoma or newer. Intel Macs are not supported. |
| Linux | x86_64 or arm64 (aarch64), glibc 2.39 or newer (Ubuntu 24.04+, Debian 13+, Fedora 40+ or similar) |
| Windows | Windows 10 or 11, x86_64 |
| x86_64 CPU | must support **AVX2** and FMA (Intel Haswell 2013+ / AMD Zen 2017+) |
| Memory | 8 GB minimum; 16 GB or more recommended for mid-size models |
| Disk | Space for the model files (about 0.5–20 GB each) |

## Platform notes

| Platform | Release file | Runs on |
|---|---|---|
| macOS (Apple Silicon) | `solomon-cli-<version>-macos-arm64.zip` | Apple GPU (Metal) |
| Linux x86_64 | `solomon-cli-<version>-linux-x86_64.tar.gz` | CPU (AVX2) |
| Linux arm64 | `solomon-cli-<version>-linux-arm64.tar.gz` | CPU (NEON) |
| Windows x86_64 | `solomon-cli-<version>-windows-x86_64.zip` | CPU (AVX2) |

On Linux and Windows the CLI runs on the CPU for now, so it is slower than on a Mac with the same model; small models (0.5B–4B) work well. GPU acceleration on Linux and Windows is planned. Commands, options and output are the same on every platform.


## Install

### macOS

1. Download `solomon-cli-<version>-macos-arm64.zip` from the [Releases](../../releases) page.
2. Optional: check the download against `SHA256SUMS`:
   ```sh
   shasum -a 256 -c SHA256SUMS --ignore-missing
   ```
3. Unzip it and move the `solomon` binary onto your PATH:
   ```sh
   unzip solomon-cli-*-macos-arm64.zip
   sudo mkdir -p /usr/local/bin
   sudo mv solomon /usr/local/bin/
   # or, without sudo:
   mkdir -p ~/bin && mv solomon ~/bin/ && echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc
   ```
4. Check that it works:
   ```sh
   solomon --version
   ```

**Gatekeeper ("cannot be opened because the developer cannot be verified")**: signed and notarized releases open normally. If macOS blocks a build anyway, remove the download quarantine flag:
```sh
xattr -d com.apple.quarantine /usr/local/bin/solomon
```
Do this only for files you downloaded from this repository's Releases page.

**Homebrew**: a tap (`brew install <owner>/tap/solomon`) is *coming soon*.

### Linux

1. Download `solomon-cli-<version>-linux-x86_64.tar.gz` (or `-linux-arm64.tar.gz` on ARM) and `SHA256SUMS` from [Releases](../../releases).
2. Optional: check the download:
   ```sh
   sha256sum -c SHA256SUMS --ignore-missing
   ```
3. Unpack and install:
   ```sh
   tar -xzf solomon-cli-*-linux-*.tar.gz
   cd solomon-cli-*-linux-*/
   chmod +x solomon
   sudo install -m 755 solomon /usr/local/bin/solomon
   # or, without sudo:
   mkdir -p ~/.local/bin && cp solomon ~/.local/bin/   # ~/.local/bin is on PATH on most distros
   ```
4. Check: `solomon --version`

To check that an x86_64 CPU has AVX2: `grep -o -m1 avx2 /proc/cpuinfo` (prints `avx2` if supported).

### Windows

1. Download `solomon-cli-<version>-windows-x86_64.zip` from [Releases](../../releases).
2. Optional: check the download in PowerShell and compare with `SHA256SUMS`:
   ```powershell
   Get-FileHash .\solomon-cli-*-windows-x86_64.zip -Algorithm SHA256
   ```
3. Right-click the zip → **Extract All…**, then move `solomon.exe` to a folder such as `C:\Program Files\Solomon` (or `%LOCALAPPDATA%\Programs\Solomon` without admin rights).
4. Add that folder to PATH: Start → "Edit environment variables for your account" → **Path** → **Edit** → **New** → paste the folder → OK. Or in PowerShell:
   ```powershell
   $dir = "$env:LOCALAPPDATA\Programs\Solomon"
   [Environment]::SetEnvironmentVariable("Path", [Environment]::GetEnvironmentVariable("Path", "User") + ";$dir", "User")
   ```
5. Open a **new** terminal (Windows Terminal, PowerShell or cmd) and run `solomon --version`.

**SmartScreen ("Windows protected your PC")**: the program is not yet code-signed for Windows, so SmartScreen may warn the first time. Click **More info** → **Run anyway**, or right-click `solomon.exe` → Properties → check **Unblock**. Do this only for files from this repository's Releases page.

Non-English text works in Windows Terminal and PowerShell; the CLI switches the console to UTF-8 itself.

## Get a model

Solomon CLI runs models in the **GGUF** file format. Thousands of them are available on [Hugging Face](https://huggingface.co/models?library=gguf). Search for a model name followed by "GGUF" and download one `.gguf` file.

Supported: popular open chat model families such as **Llama** and **Qwen** (including Qwen2, Qwen3 and Qwen3.5 text models) in GGUF format. Other families may give "unsupported" errors; please open a feature request.

Good first models:

| Your computer's memory | Try a model of about |
|---|---|
| 8 GB | 0.5B–3B parameters (file of 0.5–2 GB) |
| 16 GB | up to 8B parameters (file of up to ~6 GB) |
| 32 GB+ | 14B parameters and larger |

When a model is offered in several file variants, the smaller files are faster and use less memory, and the larger ones are a bit more accurate. Medium-sized variants are a good default.

Large models that are split across several files (`...-00001-of-00003.gguf`) are supported: pass the **first** file.

## Quick start

```sh
solomon chat --file ~/Models/Qwen3-0.6B-Q8_0.gguf
```
```
Ready. Type a message, /reset to start over, /exit to quit.

> What is the capital of France?
The capital of France is Paris.
```

## Commands

Run `solomon help` for an overview and `solomon <command> --help` for details. Every option takes a value (`--name value`).

### `solomon chat`

Interactive chat in the terminal. Type a message and press Enter; the reply streams as it is generated. The conversation is remembered until you type `/reset`.

```
solomon chat --file MODEL.gguf [--context N] [--max-tokens N] [--threads N] [--memory-mib N]
             [--temperature T] [--top-p P] [--repeat-penalty R] [--seed N]
```

| Option | Default | Meaning |
|---|---|---|
| `--file` | (required) | Path to the `.gguf` model |
| `--context` | 8192 | How much of the conversation (in tokens) the model can see |
| `--max-tokens` | 1024 | Longest reply, in tokens (max 8192) |
| `--threads` | auto | CPU threads (1–32), or `auto` = the number of performance cores |
| `--memory-mib` | 512 | Memory reserved for the conversation, in MiB |
| `--temperature` | 0.7 | Creativity, 0–5. `0` always picks the most likely word (same reply every time) |
| `--top-p` | 0.9 | Picks only among the most likely words that together reach this share |
| `--top-k` | 0 (off) | Picks only among the N most likely words |
| `--min-p` | 0 (off) | Skips words less than this share as likely as the best one |
| `--repeat-penalty` | 1.1 | Discourages repeating words from the last 256 tokens (`1` = off; `--repeat-window N` changes the 256) |
| `--seed` | random | Same seed + same input = same reply |

A reply that starts repeating the same block over and over is stopped automatically.
`solomon bench` accepts the same `--temperature`, `--top-p`, `--repeat-penalty` and `--seed` options.

In-chat commands: `/reset` clears the conversation; `/exit`, `/quit` or Ctrl-D quits.

You can also pipe messages in, one per line:
```sh
printf 'Write a haiku about the sea\n/exit\n' | solomon chat --file model.gguf --max-tokens 200
```

Some models (for example Qwen3) "think out loud" first, inside `<think>…</think>`. For Qwen3 you can add `/no_think` to the end of a message to skip that.

### `solomon complete`

Continues a piece of text (raw completion with no chat formatting). This is handy in scripts.

```
solomon complete --file MODEL.gguf [--prompt TEXT] [--count N]
```

| Option | Default | Meaning |
|---|---|---|
| `--prompt` | read from stdin | Text to continue (max 4096 bytes from stdin) |
| `--count` | 16 | Tokens to generate (1–128) |

```sh
$ solomon complete --file model.gguf --prompt "The capital of France is" --count 12
 Paris. The capital of France is also the capital of the
$ echo "Once upon a time" | solomon complete --file model.gguf --count 10
```

### `solomon bench`

Measures how fast a model runs on your computer and prints JSON.

```
solomon bench --file MODEL.gguf [--prompt TEXT] [--count N]
```

| Option | Default | Meaning |
|---|---|---|
| `--prompt` | a short built-in question | Prompt to answer |
| `--count` | 128 | Tokens to generate (1–1024) |

```sh
$ solomon bench --file Qwen3-0.6B-Q8_0.gguf --count 64
{"load_seconds":0.17,"first_token_seconds":0.49,"tokens":64,"tokens_per_second":27.23,"total_seconds":2.98}
```
Fields: model load time, time to the first token, tokens generated, generation speed and total time.

### `solomon optimize`

Makes a model smaller and faster on this computer; answer quality stays close to the original. See [Optimize a model](#optimize-a-model).

```
solomon optimize --file MODEL.gguf [--out PATH]
```

| Option | Default | Meaning |
|---|---|---|
| `--file` | (required) | The model to optimize (left unchanged) |
| `--out` | next to the model, `NAME.solomon` | Where to write the optimized copy |

### `solomon model-info`

Checks that a file is a readable GGUF model and prints basic facts.
```sh
$ solomon model-info --file Qwen3-0.6B-Q8_0.gguf
{"format":"gguf","version":3,"tensors":310,"metadata":28,"bytes":639446688}
```

### Other

- `solomon --version` prints the version.
- `solomon help` and `solomon help <command>` print help.
- The exit code is `0` on success and `1` on error. Errors go to stderr as `Error: ...`.

## Optimize a model

Large models are often too big for your computer's memory, and then they run slowly or not at all. `solomon optimize` writes a smaller copy of the model that is tuned to run on this computer. The original file is not changed.

```sh
$ solomon optimize --file ~/Models/Qwen3.6-35B-A3B-UD-Q4_K_M.gguf
Optimizing Qwen3.6-35B-A3B-UD-Q4_K_M.gguf for this computer...
Size: 20.6 GB -> ~11.7 GB
[##############################] 100%  ETA 00:00
Done in 01:20: 20.6 GB -> 11.7 GB
```

Then use the new file like any model, with `chat`, `complete` and `bench`:

```sh
solomon chat --file ~/Models/Qwen3.6-35B-A3B-UD-Q4_K_M.solomon
```

- **Best for large models** (3 GB and up), especially ones that are big compared with your memory. Smaller models already run well and are left as they are.
- **Quality stays close to the original.** Answers can differ slightly in wording.
- **Uses little memory** while it works and shows progress with the time left. You need free disk space for the new file; Solomon checks this first.
- When you start `solomon chat` with a large model, it tells you if optimizing would help, or if an optimized copy is already next to it.
- Single-file models only for now.

| Model | Before | After | Speed before | Speed after |
|---|---|---|---|---|
| Qwen3.6-35B-A3B UD-Q4_K_M | 20.6 GB | 11.7 GB | does not fit in 16 GB* | 20 tokens/s |

Measured on a MacBook Air with Apple M4 and 16 GB of memory, with the model on the internal disk. When an optimized model fits in memory, Solomon reads it in fully before the first answer ("Loading model... 45%"), so replies don't wait on the disk. *llama.cpp stopped with "Insufficient Memory" on the original 35B model.

## Performance tips

- **Automatic memory check.** Before loading, Solomon compares what the model needs with the free memory. If it fits, the model is loaded fully into memory for the best speed; if memory is tight, it prints a short note (for example *"this model needs ~6.1 GB of memory, ~4.0 GB available; close other apps for best speed"*) and reads the model from disk as needed instead.

- **Keep models on the internal disk.** External USB drives are much slower to read; a model loads and answers faster from the computer's own disk.
- **Choose the model size for your memory.** A model that doesn't fit in RAM still runs, but it slows down a lot. Keep the model file under about 60% of your computer's memory.
- **Close memory-hungry apps** (browsers with many tabs, VMs) before you run large models.
- **Smaller file variants are faster.** If generation is slow, try a smaller variant or a smaller model.
- The **first run after a download is slower** while macOS reads the file into its cache.
- If you only need short chats, a lower `--context` saves memory.
- Use `solomon bench` to compare models on your computer.
- On Linux and Windows (CPU), the default `--threads auto` (the number of performance cores) is usually fastest; Q8_0 and Q4_K files are the fastest formats on the CPU.

## Troubleshooting

| Message / symptom | Fix |
|---|---|
| `Model file not found` | Check the path. Put quotes around paths with spaces. |
| `Not a GGUF model file` | Download a `.gguf` file. Other formats (safetensors, PyTorch) are not supported. |
| `Unsupported GGUF version` / unsupported model error | That model family or file is not supported yet. Try a Llama or Qwen GGUF, or open a feature request. |
| `Option --x needs a value` | Every option takes a value, e.g. `--count 32`. |
| `zsh: command not found: solomon` | The binary is not on your PATH (see Install). |
| "cannot be opened because the developer cannot be verified" | See the Gatekeeper note under Install. |
| `zsh: bad CPU type in executable` | You are on an Intel Mac, which is not supported. |
| `Illegal instruction` (Linux) / crash at start (Windows) | The x86_64 CPU has no AVX2; it is not supported. |
| `cannot execute binary file: Exec format error` (Linux) | Wrong archive: use `linux-arm64` on ARM, `linux-x86_64` on Intel/AMD. |
| `GLIBC_2.xx not found` (Linux) | The distro is too old; use one from the Requirements table. |
| `'solomon' is not recognized...` (Windows) | The folder is not on PATH, or open a new terminal after changing PATH. |
| Very slow output | The model is too large for your memory; use a smaller one. |
| Reply stops mid-sentence | Raise `--max-tokens` (chat) or `--count` (complete). |

## FAQ

**Is it really free?** Yes. The CLI is free for personal and commercial use under the terms in [LICENSE](LICENSE).

**Does it need the internet?** No. After you download a model, everything runs offline.

**Is the source code available?** No. Solomon is proprietary software; only the binaries are published.

**Windows / Linux?** Yes: x86_64 Linux and Windows, and arm64 Linux. Intel Macs are not supported.

**Can it use my GPU?** On Macs, yes, the Apple GPU is used automatically. On Linux and Windows it runs on the CPU for now; GPU support there is planned.

**Where are my chats stored?** Nowhere. The CLI keeps the conversation in memory only and saves nothing.

## Privacy

Solomon CLI runs **entirely on your computer**:
- no network connections (the binary contains no networking code, and it works with network access blocked)
- no telemetry, analytics or crash reporting
- no account or sign-in
- it reads only the model file you pass it, and it writes nothing to disk

## Solomon for Mac (paid app)

Want more than the terminal? **Solomon** is the desktop app, built on the same engine:
- a clean, native Mac chat interface
- Solomon's own models, tuned for speed on Apple Silicon
- conversation history, projects and documents
- more features for power users and teams

Learn more: website coming soon — watch this repository for the announcement.

## License

Solomon CLI is proprietary freeware. In short: you may use it for free, including at work. You may not redistribute, modify or reverse-engineer it. It comes with no warranty. The full text is in [LICENSE](LICENSE).

## Support

- Bugs and feature requests: [Issues](../../issues)
- Security problems: see [SECURITY.md](SECURITY.md)
- Changes: [CHANGELOG.md](CHANGELOG.md)
