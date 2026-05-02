# Client-Server Communication Patterns

## REST 

REST is the standard "request-response" model where the client asks for something, the server sends it back, and the connection closes. It is stateless—meaning the server treats every request as an independent event and stores no session data—and relies on standard HTTP methods (GET, POST, PUT, DELETE).

## Request-Response (Short Polling)

In short polling, the client repeatedly sends requests to the server at fixed intervals (e.g., every 5 seconds) to check for new data.

Best for: Scenarios where real-time updates aren't critical and the server resources are low.

The Catch: It’s inefficient. Most requests return empty, wasting bandwidth and CPU.

## Long Polling

In long polling, the client sends a request, but the server **holds the request open** until new data is available or a timeout occurs.

Best for: Basic chat apps or notifications where you want "near" real-time updates without a complex setup.

The Catch: Keeping many connections "hanging" can be intensive for the server.

## Server-Sent Events (SSE)

SSE is a **one-way street**. The client establishes a long-term connection, and the server pushes data to the client whenever something happens.

Best for: Real-time dashboards, stock tickers, or social media feeds where the user doesn't need to "talk back" instantly.

The Catch: It is unidirectional. If the client needs to send data back, it has to use a separate REST request.

## WebSocket

WebSockets provide a **full-duplex (two-way)** communication channel over a single, long-lived connection.

How it works: It starts as an HTTP request ("Handshake") and upgrades to a WebSocket protocol. Both the client and server can send messages at any time.

Best for: High-frequency, low-latency applications like multiplayer gaming, collaborative editing (like Google Docs), or high-speed trading platforms.

The Catch: It's more complex to implement and requires servers that can handle thousands of persistent "open" connections.

### The "Zombie" Connection Problem

A Zombie Connection is a connection that stays "open" on a server after the client has already disconnected or crashed (could be due to Wifi/network drops), the Server may not receive a "Close" frame.

#### Solution (Ping Pong / Heartbeat): 

Ping: One party (usually the server) sends a "Ping" frame to the client.
Pong: The client is required by the WebSocket protocol to respond immediately with a "Pong" frame.
Termination: If the server doesn't receive a Pong within a specific timeframe, it assumes the connection is a "zombie" and closes it.

### The "Thundering Herd" Problem

The “Thundering Herd” problem in WebSockets is a performance issue that happens when a large number of clients react simultaneously to the same event, overwhelming your server or infrastructure.

#### Solution: 

Implement Exponential Backoff with Jitter (randomized delay) to prevent DDOSing your own infrastructure.
- Exponential Backoff: The waiting time grows exponentially, e.g., 1s, 2s, 4s, 8s.... 
- Jitter: A random value added to the backoff time (e.g., `1000ms + random(0–200ms)`) to stagger requests.
- Capping: Setting a maximum wait time (e.g., 60s) to avoid excessively long delays.

## SSE vs WebSocket

