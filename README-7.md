# Assignment 2: Harness Engineering

**Author:** Yashaswini Dinesh
**GitHub repo:** https://github.com/PASTE_YOUR_USERNAME/PASTE_REPO_NAME

## Deliverables and links

| Part | What I built | Code | YouTube walkthrough |
|------|--------------|------|---------------------|
| A | A coding-agent harness written from scratch on the OpenRouter API, built in five progressive stages and then packaged | [`part-a-harness/`](part-a-harness) | PASTE_PART_A_YOUTUBE_LINK |
| B | DeepSeek Harness in Creator mode: five or more community plugins, plus two plugins I wrote from scratch | [`part-b-deepseek-harness/`](part-b-deepseek-harness) | PASTE_PART_B_YOUTUBE_LINK |
| C | A custom ML harness for autoresearch: an LLM edits `train.py`, the harness runs it, scores it, and keeps or reverts the change | [`part-c-autoresearch/`](part-c-autoresearch) | PASTE_PART_C_YOUTUBE_LINK |

Each video is a full walkthrough of every file in that part with the code running. The Part B video also includes the
DeepSeek Harness demos.

## The idea behind the project

A language model only produces text. A **harness** is the program around it that turns that text into action. It sends
the conversation and a list of tools to the model, runs whichever tools the model asks for, feeds the results back, and
repeats until the model gives a final answer. Everything else in agent engineering, such as safety checks, session logs,
context management, plugins and experiment loops, is built on top of that loop. The three parts explore the same idea at
three levels.

- **Part A** builds the loop from nothing, one idea per stage, so each piece can be explained on its own.
- **Part B** studies the same idea as a plugin architecture, where even the tool registry and the agent loop are plugins.
- **Part C** aims the loop at a research task and keeps the control flow in code, so the model cannot wander off.

## Part A: a coding-agent harness from scratch

The harness talks to any tool-calling model through OpenRouter and can list, read, search, edit and run code inside a
workspace folder. It was built in stages, and each stage is a runnable script in `part-a-harness/stages/`.

1. **Single call:** one request to OpenRouter, to prove the key and model work.
2. **Memory:** the API is stateless, so memory means resending the message list on every turn.
3. **The agent loop:** the model returns tool calls, the harness runs them and appends the results, and this repeats until the model answers.
4. **Coding tools:** write file, exact-match edit, grep and shell. This stage is deliberately unsafe.
5. **Guardrails:** workspace confinement, shell timeout, output truncation, blocked commands, approval prompts, secret scrubbing and a step cap.
6. **Packaged harness (`harness/`):** the same ideas as a clean package, plus JSONL session logs with resume, context compaction that never splits a tool call from its result, repeated-call detection, and a CLI.

The demo task is a small shopping library with two planted bugs. The agent runs the tests, reads the code, fixes both bugs
and re-runs until the tests pass. There are 22 tests, and they run without an API key by using a scripted stand-in for the model.

## Part B: DeepSeek Harness and Creator mode

DeepSeek Harness (`dsh`) is an open-source agent runtime built on the Cordis plugin framework, and its Creator mode is used
to build new plugins and presets. I installed it, showed its four modes (Standard, Code, Minimal, Creator), added community
plugins, and wrote two plugins myself.

- **`dsh-second-brain`** gives the agent a persistent markdown notebook. It has five tools: save, search, read, list and backlinks. Notes live in a vault folder, `[[wikilinks]]` connect them, and slugs are restricted so a title cannot escape the vault.
- **`dsh-tool-guard`** is a safety net and flight recorder. It denies tool calls that match dangerous patterns (`rm -rf ~`, `curl | sh`, `.env` files, SSH keys, force-push), writes a JSONL audit entry for every call, and adds a `guard_report` tool. It uses three extension points: the tool guard, the `tools/pre-execute` and `tools/result` events, and tool registration.

Both plugins are tested against the real DeepSeek Harness runtime (14 tests, version 0.1.5-rc.2) and both load in a real
`dsh web` session. The Creator-mode prompts I used and the community plugins I installed are documented in the folder.

## Part C: a custom ML harness for autoresearch

This follows the autoresearch idea: an LLM improves a model by running experiments on its own. A chat model tends to stop
after a few tries, so the loop lives in code:

`baseline -> [ ask the model for the next train.py -> validate -> run with a time limit -> compare -> keep or revert -> log ] x N`

- `prepare.py` is fixed. It creates a dataset with non-linear signal and missing values, splits it into train, validation and a **hidden test** set, and defines the metric (ROC-AUC).
- `train.py` is the only file the agent may edit. The baseline is plain logistic regression.
- `runner.py` is the harness. It rejects unsafe or malformed proposals (bad syntax, forbidden constructs, any attempt to read the test split), enforces a per-run timeout, keeps a change only if it beats the best score by a real margin, restores the best version otherwise, and records kept experiments as git commits.
- `report_test.py` scores the final model on validation and on the hidden test split, to expose over-fitting.
- The same loop is also packaged as a Claude Code skill and an `/autoresearch` command, so a coding assistant can run it directly.

Results of my real model run: PASTE_MODEL_NAME, baseline validation AUC 0.654 to PASTE_BEST_AUC after PASTE_N experiments
(PASTE_KEEPS kept, PASTE_DISCARDS discarded, PASTE_CRASHES crashed), hidden-test AUC PASTE_TEST_AUC. See `results/progress.png`.

## How to run

```bash
# Part A (Python 3.10+)
cd part-a-harness && pip install -r requirements.txt && cp .env.example .env    # add your OpenRouter key
python -m pytest
python demo/reset_demo.py && python -m harness --workspace demo/workdir "Run the tests, find the bugs, fix them"

# Part B (Node 22+)
cd part-b-deepseek-harness && npm install && npm test
dsh plugin --profile web add /absolute/path/to/plugins/dsh-second-brain
dsh plugin --profile web add /absolute/path/to/plugins/dsh-tool-guard

# Part C (Python 3.10+)
cd part-c-autoresearch && pip install -r requirements.txt && python -m pytest
python runner.py --iters 12 --tag run1      # real run through OpenRouter
```

## Limitations

- The guardrails in Parts A, B and C are tripwires that lower risk. They are not sandboxes. For real isolation, run the harness in a container.
- DeepSeek Harness is a developer preview with compatibility-breaking changes, so the Part B plugins were tested against one specific version.
- The metric in Part C can be over-fitted through repeated tuning, which is why the hidden test split exists.
- API keys stay in `.env`, which is git-ignored and never committed.
