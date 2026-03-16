# TokenLess

Agent command outputs are one of the biggest sources of token waste.

Logs, test results, stack traces… thousands of tokens sent to an LLM just to answer a simple question.

**🔥 `TokenLess` compresses command outputs into only what the LLM actually needs.**

Save **up to 99% of tokens** without losing the signal.

## How to use

```bash
npm i -g @kswork2001/tokenless
```

You can also point `TokenLess` at OpenAI-compatible providers such as LM Studio, Jan, LocalAI, vLLM, SGLang, llama.cpp-compatible servers, MLX-based servers, and Docker Model Runner.

Add in your global agent instructions file:

```md
CRITICAL: Pipe every non-interactive shell command through `TokenLess` unless raw output is explicitly required.

CRITICAL: Your prompt to `TokenLess` must be fully explicit. State exactly what you want to know and exactly what the output must contain. If you want only filenames, say `Return only the filenames.` If you want JSON, say `Return valid JSON only.` Do not ask vague questions.

Bad:
- `TokenLess "Which files are shown?"`

Good:
- `TokenLess "Which files are shown? Return only the filenames."`

Examples:
- `bun test 2>&1 | TokenLess "Did the tests pass? Return only: PASS or FAIL, followed by failing test names if any."`
- `git diff 2>&1 | TokenLess "What changed? Return only the files changed and a one-line summary for each file."`
- `terraform plan 2>&1 | TokenLess "Is this safe? Return only: SAFE, REVIEW, or UNSAFE, followed by the exact risky changes."`
- `npm audit 2>&1 | TokenLess "Extract the vulnerabilities. Return valid JSON only."`
- `rg -n "TODO|FIXME" . 2>&1 | TokenLess "List files containing TODO or FIXME. Return only file paths, one per line."`
- `ls -la 2>&1 | TokenLess "Which files are shown? Return only the filenames."`

You may skip `TokenLess` only in these cases:
- Exact uncompressed output is required.
- Using `TokenLess` would break an interactive or TUI workflow.

CRITICAL: Wait for `TokenLess` to finish before continuing.
```

## Usage

```bash
logs | TokenLess "summarize errors"
git diff | TokenLess "what changed?"
terraform plan 2>&1 | TokenLess "is this safe?"
```

Examples with other providers:

```bash
TokenLess config provider lmstudio
TokenLess config model "your-loaded-model"

TokenLess config provider jan
TokenLess config api-key "secret-key-123"

TokenLess --provider localai --host http://127.0.0.1:8080/v1 "summarize errors"
TokenLess --provider docker-model-runner --model ai/llama3.2 "what failed?"
TokenLess --provider openai-compatible --host http://127.0.0.1:9000/v1 "summarize warnings"
```

## Configurations

You can persist defaults locally:

```bash
TokenLess config model "qwen3.5:2b"
TokenLess config timeout-ms 90000
TokenLess config thinking false
TokenLess config provider lmstudio
TokenLess config host http://127.0.0.1:1234/v1
```

Supported providers:

- `ollama`
- `openai`
- `openai-compatible`
- `lmstudio`
- `jan`
- `localai`
- `vllm`
- `sglang`
- `llama.cpp`
- `mlx-lm`
- `docker-model-runner`

For pipeline exit mirroring, use `pipefail` in your shell:

```bash
set -o pipefail
```

Interactive prompts are passed through when `TokenLess` detects simple prompt patterns like `[y/N]` or `password:`.

## Global agent instructions

If you want Codex, Claude Code, or OpenCode to prefer `TokenLess` whenever they run a command whose output will be sent to a paid LLM, add a global instruction telling the agent to pipe command output through `TokenLess`.

- Codex reads global agent instructions from `~/.codex/AGENTS.md`.
- Claude Code supports global settings in `~/.claude/settings.json`, and its official mechanism for custom behavior is global instructions via `CLAUDE.md`.
- OpenCode supports global instruction files through `~/.config/opencode/opencode.json`. Point its `instructions` field at a markdown file with the same rule.
- GitHub Copilot CLI supports local global instructions from `~/.copilot/copilot-instructions.md`.
- GitHub Copilot CLI also reads repository instructions from .github/copilot-instructions.md, and it can read AGENTS.md files from directories listed in COPILOT_CUSTOM_INSTRUCTIONS_DIRS.

## Example:

```sh 
rg -n "terminal|PERMISSION|permission|Permissions|Plan|full access|default" desktop --glob '!**/node_modules/**' | distill "find where terminal and permission UI are implemented in chat screen"
```

- **Before:** [7648 tokens 30592 characters 10218 words](./examples/1/BEFORE.md)
- **After:** [99 tokens 396 characters 57 words](./examples/1/AFTER.md)

**🔥 Saved ~98.7% tokens**