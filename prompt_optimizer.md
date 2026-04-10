# Prompt Optimization Loop — Orchestrator

## Overview

This is a continuous optimization loop that iteratively improves the LLM text polish prompt. It runs until the user stops it.

Inspired by Google DeepMind's OPRO (Optimization by PROmpting): each round, agents propose changes based on previous results, a benchmark evaluates them, and the best change is kept.

## Architecture

```
Controller (you — lightweight, runs the loop)
  │
  ├── Each Round:
  │   ├── Read current baseline (prompt + temp + score)
  │   ├── Spawn Agent Team: 4 proposers each suggest ONE change
  │   ├── Spawn Benchmark Judge: test all proposals on 50-100 samples
  │   ├── If winner beats baseline → update baseline
  │   ├── Log results to optimization_log.json
  │   └── Continue to next round
  │
  └── Repeat forever until user stops
```

## The 5 Agents

| Agent | File | Role |
|-------|------|------|
| Minimalist | prompt_minimalist.md | Proposes deletions/simplifications |
| Example Crafter | prompt_example_crafter.md | Proposes example changes |
| Rule Writer | prompt_rule_writer.md | Proposes rule additions/modifications |
| Temperature Tuner | prompt_temperature_tuner.md | Proposes parameter changes |
| Benchmark Judge | prompt_benchmark_judge.md | Runs benchmarks, declares winners |

## Files

| File | Purpose |
|------|---------|
| `~/.local/voice-input/training_data.json` | 978 input/expected pairs for benchmarking |
| `~/.local/voice-input/optimization_log.json` | History of all rounds, proposals, and scores |
| `~/.local/voice-input/best_prompt.json` | Current best prompt + parameters + score |
| `~/.local/voice-input/text_polisher.py` | Where the actual prompt lives (DO NOT MODIFY — only update best_prompt.json) |

## Round Structure

### Step 1: Load State
Read `best_prompt.json` for current baseline. If it doesn't exist, extract current prompt and parameters from `text_polisher.py`.

### Step 2: Spawn Proposers
Launch ONE agent that role-plays all 4 proposers. Give it:
- The current prompt text
- The current parameters (temp, top_p, top_k)
- The last 5 rounds of history (what was tried, what worked, what failed)
- Error analysis from the last benchmark (which test cases failed and why)

Each proposer outputs:
```json
{
  "agent": "minimalist",
  "change": "Remove Rule 7 (preserve idioms) — redundant with Example 6",
  "new_prompt": "...",
  "new_temp": 0.3,
  "new_top_p": 0.8,
  "new_top_k": 20
}
```

### Step 3: Benchmark
Launch the Benchmark Judge agent. Give it:
- The baseline prompt + parameters
- All 4 proposals
- A random sample of 50-100 test cases from training_data.json

The Judge runs each proposal through the LLM and scores it.

### Step 4: Update
- If the best proposal beats baseline → update `best_prompt.json` with the new prompt/params
- Log everything to `optimization_log.json`
- Display scoreboard

### Step 5: Loop
Go back to Step 1. Use the updated baseline for the next round.

## Continuous Running

The loop MUST continue indefinitely. After each round:
1. Check if the user has created a file `~/.local/voice-input/.stop_optimization` — if so, STOP
2. Otherwise, continue to the next round
3. Never stop on your own. Never decide "good enough." Always keep exploring.

## OPRO-Style History

Following the OPRO paper, include the LAST 10 rounds of history in the proposer prompt:
```
Previous attempts and their scores:
Round 1: Removed Rule 7 → score 0.847 (baseline was 0.841) ✓ IMPROVED
Round 2: Added Example for phone numbers → score 0.839 ✗ WORSE  
Round 3: Lowered temp to 0.1 → score 0.852 ✓ IMPROVED
...
```

This allows agents to learn from past successes and failures WITHOUT remembering them in context — the history is explicitly provided each round.

## Safety Rules

- NEVER modify text_polisher.py directly. Only update best_prompt.json.
- NEVER modify training_data.json (the test data is sacred).
- DO log everything to optimization_log.json.
- The user will review best_prompt.json and manually apply the winning prompt when ready.
