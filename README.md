<img width="60" src="./docs/src/images/EEUM_logo_EN.jpg" alt="School of Engineering, University of Minho" align="left" />

# Real-Time Speech Synthesis (PI24)

Low-latency **text-to-speech** system for Portuguese and English, built as the Master's capstone project ("Projeto de Informática") at the University of Minho, in partnership with **Agentifai**. It turns natural text into speech and returns it either as a file or as a real-time audio stream.

📖 **Full documentation:** [ddu72.github.io/PI24](https://ddu72.github.io/PI24/)

<br clear="left" />

## How it works

![Basic flow](./docs/src/images/BasicFlow.png)

1. **Input**: the user sends natural text.
2. **Normalisation**: abbreviations, numbers, dates and punctuation are expanded into words that can be spoken, using language-specific rules.
3. **Phonemisation**: the normalised text is converted into a phonetic representation.
4. **Synthesis**: a **Coqui XTTS v2** model, fine-tuned for Portuguese, generates the audio. The voice is cloned from a short reference sample, so different voices can be used.
5. **Output**: audio is streamed back in real time or saved to a file.

## Architecture

The system is a set of Python microservices that talk to each other over **gRPC**, each running in its own Docker container.

![Architecture](./docs/src/images/architecture.png)

| Service | Role |
|---|---|
| `app_client` | Terminal client that sends text and receives the audio |
| `app` + `app_api` | React web interface and the Flask REST API behind it |
| `app_proxy` | Receives requests and balances them across the synthesis servers |
| `app_server` | Runs the TTS model (three replicas in Docker Compose) |
| `app_normalizer` | Text normalisation service |

Black arrows in the diagram show text; blue arrows show audio.

## Tech stack

Python 3.12 · gRPC / Protocol Buffers · Coqui TTS (XTTS v2) · Flask · React · Docker / Docker Compose

## Getting started

### Option 1: standalone (single process)

```bash
pip install -r enviroments/server_requirements.txt
python intlex.py "Olá, mundo!" config/intlex_config.json --output out.wav --lang pt
```

| Argument | Description |
|---|---|
| `TEXT` | Text to synthesise |
| `CONFIG` | Configuration file (model, voices, normalisation rules) |
| `--output` | Output file (optional) |
| `--lang` | `pt` or `en` (optional) |

### Option 2: microservices with Docker

```bash
docker compose build
docker compose up proxy -d          # starts the proxy, synthesis servers and normaliser
docker compose run client           # opens the terminal client
```

Stop everything with `docker compose down`.

> The fine-tuned model weights are not stored in this repository. Set their location in `config/intlex_config.json` (`checkpoint_path`, `vocab_path`, `xtts_config`). The training scripts and notebook are in `model/`.

## Repository structure

```
app/              React web interface
app_api/          Flask REST API (/api/tts)
app_client/       Terminal client
app_proxy/        Load-balancing proxy
app_server/       Synthesis server
app_normalizer/   Text normalisation (numbers, dates, abbreviations, punctuation)
grpcs/            .proto definitions and generated stubs
config/           Model, proxy and normalisation configuration
model/            XTTS fine-tuning scripts and notebook
docker/           One Dockerfile per service
docs/             Documentation site
```

## Roadmap

- Automated tests
- SSML input support (parser started in `ssml_parser/`)
- Voice selection in the client
- Log files for each service
- Fix the web interface connection when running in Docker

## Team

- [Beatriz Monteiro](https://github.com/5ditto)
- [Daniel Du](https://github.com/ddu72)
- [Daniel Furtado](https://github.com/danielfurtado11)
- [Miguel Gomes](https://github.com/MayorX500)
- [Moisés Antunes](https://github.com/MoisesA14)
- [Telmo Maciel](https://github.com/telmomaciel9)

Project proposed by Agentifai · University of Minho, 2024/25.
