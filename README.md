# CarMind
On-device AI assistant for your car — allowing wireless Android Auto, bejng a conversational AI,giving you a custom UI, and analysing your vehicle diagnostics. No cloud. No subscription. Built on a pluggable layered architecture. Only uses your mobile and still runs on local AI



# 🚗 CarMind

> An open source, modular, on-device AI assistant for your car — built on a pluggable layered architecture inspired by AUTOSAR principles.

**No subscriptions. No cloud. No data leaving your vehicle.**

---

## Vision

CarMind replaces the traditional head unit experience with an intelligent, privacy-first, fully offline AI assistant that knows your car, talks to you naturally, and runs entirely on your own hardware.

Built with a pluggable modular architecture — each component can be swapped independently without touching the rest of the stack.

---

## Architecture

```
┌──────────────────────────────────────────────────┐
│                 APPLICATION LAYER                 │
│              CarBuddy Android App                 │
│                                                   │
│   ┌─────────────────┐   ┌─────────────────────┐  │
│   │  Head Unit UI   │   │  Settings &         │  │
│   │  (Car Display)  │   │  Session History    │  │
│   └─────────────────┘   └─────────────────────┘  │
├──────────────────────────────────────────────────┤
│                 AI SERVICE LAYER                  │
│            CarBuddy Core Service                  │
│           (Runs in background always)             │
│                                                   │
│   ┌──────────────┐   ┌──────────────────────┐    │
│   │  Wake Word   │   │  Conversation Engine │    │
│   │  Detection   │   │  - Interruption      │    │
│   │  "Hey        │   │  - Barge-in          │    │
│   │   CarMind"  │   │  - Streaming         │    │
│   └──────────────┘   │  - Backchanneling    │    │
│                       └──────────────────────┘    │
│   ┌──────────────┐   ┌──────────┐  ┌──────────┐  │
│   │     STT      │   │   LLM    │  │   TTS    │  │
│   │   Whisper    │──▶│  Phi-3   │─▶│  Kokoro  │  │
│   │  Streaming   │   │   Mini   │  │ (Natural │  │
│   │              │   │ (On-     │  │  Voice)  │  │
│   │              │   │ Device)  │  │          │  │
│   └──────────────┘   └──────────┘  └──────────┘  │
├──────────────────────────────────────────────────┤
│               CONNECTIVITY LAYER                  │
│                                                   │
│   ┌──────────────────┐   ┌─────────────────────┐ │
│   │  Android Auto    │   │   Local WiFi API    │ │
│   │  Protocol        │   │   Pi ↔ Phone        │ │
│   │  (OpenAuto base) │   │   (REST / gRPC)     │ │
│   └──────────────────┘   └─────────────────────┘ │
│   ┌──────────────────┐                            │
│   │  WiFi Direct     │                            │
│   │  P2P Manager     │                            │
│   └──────────────────┘                            │
├──────────────────────────────────────────────────┤
│                 HARDWARE LAYER                    │
│                                                   │
│   ┌──────────────────┐   ┌─────────────────────┐ │
│   │  Raspberry Pi 4  │   │  Samsung S25+       │ │
│   │  (Head Unit)     │   │  (AI Compute)       │ │
│   │  - OpenAuto      │   │  - Snapdragon 8     │ │
│   │  - Display HDMI  │   │    Elite            │ │
│   │  - Audio I/O     │   │  - 12GB RAM         │ │
│   └──────────────────┘   └─────────────────────┘ │
├──────────────────────────────────────────────────┤
│          VEHICLE DATA LAYER  (Phase 5)            │
│                                                   │
│   ┌──────────────────┐   ┌─────────────────────┐ │
│   │  OBD Interface   │   │  Diagnostics Engine │ │
│   │  ELM327 BT       │   │  DTC Codes          │ │
│   │  Speed/RPM/Temp  │   │  Health Reports     │ │
│   └──────────────────┘   └─────────────────────┘ │
└──────────────────────────────────────────────────┘
```

---

## Module Interfaces

Each module has a defined input/output contract. Internal implementation can be swapped without affecting other modules.

