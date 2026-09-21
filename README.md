# Assignment 2: Harness Engineering

**Author:** Yashaswini Dinesh


## Deliverables and links

| Part | What I built | Code | YouTube walkthrough 
|------|--------------|------|---------------------|
| A | A coding-agent harness written from scratch on the OpenRouter API, built in five progressive stages and then packaged | [`part-a-harness/`](part-a-harness) | link
| B | DeepSeek Harness in Creator mode: five or more community plugins, plus two plugins I wrote from scratch | [`part-b-deepseek-harness/`](part-b-deepseek-harness) | link
| C | A custom ML harness for autoresearch: an LLM edits `train.py`, the harness runs it, scores it, and keeps or reverts the change | [`part-c-autoresearch/`](part-c-autoresearch) | link

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
