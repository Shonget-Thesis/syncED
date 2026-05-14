# Call Me Backend - Anonymous Voice Chat Signaling Server

FastAPI-based WebSocket signaling server for the Call Me anonymous voice chat platform.

## Features

- Real-time WebSocket connections for signaling
- Random user matching algorithm
- Session management without database
- WebRTC signaling (SDP offer/answer, ICE candidates)
- Skip functionality to find new partners
- CORS configured for Next.js frontend

---

## Installation

### 1. Install Python

Install Python 3.8 or higher.

Check if Python is installed:

```bash
python --version
```

or

```bash
python3 --version
```

---

### 2. Create Virtual Environment (venv)

#### Windows (PowerShell)

```bash
python -m venv venv
```

Activate the virtual environment:

```bash
venv\Scripts\activate
```

#### Windows (CMD)

```bash
venv\Scripts\activate.bat
```

#### macOS / Linux

```bash
python3 -m venv venv
```

Activate the virtual environment:

```bash
source venv/bin/activate
```

When activated, your terminal should display:

```bash
(venv)
```

---

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

---

### 4. Deactivate Virtual Environment

After development:

```bash
deactivate
```

---

## Running the Server

### Run Normally

```bash
python main.py
```

### Run Using Uvicorn

```bash
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

The server will start on:

```txt
http://localhost:8000
```

---

## API Endpoints

### HTTP

- `GET /` — Server status and statistics

### WebSocket

- `WS /ws/{user_id}` — WebSocket connection for signaling

---

## WebSocket Message Types

### Client → Server

- `find_match` — Request to find a random partner
- `skip` — Skip current partner and find a new one
- `offer` — WebRTC SDP offer
- `answer` — WebRTC SDP answer
- `ice_candidate` — ICE candidate for connection setup

### Server → Client

- `connected` — Connection established
- `waiting` — Added to waiting queue
- `match_found` — Partner found
- `partner_disconnected` — Current partner left
- `offer` / `answer` / `ice_candidate` — Forwarded signaling messages

---

## Architecture

The server uses an in-memory connection manager that handles:

- Active WebSocket connections
- Waiting queue for users seeking partners
- Current matches between users
- Message routing between matched pairs

---

## Stress Testing

Run stress tests against your backend to measure performance metrics like match latency, throughput, and skip response times.

### Local Stress Test

```bash
python stress_test_metrics.py --clients 10 30 60
```

### Deployed URL Stress Test

Test the deployed instance at:

```txt
https://iqueue-gj4a.onrender.com/
```

Run:

```bash
python stress_test_metrics.py --backend-url wss://iqueue-gj4a.onrender.com/ws --clients 10 30 60
```

This generates a `stress_test_results.json` file containing:

- Average response time
- Median response time
- P95 latency
- Match throughput (matches per minute)
- Partner disconnect latency
- Success and failure rates

> **Note:** Free Render instances may experience spin-down delays. Using a health check endpoint or external pinger service is recommended before running stress tests.

---

## Development

The server runs without a database, using temporary session identifiers to manage anonymous users.

All state is stored in memory and is cleared whenever the server restarts.