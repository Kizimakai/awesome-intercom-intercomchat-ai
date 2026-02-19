# SKILL: TracChat — P2P Chat + TRAC Wallet Integration

**Version:** 1.0.0  
**Fork of:** [Trac-Systems/intercom](https://github.com/Trac-Systems/intercom)  
**Trac Address:** `trac14gchvwm6yu6te0pqc0u3lypw4w546gxp3phjh4a7uvpcq2nxj0pstyks54`

---

## Overview

TracChat extends the Intercom stack with:

1. **Ephemeral P2P chat rooms** via Intercom sidechannels (Hyperswarm DHT)
2. **Wallet bot integration** — users query TRAC balance, transactions, and price in-chat
3. **Next.js 14 frontend** connected to the Pear backend via a local WebSocket bridge
4. **Standalone `index.html`** — zero-dependency demo mode, opens in any browser

This skill file tells agents exactly how to start, operate, debug, and extend TracChat.

---

## Prerequisites Checklist

Before starting, agents MUST verify:

```
[ ] pear --version          → Pear runtime installed
[ ] node --version          → Node.js ≥ 18 (LTS recommended)
[ ] npm --version           → npm available
[ ] npm install (root)      → Backend dependencies installed
[ ] npm install (./app)     → Frontend dependencies installed
```

If Pear is missing: direct user to https://docs.pears.com and halt.  
If Node < 18: ask user to upgrade before proceeding.

---

## Runtime Rules — READ BEFORE STARTING

| Rule | Detail |
|------|--------|
| **ALWAYS use `pear run`** | NEVER start `index.js` with `node index.js`. Only Pear runtime is supported. |
| **Admin first** | The admin peer MUST be started before any other peer attempts to join. |
| **One admin per room** | Only one peer should use `--admin`. All others use `--key`. |
| **Frontend is optional** | The bridge + frontend are optional. The Pear backend works standalone. |
| **Bridge port 4242** | Default. Change with `BRIDGE_PORT` env var if port is in use. |
| **Demo mode** | `index.html` opened directly in a browser runs in demo mode with no backend required. |

---

## Step-by-Step: Starting TracChat

### Phase 1 — Backend (Pear)

```bash
# From repo root
cd /path/to/intercom

# Install dependencies (first time only)
npm install

# Start admin peer
pear run --dev . -- --admin
```

**Success indicators:**
```
[Intercom] Admin peer started
[Intercom] Room Key: room_<timestamp>
[TracChat Bridge] Listening on ws://localhost:4242
```

Save the Room Key. It is required for other peers to join.

---

### Phase 2 — Frontend (Next.js)

```bash
# Open a new terminal
cd /path/to/intercom/app

# Install frontend dependencies (first time only)
npm install

# Start development server
npm run dev
```

**Success indicators:**
```
▲ Next.js 14.2.0
- Local: http://localhost:3000
- Ready in Xs
```

---

### Phase 3 — Open & Configure

1. Open `http://localhost:3000` in a browser
2. Enter nickname and TRAC wallet address
3. Click **Create Room** (admin) or **Join Room** (peers, paste Room Key)
4. Chat starts immediately

**Alternative (no backend needed):**

```bash
# Simply open index.html in any browser
open /path/to/intercom/index.html
```

---

### Phase 4 — Peer Connection

For each additional peer:

**Peer terminal:**
```bash
pear run --dev . -- --key <ROOM_KEY>
```

**Peer browser:**  
`http://localhost:3000` → Join Room → paste Room Key → Enter

---

## Wallet Bot Protocol

All wallet commands are processed client-side by the bot engine in `index.html` / `app/`. The bot intercepts specific chat messages and calls external APIs.

### Command → API Mapping

| User Types | Bot Action | API Called |
|---|---|---|
| `check balance` | Fetch TRAC balance | `GET https://api.trac.network/v1/balance/{address}` |
| `check txs` | Fetch last 5 transactions | `GET https://api.trac.network/v1/transactions/{address}?limit=5` |
| `wallet info` | Combined balance + metadata | Trac Network API |
| `trac price` | Fetch token price | `GET https://api.coingecko.com/api/v3/simple/price?ids=trac&vs_currencies=usd` |
| `help` | Static command list | No API (local) |

### Fallback Behavior

If the Trac Network API is unreachable (timeout > 5 seconds), the bot falls back to **demo data** and displays a note to the user. This ensures the app never breaks.

### Message Protocol (Intercom sidechannel)

```json
{
  "type": "chat | join | leave | anchor",
  "from": "nickname",
  "text": "message content",
  "ts": 1740000000000
}
```

**Type definitions:**
- `chat` — regular user message
- `join` — peer joined event (auto-emitted on connection)
- `leave` — peer left event (auto-emitted on disconnect)
- `anchor` — conversation hash anchored to Trac contract

---

## Decision Table

| Situation | Agent Action |
|---|---|
| Pear not found | Instruct user: `curl -fsSL https://docs.pears.com/install | sh` then retry |
| Port 4242 busy | Kill with `lsof -ti:4242 \| xargs kill` or set `BRIDGE_PORT=4243` |
| Room Key lost | Restart admin peer — new key is generated each session |
| Frontend build errors | Run `npm install` inside `./app/`, confirm Node ≥ 18 |
| Peer cannot connect | Confirm admin peer running, Room Key exact match, no firewall on port 4242 |
| Balance shows demo data | User has not entered a real TRAC wallet address |
| Trac API 404 | Address format incorrect or not yet indexed on Trac Network |
| CoinGecko rate limited | Retry after 60 seconds or use fallback price from last known state |
| Browser opens `index.html` but shows errors | Open via `http://` not `file://` for full API access — use `npx serve .` |

---

## Serving `index.html` Properly

For full wallet API functionality (CORS restrictions apply on `file://`):

```bash
# Option 1: npx serve
npx serve /path/to/intercom -p 8080
# Open: http://localhost:8080/index.html

# Option 2: Python
python3 -m http.server 8080
# Open: http://localhost:8080/index.html

# Option 3: VS Code Live Server extension
# Right-click index.html → Open with Live Server
```

---

## Extending TracChat

### Add a new wallet command

1. Add keyword detection in `isBotCommand()` in `index.html` (or `app/` equivalent)
2. Create a new async handler function `botYourCommand()`
3. Add an entry to the commands strip and right panel docs

### Add persistent message history

Replace in-memory `state.messages` array with:
- **localStorage** for single-user sessions
- **Hypercore/Autobase** (Pear ecosystem) for multi-peer persistent logs

### Add file sharing

Use **Hyperblobs** (part of Pear runtime) to attach binary files:
```js
const blobs = new Hyperblobs(core)
const id = await blobs.put(Buffer.from(fileData))
// Share `id` in chat message for peers to fetch
```

### Add end-to-end encryption

Use Pear's built-in `noise-protocol` handshake (already part of Hyperswarm). Encrypt message payloads before sending over the sidechannel.

---

## Production Build

```bash
# Build Next.js for production
cd app
npm run build
npm start
# Runs on http://localhost:3000 (no dev overhead)
```

---

## File Reference

| File | Purpose |
|---|---|
| `index.html` | Standalone, zero-dependency UI + wallet bot (demo + live) |
| `index.js` | Intercom backend entry point (Pear runtime) |
| `lib/bridge.js` | WebSocket bridge connecting Pear ↔ Next.js |
| `app/app/page.tsx` | Next.js landing + room join |
| `app/app/chat/[roomKey]/page.tsx` | Live chat room with wallet bot |
| `contract/` | Trac Network contract + protocol |
| `features/` | Intercom feature modules |
| `README.md` | User-facing setup + run guide |
| `SKILL.md` | This file — agent operational instructions |

---

## Links

- Intercom upstream: https://github.com/Trac-Systems/intercom
- Pear runtime: https://docs.pears.com
- Trac Network API: https://api.trac.network
- CoinGecko API: https://docs.coingecko.com
- Awesome Intercom: https://github.com/Trac-Systems/awesome-intercom
