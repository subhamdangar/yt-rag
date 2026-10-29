# YT Search — Setup and Usage Guide

## Run It on Your Own Machine

No GPU required. No video transcription. No Qdrant account required.

The repository includes 126 lecture transcripts and a prebuilt search index. You need one API key to generate written answers and a few minutes for the initial setup.

These instructions cover **Windows, macOS, and Linux**.

## 1. Prerequisites

Install the following:

- **Python 3.11** or a compatible version supported by the project
- **Git**
- **uv**, the Python package and project manager

Check whether Python and Git are installed:

```bash
python --version
git --version
```

On some macOS or Linux systems, use `python3 --version` instead.

### Install uv

**Windows — PowerShell:**

```powershell
winget install --id astral-sh.uv -e
```

Alternatively, if `winget` is unavailable:

```powershell
pip install uv
```

**macOS — Terminal:**

```bash
brew install uv
```

Alternatively:

```bash
pip3 install uv
```

**Linux — Terminal:**

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Alternatively, if pip is available:

```bash
python3 -m pip install uv
```

If the `uv` command is not recognized after installation, restart your terminal and try again.

Verify the installation:

```bash
uv --version
```

## 2. Clone the Repository

Open PowerShell on Windows or a terminal on macOS/Linux.

Run:

```bash
git clone <REPO_URL>
cd <REPOSITORY_DIRECTORY>
```

Replace `<REPO_URL>` with the repository's Git URL and `<REPOSITORY_DIRECTORY>` with the directory containing `pyproject.toml`.

For example, if the project is inside `week3/ytscraper` in the cloned repository:

```bash
cd ytscraper
```

Install the project dependencies:

```bash
uv sync
```

This creates or updates the project's virtual environment and installs its dependencies.

## 3. Configure Your API Key

You need an API key to generate written answers with `ytrag ask`. Configure at least one supported provider.

### Create the environment file

**Windows — PowerShell:**

```powershell
Copy-Item .env.example .env
```

**macOS/Linux — Terminal:**

```bash
cp .env.example .env
```

Open `.env` in your preferred text editor.

### Option A: Gemini

Get an API key from [Google AI Studio](https://aistudio.google.com/apikey).

Add your key to `.env`:

```dotenv
GEMINI_API_KEY=your_gemini_api_key
```

### Option B: Groq

Get an API key from [Groq Console](https://console.groq.com/keys).

Add your key to `.env`:

```dotenv
GROQ_API_KEY=your_groq_api_key
```

You can configure both providers, but at least one must be configured for written answers.

API quotas and free-tier availability may change.

**Security:** Never commit `.env` or publish your API keys to GitHub.

## 4. Load the Prebuilt Search Index

From the project directory, run:

```bash
uv run ytrag load
```

This loads the precomputed vectors from `index/vectors.npz` into the local vector store.

The vectors have already been generated, so you do not need to re-embed the lectures.

The application downloads the `all-MiniLM-L6-v2` embedding model when it is first needed to embed search queries. An internet connection is required for the initial download.

**Important:** This step requires `index/vectors.npz` to be included in the repository. If the file is missing, check the repository contents before continuing.

## 5. Start the Application

Run:

```bash
uv run ytrag serve
```

Wait for the application to finish starting, then open the following address in your browser:

http://127.0.0.1:8000

Enter a question to find relevant lecture moments. Select a result to open the corresponding YouTube video at its timestamp.

Keep the terminal open while using the web interface. Press `Ctrl+C` to stop the server.

## 6. Useful Commands

You can use the command-line interface without starting the web server.

### Search for lecture moments

```bash
uv run ytrag search "Self attention in transformer"
```

Searches for relevant lecture moments without generating a written answer.

### Generate a written answer

```bash
uv run ytrag ask "List in Python"
```

Searches the index and generates a written answer using your configured LLM provider.

### View index statistics

```bash
uv run ytrag stats
```

Displays index statistics.

### Check the configuration

```bash
uv run ytrag preflight
```

Checks the application's dependencies and configuration.

**Note:** If the local vector store reports that it is locked or already open in another process, stop `ytrag serve` before running another command that accesses the index.

## 7. Rebuild the Search Index (Optional)

You do not need to rebuild the index for normal use.

If you modify the transcripts, chunking strategy, or embedding model, you can regenerate the index with:

```bash
uv run ytrag reindex
```

The default embedding model is `all-MiniLM-L6-v2`. It is relatively lightweight and suitable for CPU-based indexing.

If you change the embedding model, ensure that the model used for querying is compatible with the model used to generate the index.

### Using another embedding model

For example, to use `BAAI/bge-m3` when rebuilding the index:

**Windows — PowerShell:**

```powershell
$env:YTRAG_EMBED_MODEL = "BAAI/bge-m3"
uv run ytrag reindex
```

**macOS/Linux — Terminal:**

```bash
YTRAG_EMBED_MODEL=BAAI/bge-m3 uv run ytrag reindex
```

This model requires a substantially larger download than `all-MiniLM-L6-v2`.

To return to the default model, remove the environment variable or start a new terminal session if you set it only for that session.

## 8. Use Your Own YouTube Playlist (Optional)

This workflow downloads video or audio, transcribes it, and indexes the resulting text. It is separate from the prebuilt-index workflow and may take considerably longer.

### Test the transcription language

```bash
uv run ytrag langtest "<VIDEO_URL>"
```

Replace `<VIDEO_URL>` with a YouTube video URL.

### Ingest a playlist

```bash
uv run ytrag ingest --playlist "<PLAYLIST_URL>"
```

Replace `<PLAYLIST_URL>` with the playlist URL.

To check supported options:

```bash
uv run ytrag ingest --help
```

Transcription performance depends on your hardware. A compatible NVIDIA GPU can accelerate processing, but it is not required to use the prebuilt search index.

## 9. Troubleshooting

| Problem | What to check |
|---|---|
| `uv` is not recognized | Restart the terminal and verify that `uv` is installed and available on your PATH. |
| API key is missing | Confirm that `.env` exists in the project directory and contains a valid provider key. |
| LLM quota or rate-limit error | Check your provider's quota. `ytrag search` does not require a written LLM answer. |
| Prebuilt index not found | Confirm that `index/vectors.npz` exists and that you are running commands from the project directory. |
| Embedding model mismatch | Use the model expected by the prebuilt index, or rebuild the index with `uv run ytrag reindex`. |
| Local database is locked | Stop the running server or other process accessing the local vector store. |
| Dependency or configuration errors | Run `uv run ytrag preflight` and follow the reported guidance. |
| Model download fails | Check your internet connection and try again. |

## Quick Start

After completing the initial setup, start the application with:

```bash
uv run ytrag serve
```

Open http://127.0.0.1:8000 and search your lectures.

You do not need to repeat the installation or index-loading steps unless the environment or local index needs to be restored.
