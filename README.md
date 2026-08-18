# live-voice-agent

A command-line voice assistant built on [Azure AI VoiceLive](https://learn.microsoft.com/azure/ai-services/speech-service/voice-live). It streams microphone audio to a configured Azure AI Foundry agent in real time and plays back the agent's spoken response, with live transcripts printed to the console.

## Features

- Real-time, full-duplex audio streaming (24kHz, 16-bit PCM mono) via [PyAudio](https://pypi.org/project/PyAudio/)
- Server-side voice activity detection (Azure semantic VAD) with barge-in support (speaking over the agent clears queued playback)
- Echo cancellation and deep noise suppression on the input audio
- Live console transcripts for both user speech and agent responses
- Auth via `AzureCliCredential` (no API keys stored in code)

## Prerequisites

- Python 3.9+
- A working microphone and speakers
- An Azure AI Foundry project with a VoiceLive-enabled agent deployed
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), logged in via `az login` (used for authentication)
- PortAudio (required by PyAudio):
  - macOS: `brew install portaudio`

## Setup

1. Clone the repository and create a virtual environment:

   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```

2. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```

3. Log in to Azure:

   ```bash
   az login
   ```

4. Create a `.env` file in the project root with your Azure AI Foundry VoiceLive settings:

   ```env
   AZURE_VOICELIVE_ENDPOINT=<your-voicelive-endpoint>
   AZURE_VOICELIVE_PROJECT_NAME=<your-project-name>
   AZURE_VOICELIVE_AGENT_ID=<your-agent-name>
   ```

## Usage

```bash
python chat-client.py
```

Once connected, the agent is ready to listen — just start speaking. Press `Ctrl+C` to exit.

## How it works

`chat-client.py` contains two main components:

- **`VoiceAssistant`** — connects to the VoiceLive endpoint, configures the session (modalities, audio formats, VAD, noise reduction, echo cancellation), and processes the incoming event stream (session updates, transcripts, speech start/stop, audio deltas, errors).
- **`AudioProcessor`** — wraps PyAudio to capture microphone input (base64-encoded and streamed to the service) and to play back audio chunks received from the agent through a queue, with support for clearing the queue on user interruption.

## Notes

- `.env` and `.venv` are git-ignored — never commit credentials or endpoints.
- Uses the `azure-ai-voicelive` preview SDK (`1.2.0b4`) and API version `2026-01-01-preview`, so behavior may change as the service evolves.
