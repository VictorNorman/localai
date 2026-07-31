# localai

A CLI for Calvin's LocalAI (Open WebUI) instance, with filesystem/shell tool access.

The assistant can read files, write files, list directories, and run shell
commands via function calling. Destructive-looking shell commands (`rm`,
`rmdir`, `unlink`, `shred`, `find -delete`, or anything mentioning "delete")
prompt for confirmation before running; everything else executes directly, so
only point this at a model/endpoint you trust.

## Setup

```sh
pip install -r requirements.txt
```

Configure credentials via environment variables:

```sh
export LOCALAI_API_KEY="..."
export LOCALAI_MODEL="..."
export LOCALAI_URL="https://localai.calvin.edu/api/chat/completions"  # optional, this is the default
```

or persist them with:

```sh
./localai config set api_key ...
./localai config set model ...
```

which writes to `~/.localai/config.json`.

### macOS note

macOS's stdlib `readline` module is backed by libedit, which mis-renders
colored prompts (it mishandles multiple `\001`/`\002` invisible-width spans).
`gnureadline` bundles real GNU readline and fixes this — install it with
`pip install -r requirements.txt` (included for `sys_platform == "darwin"`).

## Usage

Interactive mode:

```sh
./localai
```

One-shot batch mode:

```sh
./localai "list the files in this directory"
```

Session management:

```sh
./localai session save|load|clear|info|export [file]
```

Conversation history is persisted to `~/.localai/current_session.json`
between runs. Command-line input history is kept in `~/.localai/history.json`.
