# SoundScene

**Text-to-spatial audio generation from any scene description — concrete or abstract.**

Describe a scene in text and SoundScene generates a fully layered, spatially positioned audio environment. Every sound element is individually placed in 2D space around the listener and adjustable in real time. Abstract inputs like "peace and serenity" are grounded through live Google Search before generation, enabling prompts that would otherwise fail standard sound generation APIs.

---

## Demo

> Type "rainy Tokyo street at night" → scene decomposes into rain, footsteps, traffic, neon hum → each sound generated and spatially positioned → drag elements around the listener in real time with headphones on

---

## Features

- **Abstract input grounding** — Gemini + Google Search translates vague emotional inputs ("melancholic", "tense") into concrete sound descriptors before generation
- **Scene decomposition** — LLM breaks any scene into 3–6 individual sound elements with spatial metadata (x/y position, reverb, layer, mix settings)
- **Parallel sound generation** — ElevenLabs Sound Effects API V2 called concurrently via `asyncio`, reducing generation latency by ~70% vs sequential
- **Automated QC loop** — Gemini scores each generated stem and auto-retries with an improved prompt if quality falls below threshold
- **Spatial DSP mixing** — per-element pan, volume attenuation, reverb, and EQ applied via `pedalboard` and `pydub` before export as binaural WAV
- **Image input** — upload a still frame and Gemini analyzes visible objects, spatial layout, and scene mood to generate a matching soundscape
- **Interactive spatial editor** — top-down 2D map with draggable sound sources; Web Audio API updates pan and volume in real time
- **Timeline editor** — clip trimming, fade in/out, and automation path interpolation per element
- **Per-element controls** — mute, solo, volume, reverb, regenerate with natural language edit instruction
- **Stem export** — download individual WAV files per sound element
- **User accounts** — auth, scene history, saved mixes via SQLite

---

## Tech Stack

| Layer | Tools |
|---|---|
| LLM + grounding | Gemini (google-genai) + Google Search grounding |
| Sound generation | ElevenLabs Sound Effects API V2 |
| Audio QC | Gemini audio review with auto-retry |
| Spatial DSP + mixing | pedalboard, pydub, numpy, scipy |
| Image generation | Gemini native image generation |
| Real-time spatial audio | Web Audio API |
| Backend | FastAPI (Python) |
| Frontend | React + TypeScript + Vite |
| State management | Zustand |
| Audio visualization | WaveSurfer.js |
| Database | SQLite |

---

## Architecture

```
User Input (text / voice / image)
        │
        ▼
┌─────────────────────────┐
│  Abstract Classifier    │  Gemini classifies input as concrete or abstract
└────────────┬────────────┘
             │
     ┌───────┴───────┐
     │               │
     ▼               ▼
 Abstract         Concrete
 Grounding        (skip)
 (Google Search)
     │               │
     └───────┬───────┘
             ▼
┌─────────────────────────┐
│  Scene Decomposer       │  Breaks scene into 3–6 elements with spatial metadata
└────────────┬────────────┘
             ▼
┌─────────────────────────┐
│  Prompt Refiner         │  Gemini optimizes each ElevenLabs prompt
└────────────┬────────────┘
             ▼
┌─────────────────────────┐
│  Parallel Generation    │  ElevenLabs called concurrently via asyncio
└────────────┬────────────┘
             ▼
┌─────────────────────────┐
│  Audio QC + Retry       │  Gemini reviews each stem, retries if score < 0.72
└────────────┬────────────┘
             ▼
┌─────────────────────────┐
│  Spatial DSP Mixing     │  pedalboard + pydub → binaural WAV export
└────────────┬────────────┘
             ▼
┌─────────────────────────┐
│  Spatial Editor (UI)    │  Web Audio API, draggable dots, timeline, mute/solo
└─────────────────────────┘
```

---

## Getting Started

### Prerequisites

- Python 3.11+
- Node.js 18+
- Gemini API key
- ElevenLabs API key

### Backend

```bash
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -r backend/requirements.txt
```

Copy `.env.example` to `.env` and fill in your keys:

```env
GEMINI_API_KEY=your_key_here
ELEVENLABS_API_KEY=your_key_here
```

Start the server:

```bash
uvicorn backend.app.main:app --reload
```

### Frontend

```bash
cd frontend
npm install
npm run dev
```

Open [http://localhost:5173](http://localhost:5173)

---

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/decompose` | Classify + ground + decompose text prompt |
| POST | `/api/decompose-image` | Analyze image and decompose into sound elements |
| POST | `/api/generate` | Generate audio for all scene elements |
| POST | `/api/regenerate-element` | Regenerate a single element with edit instruction |
| GET | `/api/status/{job_id}` | Poll generation job status |
| POST | `/api/auth/register` | Create account |
| POST | `/api/auth/login` | Sign in |
| GET | `/api/user/history` | Fetch saved scenes |

---

## Example Prompts

**Concrete:**
- "Rainy Tokyo street at night"
- "Busy coffee shop during a thunderstorm"
- "Deep forest at dawn"
- "Subway station, incoming train"

**Abstract (triggers Google Search grounding):**
- "Peace and serenity"
- "Tension and unease"
- "Nostalgia"
- "Chaos"

---

## Project Structure

```
soundscene/
├── backend/
│   ├── app/
│   │   ├── audio/          # ElevenLabs generation, DSP mixing, QC, spatial export
│   │   ├── main.py         # FastAPI app, middleware, routing
│   │   ├── database.py     # SQLite schema
│   │   └── user_router.py  # Auth + history endpoints
│   └── pipeline/
│       ├── classifier.py   # Abstract vs concrete detection
│       ├── grounding.py    # Google Search grounding
│       ├── decomposer.py   # Scene-to-elements decomposition
│       ├── prompt_refiner.py # ElevenLabs prompt optimization
│       └── image_analyzer.py # Image-to-soundscape analysis
└── frontend/
    └── src/
        ├── components/
        │   └── SpatialEditor/  # 2D map, timeline, sidebar, Web Audio engine
        ├── api.ts              # Backend API client
        └── types.ts            # Shared TypeScript types
```

---

## Built At

HooKEM Hacks @ University of Texas at Austin. In collaboration with Praneel Bhagavatula, Anshul Mistry, and Ashwin Vijayakumar. 
