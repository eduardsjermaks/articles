## Experiment Setup

To compare how different models analyze the same codebase, I used a fixed set of prompts and collected the responses produced by each model.

Prompts were refined with ChatGPT to make them clear and optimized for the model.

The table below shows the prompts used in the experiment and the corresponding results.

| Prompt | Result |
|--------|--------|
| [1-codex-readme-prompt.md](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/make-readme/1-codex-readme-prompt.md) | [Codex (medium thinking) response](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/make-readme/responses/1-codex-readme-prompt-response.md) |
| [1-codex-readme-prompt.md](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/make-readme/1-codex-readme-prompt.md) | [Codex (extra thinking) response](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/make-readme/responses/1-codex-readme-prompt-response-extra.md) |
| [1-make-readme-opus.md](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/make-readme/1-make-readme-opus.md) | [Claude Code response](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/make-readme/responses/1-opus-readme-response.md) |
| [1-vscode-readme-prompt.md](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/make-readme/1-vscode-readme-prompt.md) | [VSCode response](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/make-readme/responses/1-vscode-readme-response.md) |


These responses are evaluated in the next section using an AI-as-Judge approach.

# Using AI-as-Judge to Compare LLM Codebase Analysis

When comparing outputs from different LLM tools, subjective reading is unreliable.  
To make the comparison reproducible, I used an **AI-as-judge approach with a fixed rubric**, where two generated documents were evaluated against the same criteria.

The judge model used in all runs was **Gemini 3 Pro**.

---

## Reference table

| Run | Prompt file | Result file | Document A | Document B |
|------|------------|------------|-----------|-----------|
| 1 | [1-ai-judge-prompt](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/ai-judge/1-ai-judge-prompt.md) | [1-ai-judge-prompt-result](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/ai-judge/results/1-ai-judge-result.md) | Codex medium thinking (GPT-5.4)  | Claude Code (Opus 4.7) |
| 2 | [1-ai-judge-vs-code-auto-vs-codex](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/ai-judge/1-ai-judge-vs-code-auto-vs-codex.md) | [1-ai-judge-vs-code-auto-vs-codex-result](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/ai-judge/results/1-ai-judge-vs-code-auto-vs-codex-result.md) | Codex medium thinking (GPT-5.4)  | VSCode default auto mode (GPT-5.4, Opus 4.6, Sonnet 4.6) |
| 3 | [1-ai-judge-gemini-prompt-extra](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/ai-judge/1-ai-judge-gemini-prompt-extra.md) | [1-ai-judge-gemini-prompt-extra-result](https://github.com/eduardsjermaks/articles/blob/main/articles/01-LLM-SDD-Migration/prompts/ai-judge/1-ai-judge-gemini-prompt-extra.md) | Codex medium thinking (GPT-5.4)  | Codex extra thinking (GPT-5.4) |

---