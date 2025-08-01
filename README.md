# openbench-nebius-guide

End‑to‑End Guide ▸ Evaluating Nebius AI Studio models with OpenBench

This walkthrough shows you—from a clean shell prompt—to a complete, logged mmlu evaluation on a Nebius‑hosted Meta‑Llama‑3.1‑70B‑Instruct model. Every step has been tested on macOS 13 (Apple Silicon); Linux users can substitute identical commands. Windows ⌨️ users can run under WSL or PowerShell.

1  Prerequisites

Requirement

Why you need it

Nebius AI Studio account & API key

Authenticates your model calls. Grab a key at studio.nebius.com ▶ Settings ▶ API keys.

curl or wget

To bootstrap uv quickly.

Rust tool‑chain

Optional. uv distributes pre‑built wheels; if one isn’t available for your platform, compiling will require Rust ≥ 1.77.

Git

To clone the OpenBench repo (recommended).

🗒️ Terminology: uv is a blister‑fast Python package/virtual‑env manager from Astral. We’ll use it instead of pip+venv.

2  Install uv (≈ 15 s)

# macOS/Linux – one‑liner
curl -LsSf https://astral.sh/uv/install.sh | sh

# Add ~/.local/bin to PATH *for this shell* (add to ~/.zshrc later)
export PATH="$HOME/.local/bin:$PATH"

# Verify
uv --version   # ➜ uv 0.8.x

3  Install a managed Python runtime

# Pick a modern CPython (OpenBench supports 3.10+)
uv python install 3.12      # downloads ~40 MB once

4  Clone or download OpenBench

git clone https://github.com/groq/openbench.git
cd openbench

(If you can’t git‑clone, download the ZIP from GitHub and unzip to ~/openbench).

5  Create & activate a virtual environment

uv venv --python 3.12       # creates .venv and pins Python
source .venv/bin/activate   # prompt changes → (openbench)

6  Install OpenBench & deps (≈ 20 s)

uv pip install -e .         # “editable” install; omit -e if you prefer

7  Configure environment variables

# 7.1 Nebius creds
export NEBIUS_API_KEY="<paste‑your‑key‑here>"

# 7.2 Tell OpenBench / Inspect to use a plain‑vanilla OpenAI‑compatible endpoint
export OPENAI_BASE_URL="https://api.studio.nebius.com/v1/"

# 7.3 (Optional) raise concurrency limits for faster evals
export INSPECT_MAX_CONNECTIONS=40

Heads‑up (zsh): Put back‑slashes at the end of each line when you split long commands; otherwise zsh interprets # as history‐expansion rather than a comment and throws “bad pattern” errors.

8  Dry‑run sanity check (5 samples)

bench eval mmlu \
  --model openai/meta-llama/Meta-Llama-3.1-70B-Instruct \
  --limit 5

You should see a quick accuracy score (~0–1 depending on RNG) and a log file path like logs/2025‑07‑31T22‑39‑24+02‑00_mmlu_….

9  Full MMLU sweep (≈ 35 min on 70 B model @ 400 K TPM)

bench eval mmlu \
  --model openai/meta-llama/Meta-Llama-3.1-70B-Instruct \
  --temperature 0.6 \
  --max-connections 40 \
  --timeout 30000 \
  --logfile logs/full_mmlu_run.jsonl

Explanation of key flags

Flag

Purpose

--temperature 0.6

Matches Nebius docs’ default playground setting.

--max-connections 40

Opens up to 40 parallel requests (respect Nebius rate limits).

--timeout 30000

30 s per request before retrying/aborting.

--logfile

Streams JSONL traces to a file you can parse later.

The run emits a live progress bar; expect ~60 k requests for full MMLU.

10  Inspect results

# View aggregate metrics in‑terminal
bench view logs/full_mmlu_run.jsonl

# Or open the rich TUI viewer (arrow‑keys navigate)
bench view

The output summarises accuracy, stem_accuracy (case‑insensitive), token counts, and per‑subject breakdowns.

11  Advanced knobs & tips

Goal

Flag or env‑var

Notes

Evaluate another Nebius model

--model openai/deepseek/DeepSeek-R1

Use names from the Nebius dashboard.

Few‑shot / custom system prompt

--system "You are an expert …"

Any OpenAI param accepted by Inspect.

Re‑run failed samples only

--retry-on-error

Great for transient 5xx errors.

Limit working memory

--max-tokens 2048

Useful on very long‑context models.

Non‑interactive CI run

--log-buffer 0 --logfile results.jsonl --log-level WARN

Ideal for GitHub Actions.

12  Troubleshooting

Symptom

Fix

ValueError: Model API nebius … not recognised

Ensure you used the openai/ prefix (OpenBench maps Nebius to the OpenAI API schema).

zsh: bad pattern: #

Move inline # comments to their own line or escape them with \#.

429 rate‑limit errors

Dial down --max-connections, or request higher limits in Nebius Studio.

Slow run

Lower temperature, raise max-connections, ensure local bandwidth isn’t saturated.

Cleanup

deactivate        # leave venv
uv python uninstall 3.12      # if you no longer need it
uv self uninstall             # removes uv binaries

13  Next steps

Explore additional OpenBench benchmarks (bench list).

Compare models side‑by‑side by comma‑separating names: --model openai/meta-llama/70b,openai/deepseek/R1.

Push results to your own dashboard with inspect_ai’s reporting hooks.

Happy benchmarking 🚀

🎬 Quick LinkedIn demo (≤ 60 s)

Need something snappy for a screen‑capture? Run a tiny HumanEval sweep that still shows the flashy progress UI and spits out an accuracy score.

# Inside the (openbench) venv
bench eval humaneval \
  --model openai/meta-llama/Meta-Llama-3.1-70B-Instruct \
  --limit 3 \            # just 3 coding tasks → finishes in ≈15 s
  --temperature 0.4 \
  --max-connections 10 \
  --timeout 30000 \
  --logfile logs/demo_humaneval.jsonl

Watch the live tokens & progress bar—perfect b‑roll.

When it completes, pop up the summary TUI:

bench view logs/demo_humaneval.jsonl

The colourful table makes a great closing frame.



Need to abort? Press q in the TUI or Ctrl+C in the shell.

Alternative quick demos

Benchmark

Command snippet

Typical runtime*

simpleqa (factual)

bench eval simpleqa --model … --limit 10

10‑20 s

Mini‑mmlu

bench eval mmlu --model … --limit 25

30‑40 s

*Measured on Nebius Meta‑Llama‑3.1‑70B‑Instruct @ 400 k TPM with 40 concurrent connections.

Happy filming 📹

