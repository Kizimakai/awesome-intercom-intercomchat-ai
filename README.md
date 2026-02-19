# TracChat 💬 — P2P Chat + TRAC Wallet Integration
<img width="922" height="855" alt="image" src="https://github.com/user-attachments/assets/10b8b33e-d4a1-4b18-be6f-1db327de1761" />
<img width="909" height="844" alt="image" src="https://github.com/user-attachments/assets/4b672968-4d56-49bc-9330-0280d7b85a2c" />

> A decentralized peer-to-peer chat application built on [Intercom](https://github.com/Trac-Systems/intercom) (Trac Network) with **live TRAC wallet balance & transaction checking** via an integrated bot.

[![Built on Intercom](https://img.shields.io/badge/Built%20on-Intercom-00ff96?style=flat-square)](https://github.com/Trac-Systems/intercom)
[![Trac Network](https://img.shields.io/badge/Network-Trac-3de8ff?style=flat-square)](https://www.trac.network)
[![Tech](https://img.shields.io/badge/Stack-Next.js%2014-black?style=flat-square)](https://nextjs.org)

---

## 💰 Trac Payout Address

```
trac14gchvwm6yu6te0pqc0u3lypw4w546gxp3phjh4a7uvpcq2nxj0pstyks54
```

---

## ✨ What is TracChat?

TracChat is a fork of `Trac-Systems/intercom` that adds a real-time **P2P chat interface** with a built-in **wallet bot**. Users can:

- Open or join ephemeral P2P rooms over Intercom sidechannels
- Chat in real-time with no central server
- Query their **TRAC token balance** directly in chat (`check balance`)
- View **recent transactions** (`check txs`)
- Get **live TRAC/USD price** (`trac price`)
- Anchor conversation hashes to the Trac contract layer

The wallet bot responds instantly inside the chat — no external apps, no copy-pasting addresses, no switching tabs.

---

## 🖥️ Screenshots / Proof of Work

```
┌─────────────────────────────────────────────────────┐
│  TracChat  [room_1740000000000]  ● demo mode  2 peers │
├──────────────┬──────────────────────────────┬────────┤
│ TRAC Wallet  │                              │ Bot    │
│ 1,074 TRAC   │  > check balance             │ Cmds   │
│ ≈ $15.26     │                              │        │
│              │  ⬡ TracBot                   │        │
│ Recent TX    │  ┌──────────────────────┐    │        │
│ ↓ +250 TRAC  │  │ TRAC Balance         │    │        │
│ ↑ -50 TRAC   │  │ 1,074.76 TRAC        │    │        │
│ ↓ +1000 TRAC │  │ ≈ $15.26 USD         │    │        │
│              │  └──────────────────────┘    │        │
└──────────────┴──────────────────────────────┴────────┘
```

See `screenshots/` folder for full UI screenshots.

---

## 🛠️ Tech Stack

| Layer           | Technology                                 |
|-----------------|--------------------------------------------|
| P2P Transport   | Intercom (Pear runtime / Hyperswarm)       |
| Frontend        | Next.js 14, React 18, TypeScript           |
| Styling         | Tailwind CSS                               |
| Wallet Data     | Trac Network API + CoinGecko               |
| Bridge          | Local WebSocket (`ws://localhost:4242`)    |
| Standalone Demo | `index.html` (zero dependencies)           |

---

## 🚀 Quick Start — Run TracChat

### Option A: Open `index.html` directly (Demo Mode — No Setup Required)

This is the fastest way to try TracChat. No Pear, no Node, no install.

```bash
# Clone the repo
git clone https://github.com/Kizimakai/awesome-intercom-intercomchat-ai.git
cd awesome-intercom-intercomchat-ai

# Open in browser
open index.html
# or: double-click index.html in your file explorer
```

> In demo mode, the wallet bot works with simulated data. Connect a real backend (Option B) for live P2P and on-chain data.

---

### Option B: Full Stack — Intercom Backend + Next.js UI

#### Prerequisites

Before you start, make sure you have:

| Tool        | Install                                      | Check              |
|-------------|----------------------------------------------|--------------------|
| Pear Runtime | https://docs.pears.com                      | `pear --version`   |
| Node.js ≥ 18 | https://nodejs.org                          | `node --version`   |
| npm          | Included with Node.js                       | `npm --version`    |

---

#### Step 1 — Clone & Install Backend

```bash
git clone https://github.com/Kizimakai/awesome-intercom-intercomchat-ai.git
cd awesome-intercom-intercomchat-ai
npm install
```

---

#### Step 2 — Start the Intercom Backend (Admin Peer)

```bash
pear run --dev . -- --admin
```

**Expected output:**
```
[Intercom] Admin peer started
[Intercom] Room Key: room_1740000000000
[TracChat Bridge] Listening on ws://localhost:4242
```

> 📋 **Copy the Room Key** — you need it to invite other peers.

---

#### Step 3 — Install & Start the Next.js Frontend

Open a **new terminal** tab:

```bash
cd intercom/app
npm install
npm run dev
```

**Expected output:**
```
▲ Next.js 14.2.0
- Local: http://localhost:3000
```

---

#### Step 4 — Open TracChat

Navigate to **http://localhost:3000** in your browser.

On the setup screen:
1. Enter your **nickname**
2. Enter your **TRAC wallet address** (for live balance checking)
3. Click **Create Room** (you're the admin)

---

#### Step 5 — Invite Peers

Share your **Room Key** with peers. They run:

```bash
# Peer's terminal
pear run --dev . -- --key room_1740000000000
```

Then open `http://localhost:3000`, choose **Join Room**, and paste the Room Key.

---

## 💬 Wallet Bot Commands

Once inside a room, type these commands (or click the quick-chips):

| Command         | Response                                         |
|-----------------|--------------------------------------------------|
| `check balance` | Live TRAC token balance for your wallet          |
| `check txs`     | Last 5 transactions with hash, direction, amount |
| `wallet info`   | Full wallet summary: balance, price, network     |
| `trac price`    | Current TRAC/USD price from CoinGecko            |
| `help`          | List all available commands                      |

**Example flow:**
```
You:      check balance
TracBot:  ┌─────────────────────────────┐
          │ TRAC Balance  14:32         │
          │ 1,074.76 TRAC               │
          │ ≈ $15.26 USD · @$0.000014  │
          │ trac14gchv...tyks54        │
          └─────────────────────────────┘
```

---

## 📁 Project Structure

```
intercom/
├── index.html             # Standalone demo (open in browser, no install)
├── index.js               # Intercom backend entry (Pear runtime)
├── lib/
│   └── bridge.js          # WebSocket bridge: Pear ↔ Next.js frontend
├── contract/              # Trac Network contract & protocol
├── features/              # Intercom feature modules
├── app/                   # Next.js 14 chat UI
│   ├── app/
│   │   ├── page.tsx       # Landing / room join page
│   │   ├── chat/
│   │   │   └── [roomKey]/
│   │   │       └── page.tsx   # Live chat room
│   │   ├── layout.tsx
│   │   └── globals.css
│   ├── package.json
│   ├── tailwind.config.ts
│   └── tsconfig.json
├── README.md              # This file
└── SKILL.md               # Agent-oriented operational instructions
```

---

## 🔧 Troubleshooting

| Problem | Solution |
|---|---|
| `pear: command not found` | Install Pear from https://docs.pears.com |
| Port 4242 already in use | `BRIDGE_PORT=4243 pear run --dev . -- --admin` |
| `npm run dev` fails | Run `npm install` inside `./app/` first |
| Peer cannot connect | Confirm admin peer is still running and Room Key is correct |
| Balance shows demo data | Enter your real TRAC wallet address on setup screen |
| `index.html` shows no balance | Normal — wallet queries need a valid TRAC address entered |

---

## 🔌 Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Trac Network                       │
│  ┌──────────────────────────────────────────────┐   │
│  │            Hyperswarm (P2P DHT)              │   │
│  └──────────────────────────────────────────────┘   │
│              ▲                    ▲                  │
│    ┌─────────┘                    └─────────┐        │
│    │  Peer A (admin)          Peer B         │        │
│    │  pear run . --admin      pear run . --key│       │
│    │  index.js                index.js        │       │
│    │      │                       │           │       │
│    │  ws:4242                 ws:4242         │       │
│    │      │                       │           │       │
│    │  Next.js :3000           Next.js :3000   │       │
│    │  index.html              index.html      │       │
│    └─────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────┘
                         │
               ┌─────────▼──────────┐
               │  Trac Network API  │
               │  + CoinGecko       │
               │  (balance / price) │
               └────────────────────┘
```

---

## 🤖 For Agents

See [`SKILL.md`](./SKILL.md) for full agent-oriented operational instructions including step-by-step startup, decision tables, extension points, and protocol documentation.

---

## 🌐 Links

- Intercom upstream: https://github.com/Trac-Systems/intercom
- Awesome Intercom list: https://github.com/Trac-Systems/awesome-intercom
- Trac Network: https://www.trac.network
- Pear Runtime docs: https://docs.pears.com

---

## 📄 License

MIT — fork freely, build your own app on Intercom.
