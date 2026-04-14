# String Cookbook

Practical, runnable examples for SFMD and the String runtime. Start at the top and work down — each chapter builds on the previous one.

For the runtime itself, see [`string-os/string`](https://github.com/string-os/string). For the format specification, see [`string-os/sfmd`](https://github.com/string-os/sfmd).

## Chapters

| # | Chapter | What you'll learn |
|---|---------|-------------------|
| 00 | [CLI Quick Start](./00-cli-quickstart.md) | Install String and run your first command |
| 01 | [Editing](./01-editing.md) | Edit a document with block addressing and diffs |
| 02 | [Web Browsing](./02-web-browsing.md) | Read the live web as rendered markdown |
| 03 | [Single-Page App](./03-single-page-app.md) | Build a one-file SFMD app with actions |
| 04 | [Multi-Page App](./04-multi-page-app.md) | Navigate between pages using shortcuts |
| 05 | [Multi-Topic](./05-multi-topic.md) | Run the same app against multiple configs |
| 06 | [CLI App](./06-cli-app.md) | Wrap a shell CLI as an SFMD action surface |
| 07 | [Cross-Agent Portability](./07-portability.md) | **The key demo** — one file, three AI agents, same behavior |

## How to run the examples

1. Install String: see [`00-cli-quickstart.md`](./00-cli-quickstart.md).
2. Each chapter is a single markdown file you can read or copy. Code blocks marked `bash` are commands you can paste into your terminal. Code blocks marked `markdown` are SFMD content.
3. Chapters that demonstrate multi-agent behavior (notably [07](./07-portability.md)) will also walk you through setting up additional agents like Claude Desktop or Cursor.

## Contributing an example

Open a PR with a new chapter. Keep it short — the shortest useful example is best. The chapter should:

1. State the goal in one sentence at the top.
2. Show the SFMD file(s) in full.
3. Show the commands to run.
4. Show the expected output.
5. Explain what just happened.

See existing chapters for the shape.

## License

The cookbook is MIT licensed. You may copy any example directly into your own project without attribution.
