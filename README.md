<div align="center">

# 🔐 ChatFlow
### End-to-End Encrypted Real-Time Chat Application

[![Node.js](https://img.shields.io/badge/Node.js-18+-339933?style=for-the-badge&logo=node.js&logoColor=white)](https://nodejs.org/)
[![React](https://img.shields.io/badge/React-18-61DAFB?style=for-the-badge&logo=react&logoColor=black)](https://react.dev/)
[![MongoDB](https://img.shields.io/badge/MongoDB-Atlas-47A248?style=for-the-badge&logo=mongodb&logoColor=white)](https://www.mongodb.com/)
[![Socket.IO](https://img.shields.io/badge/Socket.IO-4.8-010101?style=for-the-badge&logo=socket.io&logoColor=white)](https://socket.io/)
[![Deployed on Render](https://img.shields.io/badge/Backend-Render-46E3B7?style=for-the-badge&logo=render&logoColor=white)](https://render.com/)
[![Deployed on Vercel](https://img.shields.io/badge/Frontend-Vercel-000000?style=for-the-badge&logo=vercel&logoColor=white)](https://vercel.com/)

**A production-ready, full-stack chat app where the server never sees your messages.**  
Built with classical cryptography — Diffie-Hellman Key Exchange, Caesar Cipher & Vernam Cipher — implemented entirely from scratch.

[🚀 Live Demo](https://chatflow.vercel.app) · [📖 API Docs](#-api-endpoints) · [🧪 Run Tests](#-testing) · [📦 Deploy](#️-deployment)

</div>

---

## ✨ What Makes This Different

Most chat apps store your messages in plaintext on their servers. **ChatFlow doesn't.**

- 🔒 **Zero plaintext on the server** — messages are encrypted in the browser before being sent
- 🤝 **Manual Diffie-Hellman** — 2048-bit key exchange built from scratch with `BigInt` and square-and-multiply (RFC 3526 Group 14)
- 🔑 **Private keys never leave your device** — stored only in `localStorage`, never transmitted
- 🔄 **Real-time & encrypted** — Socket.IO + E2EE working seamlessly together
- 🎓 **Educational** — every algorithm is hand-rolled, readable, and easy to explain

---

## 🏗️ Architecture

```
┌──────────────────────┐         ┌──────────────────────┐
│    Alice's Browser   │         │    Bob's Browser     │
│                      │         │                      │
│  🔑 DH Private Key   │         │  🔑 DH Private Key   │
│  (localStorage only) │         │  (localStorage only) │
│                      │         │                      │
│  masterKey = SHA256( │         │  masterKey = SHA256( │
│   DH shared secret ) │         │   DH shared secret ) │
│         ↓            │         │         ↑            │
│  Encrypt(plaintext)  │         │  Decrypt(ciphertext) │
└──────────┬───────────┘         └──────────┬───────────┘
           │  { text: "5f3a...", cipher, nonce }
           ▼                                ▲
┌──────────────────────────────────────────────────────┐
│              Node.js + Express + Socket.IO            │
│                                                      │
│  ✅ Stores ciphertext   ✅ Broadcasts ciphertext      │
│  ❌ Never decrypts      ❌ Never sees plaintext        │
└──────────────────────────────┬───────────────────────┘
                               │
                    ┌──────────▼──────────┐
                    │    MongoDB Atlas     │
                    │  messages.text =    │
                    │  "5f3a1c2d9e4b..."  │  ← ciphertext only
                    │  chats.lastMsg  =   │
                    │  "🔒 Encrypted msg" │
                    └─────────────────────┘
```

---

## 🚀 Features

### 🔐 Security & Cryptography
- **Diffie-Hellman Key Exchange** — RFC 3526 2048-bit Group 14, square-and-multiply `modPow` with `BigInt`
- **Caesar Cipher** — shift `= (masterKey[0] % 94) + 1`, operates on printable ASCII 32–126
- **Vernam Cipher** — SHA-256 keystream XOR with a fresh 12-byte nonce per message (two-time-pad safe)
- **Key Derivation** — `SHA-256("chatflow-dh-v1" ∥ sharedSecretBytes)` → 32-byte master key
- **Visual Fingerprint** — `SHA-256(masterKey)` first 16 hex chars for out-of-band verification

### 💬 Real-Time Chat
- ⚡ Instant messaging via Socket.IO WebSockets with REST fallback
- ✍️ Live typing indicators
- 👁️ Read receipts (sent / delivered / seen)
- 🟢 Real-time online / offline presence
- 🔢 Unread message badge counts

### 🎨 User Experience
- 🔀 **Cipher selector** — switch Caesar ↔ Vernam mid-conversation; each message decrypts with its own cipher
- 🔍 **Server View Mode** — toggle raw ciphertext view to see exactly what the server stores
- 📊 **Key Exchange Panel** — prime p, generator g, public keys, shared fingerprint, Caesar shift
- 🔑 **Auto key setup** — DH key pair generated and uploaded silently on first login
- ⚠️ **New device warning** — alerts when old messages can't be decrypted on a new device

---

## 🧪 Testing

**24 unit tests** — Node.js built-in test runner, zero extra dependencies.

```bash
cd frontend
npm run test:crypto
```

```
▶ DH Prime Validation         ✔ 2048-bit  ✔ top 64 bits all 1s  ✔ bottom 64 bits all 1s
▶ DH Shared Secret            ✔ both sides match  ✔ secret is large
▶ Partner Key Validation      ✔ rejects 0  ✔ rejects 1  ✔ rejects p-1
▶ Caesar Cipher               ✔ English  ✔ symbols  ✔ spaces  ✔ empty  ✔ emoji pass-through
▶ Vernam Cipher               ✔ ASCII  ✔ emoji  ✔ multilingual  ✔ >32 bytes  ✔ >1000 bytes
▶ Vernam Nonce Uniqueness     ✔ same plaintext → different ciphertext each time
▶ Key Mismatch                ✔ wrong key fails for both ciphers
▶ Key Fingerprint             ✔ both sides identical  ✔ correct format

ℹ tests 24  |  pass 24  |  fail 0  ✅
```

---

## 🗄️ Database Schema

<details>
<summary><strong>Click to expand MongoDB collections</strong></summary>

### `users`
```js
{
  name: String,
  email: String,        // unique, indexed
  password: String,     // bcrypt hash — never returned in API responses
  publicKey: String,    // DH public key (hex) — uploaded on login
  isOnline: Boolean,
  lastSeen: Date
}
```

### `messages`
```js
{
  chatId: ObjectId,
  sender: ObjectId,
  receiver: ObjectId,
  text: String,         // ← CIPHERTEXT (no trim, no plaintext ever stored)
  cipher: String,       // "caesar" | "vernam" | "none"
  nonce: String,        // 24-char hex — Vernam only
  isEncrypted: Boolean,
  status: String,       // "sent" | "delivered" | "seen"
  seenAt: Date
}
```

### `chats`
```js
{
  participants: [ObjectId],
  lastMessage: ObjectId,
  lastMessageText: String,  // always "🔒 Encrypted message" — never plaintext
  lastMessageTime: Date,
  unreadCount: Map          // { userId → unreadCount }
}
```
</details>

---

## 📡 API Endpoints

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/api/auth/register` | ❌ | Register new user |
| `POST` | `/api/auth/login` | ❌ | Login, receive JWT |
| `GET` | `/api/users/me` | ✅ | Get current user |
| `GET` | `/api/users?search=` | ✅ | Search users |
| `PUT` | `/api/users/public-key` | ✅ | Upload DH public key |
| `GET` | `/api/chats` | ✅ | Get all chats |
| `POST` | `/api/chats` | ✅ | Create or open 1-to-1 chat |
| `GET` | `/api/messages/:chatId` | ✅ | Paginated message history |
| `POST` | `/api/messages` | ✅ | Send message (REST fallback) |
| `PUT` | `/api/messages/seen/:chatId` | ✅ | Mark messages as seen |

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| **Frontend** | React 18, Vite, React Router v7, Axios |
| **Real-time** | Socket.IO (client + server) |
| **Backend** | Node.js, Express.js |
| **Database** | MongoDB with Mongoose ODM |
| **Auth** | JSON Web Tokens + bcryptjs |
| **Crypto** | Web Crypto API (SHA-256 + `getRandomValues` only) + manual BigInt DH |
| **Hosting** | Render (backend) · Vercel (frontend) · MongoDB Atlas |

> **Zero crypto libraries used for ciphers or key exchange** — every algorithm is implemented from scratch.

---

## ⚡ Quick Start

### Prerequisites
- Node.js 18+ · MongoDB Atlas account

```bash
git clone https://github.com/YOUR_USERNAME/chatflow.git
cd chatflow
```

### Backend Setup
```bash
cd backend
npm install
cp .env.example .env   # fill in your MONGO_URI and JWT_SECRET
npm run dev            # http://localhost:5000
```

### Frontend Setup
```bash
cd frontend
npm install
# create frontend/.env with:
# VITE_API_URL=http://localhost:5000
npm run dev            # http://localhost:5173
```

> **Note:** `crypto.subtle` requires a secure context. `localhost` works fine; never serve over plain HTTP in production.

---

## ☁️ Deployment

| | Render (Backend) | Vercel (Frontend) |
|--|--|--|
| **Root Dir** | `backend` | `frontend` |
| **Build** | `npm install` | `npm run build` |
| **Start** | `node server.js` | — |
| **Key Env Var** | `MONGO_URI`, `CLIENT_URL` | `VITE_API_URL` |

Set `CLIENT_URL` on Render = your Vercel URL, and `VITE_API_URL` on Vercel = your Render URL.

---

## 📁 Project Structure

```
chatflow/
├── backend/
│   ├── config/db.js            # MongoDB connection
│   ├── controllers/            # Auth, User, Message, Chat
│   ├── middleware/authMiddleware.js
│   ├── models/                 # User, Message, Chat schemas
│   ├── routes/                 # Express routers
│   ├── services/messageService.js  # Core message logic + cipher validation
│   ├── socket/socketHandler.js     # All Socket.IO events
│   └── server.js
│
├── frontend/src/
│   ├── crypto/                 # 🔐 Hand-rolled cryptography
│   │   ├── dh.js               # Diffie-Hellman (modPow, key gen, validation)
│   │   ├── ciphers.js          # Caesar + Vernam + key derivation + fingerprint
│   │   ├── keyStore.js         # localStorage key management + master key cache
│   │   └── __tests__/          # 24 unit tests (Node built-in runner)
│   ├── components/
│   │   ├── KeyExchangePanel.jsx  # DH info panel with fingerprint
│   │   ├── MessageInput.jsx      # Input + cipher selector
│   │   ├── MessageList.jsx       # Messages + lock badge + server view
│   │   └── ChatList.jsx
│   ├── context/                # Auth + Socket providers
│   ├── pages/Chat.jsx          # Main encrypted chat page
│   └── services/               # API client + chat helpers
│
└── docs/DEMO.md                # MongoDB + Wireshark demo guide
```

---

## ⚠️ Limitations

| Issue | Detail |
|-------|--------|
| No key authentication | A malicious server could MITM — mitigated by fingerprint comparison |
| localStorage storage | Clearing browser data loses access to encrypted message history |
| Caesar is weak | 94 possible shifts — educational only, not production-secure |
| PRF-based Vernam | SHA-256 keystream, not a true one-time pad |
| 1-to-1 only | Group chat encryption requires a separate group key protocol |

---

## 📜 License

MIT © 2026 Shaleen Singh

---

<div align="center">

**If this project helped you understand end-to-end encryption, drop a ⭐**

*Made with ❤️ and a lot of BigInt arithmetic*

</div>
