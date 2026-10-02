# AI Video Assistant

Turn a YouTube video or a local audio/video file into a searchable meeting record. The app transcribes the media, creates a title and summary, extracts follow-ups, and lets you ask questions about the transcript.

## What It Does

- Accepts a YouTube URL or a local media file path.
- Converts media to mono, 16 kHz WAV and processes it in 10-minute chunks.
- Transcribes English locally with Whisper. For `hinglish`, sends 25-second audio pieces to Sarvam's speech-to-text-translate API and returns an English transcript.
- Uses Google Gemini (`gemini-2.5-flash`) to generate a session title, meeting summary, action items, key decisions, and open questions.
- Builds a local Chroma vector store from transcript chunks and supports transcript-grounded Q&A in the Streamlit app.

## Requirements

- Python 3.10 or newer
- FFmpeg installed and available on `PATH` (used by media conversion and YouTube audio extraction)
- A Google AI API key for Gemini-powered analysis and chat
- A Sarvam API key only when using the `hinglish` transcription option

Whisper runs locally and downloads its selected model the first time it is used. The default is `small`; expect the first run to take longer and require additional disk space and memory.

## Setup

Create and activate a virtual environment from the repository root.

**Windows PowerShell**

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
```

**macOS / Linux**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Install the Python dependencies and create your private environment file:

```bash
python -m pip install --upgrade pip
python -m pip install -r Requirements.txt
```

PowerShell:

```powershell
Copy-Item .env.example .env
```

macOS / Linux:

```bash
cp .env.example .env
```

Edit `.env` and set the keys you need:

```dotenv
GOOGLE_API_KEY=your_google_ai_api_key
SARVAM_API_KEY=your_sarvam_api_key
SARVAM_STT_MODEL=saaras:v2.5
WHISPER_MODEL=small
```

`GOOGLE_API_KEY` is required for title generation, summaries, extraction, and chat. `SARVAM_API_KEY` is only required for Hinglish transcription. The `.env` file is excluded from Git; do not commit or share real API keys.

## Run

Start the web app:

```bash
streamlit run app.py
```

In the sidebar, enter a YouTube URL or a path to a local audio/video file, choose `english` or `hinglish`, and select **Analyse**. Once processing finishes, the page shows the title, summary, transcript, action items, decisions, open questions, and a chat box for questions about the transcript.

The command-line pipeline is also available:

```bash
python main.py
```

It prompts for a YouTube URL or local file path and a language, prints the generated results, and opens a transcript Q&A loop in the terminal.

## Configuration

| Variable | Purpose | Default |
| --- | --- | --- |
| `GOOGLE_API_KEY` | Authenticates Gemini requests for analysis and Q&A | None |
| `SARVAM_API_KEY` | Authenticates Sarvam Hinglish transcription requests | None |
| `SARVAM_STT_MODEL` | Sarvam speech-to-text-translate model | `saaras:v2.5` |
| `WHISPER_MODEL` | Local Whisper model used for English transcription | `small` |

## Project Structure

```text
app.py                    Streamlit interface and interactive pipeline
main.py                   Command-line pipeline
core/
  extractor.py            Action items, decisions, and questions via Gemini
  rag_engine.py           Transcript-grounded retrieval and chat
  summarizer.py            Title and summary generation via Gemini
  transcriber.py          Whisper and Sarvam transcription routing
  vector_store.py          Chroma storage and local embeddings
utils/
  audio_processor.py      YouTube download, media conversion, and chunking
Requirements.txt          Python dependencies
```

## Data and Troubleshooting

- YouTube downloads and converted/chunked audio are written locally under `downloades/`. The Chroma database is stored under `vector_db/`.
- Hinglish mode sends audio to Sarvam. Gemini receives transcript text for analysis and question answering.
- If media conversion fails, verify that `ffmpeg` is installed and available in the same shell that runs Streamlit.
- If Hinglish transcription reports a missing key, add `SARVAM_API_KEY` to `.env`. English transcription does not require Sarvam.
- If Gemini requests fail due to authentication, verify `GOOGLE_API_KEY` and restart the app after changing `.env`.

## License

No license file is currently included in this repository. Contact the repository owner for reuse or distribution terms.