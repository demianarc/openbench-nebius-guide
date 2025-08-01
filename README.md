# 🧪 End‑to‑End Guide ▸ Evaluating Nebius AI Studio Models with OpenBench

OpenBench is a fast, standardized evaluation framework built by [@groqinc](https://github.com/groq) for reproducible LLM benchmarking.

This guide shows how to evaluate **Nebius AI Studio-hosted open models** (like Meta Llama 3 and Qwen) on benchmarks like **MMLU**, using OpenBench and a single terminal command.

---

## 🧠 What is Model Benchmarking?

> Benchmarking lets you measure how well a language model performs on tasks like logic, math, code, or knowledge recall.  
It’s how we compare models like Llama 3, GPT-4, Claude, or Qwen using standardized tests (e.g. MMLU, GPQA, HumanEval).

---

## ⚡ Quick Preview

We'll run a short evaluation on `Llama-3.3-70B-Instruct-fast` hosted by Nebius AI Studio:

```bash
bench eval mmlu \
  --model openai/meta-llama/Llama-3.3-70B-Instruct-fast \
  --limit 12 \
  --temperature 0.6 \
  --timeout 30000 \
  --max-connections 40 \
  --logfile logs/mmlu_sample.jsonl
