

# 📌 MYN Scout — Lightweight Explorer + Telegram Assistant

**A fast, minimal, user-focused MYN explorer and Telegram bot providing real-time alerts, address lookups, and transaction search.**

MYN Scout helps everyday users, traders, and developers instantly inspect MYN transactions, check addresses, view network activity, and receive notifications directly on Telegram.

---

## 🚀 Key Features

### 🕸️ Lightweight Web Explorer

* Global search bar (address, tx hash, block)
* Recent transaction feed (auto-refresh)
* Address view: balance, recent transactions, activity summary
* Transaction view: status, input/output, timestamp, value
* Micro sparkline charts for lightweight analytics
* Mobile-first responsive UI

### 🤖 Telegram Bot

* `/price` — get MYN price
* `/addr <address>` — quick lookup + summary
* `/tx <hash>` — fetch transaction details
* `/watch <address>` — receive alerts for incoming/outgoing transactions
* Realtime notifications powered by a background worker

---

## 🧩 Architecture Overview

```
myn-scout/
├── frontend/          # React + Vite + Tailwind Explorer UI
├── backend/           # Fastify/FastAPI API (search, addr, tx)
├── bot/               # Telegram Bot
├── worker/            # Transaction watcher & notifier
├── data/              # Mock or cached data
└── README.md
```

**Frontend**: React + Vite + Tailwind (host on Vercel or GitHub Pages)<br>
**Backend**: Node.js Fastify (JSON API)<br>
**Bot**: node-telegram-bot-api (polling mode)<br>
**Worker**: Node script that monitors watched addresses<br>
**Database**: SQLite or JSON file (MVP-friendly)<br>

---

# 📁 Code Skeleton

Below is the starter code for an MVP-ready system.

---

## 🌐 `backend/index.js`

```js
import Fastify from "fastify";
import cors from "@fastify/cors";
import fs from "fs";

const app = Fastify();
app.register(cors);

// Mock DB
const db = JSON.parse(fs.readFileSync("./data/mock.json", "utf8"));

app.get("/recent-tx", () => db.recentTx);
app.get("/addr/:address", (req) => db.addresses[req.params.address] || { error: "Address not found" });
app.get("/tx/:hash", (req) => db.txs[req.params.hash] || { error: "Transaction not found" });

app.get("/search", (req) => {
  const q = req.query.q;
  if (db.addresses[q]) return { type: "address", data: db.addresses[q] };
  if (db.txs[q]) return { type: "tx", data: db.txs[q] };
  return { error: "Not found" };
});

app.listen({ port: 3000 }, () => console.log("Backend running on :3000"));
```

---

## 🖥️ `frontend/src/App.jsx`

```jsx
import { useState, useEffect } from "react";

export default function App() {
  const [query, setQuery] = useState("");
  const [result, setResult] = useState(null);

  const search = async () => {
    const r = await fetch(`/search?q=${query}`).then(r => r.json());
    setResult(r);
  };

  return (
    <div className="p-4 max-w-xl mx-auto text-white">
      <h1 className="text-2xl font-bold mb-4">MYN Scout</h1>
      <input className="w-full p-2 bg-gray-800 rounded" placeholder="Search address or tx..." value={query} onChange={e => setQuery(e.target.value)} />
      <button onClick={search} className="mt-2 px-4 py-2 bg-blue-600 rounded">Search</button>

      <pre className="mt-4 bg-gray-900 p-4 rounded text-sm overflow-auto">
        {JSON.stringify(result, null, 2)}
      </pre>
    </div>
  );
}
```

---

## 🤖 `bot/bot.js`

```js
import TelegramBot from "node-telegram-bot-api";
import fetch from "node-fetch";

const bot = new TelegramBot(process.env.BOT_TOKEN, { polling: true });
const backend = "https://YOUR_BACKEND_URL";

bot.onText(/\/addr (.+)/, async (msg, match) => {
  const addr = match[1];
  const r = await fetch(`${backend}/addr/${addr}`).then(r => r.json());
  bot.sendMessage(msg.chat.id, "Address Info:\n" + JSON.stringify(r, null, 2));
});

bot.onText(/\/tx (.+)/, async (msg, match) => {
  const tx = match[1];
  const r = await fetch(`${backend}/tx/${tx}`).then(r => r.json());
  bot.sendMessage(msg.chat.id, "TX Details:\n" + JSON.stringify(r, null, 2));
});
```

---

## 🔄 `worker/watch.js`

```js
import fs from "fs";
import fetch from "node-fetch";
import TelegramBot from "node-telegram-bot-api";

const bot = new TelegramBot(process.env.BOT_TOKEN, { polling: false });
const backend = "https://YOUR_BACKEND_URL";

let subs = JSON.parse(fs.readFileSync("./data/subscribers.json"));
let lastSeen = {};

setInterval(async () => {
  for (let addr of Object.keys(subs)) {
    const data = await fetch(`${backend}/addr/${addr}`).then(r => r.json());
    if (!lastSeen[addr]) lastSeen[addr] = data.lastTx;

    if (data.lastTx !== lastSeen[addr]) {
      lastSeen[addr] = data.lastTx;
      for (let chatId of subs[addr]) {
        bot.sendMessage(chatId, `New transaction for ${addr}: ${data.lastTx}`);
      }
    }
  }
}, 10000);
```

---

# 🎥 Demo Script (for 60–90s presentation)

**1. Intro (5s)**
“Introducing MYN Scout — a lightweight explorer and Telegram assistant for the MYN ecosystem.”

**2. Web demo (20–30s)**

* Show search
* Open address page
* Open transaction page
* Show recent transactions feed

**3. Telegram bot demo (20–25s)**

* `/addr <address>` returns summary
* `/tx <hash>` displays transaction
* `/watch <address>`
* Trigger a new transaction → bot sends alert

**4. Closing (5–10s)**
“Fast, simple, and built for the MYN community. MYN Scout — ready to use today.”

---

# 🔧 Deployment Guide

### **Frontend (Vercel)**

```bash
npm install
npm run build
vercel deploy
```

### **Backend (Replit / Railway):**

Just import the folder and run:

```bash
npm install
node index.js
```

### **Telegram Bot (.env)**

```
BOT_TOKEN=YOUR_TELEGRAM_BOT_TOKEN
```

### **Worker**

Run on the same backend or separately:

```bash
node watch.js
```

---

# ✔️ Final Note
