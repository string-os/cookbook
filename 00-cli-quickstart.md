# String CLI Quick Start

The fastest path from zero to a running SFMD command. Takes about 2 minutes.

## Install

**From npm** (once packages are published):

```bash
npm install -g @string-os/string
```

**From source** (while packages are in prep):

```bash
git clone https://github.com/string-os/string.git
cd string
pnpm install
pnpm -r build
alias string='node packages/string/dist/cli.js'
```

Verify:

```bash
string --help
```

You should see the help banner. If you see `command not found`, make sure the alias is set or the global install succeeded.

## Basic usage

String has two modes:

```bash
string <topic> '<body>'     # run one command (one-shot)
string <topic>              # enter the interactive REPL
```

A topic takes the form `type:name`. If you write only a name, it defaults to `file:<name>`.

| Topic example | Meaning |
|----------------|---------|
| `file:main` or `main` | File-based session (default) |
| `app:weather` | App session |
| `app:weather:korea` | App session with a named config |
| `web:docs` | Web document session |
| `bash:dev` | Shell session (opt-in; disabled by default) |

## One-shot execution

```bash
string app:weather '/act.now --city Seoul'
string file:main '/open ./index.md'
string web:docs '/open https://developer.mozilla.org'
string main '/nav'
```

Output is wrapped in ChanFlow tags so the boundaries of each response are machine-parseable:

```
<𝒞=string:app:weather>
Seoul: Sunny 22°C
</𝒞>
```

## Interactive REPL

Pass only the topic to enter the REPL. Prompts go to `stderr`; responses go to `stdout` (still wrapped in ChanFlow).

```bash
string app:weather
```

```
[app:weather] > /act.now --city Seoul
<𝒞=string:app:weather>
Seoul: Sunny 22°C
</𝒞>
[app:weather] > /exit
```

Piped input also works:

```bash
echo '/act.now --city Seoul' | string app:weather
```

## Core commands

| Command | Description |
|---------|-------------|
| `/open <path>` | Open a document (path, URI, `@shortcut`, or `file.md#block`) |
| `/act` | List actions for the current document |
| `/act.name` | Run an action (executes immediately if no required params) |
| `/act.name --flag value` | Run an action with parameters |
| `/act.name --help` | Show an action's parameter schema |
| `/nav` | List the navigation menus |
| `/nav page` | List the shortcuts on the current page |
| `/back` | Go back to the previous document |
| `/refresh` | Reload the current document |
| `/info` | Show session and document state |
| `/set` | List variables |
| `/set {var} = "value"` | Set a session variable (memory) |
| `/set $VAR = "value"` | Set a persistent variable (disk) |
| `/tool:name` | Run a tool |
| `/install <file>` | Install an app or tool |

## Flags

| Flag | Description |
|------|-------------|
| `--json` | Output a JSON envelope instead of ChanFlow |
| `--help` | Show usage |

```bash
string --json app:weather '/act.now --city Seoul'
# {"ok":true,"topic":"app:weather","content":"Seoul: Sunny 22°C"}
```

## Daemon management

The daemon starts automatically when you run a command. You can manage it manually if you want:

```bash
string --daemon start      # start
string --daemon stop       # stop
string --daemon status     # check status
```

Configure via environment variables:

```bash
export STRINGD_PORT=3100    # default: 3100
export STRINGD_USER=default # default: "default"
export STRINGD_HOME=~       # default: os.homedir()
```

## Example: using an installed app

```bash
# Run a single command
string app:weather '/act.now --city Seoul'

# Or chain commands in the REPL
string app:weather
[app:weather] > /act.now --city Seoul
[app:weather] > /act.now --city Tokyo
[app:weather] > /exit
```

## Next

- [01-editing.md](./01-editing.md) — edit a document with diffs
- [02-web-browsing.md](./02-web-browsing.md) — read the live web as rendered markdown
- [07-portability.md](./07-portability.md) — run one SFMD file in multiple AI agents
