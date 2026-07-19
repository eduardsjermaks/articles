During Fable's promotion period, I continued experimenting and used it to build a tool for a small but recurring frustration: terminal command mistakes

I switch between Windows and Unix-like shells often enough to type the right command in the wrong environment, misspell a Git subcommand, or use a real CLI with the wrong arguments. The correction is usually simple, but the context switching adds up.

The idea is not new. There are projects that help with failed commands, as well as terminal-integrated LLMs such as aichat. However, I wanted something that does one thing well: fix commands in a seamless workflow powered by an LLM.

The first usable Go prototype took roughly an hour with minimal steering.

The result is nudge: https://github.com/eduardsjermaks/nudge

One development session reported about 382k output tokens, 419k cache writes, and 35.7M cache reads. My rough API-price estimate for that session is about $61.

For me, the tool was already useful at that point. I decided to spend a few more hours testing it across operating systems and making setup more user-friendly, in case others find it useful too.

But it is a good example of where LLM agents change the economics of small developer tools.

The experiment is now an open-source tool rather than a demo. I am curious which small, repetitive friction point you would turn into a tool if the first working version took an afternoon instead of a weekend.

Full article: https://dev.to/eduardj_67dc3f850/i-asked-an-llm-to-fix-my-terminal-commands-it-built-a-tool-2hj

#AI #DeveloperTools #CLI #Golang #SoftwareEngineering