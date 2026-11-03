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

[🚀 Live Demo](https://no-sql-project.vercel.app/login) · [📖 API Docs](#api-endpoints) · [🧪 Run Tests](#testing) · [📦 Deploy Guide](#deployment)

<img src="https://raw.githubusercontent.com/YOUR_USERNAME/chatflow/main/docs/preview.png" alt="ChatFlow Preview" width="800"/>

</div>

---

## ✨ What Makes This Different

Most chat apps store your messages in plaintext on their servers. **ChatFlow doesn't.**

- 🔒 **Zero plaintext on the server** — messages are encrypted in the browser before being sent
- 🤝 **Manual Diffie-Hellman** — 2048-bit key exchange built from scratch with BigInt arithmetic (RFC 3526 Group 14)
- 🔑 **Private keys never leave your device** — stored only in localStorage, never transmitted
- 🔄 **Real-time & encrypted** — Socket.IO + E2EE working seamlessly together
- 🎓 **Educational** — every algorithm is hand-rolled and easy to understand and explain

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
                    │  "5f3a1c2d9e4b..."  │ ← ciphertext
                    │  chats.lastMsg  =   │
                    │  "🔒 Encrypted msg" │
                    └─────────────────────┘
```

---

## 🚀 Features

### 🔐 Security & Cryptography

- **Diffie-Hellman Key Exchange** — RFC 3526 2048-bit Group 14 prime, implemented with `BigInt` and square-and-multiply modular exponentiation
- **Caesar Cipher** — key-derived shift `(masterKey[0] % 94) + 1`, operates on printable ASCII 32–126
- **Vernam Cipher** — SHA-256 keystream XOR with a fresh 12-byte random nonce per message (two-time-pad safe)
- **Key Derivation** — `SHA-256("chatflow-dh-v1" ∥ sharedSecretBytes)` → 32-byte master key
- **Visual Fingerprint** — first 16 hex chars of `SHA-256(masterKey)`, formatted `xxxx xxxx xxxx xxxx` for out-of-band verification

### 💬 Real-Time Chat

- ⚡ **Instant messaging** via Socket.IO WebSockets
- ✍️ **Typing indicators** — live "is typing..." status
- 👁️ **Read receipts** — seen / delivered status with timestamps
- 🟢 **Online/offline presence** — real-time user status
- 🔢 **Unread message counts** on chat list

### 🎨 User Experience

- 🔀 **Cipher selector** — switch between Caesar and Vernam mid-conversation
- 🔍 **Server View Mode** — toggle to reveal raw ciphertext as the server sees it
- 📊 **Key Exchange Panel** — shows prime p, generator g, public keys, shared fingerprint, and Caesar shift
- 🔑 **Auto key setup** — DH key pair generated and uploaded silently on login
- ⚠️ **New device warning** — alerts when a new key was created and old messages can't be decrypted

### 🛠️ Engineering

- 🔄 **REST + Socket.IO** — graceful fallback to HTTP if WebSocket drops
- 📄 **Pagination** — efficient message history loading
- 🔍 **User search** — real-time search by name or email
- 🛡️ **JWT authentication** with bcrypt password hashing
- 📱 **Responsive design** — works on mobile and desktop

---

## 🧪 Testing

**24 unit tests** with Node.js built-in test runner — zero additional dependencies.

```bash
cd frontend
npm run test:crypto
```

```
▶ DH Prime Validation         ✔ 2048-bit prime  ✔ top 64 bits  ✔ bottom 64 bits
▶ DH Shared Secret            ✔ both sides match  ✔ secret is large
▶ Partner Key Validation      ✔ rejects 0  ✔ rejects 1  ✔ rejects p-1
▶ Caesar Cipher               ✔ English  ✔ digits/symbols  ✔ spaces  ✔ empty  ✔ emoji  ✔ encryption works
▶ Vernam Cipher               ✔ ASCII  ✔ emoji  ✔ multilingual  ✔ >32 bytes  ✔ >1000 bytes
▶ Vernam Nonce Uniqueness      ✔ same plaintext → different ciphertext
▶ Key Mismatch                ✔ wrong key fails (Vernam)  ✔ wrong key fails (Caesar)
▶ Key Fingerprint             ✔ both sides match  ✔ correct format

ℹ tests 24  |  pass 24  |  fail 0  ✅
```

---

## 🗄️ Database Schema

<details>
<summary><strong>📋 MongoDB Collections (click to expand)</strong></summary>

### `users`

```js
{
  name: String,
  email: String,        // unique, indexed
  password: String,     // bcrypt hash, never returned in queries
  publicKey: String,    // DH public key hex — NEW
  isOnline: Boolean,
  lastSeen: Date,
  profilePic: String
}
```

### `messages`

```js
{
  chatId: ObjectId,
  sender: ObjectId,
  receiver: ObjectId,
  text: String,         // CIPHERTEXT — no trim, no plaintext
  cipher: String,       // "caesar" | "vernam" | "none" — NEW
  nonce: String,        // 24-char hex, Vernam only — NEW
  isEncrypted: Boolean, // NEW
  status: String,       // "sent" | "delivered" | "seen"
  seenAt: Date
}
```

### `chats`

```js
{
  participants: [ObjectId],
  lastMessage: ObjectId,
  lastMessageText: String,  // "🔒 Encrypted message" — never stores plaintext
  lastMessageTime: Date,
  unreadCount: Map,         // { userId: count }
  directKey: String         // unique key for 1-to-1 chats
}
```

</details>

---

## 📡 API Endpoints

| Method | Endpoint                     | Auth | Description                  |
| ------ | ---------------------------- | ---- | ---------------------------- |
| `POST` | `/api/auth/register`         | ❌   | Register new user            |
| `POST` | `/api/auth/login`            | ❌   | Login, receive JWT           |
| `GET`  | `/api/users/me`              | ✅   | Get current user profile     |
| `GET`  | `/api/users?search=`         | ✅   | Search users                 |
| `PUT`  | `/api/users/public-key`      | ✅   | Upload DH public key         |
| `GET`  | `/api/chats`                 | ✅   | Get all chats                |
| `POST` | `/api/chats`                 | ✅   | Create or open 1-to-1 chat   |
| `GET`  | `/api/messages/:chatId`      | ✅   | Paginated message history    |
| `POST` | `/api/messages`              | ✅   | Send message (REST fallback) |
| `PUT`  | `/api/messages/seen/:chatId` | ✅   | Mark messages as seen        |

---

## 🛠️ Tech Stack

| Layer         | Technology                                                        |
| ------------- | ----------------------------------------------------------------- |
| **Frontend**  | React 18, Vite, React Router v7, Axios                            |
| **Real-time** | Socket.IO (client + server)                                       |
| **Backend**   | Node.js, Express.js (CommonJS)                                    |
| **Database**  | MongoDB with Mongoose ODM                                         |
| **Auth**      | JSON Web Tokens + bcryptjs                                        |
| **Crypto**    | Web Crypto API (SHA-256, getRandomValues only) + manual BigInt DH |
| **Hosting**   | Render (backend) + Vercel (frontend) + MongoDB Atlas              |

> **Zero crypto libraries used for ciphers or DH** — every algorithm is implemented from scratch for educational transparency.

---

## ⚡ Quick Start (Local)

### Prerequisites

- Node.js 18+
- MongoDB Atlas account (or local MongoDB)

### 1. Clone & Install

```bash
git clone https://github.com/YOUR_USERNAME/chatflow.git
cd chatflow

# Install backend dependencies
cd backend && npm install

# Install frontend dependencies
cd ../frontend && npm install
```

### 2. Configure Environment

**`backend/.env`**

```env
PORT=5000
MONGO_URI=mongodb+srv://<user>:<password>@<cluster>.mongodb.net/chatflow
JWT_SECRET=your_jwt_secret_here
JWT_EXPIRES_IN=7d
CLIENT_URL=http://localhost:5173
```

**`frontend/.env`** (create this file)

```env
VITE_API_URL=http://localhost:5000
```

### 3. Run

```bash
# Terminal 1 — Backend
cd backend && npm run dev

# Terminal 2 — Frontend
cd frontend && npm run dev
```

Open **http://localhost:5173** 🚀

> ⚠️ `crypto.subtle` requires a **secure context** — `localhost` works fine. Do not serve over plain HTTP in production.

---

## ☁️ Deployment

### Backend → [Render](https://render.com)

| Setting        | Value                    |
| -------------- | ------------------------ |
| Root Directory | `backend`                |
| Build Command  | `npm install`            |
| Start Command  | `node server.js`         |
| Environment    | Add all vars from `.env` |

### Frontend → [Vercel](https://vercel.com)

| Setting          | Value                               |
| ---------------- | ----------------------------------- |
| Root Directory   | `frontend`                          |
| Framework        | Vite                                |
| Build Command    | `npm run build`                     |
| Output Directory | `dist`                              |
| `VITE_API_URL`   | `https://your-backend.onrender.com` |

---

## 🔏 How the Encryption Works (Quick Explainer)

```
1. Alice logs in  →  generates 2048-bit DH key pair
                  →  uploads public key to server
                  →  private key stays in localStorage ONLY

2. Bob does the same.

3. Alice opens chat with Bob:
   sharedSecret = modPow(bob_publicKey, alice_privateKey, prime)
   masterKey    = SHA-256("chatflow-dh-v1" || sharedSecret)

4. Bob computes the same:
   sharedSecret = modPow(alice_publicKey, bob_privateKey, prime)
   masterKey    = same SHA-256 hash  ✓  (DH property)

5. Alice sends a message:
   ciphertext = Caesar_Encrypt(plaintext, masterKey)
             OR Vernam_Encrypt(plaintext, masterKey, random_nonce)
   → server receives ciphertext, stores ciphertext, forwards ciphertext

6. Bob decrypts:
   plaintext = Caesar_Decrypt(ciphertext, masterKey)
            OR Vernam_Decrypt(ciphertext, masterKey, nonce)
```

---

## ⚠️ Known Limitations

| Limitation                    | Notes                                                                        |
| ----------------------------- | ---------------------------------------------------------------------------- |
| No public key authentication  | A compromised server could MITM. Mitigated by visual fingerprint comparison. |
| Private key in localStorage   | Clearing browser data or using a new device loses access to old messages.    |
| Caesar is easily brute-forced | Only 94 possible shifts. Included for educational purposes.                  |
| Vernam uses PRF, not true OTP | Keystream is SHA-256-derived, not a true random pad.                         |
| 1-to-1 chats only             | Group chat encryption would require a group key protocol.                    |

---

## 📁 Project Structure

```
chatflow/
├── backend/
│   ├── config/         # DB connection (with Google DNS fix)
│   ├── controllers/    # Auth, User, Message, Chat logic
│   ├── middleware/     # JWT auth middleware
│   ├── models/         # Mongoose schemas (User, Message, Chat)
│   ├── routes/         # Express routers
│   ├── services/       # messageService.js (core business logic)
│   ├── socket/         # Socket.IO event handler
│   └── server.js       # App entry point
│
├── frontend/
│   ├── src/
│   │   ├── crypto/         # 🔐 All cryptography
│   │   │   ├── dh.js           # Diffie-Hellman (modPow, key gen)
│   │   │   ├── ciphers.js      # Caesar, Vernam, key derivation
│   │   │   ├── keyStore.js     # localStorage + master key cache
│   │   │   └── __tests__/      # 24 unit tests
│   │   ├── components/     # React UI components
│   │   ├── context/        # Auth + Socket context providers
│   │   ├── pages/          # Chat, Login, Register pages
│   │   └── services/       # API client, chat helpers
│   └── package.json
│
└── docs/
    └── DEMO.md         # MongoDB & Wireshark demo guide
```

---

## 📜 License

MIT © 2026 Shaleen Singh

---

<div align="center">

**If this project helped you understand end-to-end encryption, give it a ⭐**

Made with ❤️ and a lot of BigInt arithmetic

</div>
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

[🚀 Live Demo](https://chatflow.vercel.app) · [📖 API Docs](#api-endpoints) · [🧪 Run Tests](#testing) · [📦 Deploy Guide](#deployment)

<img src="https://raw.githubusercontent.com/YOUR_USERNAME/chatflow/main/docs/preview.png" alt="ChatFlow Preview" width="800"/>

</div>

---

## ✨ What Makes This Different

Most chat apps store your messages in plaintext on their servers. **ChatFlow doesn't.**

- 🔒 **Zero plaintext on the server** — messages are encrypted in the browser before being sent
- 🤝 **Manual Diffie-Hellman** — 2048-bit key exchange built from scratch with BigInt arithmetic (RFC 3526 Group 14)
- 🔑 **Private keys never leave your device** — stored only in localStorage, never transmitted
- 🔄 **Real-time & encrypted** — Socket.IO + E2EE working seamlessly together
- 🎓 **Educational** — every algorithm is hand-rolled and easy to understand and explain

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
                    │  "5f3a1c2d9e4b..."  │ ← ciphertext
                    │  chats.lastMsg  =   │
                    │  "🔒 Encrypted msg" │
                    └─────────────────────┘
```

---

## 🚀 Features

### 🔐 Security & Cryptography

- **Diffie-Hellman Key Exchange** — RFC 3526 2048-bit Group 14 prime, implemented with `BigInt` and square-and-multiply modular exponentiation
- **Caesar Cipher** — key-derived shift `(masterKey[0] % 94) + 1`, operates on printable ASCII 32–126
- **Vernam Cipher** — SHA-256 keystream XOR with a fresh 12-byte random nonce per message (two-time-pad safe)
- **Key Derivation** — `SHA-256("chatflow-dh-v1" ∥ sharedSecretBytes)` → 32-byte master key
- **Visual Fingerprint** — first 16 hex chars of `SHA-256(masterKey)`, formatted `xxxx xxxx xxxx xxxx` for out-of-band verification

### 💬 Real-Time Chat

- ⚡ **Instant messaging** via Socket.IO WebSockets
- ✍️ **Typing indicators** — live "is typing..." status
- 👁️ **Read receipts** — seen / delivered status with timestamps
- 🟢 **Online/offline presence** — real-time user status
- 🔢 **Unread message counts** on chat list

### 🎨 User Experience

- 🔀 **Cipher selector** — switch between Caesar and Vernam mid-conversation
- 🔍 **Server View Mode** — toggle to reveal raw ciphertext as the server sees it
- 📊 **Key Exchange Panel** — shows prime p, generator g, public keys, shared fingerprint, and Caesar shift
- 🔑 **Auto key setup** — DH key pair generated and uploaded silently on login
- ⚠️ **New device warning** — alerts when a new key was created and old messages can't be decrypted

### 🛠️ Engineering

- 🔄 **REST + Socket.IO** — graceful fallback to HTTP if WebSocket drops
- 📄 **Pagination** — efficient message history loading
- 🔍 **User search** — real-time search by name or email
- 🛡️ **JWT authentication** with bcrypt password hashing
- 📱 **Responsive design** — works on mobile and desktop

---

## 🧪 Testing

**24 unit tests** with Node.js built-in test runner — zero additional dependencies.

```bash
cd frontend
npm run test:crypto
```

```
▶ DH Prime Validation         ✔ 2048-bit prime  ✔ top 64 bits  ✔ bottom 64 bits
▶ DH Shared Secret            ✔ both sides match  ✔ secret is large
▶ Partner Key Validation      ✔ rejects 0  ✔ rejects 1  ✔ rejects p-1
▶ Caesar Cipher               ✔ English  ✔ digits/symbols  ✔ spaces  ✔ empty  ✔ emoji  ✔ encryption works
▶ Vernam Cipher               ✔ ASCII  ✔ emoji  ✔ multilingual  ✔ >32 bytes  ✔ >1000 bytes
▶ Vernam Nonce Uniqueness      ✔ same plaintext → different ciphertext
▶ Key Mismatch                ✔ wrong key fails (Vernam)  ✔ wrong key fails (Caesar)
▶ Key Fingerprint             ✔ both sides match  ✔ correct format

ℹ tests 24  |  pass 24  |  fail 0  ✅
```

---

## 🗄️ Database Schema

<details>
<summary><strong>📋 MongoDB Collections (click to expand)</strong></summary>

### `users`

```js
{
  name: String,
  email: String,        // unique, indexed
  password: String,     // bcrypt hash, never returned in queries
  publicKey: String,    // DH public key hex — NEW
  isOnline: Boolean,
  lastSeen: Date,
  profilePic: String
}
```

### `messages`

```js
{
  chatId: ObjectId,
  sender: ObjectId,
  receiver: ObjectId,
  text: String,         // CIPHERTEXT — no trim, no plaintext
  cipher: String,       // "caesar" | "vernam" | "none" — NEW
  nonce: String,        // 24-char hex, Vernam only — NEW
  isEncrypted: Boolean, // NEW
  status: String,       // "sent" | "delivered" | "seen"
  seenAt: Date
}
```

### `chats`

```js
{
  participants: [ObjectId],
  lastMessage: ObjectId,
  lastMessageText: String,  // "🔒 Encrypted message" — never stores plaintext
  lastMessageTime: Date,
  unreadCount: Map,         // { userId: count }
  directKey: String         // unique key for 1-to-1 chats
}
```

</details>

---

## 📡 API Endpoints

| Method | Endpoint                     | Auth | Description                  |
| ------ | ---------------------------- | ---- | ---------------------------- |
| `POST` | `/api/auth/register`         | ❌   | Register new user            |
| `POST` | `/api/auth/login`            | ❌   | Login, receive JWT           |
| `GET`  | `/api/users/me`              | ✅   | Get current user profile     |
| `GET`  | `/api/users?search=`         | ✅   | Search users                 |
| `PUT`  | `/api/users/public-key`      | ✅   | Upload DH public key         |
| `GET`  | `/api/chats`                 | ✅   | Get all chats                |
| `POST` | `/api/chats`                 | ✅   | Create or open 1-to-1 chat   |
| `GET`  | `/api/messages/:chatId`      | ✅   | Paginated message history    |
| `POST` | `/api/messages`              | ✅   | Send message (REST fallback) |
| `PUT`  | `/api/messages/seen/:chatId` | ✅   | Mark messages as seen        |

---

## 🛠️ Tech Stack

| Layer         | Technology                                                        |
| ------------- | ----------------------------------------------------------------- |
| **Frontend**  | React 18, Vite, React Router v7, Axios                            |
| **Real-time** | Socket.IO (client + server)                                       |
| **Backend**   | Node.js, Express.js (CommonJS)                                    |
| **Database**  | MongoDB with Mongoose ODM                                         |
| **Auth**      | JSON Web Tokens + bcryptjs                                        |
| **Crypto**    | Web Crypto API (SHA-256, getRandomValues only) + manual BigInt DH |
| **Hosting**   | Render (backend) + Vercel (frontend) + MongoDB Atlas              |

> **Zero crypto libraries used for ciphers or DH** — every algorithm is implemented from scratch for educational transparency.

---

## ⚡ Quick Start (Local)

### Prerequisites

- Node.js 18+
- MongoDB Atlas account (or local MongoDB)

### 1. Clone & Install

```bash
git clone https://github.com/YOUR_USERNAME/chatflow.git
cd chatflow

# Install backend dependencies
cd backend && npm install

# Install frontend dependencies
cd ../frontend && npm install
```

### 2. Configure Environment

**`backend/.env`**

```env
PORT=5000
MONGO_URI=mongodb+srv://<user>:<password>@<cluster>.mongodb.net/chatflow
JWT_SECRET=your_jwt_secret_here
JWT_EXPIRES_IN=7d
CLIENT_URL=http://localhost:5173
```

**`frontend/.env`** (create this file)

```env
VITE_API_URL=http://localhost:5000
```

### 3. Run

```bash
# Terminal 1 — Backend
cd backend && npm run dev

# Terminal 2 — Frontend
cd frontend && npm run dev
```

Open **http://localhost:5173** 🚀

> ⚠️ `crypto.subtle` requires a **secure context** — `localhost` works fine. Do not serve over plain HTTP in production.

---

## ☁️ Deployment

### Backend → [Render](https://render.com)

| Setting        | Value                    |
| -------------- | ------------------------ |
| Root Directory | `backend`                |
| Build Command  | `npm install`            |
| Start Command  | `node server.js`         |
| Environment    | Add all vars from `.env` |

### Frontend → [Vercel](https://vercel.com)

| Setting          | Value                               |
| ---------------- | ----------------------------------- |
| Root Directory   | `frontend`                          |
| Framework        | Vite                                |
| Build Command    | `npm run build`                     |
| Output Directory | `dist`                              |
| `VITE_API_URL`   | `https://your-backend.onrender.com` |

---

## 🔏 How the Encryption Works (Quick Explainer)

```
1. Alice logs in  →  generates 2048-bit DH key pair
                  →  uploads public key to server
                  →  private key stays in localStorage ONLY

2. Bob does the same.

3. Alice opens chat with Bob:
   sharedSecret = modPow(bob_publicKey, alice_privateKey, prime)
   masterKey    = SHA-256("chatflow-dh-v1" || sharedSecret)

4. Bob computes the same:
   sharedSecret = modPow(alice_publicKey, bob_privateKey, prime)
   masterKey    = same SHA-256 hash  ✓  (DH property)

5. Alice sends a message:
   ciphertext = Caesar_Encrypt(plaintext, masterKey)
             OR Vernam_Encrypt(plaintext, masterKey, random_nonce)
   → server receives ciphertext, stores ciphertext, forwards ciphertext

6. Bob decrypts:
   plaintext = Caesar_Decrypt(ciphertext, masterKey)
            OR Vernam_Decrypt(ciphertext, masterKey, nonce)
```

---

## ⚠️ Known Limitations

| Limitation                    | Notes                                                                        |
| ----------------------------- | ---------------------------------------------------------------------------- |
| No public key authentication  | A compromised server could MITM. Mitigated by visual fingerprint comparison. |
| Private key in localStorage   | Clearing browser data or using a new device loses access to old messages.    |
| Caesar is easily brute-forced | Only 94 possible shifts. Included for educational purposes.                  |
| Vernam uses PRF, not true OTP | Keystream is SHA-256-derived, not a true random pad.                         |
| 1-to-1 chats only             | Group chat encryption would require a group key protocol.                    |

---

## 📁 Project Structure

```
chatflow/
├── backend/
│   ├── config/         # DB connection (with Google DNS fix)
│   ├── controllers/    # Auth, User, Message, Chat logic
│   ├── middleware/     # JWT auth middleware
│   ├── models/         # Mongoose schemas (User, Message, Chat)
│   ├── routes/         # Express routers
│   ├── services/       # messageService.js (core business logic)
│   ├── socket/         # Socket.IO event handler
│   └── server.js       # App entry point
│
├── frontend/
│   ├── src/
│   │   ├── crypto/         # 🔐 All cryptography
│   │   │   ├── dh.js           # Diffie-Hellman (modPow, key gen)
│   │   │   ├── ciphers.js      # Caesar, Vernam, key derivation
│   │   │   ├── keyStore.js     # localStorage + master key cache
│   │   │   └── __tests__/      # 24 unit tests
│   │   ├── components/     # React UI components
│   │   ├── context/        # Auth + Socket context providers
│   │   ├── pages/          # Chat, Login, Register pages
│   │   └── services/       # API client, chat helpers
│   └── package.json
│
└── docs/
    └── DEMO.md         # MongoDB & Wireshark demo guide
```

---

## 📜 License

MIT © 2026 Shaleen Singh

---

<div align="center">

**If this project helped you understand end-to-end encryption, give it a ⭐**

Made with ❤️ and a lot of BigInt arithmetic

</div>
