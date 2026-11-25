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

Show the current settings with `localai config` (or `config` at the interactive
prompt, which also shows the model in use this session). The API key is masked.

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

### Sessions

Every run starts with a clean conversation; nothing is loaded automatically,
and a conversation is lost on exit unless you save it under a name. Saved
sessions live in `~/.localai/sessions/<name>.json`. Command-line input history
(not the conversation) is kept in `~/.localai/history.json`.

Inside the interactive prompt:

| Command                | Effect                                                            |
| ---------------------- | ----------------------------------------------------------------- |
| `session save [name]`  | Save the conversation (under the active name if none is given)    |
| `session load <name>`  | Replace the current conversation with a saved one                 |
| `session list`         | List saved sessions (the active one is marked `*`)                |
| `session info [name]`  | Show a session's file, last-modified time, and message count      |
| `session clear <name>` | Delete a saved session                                            |

Once you save or load a session, it becomes the active session and is
re-saved automatically after every turn. `clear` (on its own) starts a fresh
conversation.

From the shell, `./localai session list|info|clear <name>` manage saved
sessions, and `./localai session export <name> [file]` writes a saved
session's messages to a JSON file (default `<name>.json`). The shell forms of
`save` and `load` don't carry a conversation between runs, so use them from
the interactive prompt instead.

## Model suggestions

If the configured model isn't served by the instance, `localai` queries the
server's model list (`/api/models`, derived from `LOCALAI_URL`, using the same
API key) and suggests alternatives instead of showing a raw HTTP error.

- **Interactive mode** checks the model at startup and again if a request
  fails. It shows a numbered list of available models; pick one by number or
  name. The choice applies to the current run only and is not saved; use
  `config set model <name>` to make it permanent. After a failed request, your
  message is retried with the new model. A blank answer cancels.
- **Batch mode** prints the available models and a
  `localai config set model <name>` hint, then exits.

List the available models at any time with `localai config list model`
(or `config list model` at the interactive prompt).

The list includes only chat models. The server doesn't label models by type,
so embedding, reranking, speech, and image-generation models are filtered out
by name (see `NON_CHAT_PATTERN` in `localai`), along with models marked
inactive. If the model list can't be fetched, the original error is shown.

If `LOCALAI_MODEL` is set in your environment, it overrides the saved choice on
the next run.
