# 🌐 SyncSphere

<div align="center">

![SyncSphere Hero Banner](https://img.shields.io/badge/SyncSphere-Real--Time_Collaboration-4f46e5?style=for-the-badge&logo=webrtc&logoColor=white)

**Enterprise-Grade Real-Time Communication Platform**  
*Ultra-Low Latency Multi-Party Video (Mediasoup WebRTC SFU) • Asynchronous Message Persistence (Apache Kafka) • Horizontally Scalable Distributed State (Redis Pub/Sub) • Next.js 15 App Router*

<br/>

[![Next.js](https://img.shields.io/badge/Next.js_15-black?style=flat-square&logo=next.js&logoColor=white)](https://nextjs.org/)
[![React](https://img.shields.io/badge/React_19-20232A?style=flat-square&logo=react&logoColor=61DAFB)](https://react.dev/)
[![TypeScript](https://img.shields.io/badge/TypeScript_5-3178C6?style=flat-square&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Node.js](https://img.shields.io/badge/Node.js-339933?style=flat-square&logo=node.js&logoColor=white)](https://nodejs.org/)
[![Express](https://img.shields.io/badge/Express.js-000000?style=flat-square&logo=express&logoColor=white)](https://expressjs.com/)
[![WebRTC Mediasoup](https://img.shields.io/badge/Mediasoup_SFU-FF4088?style=flat-square&logo=webrtc&logoColor=white)](https://mediasoup.org/)
[![Apache Kafka](https://img.shields.io/badge/Apache_Kafka-231F20?style=flat-square&logo=apache-kafka&logoColor=white)](https://kafka.apache.org/)
[![Redis](https://img.shields.io/badge/Redis_7-DC382D?style=flat-square&logo=redis&logoColor=white)](https://redis.io/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL_15-4169E1?style=flat-square&logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Prisma ORM](https://img.shields.io/badge/Prisma_6-2D3748?style=flat-square&logo=prisma&logoColor=white)](https://www.prisma.io/)
[![Docker](https://img.shields.io/badge/Docker_Compose-2496ED?style=flat-square&logo=docker&logoColor=white)](https://www.docker.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](LICENSE)

</div>

---

## 📖 Executive Summary & Engineering Philosophy

**SyncSphere** is a production-hardened full-stack monorepo designed to solve the three most critical failure points in real-time collaboration systems:

1. **Quadratic Video Bandwidth Explosion ($O(N^2)$)**: Mesh topologies require each participant to encode and transmit streams to every other peer. SyncSphere implements **Mediasoup**, a high-performance C++ Selective Forwarding Unit (SFU), achieving linear $O(N)$ bandwidth scaling and dynamic bitrate adaptation.
2. **Database Write Amplification Under Bursty Traffic**: High-frequency group chats (hundreds of messages/sec) cause immediate PostgreSQL connection pool saturation and lock contention. SyncSphere decouples ephemeral WebSocket broadcasts from permanent storage by ingesting messages into **Apache Kafka** and batching writes via background consumers.
3. **Cross-Instance State Desynchronization**: Multiple Node.js backend instances are unified into a cohesive cluster using **Redis Pub/Sub**, ensuring instant sub-10ms event distribution across independent server nodes.

---

## 🚀 Key Feature Matrix

### 1. WebRTC SFU Multi-Party Video & Voice Calling
- **Mediasoup C++ Worker Binding**: Operates a dedicated media router with dynamic transport allocation and spatial/temporal layer routing.
- **Spotlight Pinned View & Responsive Carousel**: Dynamically detects speaker activity or user-pinned streams, maximizing focus while arranging remote peers in a smooth, hardware-accelerated carousel.
- **Native MediaStream Rendering**: Bypasses heavy HTML5 `<canvas>` rendering loops. Audio and video tracks bind directly to native elements, preventing browser tab throttling in Chromium.
- **Screen Sharing**: Seamless negotiation of high-resolution screen capture streams alongside live webcam and audio tracks.

### 2. High-Throughput Real-Time Messaging
- **Dual-Channel Architecture**: Real-time broadcasts fire instantly over WebSocket (`socket.io`), while transactional durability is handled asynchronously via Kafka.
- **Redis Pub/Sub State Sync**: Distributes chat events across horizontally scaled Node.js servers with zero user-perceived latency.
- **Kafka Batch Persistence**: Writes batched database transactions, protecting the primary relational store from I/O bottlenecks.

### 3. $O(1)$ Scalable Unseen Message Tracking
- **Smart Cursor Engine**: Replaces traditional message-by-message receipt tables (which explode to millions of rows) with a `lastSeenAt` timestamp cursor and an atomic `unseenCount` incrementer per `Participant`.
- **Instant Badge Rendering**: Scanning a single indexed junction row renders accurate unread counts with zero query overhead.

### 4. Cloud Media Storage
- **Direct-to-S3 Pre-Signed Uploads**: Media attachments bypass the application server, streaming directly from the client to AWS S3 (or Cloudflare R2) via cryptographically signed URLs.

---

## 🧠 System Architecture

```mermaid
flowchart TD
    ClientA["Client A (Next.js 15)"]
    ClientB["Client B (Next.js 15)"]
    ClientC["Client C (Next.js 15)"]

    subgraph BackendGateway ["Node.js / Express Backend Layer"]
        API["Express REST API"]
        SocketServer["Socket.io Server"]
        Mediasoup["Mediasoup C++ SFU Worker"]
    end

    subgraph StateAndSync ["Volatile State & Distributed Pub/Sub"]
        Redis[("Redis 7 (Pub/Sub & Cache)")]
    end

    subgraph DurabilityLayer ["Asynchronous Persistence Pipeline"]
        Kafka["Apache Kafka (KRaft Mode)"]
        KafkaConsumer["Background Batch Consumer"]
        Postgres[("PostgreSQL 15 (Prisma ORM)")]
    end

    subgraph CloudStorage ["Object Storage"]
        S3[("AWS S3 / Cloudflare R2")]
    end

    %% Client Interactions
    ClientA <-->|WebSocket WSS| SocketServer
    ClientB <-->|WebSocket WSS| SocketServer
    ClientC <-->|WebSocket WSS| SocketServer

    ClientA <-->|RTP / RTCP UDP| Mediasoup
    ClientB <-->|RTP / RTCP UDP| Mediasoup
    ClientC <-->|RTP / RTCP UDP| Mediasoup

    ClientA -->|Pre-Signed HTTPS PUT| S3

    %% Backend Internals
    SocketServer <-->|Event Broadcast| Redis
    SocketServer -->|Produce Event| Kafka
    Kafka -->|Consume Batch| KafkaConsumer
    KafkaConsumer -->|Bulk Insert| Postgres
    API <-->|Read / Write| Postgres
```

---

## 📁 Repository Structure

```
SyncSphere/
├── backend/                        # Node.js & Express API Gateway + SFU
│   ├── prisma/
│   │   ├── migrations/             # Relational migration history
│   │   └── schema.prisma           # Relational schema (User, Chat, Participant, Message)
│   ├── src/
│   │   ├── config/                 # Typed environment configurations
│   │   ├── controllers/            # Auth, User, Chat, and Media controllers
│   │   ├── lib/                    # Prisma & Logger singletons
│   │   ├── mediasoup/              # SFU Worker, Router, and Transport logic
│   │   ├── middlewares/            # JWT verification & CORS
│   │   ├── routes/                 # Express route definitions
│   │   ├── services/               # Kafka producer & consumer batch handlers
│   │   ├── types/                  # Shared TypeScript interfaces
│   │   ├── utils/                  # S3 Pre-signed URL utilities & tokens
│   │   ├── index.ts                # Server entry point
│   │   └── socket.ts               # Socket.io connection & room management
│   ├── Dockerfile                  # Production container configuration
│   ├── docker-compose.yml          # Backend infrastructure services
│   └── package.json
│
├── frontend/                       # Next.js 15 App Router + Tailwind CSS
│   ├── public/                     # Static assets & icons
│   ├── src/
│   │   ├── app/                    # Layout, Page, and Route declarations
│   │   ├── components/             # Dynamic UI components (ChatBox, VideoGrid, etc.)
│   │   ├── context/                # Chat, Auth, and WebRTC Context Providers
│   │   ├── hooks/                  # Custom React hooks (useAuth, useWebRTC)
│   │   ├── lib/                    # Axios API client & styling utils
│   │   └── utils/                  # Client-side helper functions
│   └── package.json
│
├── docs/                           # In-depth architectural documentation
│   ├── architecture.md             # UML Component diagrams & scaling topology
│   ├── DATABASE_SCHEMA.md          # ER diagrams & unseen notification logic
│   ├── MESSAGING_AND_KAFKA.md      # Asynchronous queuing & event handling
│   └── WEBRTC_SFU_IMPLEMENTATION.md # Mediasoup state machine & signaling
│
├── docker-compose.yml              # Root infrastructure orchestration
├── CONTRIBUTING.md                 # Contribution guidelines
├── CODE_OF_CONDUCT.md              # Community standards
├── LICENSE                         # MIT License
└── README.md                       # Master project overview
```

---

## 🛠️ Tech Stack Breakdown

| Layer | Technologies | Purpose |
| :--- | :--- | :--- |
| **Frontend** | Next.js 15, React 19, TypeScript, Tailwind CSS v4, Framer Motion, Radix UI | Modern, high-performance responsive client interface |
| **Backend API** | Node.js, Express.js 5, TypeScript | RESTful Gateway & Authentication services |
| **Media Routing (SFU)** | Mediasoup v3, mediasoup-client | Low-latency WebRTC video & audio router ($O(N)$ scaling) |
| **Real-Time Layer** | Socket.io 4, ioredis | Bidirectional signaling & instant message transport |
| **Event Bus** | Apache Kafka (KRaft mode) | Asynchronous buffer for write amplification protection |
| **Relational Database**| PostgreSQL 15, Prisma ORM 6 | Strongly typed ACID persistence |
| **Cloud Storage** | AWS SDK S3 (Pre-Signed URLs) | Direct-to-bucket media file streaming |
| **Containerization** | Docker, Docker Compose | Infrastructure orchestration (Postgres, Redis, Kafka) |

---

## ⚡ Getting Started Locally

### Prerequisites
- **Node.js**: v18.0.0 or higher
- **Docker & Docker Compose**: Required for PostgreSQL, Redis, and Kafka
- **C++ Build Tools & Python 3**: Required on Windows for building native `mediasoup` worker binaries (`windows-build-tools` or Visual Studio C++ workload)

---

### Step 1: Clone the Repository
```bash
git clone https://github.com/<your-username>/SyncSphere.git
cd SyncSphere
```

---

### Step 2: Spin Up Infrastructure Containers
Start PostgreSQL, Redis, and Kafka via Docker Compose:
```bash
docker compose up -d
```

Verify that all containers are healthy:
```bash
docker compose ps
```
*Port mappings: PostgreSQL (`5433`), Redis (`6380`), Kafka (`9092` / `9093`).*

---

### Step 3: Configure Environment Variables

#### Backend (`backend/.env`):
```env
PORT=5000
NODE_ENV=development
FRONTEND_URL=http://localhost:3000

DATABASE_URL="postgresql://postgres:postgres@localhost:5433/syncsphere"
JWT_SECRET="syncsphere-super-secure-jwt-key-for-development-32chars"
REDIS_URL="redis://127.0.0.1:6380"
KAFKA_BROKER="127.0.0.1:9092"

# AWS S3 (Optional for local development)
AWS_ACCESS_KEY_ID=""
AWS_SECRET_ACCESS_KEY=""
AWS_REGION="us-east-1"
S3_BUCKET_NAME="syncsphere-uploads"

# Mediasoup Local Binding
ANNOUNCED_IP="127.0.0.1"
```

#### Frontend (`frontend/.env.local`):
```env
NEXT_PUBLIC_BACKEND_URL="http://127.0.0.1:5000"
NEXT_PUBLIC_BUCKET_NAME="syncsphere-uploads"
```

---

### Step 4: Setup & Run Backend

```bash
cd backend
npm install

# Run database migrations and generate Prisma Client
npx prisma generate
npx prisma db push

# Rebuild Mediasoup native bindings (Windows specific)
npm rebuild mediasoup

# Start dev server
npm run dev
```
Backend will be listening on `http://localhost:5000`.

---

### Step 5: Setup & Run Frontend

```bash
cd ../frontend
npm install
npm run dev
```
Open [http://localhost:3000](http://localhost:3000) in your browser.

---

## 📚 Deep Dive Architecture Documents

Explore the full theoretical specifications in the `docs/` folder:

1. **[System Architecture & UML Topologies](./docs/architecture.md)**: Horizontal scaling, load balancing with sticky sessions, and component boundaries.
2. **[Database Schema & ER Modeling](./docs/DATABASE_SCHEMA.md)**: Relational schema breakdown and the $O(1)$ unseen notification cursor design.
3. **[WebRTC SFU Implementation Guide](./docs/WEBRTC_SFU_IMPLEMENTATION.md)**: Transport signaling flows, ICE candidates, DTLS parameters, and Spotlight Pinned layout mechanics.
4. **[Messaging & Kafka Event Pipeline](./docs/MESSAGING_AND_KAFKA.md)**: Protecting PostgreSQL from write contention via Kafka batching.

---

## 🤝 Contributing

Contributions are welcomed! Please read [CONTRIBUTING.md](CONTRIBUTING.md) and adhere to our [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).

---

## 📄 License

This project is open-source and distributed under the [MIT License](LICENSE).
