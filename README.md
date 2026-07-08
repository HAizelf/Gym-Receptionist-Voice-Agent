# Gym Receptionist — Hinglish Voice AI Agent

A real-time voice receptionist for **Hype The Gym, Sector 93 (Delhi)** that answers calls in natural **Hinglish** — the code-mixed Hindi-English actually spoken in urban India. Callers ask about timings, membership plans, trainers, or equipment over the phone or browser, and "Sarthak" answers conversationally, grounded in real gym data via function calling.

> *"Hype The Gym में आपका स्वागत है! आज मैं आपकी कैसे help कर सकता हूँ?"*

Built with [LiveKit Agents](https://github.com/livekit/agents); works over both **telephony (SIP)** and **WebRTC** (browser/mobile).

## Demo

<video controls src="https://github.com/HAizelf/Gym-Receptionist/raw/main/.github/assets/demo.mp4" title="Presentation"></video>

*(2-minute conversation — code-mixed Hinglish, tool-grounded answers, natural interruptions.)*


## Why this is harder than an English voice bot

- **Code-mixed speech in, code-mixed speech out.** STT runs Sarvam **Saaras v3** in `codemix` mode to transcribe mid-sentence Hindi↔English switching; the system prompt enforces Devanagari-for-Hindi / Latin-for-English output; TTS is Sarvam **Bulbul v3**, which renders that mixed script naturally.
- **Turn-taking for multilingual speech.** LiveKit's multilingual turn-detector model + Silero VAD decide when the caller is done — with **preemptive generation** enabled, the LLM starts producing a reply before end-of-turn is fully confirmed, cutting response latency.
- **Grounded answers, no hallucinated prices.** GPT-4.1-mini (via LiveKit Inference) answers only from four function tools (`get_gym_timings`, `get_membership_plans`, `get_trainers`, `get_equipment_list`) — the prompt forbids guessing, and guardrails keep it on-topic (no medical advice, no fabricated classes).
- **Channel-aware noise cancellation.** Krisp `BVCTelephony` for SIP callers, `BVC` for WebRTC participants — selected per-participant at session start.
- **Voice-native output rules.** Plain text only, numbers spelled out, 1–3 sentence replies — so the TTS never reads markdown or digit soup aloud.

## Pipeline

```
Caller (phone / browser)
  └─ LiveKit room (WebRTC / SIP)
       ├─ Sarvam Saaras v3 STT (codemix)          — Hinglish transcription
       ├─ Silero VAD + multilingual turn detector — end-of-turn detection
       ├─ GPT-4.1-mini + function tools           — grounded, preemptive generation
       └─ Sarvam Bulbul v3 TTS (speaker: shubh)   — code-mixed speech out
```

## Run it

Prerequisites: Python 3.11+, [`uv`](https://docs.astral.sh/uv/), a [LiveKit Cloud](https://cloud.livekit.io/) project, and a [Sarvam](https://www.sarvam.ai/) API key.

```bash
uv sync
# .env.local needs: LIVEKIT_URL, LIVEKIT_API_KEY, LIVEKIT_API_SECRET, SARVAM_API_KEY
uv run python src/agent.py download-files   # VAD + turn-detector models

# talk to it from your terminal
uv run python src/agent.py console

# or run as a worker and connect any LiveKit frontend / SIP trunk
uv run python src/agent.py dev
```

Deploy to LiveKit Cloud with the included production `Dockerfile`:

```bash
lk agent create
```

## Project structure

```
src/
├── agent.py     # session pipeline, system prompt, function tools
└── gym_data.py  # timings, plans, trainers, equipment (the tool data source)
tests/           # evaluation suite
```

---

Scaffolded from [livekit-examples/agent-starter-python](https://github.com/livekit-examples/agent-starter-python) (MIT).
