# Security Policy

## Supported versions

Only the latest release of Solomon CLI gets security fixes.

## Reporting a vulnerability

**Please do not open a public issue for security problems.**

Report them privately:
- through GitHub's **"Report a vulnerability"** button (Security tab → Advisories), or
- privately via GitHub: https://github.com/Roholevs/solomon-cli/security/advisories/new

Please include:
- your Solomon CLI version (`solomon --version`) and macOS version
- the steps to reproduce, and a sample file if one is needed (for example a crafted model file)
- what you think the impact is

We aim to reply within 5 business days and will tell you when the issue is fixed. Please give us a reasonable amount of time to release a fix before you disclose the issue publicly.

## Scope

Examples: crashes or memory-safety bugs triggered by malformed model files, or the binary doing anything outside the documented behaviour (such as network access or writing files).

Out of scope: what a model writes in its answers. The model's output comes from the model you chose, not from Solomon CLI.