| Feature                | SSE (Server-Sent Events)          | WebSockets                           |
| ---------------------- | --------------------------------- | ------------------------------------ |
| Communication          | Unidirectional (Server → Client)  | Bidirectional (Full-Duplex)          |
| Protocol               | Standard HTTP                     | Binary Protocol (upgraded from HTTP) |
| Data Format            | Text-only (UTF-8)                 | Binary & Text                        |
| Automatic Reconnection | Built-in by default               | Must be implemented manually         |
| Firewall Friendly      | Yes (It's just standard HTTP)     | Sometimes blocked by strict proxies  |
| Max Connections        | Limited by browser (6 per domain) | Much higher limits                   |


## Lifecycle Comparison

| Action           | Standard REST | Short Polling   | Long Polling                | SSE           | WebSockets     |
| ---------------- | ------------- | --------------- | --------------------------- | ------------- | -------------- |
| TCP Handshake    | Every request | Every poll      | Every poll/event            | Once at start | Once at start  |
| HTTP Headers     | Every request | Every poll      | Every poll/event            | Once at start | Once (Upgrade) |
| Connection State | Closed        | Closed          | Closed after data / timeout | Kept Alive    | Kept Alive     |
| Data Flow        | Client Pull   | Client Pull     | Client Pull (delayed)       | Server Push   | Bi-directional |
| Ideal Wait Time  | None          | Fixed (e.g. 5s) | Variable (Hanging)          | Continuous    | Continuous     |

## Push Notifications

Push notifications allow a server to send data to a client application (mobile or desktop) even when the application is in the background or completely closed.

Unlike the other patterns, this usually requires a third-party intermediary (Push Service) to manage the delivery to the device.

How it works:

- Registration: The client app asks the OS (iOS/Android/Browser) for a unique "push token."
- Storage: The client sends this token to your server, which stores it in a database.
- Trigger: When an event occurs, your server sends a message + the token to a Push Service (like Firebase Cloud Messaging for Android or Apple Push Notification service for iOS).
- Delivery: The Push Service routes the message to the specific device.

Best for: User engagement, critical alerts (breaking news, transaction alerts), and messaging apps where the user isn't currently looking at the screen.

The Catch: Delivery is "best-effort" (not 100% guaranteed) and you are dependent on third-party infrastructure.

# Peer-to-peer (P2P) Communication Patterns

## WebRTC

WebRTC (Web Real-Time Communication) enables Peer-to-Peer (P2P) audio, video, and data exchange directly between browsers with sub-second latency.

The most common P2P pattern in the browser. Because peers sit behind NATs and firewalls, they cannot "find" each other automatically.

### 1. The Core APIs

- `getUserMedia`: Accesses the camera and microphone.
- `RTCPeerConnection`: Manages the connection, security, and streaming.
- `RTCDataChannel`: Sends non-media data (chat, files, game stats) directly.

### 2. Establishing a Connection

Since peers are usually hidden behind firewalls, they connect via a three-step process:

#### A. Signaling (The Introduction)

Peers use a Signaling Server (usually via WebSockets) to swap "business cards" called SDP (Session Description Protocol). They agree on:

- Media types (Video/Audio) and Codecs (VP9, H.264).
- Security keys and connection capabilities. 

#### B. ICE (The Negotiator)

ICE (Interactive Connectivity Establishment) is a framework used to find the best path between peers. It gathers "candidates" (potential connection paths) and tests them in order of efficiency.

#### C. NAT Traversal (The "Open Doors")

**Network Address Translation (NAT)** is a technique used by routers to map multiple private, local IP addresses to a single public IP address for internet access. 

To get through routers and firewalls, ICE uses two types of servers:

1. **STUN (Session Traversal Utilities for NAT)**: Tells you your Public IP address. It allows for a direct P2P connection. (Used 80% of the time).
   How it works: Your device sends a request to a STUN server. The server looks at the incoming packet and replies: "Hey, I see you're coming from Public IP 203.0.113.5 on Port 50001."
   
2. **TURN (Traversal Using Relays around NAT)**: Acts as a Relay. If a direct path is blocked, data flows through this server. It is a fallback and not true P2P.
   How it works: If STUN fails, the peers give up on a direct connection. Instead, they both connect to a TURN Server. User A sends their video to the TURN server, and the server "turns" (relays) it to User B.

### 3. Architecture Models

| Model                      | How it Works                                                         | Best For                          |
| -------------------------- | -------------------------------------------------------------------- | --------------------------------- |
| Mesh (Pure P2P)            | Every user sends their video to everyone else.                       | 1-on-1 calls, small groups        |
| SFU (Selective Forwarding) | Each user sends one stream to a server, which forwards it to others. | Zoom, Google Meet, large rooms    |
| MCU (Multipoint Control)   | The server mixes all videos into one single stream for you.          | Low-power devices, legacy systems |

### 4. Security & Performance

- Encryption: Mandatory and "always-on" using SRTP (Secure Real-time Transport Protocol) for media and Datagram Transport Layer Security (Data).
- Protocol: Primarily uses UDP (User Datagram Protocol) for speed (dropping lost packets is better than lagging in a live call).