# 🐳 Dockerized Chat Application

Cloud‑oriented Web Applications Course

A fully containerized, real‑time chat application using React (frontend), Node.js/Express (backend), MongoDB, Apache Kafka, and WebSockets. Everything runs with Docker Compose.

## Changelog
- Bug fixes applied to UI refresh, friend requests title duplication, and auth page background.

---

## 📋 Table of Contents

- 🏗️ Architecture Overview
- 🛠️ Technologies Used
- 📁 Project Structure
- ✅ Prerequisites
- 🚀 Installation & Setup
- 💬 Usage
- 🔌 API Endpoints
- 🏛️ Architecture Details
- 🐛 Troubleshooting

---

## 🏗️ Architecture Overview

```
+------------+     HTTP/WS      +------------+     MongoDB     +------------+
|  Frontend  | ---------------> |  Backend   |  ------------> |  Database  |
|  (React)   | <--------------- | (Node.js)  |  <------------ | (MongoDB)  |
|   :3000    |    WebSocket     |   :5000    |    Queries     |   :27017   |
+------------+                   +------------+                +------------+
                                     |
                                     |
                                     | Pub/Sub
                                     v
                               +------------+
                               |   Kafka    |
                               |  (Broker)  |
                               |   :9092    |
                               +------------+
```

Real‑time updates: backend persists and publishes messages to Kafka; the consumer broadcasts them to all clients over WebSocket.

---

## 🛠️ Technologies Used

- Frontend: React 18.2.0
- Backend: Node.js 18 + Express 4.18.2
- Database: MongoDB 6.0 + Mongoose 7.5.0
- Message Broker: Apache Kafka 7.4.0 (Confluent images)
- WebSocket: ws 8.13.0
- Kafka Client: KafkaJS 2.2.4
- Containerization: Docker & Docker Compose v2

---

## 📁 Project Structure

```
dockerized-chat-app/
├─ docker-compose.yml
├─ mongo-init.js
├─ backend/
│  ├─ Dockerfile
│  ├─ package.json
│  ├─ server.js
│  ├─ lib/
│  │  └─ kafka.js
│  ├─ middleware/
│  │  └─ authMiddleware.js
│  ├─ models/
│  │  ├─ FriendRequest.js
│  │  ├─ Message.js
│  │  └─ User.js
│  └─ routes/
│     ├─ auth.js
│     ├─ friends.js
│     ├─ messages.js
│     └─ privateMessages.js
└─ frontend/
   ├─ Dockerfile
   ├─ nginx.conf
   ├─ public/
   │  └─ index.html
   └─ src/
      ├─ App.css
      ├─ App.js
      ├─ index.css
      ├─ index.js
      └─ components/
         ├─ ChatRoom.js
         ├─ FriendRequests.js
         ├─ FriendsSidebar.js
         ├─ LoginRegister.js
         └─ PrivateChat.js
```

---

## ✅ Prerequisites

- Docker 20.10+ and Docker Compose v2
- Free ports: 3000, 5000, 9092, 27017
- ~4 GB RAM available for Docker

---

## 🚀 Installation & Setup

From the project root:

```
docker-compose up -d --build
```

First boot may take ~1–2 minutes (Kafka startup). Then open:

- Frontend: http://localhost:3000
- Health:   http://localhost:5000/health

Stop / clean:

```
docker-compose down        # remove containers
docker-compose down -v     # containers + volumes (fresh DB)
```

---

## 💬 Usage

1) Register then Login (username + password)
2) “All Users” → Add to send friend requests
3) “Friend Requests” (receiver) → Accept; friend moves to “Friends”
4) Click a friend to open a private chat
5) “Global Chat” item under Friends opens the public room

Notes:
- Friend requests are polled every 5s, so they appear without manual refresh
- Sender‑side duplicate messages are avoided (WS de‑dup)

---

## 🔌 API Endpoints

- Auth
  - POST `/api/auth/register` — { username, password }
  - POST `/api/auth/login` — { username, password } → { token }

- Messages (JWT)
  - GET `/api/messages`
  - POST `/api/messages` — { text, roomId: "general" }
  - DELETE `/api/messages` — testing only

- Private Messages (JWT)
  - GET `/api/private/:roomId`
  - POST `/api/private/:roomId` — { text }

- Friends (JWT)
  - GET `/api/friends/users` — all users except current
  - POST `/api/friends/request` — { to }
  - GET `/api/friends/requests` — pending for me
  - POST `/api/friends/accept` — { from } → returns normalized `roomId`
  - GET `/api/friends/list` — accepted friends

- Health
  - GET `/health`

---

## 🏛️ Architecture Details

- Frontend → Backend: REST over 5000 and WebSocket for updates
- Backend → MongoDB: 27017
- Backend → Kafka: `kafka:29092` (internal)
- Real‑time: Kafka consumer → WebSocket broadcast

---

## 🐛 Troubleshooting

- Kafka may need 30–60s on first boot
- Logs:
  - `docker-compose logs -f backend`
  - `docker-compose logs -f kafka`
  - `docker-compose logs -f database`
- Fresh start: `docker-compose down -v && docker-compose up --build`
- If base image pulls fail (DNS/proxy), configure DNS/proxy or pull manually:
  - `docker pull node:18-alpine`
  - `docker pull nginx:alpine`
