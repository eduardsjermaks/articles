---
title: I Asked an LLM to Fix My Terminal Commands. It Built a Tool.
published: false
description: A practical experiment in using an LLM agent to turn one terminal frustration into a working, cross-platform CLI.
tags: ai, cli, devtools, go
---

I regularly lose time to terminal muscle memory.

I work across Windows and Unix-like shells, so I will remember the right command in the wrong environment, transpose a Git subcommand, or use a valid binary with an invalid subcommand. The fix is usually easy to find. The interruption is the costly part: stop, search, translate the answer back into the current shell, and try again.

The idea is not new. There are projects that help with failed commands, as well as terminal-integrated LLMs such as aichat. However, I wanted something that does one thing well: fix commands in a seamless workflow powered by an LLM.

I wanted a deliberately narrow tool: when a command fails, suggest the command I probably meant, let me inspect it, and run it only after confirmation.

During a Fable promotion period, I used an LLM coding agent to see how far a well-scoped prompt could get me. With very little steering, it produced a usable Go prototype in roughly an hour.

The project is now open source: [nudge](https://github.com/eduardsjermaks/nudge).

## The workflow I wanted

The core interaction is intentionally small:

1. Run a command as usual.
2. If it fails, type `fix` (or bare `nudge`) to get a suggested correction.
3. Review it, then press Enter to run, `e` to edit it first, or `n` to cancel.

The cheapest case is a plain typo, which never reaches a model at all:

```text
PS> git pshu
git: 'pshu' is not a git command. See 'git --help'.

PS> fix
`git pshu` isn't a valid command. Did you mean:
  → git push    (typo fix for `git pshu`)
Run it? [Enter = yes / n = no / e = edit]
```

The `(typo fix for ...)` label is the tool telling me it answered locally, in
under 10 ms, without a network call.

The cross-shell version of the same mistake is reaching for a binary that does
not exist here. Because the binary is missing, the shell's command-not-found
hook fires and I do not have to type anything at all:

```text
PS> printenv
`printenv` isn't a valid command. Did you mean:
  → Get-ChildItem env:    (list all environment variables)
Run it? [Enter = yes / n = no / e = edit]
```

The case that actually motivated the tool is narrower: the binary exists, so no
hook fires and no spell-checker helps — the invocation is just wrong. This is
what bare `fix` is for, since it reads the previous command and its exit code
from shell history:

```text
PS> dotnet install dotnet-ef
Could not execute because the specified command or file was not found.

PS> fix
`dotnet install dotnet-ef` isn't a valid command. Did you mean:
  → dotnet tool install --global dotnet-ef    (install dotnet-ef as a global tool)
Run it? [Enter = yes / n = no / e = edit]
```

It also accepts an intent stated in plain words, which is the same loop with a
different input:

```text
PS C:\repos> just create dir mynewproj and init repo there
`just create dir mynewproj and init repo there` isn't a valid command. Did you mean:
  → mkdir mynewproj; cd mynewproj; git init    (create directory, enter it, initialize git repo)
Run it? [Enter = yes / n = no / e = edit]

PS C:\repos\mynewproj>
```

The prompt in that last line is the point: the `cd` applied to my actual
session. A suggestion that runs in a child process would have left me back in
`C:\repos`.

The important constraint was context switching. I did not want a general chat
experience embedded in the terminal. I wanted one quick repair loop that stayed
close to the failed command.

## What the agent produced

The initial prototype was enough for my own daily use. The subsequent work turned it into something I could reasonably share:

- A fast local matcher handles simple typos such as `git pshu` without calling a model.
- Model-backed suggestions handle unknown commands, incorrect subcommands, and plain-English intent.
- Shell integration supports PowerShell, bash, zsh, and fish, so the correction can use the failed command and its exit code.
- Suggestions that can be destructive, such as `rm -rf`, force-pushes, or hard resets, require an explicit typed `y`; Enter is not enough.
- The tool can use either cloud providers or a local Ollama-compatible model, depending on quality, privacy, and cost requirements.

The safety guard is the one place where the interaction deliberately gets
slower. When the model flags a suggestion as destructive, the prompt changes
and a reflexive Enter no longer accepts it:

```text
PS> and uninstall dotnet-ef globally
`and uninstall dotnet-ef globally` isn't a valid command. Did you mean:
  → dotnet tool uninstall --global dotnet-ef    (uninstall dotnet-ef tool globally)
  ! destructive: model flagged this as destructive - requires an explicit 'y'
Run it? [y = yes / Enter or n = no / e = edit]
```

That split matters. A model call is useful when the problem requires interpretation; it is wasteful when the command is simply a typo. The tool should be instant and offline for the easy case, then use an LLM only when it earns its place.

## The engineering was after the first hour

"The agent built it in an hour" is true for the first usable version, but incomplete as a development story.

I spent additional time testing shell behavior across operating systems, improving installation and setup, and making unsafe actions harder to execute accidentally. In particular, shell state is tricky: a suggested `cd` must affect the current shell session, not a child process that disappears immediately. Installation is also part of the product. A CLI is not really useful if it only works in the environment where it was generated.

This is the pattern I find most useful with coding agents:

- Give the agent a narrow, observable problem.
- Get to a working vertical slice quickly.
- Treat the output as a starting point for testing, failure handling, setup, and review.

The agent accelerated implementation. It did not remove the need to decide what should happen when a command is uncertain, destructive, shell-specific, or sent to a cloud model.

## What did the session cost?

One session on July 15 reported the following token usage:

| Output | Cache write | Cache read | Fresh input |
| ---: | ---: | ---: | ---: |
| 382,094 | 418,616 | 35,691,065 | 551 |

I used a Pro subscription during the promotion period, so I was not billed per token. Priced against the API rates for the model I was using, the same session would have cost roughly **$63**.

That number needs context, and the shape of it is more interesting than the total. Almost all of the input was cache reads, billed at a tenth of the normal input rate. The same 35.7 million tokens at full price would have put the session near $375 — caching is doing about a 6x reduction, and any estimate like this moves with the model, provider, and cache pricing. It is also not the cost of using the CLI; a single correction is roughly 300-500 input and 60 output tokens, fractions of a cent. This is the cost of an extended agentic development session that explored, generated, and refined the project.

For a personal tool, $63 is not automatically cheap. But it is a useful comparison point: I got from a recurring friction point to a working tool much faster than I expect I would have by building every part from scratch. Whether that is worth it depends on how often the tool saves time and how much you value the experiment itself.

## Where this approach works well

LLM-assisted development feels most compelling when all of these are true:

- The problem is concrete enough to demonstrate and test.
- The first version can be deliberately small.
- The developer can judge whether the result is correct.
- The cost of a wrong answer is controlled by review, confirmation, and tests.

A terminal command helper fits those conditions. It has clear inputs and outputs, and it can require confirmation before it executes anything risky.

It would be a mistake to generalize this result into "one prompt builds production software." The useful lesson is smaller: agents can make small tools economically viable when the developer supplies a precise problem, defines the safety boundaries, and does the final engineering work.

## Try it or inspect the code

The repository includes installation instructions, shell integration, privacy notes, supported providers, and development tests:

[github.com/eduardsjermaks/nudge](https://github.com/eduardsjermaks/nudge)

I am continuing to use it as a real terminal workflow rather than treating it as a one-off demo. That is the real test for this kind of project: not whether an LLM can generate a repository, but whether the resulting tool remains useful after the novelty wears off.