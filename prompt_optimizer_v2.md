# Prompt Optimization V2 — Evolutionary Collaborative Framework

## Philosophy

Based on EvoPrompt (ICLR 2024) + MAEF (multi-agent evolutionary framework):
- **Population-based**: Maintain multiple candidate prompts, not just one "best"
- **Bold mutations**: Agents can make BIG changes, not just one-line edits
- **Collaborative breeding**: Agents can combine their ideas (crossover)
- **Beat yourself**: Each agent competes against their OWN previous best, not others
- **Speed matters**: Prompt length and inference speed are part of the score
- **Shared knowledge**: All history, all scores, all failures are PUBLIC to all agents
- **No limits on discussion**: Agents can debate and collaborate freely
- **Last 50 rounds of history**: Enough context to see patterns

## Architecture

```
Controller (lightweight)
  │
  ├── Spawn ONE agent per round
  │     This agent role-plays ALL team members:
  │     
  │     Phase 1: REVIEW (all agents see everything)
  │       - Read current population (top 5 prompts)
  │       - Read last 50 rounds of history
  │       - Read failure analysis
  │       - Each agent identifies what to improve
  │     
  │     Phase 2: PROPOSE (bold changes)
  │       - Each agent proposes their change (can be BIG)
  │       - Agents can propose COMBINATIONS with each other
  │       - No limit on how much to change
  │       - Must justify: "this will fix X cases because Y"
  │     
  │     Phase 3: BREED (crossover)
  │       - Take best ideas from multiple proposals
  │       - Create COMBINATION proposals (agent A's rule + agent B's example)
  │       - Generate 2-3 hybrid proposals in addition to individual ones
  │     
  │     Phase 4: BENCHMARK
  │       - Test ALL proposals (individual + hybrids) against 4000 cases
  │       - Score = accuracy * speed_factor
  │       - speed_factor = 1.0 if prompt < 4000 chars, decreases for longer
  │     
  │     Phase 5: UPDATE POPULATION
  │       - Keep top 5 prompts (not just top 1)
  │       - Replace worst in population if new prompt is better
  │       - Record everything
  │
  └── Repeat
```

## The 5 Agents (Evolved from V1)

### 1. The Radical Restructurer (was Minimalist)
Not just "delete one line" — can COMPLETELY rewrite the prompt structure.
Can merge rules, reorganize sections, change the entire format.
Goal: shorter prompt = faster inference = better speed score.
Can propose deleting 5 rules and replacing with 2 better ones.

### 2. The Pattern Synthesizer (was Example Crafter)  
Not just "add one example" — can redesign the entire example strategy.
Can replace ALL examples with fewer, more powerful ones.
Can create "meta-examples" that teach multiple patterns at once.
Goal: maximum teaching power per example.

### 3. The Rule Architect (was Rule Writer)
Not just "add one rule" — can redesign the rule hierarchy.
Can merge overlapping rules, create sub-rule trees, add priorities.
Can propose a completely different rule organization.
Goal: clearer rules = fewer model mistakes.

### 4. The Parameter Explorer (was Temperature Tuner)
Not just "change temp by 0.1" — can propose radically different parameter combinations.
Can test temp=0.0 (greedy), temp=0.5 (creative), different top_p/top_k combos.
Can propose using different parameters for different input lengths.
Goal: find the sweet spot between consistency and flexibility.

### 5. The Devil's Advocate (NEW)
The agent who tries to BREAK the other agents' proposals.
Generates adversarial inputs that would fail under each proposal.
Forces the team to think about regression risks.
Can propose "defensive" changes that prevent known failure patterns.
Goal: prevent regressions, find blind spots.

## Scoring Formula

```
final_score = accuracy_score * speed_factor

accuracy_score = (exact_matches + partial_scores) / total_cases

speed_factor:
  prompt < 3000 chars → 1.00 (bonus for being concise)
  prompt 3000-5000 chars → 0.98
  prompt 5000-7000 chars → 0.95
  prompt > 7000 chars → 0.90 (penalty for being too long)
```

This incentivizes the Radical Restructurer to find shorter prompts.

## Population Management

Instead of keeping only the #1 best prompt, maintain a **population of 5**:
```json
{
  "population": [
    {"id": 1, "prompt": "...", "params": {...}, "score": 0.997, "round": 15},
    {"id": 2, "prompt": "...", "params": {...}, "score": 0.995, "round": 12},
    {"id": 3, "prompt": "...", "params": {...}, "score": 0.993, "round": 8},
    {"id": 4, "prompt": "...", "params": {...}, "score": 0.990, "round": 14},
    {"id": 5, "prompt": "...", "params": {...}, "score": 0.985, "round": 13}
  ]
}
```

Why? Crossover works better with diverse parents. Two mediocre prompts can breed a great one.

## Collaboration Rules

- All agents see ALL history, ALL scores, ALL failure cases
- Any 2+ agents can propose a JOINT change (crossover)
- No limit on discussion rounds between agents
- No limit on how much to change in one round
- The ONLY constraint: must beat at least one prompt in the population to survive

## History: Last 50 Rounds

Each round records:
- All proposals (individual + hybrids)
- All scores
- Which cases improved / regressed
- Which agent(s) proposed the winner
- Collaboration logs (which agents combined)

Agents see the last 50 rounds to identify long-term patterns:
- "Deleting examples always causes regression" → stop trying that
- "Adding more dollar amounts always helps" → try more
- "Crossover between Restructurer + Synthesizer produces best results"

## Files

- `~/.local/voice-input/training_data_v2.json` — 4000 test cases (SACRED)
- `~/.local/voice-input/prompt_population.json` — top 5 prompts
- `~/.local/voice-input/optimization_log_v2.json` — full history
- `~/.local/voice-input/text_polisher.py` — DO NOT MODIFY directly

## Stop Condition

None. Runs forever until user says stop.
