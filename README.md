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

## Model suggestions

If the configured model isn't served by the instance, `localai` queries the
server's model list (`/api/models`, derived from `LOCALAI_URL`, using the same
API key) and suggests alternatives instead of showing a raw HTTP error.

- **Interactive mode** checks the model at startup and again if a request
  fails. It shows a numbered list of available models; pick one by number or
  name, and the choice is saved to `~/.localai/config.json`. After a failed
  request, your message is retried with the new model. A blank answer cancels.
- **Batch mode** prints the available models and a
  `localai config set model <name>` hint, then exits.

The list includes only chat models. The server doesn't label models by type,
so embedding, reranking, speech, and image-generation models are filtered out
by name (see `NON_CHAT_PATTERN` in `localai`), along with models marked
inactive. If the model list can't be fetched, the original error is shown.

If `LOCALAI_MODEL` is set in your environment, it overrides the saved choice on
the next run.
