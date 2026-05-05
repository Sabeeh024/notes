# Client-Server Communication Patterns

---

## 1. REST

**REST** is the standard **request-response model**:

- Client requests data
- Server responds
- Connection closes

**Key Characteristics:**

- Stateless (no session stored)
- Uses HTTP methods:
  - `GET`
  - `POST`
  - `PUT`
  - `DELETE`

---

## 2. Request-Response (Short Polling)

Client repeatedly sends requests at fixed intervals (e.g., every 5 seconds).

**Best For:**
- Non real-time systems
- Low server complexity

**The Catch:**
- Inefficient
- Most responses are empty → wasted bandwidth & CPU

---

## 3. Long Polling

Client sends request → server **holds it open** until:

- New data arrives OR
- Timeout occurs

**Best For:**
- Chat apps
- Notifications (near real-time)

**The Catch:**
- Many open connections → server strain

---

## 4. Server-Sent Events (SSE)

**One-way communication (Server → Client)**

- Long-lived connection
- Server pushes updates

**Best For:**
- Dashboards
- Stock tickers
- Social feeds

**The Catch:**
- Unidirectional
- Client must use separate REST calls to send data

---

## 5. WebSocket

**Full-duplex (two-way) communication** over a single persistent connection.

### How It Works

- Starts as HTTP request (**Handshake**)
- Upgrades to WebSocket protocol
- Both client & server can send messages anytime

**Best For:**
- Multiplayer games
- Collaborative tools (Google Docs-like)
- High-frequency trading apps

**The Catch:**
- Complex implementation
- Requires handling thousands of persistent connections

---

### 5.1 The "Zombie" Connection Problem

A **Zombie Connection** stays open after client disconnects (e.g., network drop).

#### Solution: Ping / Pong (Heartbeat)

- **Ping** → Sent by server
- **Pong** → Client must respond immediately
- **Termination** → No pong → connection closed

---

### 5.2 The "Thundering Herd" Problem

Occurs when many clients react simultaneously → overload.

#### Solution: Exponential Backoff + Jitter

- **Exponential Backoff**
  - `1s → 2s → 4s → 8s`

- **Jitter**
  - Random delay:
    - `1000ms + random(0–200ms)`

- **Capping**
  - Max wait time (e.g., `60s`)

---

## 6. SSE vs WebSocket

| Feature                | SSE (Server-Sent Events)         | WebSockets                          |
|-----------------------|----------------------------------|-------------------------------------|
| Communication         | Unidirectional (Server → Client) | Bidirectional (Full-Duplex)         |
| Protocol              | HTTP                             | Binary (Upgraded from HTTP)         |
| Data Format           | Text (UTF-8)                     | Text + Binary                       |
| Auto Reconnect        | Built-in                         | Manual                              |
| Firewall Friendly     | Yes                              | Sometimes blocked                   |
| Max Connections       | ~6 per domain                    | Much higher                         |

---

## 7. Lifecycle Comparison

| Action           | REST         | Short Polling | Long Polling          | SSE          | WebSockets     |
|------------------|--------------|---------------|-----------------------|--------------|----------------|
| TCP Handshake    | Every request| Every poll    | Every poll/event      | Once         | Once           |
| HTTP Headers     | Every request| Every poll    | Every poll/event      | Once         | Once (Upgrade) |
| Connection State | Closed       | Closed        | Closed after response | Kept Alive   | Kept Alive     |
| Data Flow        | Client Pull  | Client Pull   | Delayed Pull          | Server Push  | Bi-directional |
| Ideal Wait Time  | None         | Fixed         | Variable              | Continuous   | Continuous     |

---

## 8. Push Notifications

Allows server to send data even when app is **backgrounded or closed**.

**Requires third-party push services**

### How It Works

1. **Registration**
   - The client app asks the OS (iOS/Android/Browser) for a unique "push token."

2. **Storage**
   - The client sends this token to your server, which stores it in a database.

3. **Trigger**
   - Server sends message + token to push service

4. **Delivery**
   - Service routes to device

**Example Services:**
- Firebase Cloud Messaging (Android)
- Apple Push Notification Service (iOS)

**Best For:**
- Alerts
- Engagement
- Messaging apps

**The Catch:**
- Not guaranteed delivery
- Depends on third-party infra

---

# Peer-to-Peer (P2P) Communication Patterns

---

## 1. WebRTC

**WebRTC** enables real-time **P2P audio, video, and data** between browsers.

### Why It's Needed

- Peers are behind NAT/firewalls
- Cannot connect directly without assistance

---

### 1.1 Core APIs

- `getUserMedia` → Camera & mic access  
- `RTCPeerConnection` → Connection management  
- `RTCDataChannel` → Data transfer  

---

### 1.2 Establishing a Connection

#### A. Signaling (Introduction)

Peers exchange **SDP (Session Description Protocol)** via a signaling server.

They agree on:

- Media types (audio/video)
- Codecs (`VP9`, `H.264`)
- Security keys

---

#### B. ICE (Negotiation)

**ICE (Interactive Connectivity Establishment)**:

- Collects connection paths (**candidates**)
- Tests them for best route

---

#### C. NAT Traversal

**NAT (Network Address Translation)** maps local IPs → public IP.

##### STUN (Direct Connection)

- Reveals public IP
- Enables direct P2P (~80% cases)

**Flow:**
- Request → STUN server
- Response → "Your public IP is X"

---

##### TURN (Relay Fallback)

- Used when direct connection fails
- Acts as relay server

**Flow:**
- Both peers connect to TURN
- Data routed via server

---

### 1.3 Architecture Models

| Model        | How It Works                          | Best For                     |
|--------------|---------------------------------------|------------------------------|
| Mesh         | Everyone sends to everyone            | 1:1, small groups            |
| SFU          | Server forwards streams               | Zoom, Google Meet            |
| MCU          | Server mixes streams into one         | Low-power / legacy systems   |

---

### 1.4 Security & Performance

- **Encryption**
  - Always-on
  - Uses:
    - `SRTP`
    - `DTLS`

- **Protocol**
  - Uses `UDP`
  - Drops packets instead of delaying