```
┌─────────────────────────────────────────────────────┐
│                  MODULE CONTRACTS                    │
├──────────────────┬──────────────────────────────────┤
│ STT Module       │ OUT: { text: string,             │
│                  │        confidence: float,         │
│                  │        is_final: bool }           │
├──────────────────┼──────────────────────────────────┤
│ LLM Module       │ IN:  { text, context,            │
│                  │        vehicle_data? }            │
│                  │ OUT: { response: string,          │
│                  │        action: enum,              │
│                  │        stream: bool }             │
├──────────────────┼──────────────────────────────────┤
│ TTS Module       │ IN:  { text: string,             │
│                  │        stream: bool }             │
│                  │ OUT: audio stream                 │
├──────────────────┼──────────────────────────────────┤
│ Wake Word Module │ OUT: { triggered: bool,          │
│                  │        confidence: float }        │
├──────────────────┼──────────────────────────────────┤
│ AA Module        │ OUT: { display_active: bool,     │
│                  │        audio_channel: active }    │
├──────────────────┼──────────────────────────────────┤
│ OBD Module       │ OUT: { speed: float,             │
│  (Phase 5)       │        rpm: float,               │
│                  │        temp: float,               │
│                  │        fuel: float,               │
│                  │        dtc_codes: string[] }      │
└──────────────────┴──────────────────────────────────┘
```

---

## AUTOSAR Parallel

CarMind's architecture is deliberately inspired by AUTOSAR principles — the same philosophy that drives modern automotive ECU software design.

| AUTOSAR Concept | CarMind Equivalent |
|---|---|
| Basic Software (BSW) | Hardware + Connectivity Layers |
| Runtime Environment (RTE) | AI Service Layer interfaces |
| Software Components (SWCs) | AA module, LLM module, OBD module |
| ARXML configuration | JSON/YAML config per module |
| Standardized interfaces | Defined API contracts between layers |
| Layered architecture | Pluggable layer stack |

---

## Build Phases

- [x] **Phase 0** — Architecture defined, system prompt working on Gemma 4 (S25+)
- [ ] **Phase 1A** — AI pipeline: Whisper STT + Phi-3 Mini LLM + Kokoro TTS on S25+
- [ ] **Phase 1B** — CarBuddy Core Service: background Android service, streaming, interruption handling, barge-in detection
- [ ] **Phase 2** — Connectivity Layer: OpenAuto on Pi, Pi ↔ Phone local API
- [ ] **Phase 3** — Application Layer: Custom UI on car screen via Pi HDMI
- [ ] **Phase 4** — Hardware optimization: boot time, power, latency tuning
- [ ] **Phase 5** — Vehicle Data Layer: ELM327 OBD integration, diagnostics

---

## Conversation Experience Target

Inspired by Gemini Live's seamless full-duplex conversation feel — achieved fully offline:

```
User speaks
    ↓
Wake word detected (ESP32 / phone mic)
    ↓
Whisper streaming transcription (real-time)
    ↓
LLM begins generating (streams tokens)
    ↓
Kokoro TTS speaks while generating
    ↓
Barge-in detection — user can interrupt anytime
    ↓
Backchanneling — "mm", "got it" while listening
```

**Target latency: < 2 seconds from speech end to first word of response**

---

## Hardware Bill of Materials

| Component | Purpose | Cost (INR) |
|---|---|---|
| Raspberry Pi 4 4GB | Head unit, OpenAuto, display | ~4,500 |
| Samsung S25+ (owned) | AI inference engine | 0 |
| MicroSD 32GB | Pi OS | ~400 |
| HDMI cable | Pi to car display | ~300 |
| USB-C power (car) | Pi power via car USB | ~200 |
| ELM327 BT dongle | OBD vehicle data (Phase 5) | ~600 |
| **Total** | | **~6,000** |

---

## Tech Stack

| Layer | Technology |
|---|---|
| LLM Inference | MLC LLM + Phi-3 Mini 4K (quantized) |
| Speech to Text | Whisper (streaming, on-device) |
| Text to Speech | Kokoro TTS (offline, natural voice) |
| Wake Word | OpenWakeWord |
| Android Auto | OpenAuto (C++, Linux) |
| Head Unit OS | Raspberry Pi OS |
| Android App | Kotlin + Android Services |
| Pi ↔ Phone API | REST / gRPC over WiFi Direct |
| OBD (Phase 5) | Python-OBD + ELM327 |

---

## Why CarBuddy

- **Privacy first** — zero cloud dependency, all inference on-device
- **Modular** — swap any component without touching the stack
- **Automotive native** — built by someone who knows BSW, not just ML
- **SDV aligned** — targets the Software Defined Vehicle future
- **Open source** — every layer visible, forkable, improvable

---

## Status

> 🚧 Active development — Phase 1A in progress  
> Started: May 2026  
> Builder: [@avinash-gopalabhatla](https://github.com/avinash-gopalabhatla)

---

*CarMind is an independent open source project. Not affiliated with Google, Android Auto, or any automotive OEM.*
