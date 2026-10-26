# YT Lecture RAG

Semantic search and question answering over YouTube DSA lecture transcripts. Ask a question in English or Hinglish and receive an answer grounded in relevant lecture segments, with timestamp links that can jump to the corresponding point in the YouTube video.

## Features

- Semantic search over lecture transcripts.
- Retrieval-augmented question answering with timestamped source citations.
- CLI commands for searching, asking questions, checking statistics, and evaluating retrieval.
- FastAPI web interface with an embedded YouTube player.
- Reuse bundled transcripts so users do not need to download videos or transcribe the playlist themselves.

## How It Works

1. Lecture transcripts are stored as local JSON files.
2. The application splits transcript text into time-based chunks.
3. An embedding model converts chunks into vectors.
4. Vectors are indexed in Qdrant for similarity search.
5. Relevant chunks are passed to the configured language model.
6. The answer includes citations that link to the relevant video timestamps.

## Technology Stack

- Python 3.11
- `uv` for environment and dependency management
- `yt-dlp` and `faster-whisper` for optional ingestion/transcription
- `sentence-transformers` for embeddings
- Qdrant for vector search
- Groq for language-model inference
- Typer and Rich for the command-line interface
- FastAPI and the YouTube IFrame Player API for the web interface

## Requirements

For the standard workflow using the bundled transcripts:

- Python 3.11
- [`uv`](https://docs.astral.sh/uv/)
- A Groq API key
- A Qdrant instance and its URL/API key, if required by your project configuration

A GPU is **not required** to search or ask questions. GPU support is relevant mainly to optional transcription and can speed up some workloads.

## Installation

### 1. Clone the repository

Replace `<REPO_URL>` with your GitHub repository URL.

```bash
git clone <REPO_URL>
cd <REPOSITORY_DIRECTORY>
```

Run the commands from the directory containing `pyproject.toml`.

### 2. Create the environment and install dependencies

```bash
uv venv --python 3.11
uv sync
```

If you intentionally plan to run GPU-accelerated transcription and your project defines a CUDA extra, install it with:

```bash
uv sync --extra cuda
```

The CUDA extra is optional; it is not needed for the standard transcript-search workflow.

### 3. Configure environment variables

Create a `.env` file in the project root. Use the variable names expected by your current application configuration. A typical setup may include:

```dotenv
GROQ_API_KEY=your_groq_api_key
QDRANT_URL=your_qdrant_url
QDRANT_API_KEY=your_qdrant_api_key
```

Do not commit `.env` or real API keys to GitHub. If the project includes an `.env.example`, copy it to `.env` and fill in your own values.

> Configuration names and required values depend on the current code. Check the application's configuration module and `.env.example` before running the commands below.

## Use the Bundled Transcripts

If the repository includes a `transcripts/` directory, you can build the search index from those files without downloading videos or transcribing audio.

For a smaller embedding model that is more practical on CPU, set:

**PowerShell**
```powershell
$env:YTRAG_EMBED_MODEL="all-MiniLM-L6-v2"
uv run ytrag reindex
```

**macOS / Linux**
```bash
YTRAG_EMBED_MODEL=all-MiniLM-L6-v2 uv run ytrag reindex
```

The first run may download the embedding model. Reindexing creates the vector index using the bundled transcript data. If your current project configuration requires Qdrant Cloud, make sure `QDRANT_URL` and `QDRANT_API_KEY` are set before running the command.

If the repository does **not** include `transcripts/`, this workflow cannot work until you obtain the transcript files or ingest a playlist.

## Basic Usage

### Ask a question

```bash
uv run ytrag ask "What is the difference between memoization and tabulation?"
```

You can also ask in Hinglish, for example:

```bash
uv run ytrag ask "memoization aur tabulation ka difference kya hai?"
```

### Search for a topic

```bash
uv run ytrag search "adjacency list"
```

### View index statistics

```bash
uv run ytrag stats
```

### Run the evaluation

```bash
uv run ytrag eval --verbose
```

### Start the web interface

```bash
uv run ytrag serve
```

Open the local address shown by the server. The default address described by the project is:

```text
http://127.0.0.1:8000
```

Select a cited result to jump to the relevant timestamp in the embedded YouTube player.

## Optional: Ingest Your Own Playlist

Ingestion downloads or processes lecture media and may require additional dependencies, configuration, API access, and substantial time. It is **not required** when the bundled transcripts are available.

Check configuration and connectivity first:

```bash
uv run ytrag preflight --playlist "<PLAYLIST_URL>"
```

Test transcription on one lecture before starting a larger job:

```bash
uv run ytrag langtest "<LECTURE_URL>"
```

Ingest one video as a small test:

```bash
uv run ytrag ingest --playlist "<PLAYLIST_URL>" --limit 1
```

After confirming that the setup works, ingest the full playlist:

```bash
uv run ytrag ingest --playlist "<PLAYLIST_URL>"
```

Use only media and transcripts that you have permission to access and redistribute. Follow YouTube's terms and the rights holders' applicable requirements.

## Useful Commands

| Command | Purpose |
|---|---|
| `uv run ytrag --help` | List available commands |
| `uv run ytrag preflight` | Check configured services and dependencies; consult `--help` for options |
| `uv run ytrag search "<query>"` | Search for relevant lecture segments |
| `uv run ytrag ask "<question>"` | Ask a question using retrieved transcript context |
| `uv run ytrag stats` | Show index statistics |
| `uv run ytrag eval --verbose` | Evaluate retrieval/answering |
| `uv run ytrag reindex` | Build or rebuild the vector index from available transcripts |
| `uv run ytrag serve` | Start the web interface |
| `uv run ytrag export-transcripts` | Export local transcripts, if supported by the current version |
| `uv run ytrag clean-audio` | Clean local audio files, if supported by the current version |

Use `uv run ytrag <command> --help` to check the exact options available in your installed version.

## Troubleshooting

**The application cannot find the transcripts**

- Confirm that the repository contains the expected `transcripts/` directory.
- Run commands from the project root.
- Check the project's configuration for the transcript directory and data root.

**Qdrant connection fails**

- Verify `QDRANT_URL` and `QDRANT_API_KEY`.
- Confirm that the Qdrant instance is running and accessible.
- Ensure the collection's vector dimensions match the selected embedding model. Changing embedding models may require rebuilding the index.

**The model is slow on CPU**

- Use a smaller embedding model such as `all-MiniLM-L6-v2` if the code supports it.
- The first run may take longer while model files are downloaded and the index is built.
- A GPU is not necessary for search, but may help with optional transcription.

**The CLI command is not found**

- Run commands with `uv run ytrag ...` from the directory containing `pyproject.toml`.
- Confirm that `uv sync` completed successfully.
- Check `uv run ytrag --help` for the installed command name.

**API or authentication errors**

- Confirm that the required keys are present in `.env`.
- Check that the variable names match the application's configuration.
- Never paste API keys into public issues, screenshots, or commits.

## Project Data and Secrets

Do not commit any of the following:

- `.env` files or API keys
- `cookies.txt` or other browser/session cookies
- Virtual environments such as `.venv/`
- Python cache files such as `__pycache__/`
- Local Qdrant databases, downloaded audio, model caches, or other generated files unless they are intentionally part of the release

If you choose to distribute bundled transcript files, verify that you are permitted to share them. Transcripts and other derived content may be subject to copyright or platform terms.

## License

Add the project's license here before publishing. If no license file is included, users should not assume that the source code is licensed for reuse.
