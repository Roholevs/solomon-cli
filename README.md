# Solomon CLI

**Fast, private AI on your own computer.** Solomon CLI runs open-source AI chat models directly on your Mac, Linux PC or Windows PC. No account, no cloud, no internet connection needed.

*Українською: [README.uk.md](README.uk.md)*

> Solomon CLI is **free to use** and **closed source**. This repository has the documentation, release downloads and issue tracker only. See [LICENSE](LICENSE).

## Features

- Chat with a local AI model in your terminal
- One-shot text completion that works in shell scripts and pipes
- A built-in speed benchmark
- Uses the Apple GPU automatically on Macs; runs on the CPU on Linux and Windows
- **100% local**: your prompts and answers never leave your computer

## Requirements

| | |
|---|---|
| macOS | Apple Silicon (M1, M2, M3, M4 or later), macOS 14 Sonoma or newer. Intel Macs are not supported. |
| Linux | x86_64 or arm64 (aarch64), glibc 2.28 or newer (Ubuntu 20.04+, Debian 10+, RHEL/Rocky 8+ or similar) |
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

> **Release status:** since 0.1.1 there are builds for macOS (Apple Silicon), Linux (x86_64, arm64) and Windows (x86_64). Linux and Windows builds are new; please report any problems in Issues.

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
solomon chat --file MODEL.gguf [--context N] [--max-tokens N] [--threads N] [--memory-mib N] [--thinking show|hide]
```

| Option | Default | Meaning |
|---|---|---|
| `--file` | (required) | Path to the `.gguf` model |
| `--context` | 8192 | How much of the conversation (in tokens) the model can see |
| `--max-tokens` | 1024 | Longest reply, in tokens (max 8192) |
| `--threads` | auto | CPU threads (`auto` = performance cores, or 1–32) |
| `--memory-mib` | 512 | Memory reserved for the conversation, in MiB |

In-chat commands: `/reset` clears the conversation; `/exit`, `/quit` or Ctrl-D quits.

You can also pipe messages in, one per line:
```sh
printf 'Write a haiku about the sea\n/exit\n' | solomon chat --file model.gguf --max-tokens 200
```

Some models (for example Qwen3) "think out loud" first. Solomon hides that reasoning by default; add `--thinking show` to see it. For Qwen3 you can also add `/no_think` to the end of a message to skip it.

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

## Performance tips

- **Automatic memory check.** Before loading, Solomon compares what the model needs with the free memory. If it fits, the model is loaded fully into memory for the best speed; if memory is tight, it prints a short note and reads the model from disk as needed.
- **Choose the model size for your memory.** A model that doesn't fit in RAM still runs, but it slows down a lot. Keep the model file under about 60% of your computer's memory.
- **Close memory-hungry apps** (browsers with many tabs, VMs) before you run large models.
- **Smaller file variants are faster.** If generation is slow, try a smaller variant or a smaller model.
- The **first run after a download is slower** while macOS reads the file into its cache.
- If you only need short chats, a lower `--context` saves memory.
- Use `solomon bench` to compare models on your computer.
- On Linux and Windows (CPU), `--threads` set to the number of *physical* cores is usually fastest; Q8_0 and Q4_K files are the fastest formats on the CPU.

## Performance

Measured with `solomon bench` (128 tokens) on a MacBook with Apple M4 and 16 GB of memory, with other apps open.

| Model | 0.1.1 tokens/s | 0.1.2 tokens/s | 0.1.2 first token |
|---|---|---|---|
| Qwen3-0.6B Q8_0 | 62 | 60 | 0.24 s |
| Qwen3-1.7B Q4_K_M | 36 | 36 | 0.55 s |
| Qwen3.5-2B Q4_K_M | 19 | 35–51 | 0.15 s |
| Qwen2.5-Coder-7B Instruct Q4_K_M | 14.6 | 15.2 | 0.75 s |

Replies are the same as in 0.1.1; only the speed changed.

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

**Windows / Linux?** Yes, since 0.1.1: x86_64 Linux and Windows, and arm64 Linux (CPU for now). Intel Macs are not supported.

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
