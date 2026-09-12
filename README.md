# Local AI Agent with Ollama and Pydantic AI

A minimal, fully local personal assistant built with [`pydantic-ai`](https://ai.pydantic.dev/) and [Ollama](https://ollama.com/). The agent runs entirely on your machine — no API keys, no cloud calls, no data leaving your computer — and can use a small set of tools to answer questions, do arithmetic, and keep notes across turns.

## Features

- **100% local inference** via Ollama (default model: `llama3.2:latest`).
- **Tool use** with automatic tool selection by the model:
  - `get_current_time` — returns the current date and time.
  - `calculate` — safely evaluates basic arithmetic expressions.
  - `save_notes` — appends a note to a local `notes.txt` file.
  - `read_notes` — reads back all saved notes.
- **Conversational memory** within a session (message history is passed back on every turn).
- **Simple REPL** you can quit anytime with `quit` or `exit`.

## Requirements

- Python 3.10+
- [Ollama](https://ollama.com/download) installed and running locally
- The `llama3.2` model pulled locally
- Python packages: `pydantic-ai`

## Installation

1. **Install Ollama** and start the server:

   ```bash
   # macOS / Linux install instructions: https://ollama.com/download
   ollama serve
   ```

2. **Pull the model** used by the script:

   ```bash
   ollama pull llama3.2:latest
   ```

3. **Clone this repo and install dependencies:**

   ```bash
   git clone <your-repo-url>
   cd <repo-folder>
   pip install pydantic-ai
   ```

## Usage

Run the script:

```bash
python agent.py
```

You'll see:

```
Local agent ready! Type 'quit' to exit.

You:
```

Type any question. The agent will decide when to use its tools.

### Example session

```
You: What time is it?
Agent: It's Saturday, September 12, 2026 at 03:14 PM.

You: What's 234 * 17 + 9?
Agent: 234 * 17 + 9 = 3987.

You: Save a note that I need to email Prof. Smith about the reading list.
Agent: Note saved.

You: What notes do I have?
Agent:
- I need to email Prof. Smith about the reading list.
```

## How it works

The script wires together three pieces:

1. **`OllamaModel`** — a `pydantic-ai` wrapper around Ollama's local OpenAI-compatible endpoint (`http://localhost:11434/v1`).
2. **Tools** — plain Python functions with type hints and docstrings. `pydantic-ai` reads the signatures and docstrings to expose them to the model as callable tools.
3. **`Agent`** — orchestrates the loop: it sends the user's message plus the message history to the model, executes any tool calls the model requests, and returns the final response.

The instruction prompt tells the model it is a local personal assistant and that it should use the available tools whenever helpful.

## Project structure

```
.
├── agent.py       # main script
├── notes.txt      # created on first save_notes call
└── README.md
```

## Configuration

- **Change the model:** edit the `OllamaModel(...)` call to any model you have pulled locally, e.g. `"qwen2.5:7b"` or `"mistral:latest"`.
- **Change the notes file:** edit the `NOTES_FILE` constant at the top of `agent.py`.

## Security note

The `calculate` tool restricts input characters to digits and basic operators before evaluating, so arbitrary code cannot be executed through it. Still, `eval` is used under the hood — if you plan to extend the tool or expose the agent to untrusted input, replace it with a proper expression parser (for example [`asteval`](https://newville.github.io/asteval/) or Python's `ast.literal_eval` for stricter cases).

## Possible next steps

- Add more tools (file search, web fetching, calendar lookup).
- Persist message history across sessions.
- Swap the REPL for a small web UI (Gradio, Streamlit, or FastAPI + HTMX).
- Experiment with different local models and compare tool-use reliability.

## License

MIT
