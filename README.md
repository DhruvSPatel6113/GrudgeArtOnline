<div align="center">

```
╔═══════════════════════════════════════════════════════════════╗
║                                                               ║
║   ██████╗ ██████╗ ██╗   ██╗██████╗  ██████╗ ███████╗        ║
║  ██╔════╝ ██╔══██╗██║   ██║██╔══██╗██╔════╝ ██╔════╝        ║
║  ██║  ███╗██████╔╝██║   ██║██║  ██║██║  ███╗█████╗          ║
║  ██║   ██║██╔══██╗██║   ██║██║  ██║██║   ██║██╔══╝          ║
║  ╚██████╔╝██║  ██║╚██████╔╝██████╔╝╚██████╔╝███████╗        ║
║   ╚═════╝ ╚═╝  ╚═╝ ╚═════╝ ╚═════╝  ╚═════╝ ╚══════╝        ║
║                                                               ║
║              A R T   O N L I N E                              ║
║                                                               ║
╚═══════════════════════════════════════════════════════════════╝
```

### *The NPCs remember. The world holds a grudge.*

[![Download EXE](https://img.shields.io/badge/⬇%20Download%20Game%20EXE-7B2FBE?style=for-the-badge&logoColor=white)](https://drive.google.com/file/d/1cgn75xP1pR7WPUj-py6zGoUZS3d22zq4/view?usp=sharing)
[![Python](https://img.shields.io/badge/Python-3.x-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![Flask](https://img.shields.io/badge/Flask-Backend-000000?style=for-the-badge&logo=flask&logoColor=white)](https://flask.palletsprojects.com)
[![Unity](https://img.shields.io/badge/Unity-6-FFFFFF?style=for-the-badge&logo=unity&logoColor=black)](https://unity.com)
[![GPT-4o](https://img.shields.io/badge/GPT--4o-Powered-412991?style=for-the-badge&logo=openai&logoColor=white)](https://openai.com)

</div>

---

## ⚔️ What is Grudge Art Online?

> *"You punched the merchant. The System does not forget."*

**Grudge Art Online** is a dark fantasy RPG inspired by **Solo Leveling**, where the world's NPCs have **genuine, persistent memory**. Every insult, every apology, every crime you commit is remembered — not just for your session, but forever.

Most games forget you the moment you close them. This one doesn't.

Built using **Backboard.io's** memory-retaining thread system, the game's AI Dungeon Master tracks your actions across sessions and makes the entire world react. Be rude to an NPC? That NPC will refuse to help you next time. Push the world's anger past 100%? **You get banned from the server.** Permanently.

---

## 🎮 Download & Play

<div align="center">

### **[⬇ Download GrudgeArtOnline.exe](https://drive.google.com/file/d/1cgn75xP1pR7WPUj-py6zGoUZS3d22zq4/view?usp=sharing)**

*Windows x64 · Unity 6 · ~652 KB launcher*

</div>

**Requirements to run the full game:**
- Windows 10 / 11 (64-bit)
- The backend server must be running (see [Running the Backend](#-running-the-backend) below)
- A valid `BACKBOARD_API_KEY` in your `.env` file

---

## ✨ Core Features

| Feature | Description |
|---|---|
| 🧠 **Memory-Retaining NPCs** | NPCs remember your past actions across every session via Backboard.io threads |
| 🌍 **Global Anger System** | One player's crime raises the world's anger level — affecting **all** players |
| ⚡ **Real-Time Streaming** | AI responses stream live, chunk by chunk, for immersive dialogue |
| 🔨 **The Ban Hammer** | Push global anger to 100% and get permanently banned from the world |
| 🎭 **Action Tag Parsing** | The AI outputs structured tags (`[ACTION: CRIME]`, `[ACTION: ATONE]`) parsed server-side to mutate world state |
| 📊 **Live World Status** | Poll `/status` to see real-time anger level, last crime committed, and banned player count |

---

## 🏗️ Architecture

```
┌─────────────────────┐         HTTP          ┌──────────────────────────┐
│                     │ ◄──────────────────── │                          │
│   Unity 6 Client    │                       │   Flask Backend (app.py) │
│   (Game EXE)        │ ──────────────────── ► │   Python 3 · Port 5000   │
│                     │   POST /chat           │                          │
└─────────────────────┘   GET  /status         └───────────┬──────────────┘
                                                           │
                                                           │ BackboardClient
                                                           ▼
                                               ┌──────────────────────────┐
                                               │     Backboard.io         │
                                               │  Persistent AI Threads   │
                                               │  ┌────────────────────┐  │
                                               │  │  Dungeon Master    │  │
                                               │  │  (GPT-4o)          │  │
                                               │  └────────────────────┘  │
                                               │  Thread A · Thread B ... │
                                               └──────────────────────────┘
```

### How it works — the request lifecycle

1. Client sends `POST /chat` with `{message, user_id}`
2. Server checks the **ban list** — banned users are hard-rejected
3. **World State** (`anger_level`, `last_crime`) is injected as a hidden `[SYSTEM DATA]` header into the prompt
4. Message is routed to the user's **personal Backboard thread** (created on first contact, reused forever)
5. The AI streams its response back in real-time
6. Server **parses action tags** from the response to update world state
7. If anger hits 100, the offending user is added to the permanent ban list
8. `{reply, anger}` is returned to the client

---

## 🧠 The Dungeon Master AI

The game's NPC is a single Backboard **Assistant** — the *System* — a cold, omniscient overseer of the RPG world. Its behaviour is governed by hard rules embedded in its system prompt:

- 📖 Reads injected `[SYSTEM DATA]` on every message — always contextually aware
- 😡 When global anger > 50, becomes hostile to **every** player, not just the offender
- 🚫 At anger = 100, **refuses to help anyone**
- 📢 Must explicitly name and shame the last player who committed a crime
- 🏷️ Outputs `[ACTION: CRIME]` or `[ACTION: ATONE]` tags, parsed silently by the server

### Memory: Global vs. Per-User

| Memory Type | Scope | Mechanism | Persistence |
|---|---|---|---|
| **Thread Memory** | Per-user | Backboard thread history | Permanent (thread IDs stored) |
| **World State** | Global | In-memory Python dict | Session (resets on restart) |
| **Ban List** | Global | List inside `WORLD_STATE` | Session (resets on restart) |

---

## 🗂️ Project Structure

```
GrudgeArtOnline/
│
├── app.py              # ⭐ Full production backend
│                       #    Global anger system, ban logic, action tag parsing
│
├── app_basic.py        # Simplified baseline — per-user memory, no anger system
│
├── first-message.py    # Minimal Backboard SDK proof-of-concept / reference
│
├── threads_db.json     # Persistent store: user_id → Backboard thread_id
│
├── templates/
│   └── index.html      # Flask frontend UI (served to game client)
│
└── .gitignore
```

---

## 🚀 Running the Backend

### 1. Clone & Install

```bash
git clone https://github.com/DhruvSPatel6113/GrudgeArtOnline.git
cd GrudgeArtOnline
pip install flask python-dotenv backboard-sdk
```

### 2. Set up your API key

Create a `.env` file in the project root:

```env
BACKBOARD_API_KEY=your_backboard_api_key_here
```

### 3. Run the server

```bash
python app.py
```

The Dungeon Master will initialize, and the server starts on `http://0.0.0.0:5000`.

### 4. Launch the game

Download and run **[GrudgeArtOnline.exe](https://drive.google.com/file/d/1cgn75xP1pR7WPUj-py6zGoUZS3d22zq4/view?usp=sharing)** — the Unity frontend will connect to your local server automatically.

---

## 🔌 API Reference

| Method | Route | Description | Returns |
|---|---|---|---|
| `GET` | `/` | Serves the main game UI | HTML |
| `POST` | `/chat` | Send `{message, user_id}` — runs AI logic | `{reply, anger}` |
| `GET` | `/status` | Live world state poll | `{anger, last_crime, banned_count}` |

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| 🎮 Game Client | Unity 6 (Windows x64) |
| 🐍 Backend | Python 3 + Flask |
| 🧠 AI Memory | Backboard.io (Persistent Assistants + Threads) |
| 🤖 LLM | OpenAI GPT-4o |
| ⚡ Async | Python `asyncio` |
| 🔐 Config | `python-dotenv` |

---

## 💡 Inspiration

Grudge Art Online is built around a simple question:

> *What if NPCs actually remembered what you did to them?*

Inspired by the **Solo Leveling** manhwa and its iconic "System" — an omniscient, cold game overseer — and powered by Backboard.io's memory-retaining thread model, this project explores what truly persistent NPC cognition feels like in practice.

---

## 📄 License

This project is for personal/portfolio use. All rights reserved © Dhruv S Patel, 2026.

---

<div align="center">

*The System is watching. Choose your words carefully.*

**[⬇ Download & Play](https://drive.google.com/file/d/1cgn75xP1pR7WPUj-py6zGoUZS3d22zq4/view?usp=sharing)**

</div>
