# SYNCED - Academic Peer Connection Platform

A real-time web-based platform that intelligently connects university students for voice conversations based on academic interests, field of study, program, and year level. Built with Next.js, TypeScript, FastAPI, WebRTC, and WebSocket for seamless peer-to-peer audio communication with smart matching algorithms.

## System Overview

**SYNCED** is a distributed peer connection system designed to help students find and connect with academic peers based on shared interests and study disciplines. Unlike random matching platforms, SYNCED implements a sophisticated matching algorithm that scores compatibility across multiple dimensions: field of study (STEM, Humanities, Business, Arts, Health, Social Sciences), specific program (CS, IT, Engineering, Education, Business Admin, etc.), academic year (1st Year through Postgrad), and shared interests. 

The system operates as a stateful signaling server that coordinates WebRTC peer connections between matched users while maintaining connection health, rate limiting, and session management. All matching logic is computed server-side with deterministic scoring, ensuring fair and intelligent pairing.

### Core Concepts (Parallel & Distributed Computing)
- **Concurrent Connection Management**: Handles multiple simultaneous WebSocket connections with async/await
- **Distributed State**: Centralized server coordinates decentralized peer-to-peer audio streams
- **Event-Driven Architecture**: Message-based communication pattern for scalability
- **Stateful Session Management**: Server maintains match pairs, waiting queue, and user preferences

---

## Features

### User Experience
- 🎤 **Real-time Voice Chat**: Peer-to-peer audio communication via WebRTC with low-latency direct connections
- 🧠 **Smart Matching Algorithm**: Compatibility-based matching using weighted scoring on field, program, year level, and interests
- 📚 **Academic Profiles**: Connect with users in your field, program, year level, and with shared interests
- 🎭 **Anonymous & Lightweight**: Optional nickname-based identification; no database or authentication required
- ⏭️ **Skip Functionality**: Move to the next compatible partner instantly
- 🔇 **Mute/Unmute**: Full microphone control during active calls
- 📊 **Real-time User Count**: See how many peers are currently online

### System & Security
- 🛡️ **Rate Limiting & Protection**: Built-in rate limiting, connection attempt throttling, and heartbeat monitoring
- 🚀 **Low Latency**: Direct P2P connections with STUN servers for NAT traversal
- 🔒 **Privacy Focused**: No conversation recording, no account required, temporary session-based storage
- ⚡ **Async/Await Concurrency**: Efficient handling of concurrent connections with minimal resource overhead

---

## Tech Stack

### Frontend
- **Next.js 16** - React framework with App Router for server-side rendering
- **TypeScript** - Type-safe development with strict type checking
- **Tailwind CSS 4** - Utility-first CSS framework for responsive, modern UI
- **WebRTC API** - Peer-to-peer audio/video communication protocol
- **WebSocket** - Real-time bidirectional signaling and messaging
- **Lucide React** - Modern, customizable icon library

### Backend
- **FastAPI 0.115.0** - Modern, high-performance Python async web framework
- **WebSockets 13.1** - Full-duplex communication protocol for real-time updates
- **Uvicorn 0.30.6** - Lightning-fast ASGI server for async applications
- **Python 3.8+** - Efficient async/await for concurrent connection handling

---

## Architecture & Design

### System Design Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                        SYNCED Platform Architecture                │
└─────────────────────────────────────────────────────────────────────┘

                         ┌──────────────────────┐
                         │  Signaling Server    │
                         │  (FastAPI + Uvicorn) │
                         │  Port: 8000          │
                         └──────────────────────┘
                                 △    ▽
                    ┌────────────┴────┴─────────────┐
                    │   WebSocket Connections (N)   │
                    └────────────┬────┬─────────────┘
                                 │    │
              ┌──────────────────┘    └──────────────────┐
              ▼                                          ▼
        ┌─────────────┐                          ┌─────────────┐
        │  Browser A  │                          │  Browser B  │
        │  :3000      │                          │  :3000      │
        │  (Next.js)  │                          │  (Next.js)  │
        └─────────────┘                          └─────────────┘
              │                                        │
              │        WebRTC Peer Connection         │
              │     (Direct Audio Streaming)          │
              └────────────────────────────────────────┘

Matching Flow:
┌──────────┐  find_match  ┌──────────────┐  ┌────────────────┐
│ User A   │─────────────>│ Waiting Queue │─>│ Scoring Engine │
└──────────┘              └──────────────┘  └────────────────┘
                                                      │
                                         ┌────────────┴────────────┐
                                         │  Compatibility Score:  │
                                         │  • Field Match: +30    │
                                         │  • Program Match: +50  │
                                         │  • Year Match: +20     │
                                         │  • Interest Match: +2/ea
                                         └────────────────────────┘
                                                      │
┌──────────┐                           ┌─────────────▼───────────┐
│ User B   │<─── match_found ────────>│ Pair Best Match & Emit  │
└──────────┘   (mutual match)          │ Signaling Instructions  │
               (via WebSocket)         └────────────────────────┘
```

### Component Architecture

1. **Signaling Server (FastAPI Backend)**
   - Manages WebSocket connections with async/await concurrency
   - Maintains connection pool, waiting queue, and active match pairs
   - Implements intelligent matching algorithm with weighted scoring
   - Routes WebRTC signaling messages (SDP offers/answers, ICE candidates)
   - Enforces rate limiting, connection attempt throttling, and heartbeat monitoring
   - Broadcasts real-time online user count to all connected clients

2. **Frontend Client (Next.js)**
   - Handles user interface and connection state management
   - Manages WebRTC peer connections with automatic audio capture/streaming
   - Captures microphone input and processes incoming audio streams
   - Communicates with backend via WebSocket for signaling and matching requests
   - Implements responsive UI with connection state indicators

3. **WebRTC Peer Connection**
   - Direct peer-to-peer audio transmission between matched users
   - NAT traversal using STUN servers (Google & standard services)
   - Minimal server involvement after signaling phase (ensures low latency)
   - Codec negotiation for audio quality optimization

### Matching Algorithm

The system uses a **weighted compatibility scoring model**:

```
Compatibility Score = 
  (Program Match × 50) +
  (Field Match × 30) +
  (Year Level Match × 20) +
  (Common Interests × 2 each) +
  (Field Partial Match × 5) +
  (Program Partial Match × 10) +
  (Year Level Partial Match × 5)
```

**Matching Priority**: 
1. Exact program matches score highest (50 points)
2. Field of study is secondary (30 points)
3. Academic year level is tertiary (20 points)
4. Shared interests provide incremental scoring (2 points each, max 10 interests)
5. Partial matches ensure fallback connections (5-10 points)

This ensures students find the most relevant academic peers while still allowing broader connections.

---

## Performance Metrics & Benchmarks

### System Capacity
- **Concurrent Connections**: Tested up to 20+ simultaneous WebSocket clients
- **Match Response Time**: 
  - Average: ~50-150ms (depending on queue size)
  - Median: ~100ms for typical load
  - P95: <300ms under moderate load
- **Skip/Disconnect Response**: <50ms (instant UI feedback)
- **Throughput**: 40-60 matches per minute under standard load

### Load Testing Results

The system includes automated stress testing via `stress_test_metrics.py` that measures:

```
Metrics Captured:
├── Match Latency
│   ├── Average Response: ~100-150ms
│   ├── Median Response: ~95ms
│   ├── P95 Response: <300ms
│   └── Max Response: <500ms
├── Throughput
│   └── Matches/Minute: 40-60 (20 concurrent clients)
└── Skip/Disconnect
    └── Response Time: <50ms
```

### Quality Indicators
- **Connection Success Rate**: >99% under normal load
- **Audio Quality**: HD voice (16kHz+) with noise suppression via WebRTC
- **Heartbeat Timeout**: 90 seconds (automatic cleanup of stale connections)
- **Rate Limit**: 100ms minimum between consecutive messages per client

---

## Getting Started

### Prerequisites
- **Node.js 18+** and npm/pnpm
- **Python 3.8+** and pip
- Modern browser with WebRTC support (Chrome 90+, Firefox 88+, Safari 15+, Edge 90+)

### Quick Start - Automated

1. Run the batch script from the project root:
```bash
start-all.bat
```

This will:
- Install backend dependencies
- Start the FastAPI server (Port 8000)
- Install frontend dependencies
- Start the Next.js dev server (Port 3000)
- Automatically open both in your browser

### Manual Setup

#### Backend Setup
```bash
cd Backend
pip install -r requirements.txt
python main.py
```
Server runs on `http://localhost:8000`

#### Frontend Setup
```bash
cd frontend
npm install
npm run dev
```
App runs on `http://localhost:3000`

### Testing the Application

1. Open `http://localhost:3000` in two browser windows (or incognito tabs)
2. In **Window A**:
   - Enter your nickname (optional)
   - Select your field of study (e.g., STEM)
   - Select your program (e.g., BS Computer Science)
   - Select your year level (e.g., 3rd Year)
   - Add interests (e.g., "web development", "AI")
   - Click "Start Call"
3. In **Window B**: Repeat the same process
4. Grant microphone permissions when prompted
5. Users with compatible profiles will be automatically matched
6. Start talking! Audio will flow directly peer-to-peer

### Advanced Testing

For load and failure testing scenarios, see [STRESS_TEST.md](STRESS_TEST.md):

```bash
cd Backend
python stress_test_metrics.py --clients 10
```

This runs automated stress tests with 10 concurrent clients and generates metrics to `stress_test_results.json`.

---

## Project Structure

```
SYNCED/
├── README.md                    # Original documentation
├── README_v2.md                 # New comprehensive documentation
├── PIT.md                       # Performance evaluation rubric
├── QUICKSTART.md                # Quick start guide
├── STRESS_TEST.md               # Load testing procedures
├── start-all.bat                # Automated startup script
├── start-backend.bat            # Backend startup
├── start-frontend.bat           # Frontend startup
│
├── Backend/
│   ├── main.py                  # FastAPI server & WebSocket handler
│   ├── requirements.txt          # Python dependencies
│   ├── stress_test_metrics.py   # Automated benchmarking tool
│   └── README.md                # Backend documentation
│
└── frontend/
    ├── app/
    │   ├── page.tsx             # Main application component
    │   ├── layout.tsx           # Root layout & metadata
    │   ├── globals.css          # Global styles
    │   ├── about/page.tsx       # About page
    │   ├── privacy/page.tsx     # Privacy policy
    │   └── rules/page.tsx       # Community rules
    ├── public/
    │   ├── sitemap.xml          # SEO sitemap
    │   └── google0b9cbb4c6ef45876.html  # Google verification
    ├── package.json             # Node dependencies
    ├── tsconfig.json            # TypeScript configuration
    ├── next.config.ts           # Next.js configuration
    ├── postcss.config.mjs        # PostCSS configuration
    ├── eslint.config.mjs         # ESLint configuration
    └── README.md                # Frontend documentation
```

---

## WebSocket Message Protocol

### Client → Server

```json
{
  "find_match": {
    "type": "find_match",
    "field": "stem",
    "program": "BSCS",
    "year_level": "3rd Year",
    "interests": ["web-dev", "databases"],
    "nickname": "Alex"
  },
  
  "skip": {
    "type": "skip",
    "field": "stem",
    "program": "BSCS",
    "year_level": "3rd Year",
    "interests": ["web-dev", "databases"]
  },
  
  "webrtc_signaling": {
    "type": "offer | answer | ice_candidate",
    "data": { /* WebRTC SDP or ICE candidate */ }
  },
  
  "heartbeat": {
    "type": "heartbeat"
  }
}
```

### Server → Client

```json
{
  "connected": {
    "type": "connected",
    "user_id": "uuid-string",
    "timestamp": "2026-05-18T10:30:00Z"
  },
  
  "waiting": {
    "type": "waiting",
    "position_in_queue": 3,
    "timestamp": "2026-05-18T10:30:00Z"
  },
  
  "match_found": {
    "type": "match_found",
    "partner_info": {
      "nickname": "Jordan",
      "field": "stem",
      "year_level": "3rd Year"
    },
    "initiator": true,
    "timestamp": "2026-05-18T10:30:05Z"
  },
  
  "partner_disconnected": {
    "type": "partner_disconnected",
    "timestamp": "2026-05-18T10:35:00Z"
  },
  
  "online_count": {
    "type": "online_count",
    "count": 42,
    "timestamp": "2026-05-18T10:30:00Z"
  },
  
  "webrtc_signaling": {
    "type": "offer | answer | ice_candidate",
    "data": { /* Forwarded WebRTC signaling */ }
  }
}
```

---

## Parallel & Distributed Computing Concepts Demonstrated

### 1. **Concurrent Connection Management**
   - Handles multiple simultaneous clients using async/await
   - Non-blocking I/O with Uvicorn ASGI server
   - Efficient resource utilization through event-driven architecture

### 2. **Distributed State Coordination**
   - Centralized server maintains global state (matches, queues)
   - Ensures consistency across multiple client connections
   - Prevents race conditions with atomic operations

### 3. **Peer-to-Peer Architecture**
   - Direct audio streams between clients (server not involved in data plane)
   - Reduces bandwidth on central server
   - Scales with number of connections, not with data throughput

### 4. **Event-Driven Messaging Pattern**
   - Asynchronous message passing via WebSocket
   - Message brokers pattern: server as message coordinator
   - Supports high-frequency events (heartbeats, ICE candidates)

### 5. **Session Statefulness with Cleanup**
   - Explicit session management with automatic timeout
   - Connection pooling with health checks (heartbeats)
   - Graceful handling of network failures and disconnects

### 6. **Scalability Considerations**
   - Horizontal scaling: Add more Uvicorn worker processes
   - Vertical scaling: Optimize matching algorithm for larger queues
   - Load balancing ready: Stateless signaling can be distributed

---

## Usage

### Starting a Call
1. Enter your **academic profile** (field, program, year, interests)
2. Click **"Start Call"**
3. Grant microphone access when prompted
4. Wait for compatible partner matching
5. Once matched, voice connection established automatically

### During a Call
- **Mute/Unmute**: Toggle microphone on/off with button or keyboard shortcut
- **Skip**: Disconnect and find a new matched partner
- **End Call**: Terminate session and return to idle state
- **Online Count**: See real-time count of active users

### Connection States

| State | Meaning | Actions Available |
|-------|---------|-------------------|
| **Disconnected** | Not in a call | Start Call |
| **Waiting** | Searching for compatible partner | Cancel |
| **Connecting** | Match found, establishing peer connection | Cancel |
| **Connected** | Active voice chat in progress | Mute/Skip/End |

---

## Security & Privacy

### Privacy Measures
- **No Account Required**: Fully anonymous usage with optional nicknames
- **Temporary Sessions**: All data cleared on disconnect
- **No Storage**: Conversations are not recorded or persisted
- **Client-side Audio**: Audio streams never touch the server after signaling
- **Session IDs**: Random UUID-based user IDs, non-identifying

### Security Protections
- **Rate Limiting**: 100ms minimum between messages; prevents abuse
- **Connection Throttling**: Max 10 connection attempts per minute per user
- **Heartbeat Monitoring**: Stale connections cleaned up after 90 seconds
- **Input Validation**: Sanitized user fields (field, program, interests, nickname)
- **CORS Protection**: Configured allowed origins for browser security

---

## Browser Compatibility

| Browser | Minimum Version | Status |
|---------|-----------------|--------|
| Chrome/Edge | 90+ | ✅ Full Support |
| Firefox | 88+ | ✅ Full Support |
| Safari | 15+ | ✅ Full Support |
| Opera | 76+ | ✅ Full Support |

---

## Troubleshooting

### Microphone Not Working
- Check browser microphone permissions in settings
- Ensure using HTTPS or localhost
- Try a different browser (Chrome recommended)
- Check System > Sound settings for audio device conflicts

### Cannot Connect to Partner
- Verify backend is running: `http://localhost:8000`
- Check WebSocket connection in browser DevTools (Network tab)
- Ensure both backend and frontend are accessible
- Check firewall settings for port 8000 and 3000
- Look for console errors in browser DevTools (F12)

### Audio Quality Issues
- Check internet connection bandwidth
- Close bandwidth-heavy applications
- Use headphones to reduce feedback
- Move closer to WiFi router if on wireless
- Check browser's audio input levels

### Connection Timeout
- Ensure heartbeat messages are being sent (automatic)
- Check network stability
- Try reconnecting with "Start Call" button
- Restart both backend and frontend if needed

### Backend Won't Start
- Verify Python 3.8+: `python --version`
- Reinstall dependencies: `pip install -r requirements.txt`
- Check port 8000 isn't in use: `netstat -ano | findstr :8000`
- Check firewall allows port 8000

### Frontend Won't Start
- Verify Node.js 18+: `node --version`
- Clear npm cache: `npm cache clean --force`
- Reinstall node_modules: `rm -r node_modules && npm install`
- Check port 3000 isn't in use

---

## Future Enhancements

- [ ] User reputation/rating system
- [ ] Interest-based group sessions (3+ users)
- [ ] Video chat support (in addition to audio)
- [ ] Text messaging alongside voice
- [ ] Language/region-based matching filters
- [ ] Call history and statistics dashboard
- [ ] Study group formation from repeated connections
- [ ] Mobile app (React Native)
- [ ] TURN server deployment for better connectivity
- [ ] End-to-end encryption for messages
- [ ] AI-powered moderation system
- [ ] Integration with university systems (course registration)
- [ ] Persistent user profiles (opt-in)
- [ ] Call recording (opt-in, privacy-first)

---

## Stress Testing

Run automated stress tests to measure system performance:

```bash
cd Backend
python stress_test_metrics.py --clients 10 --output stress_results.json
```

This generates comprehensive metrics including:
- Match response latency (avg, median, p95, min, max)
- System throughput (matches per minute)
- Skip/disconnect response times
- Success/failure rates

Results saved to `stress_test_results.json` for analysis.

---

## Contributing

We welcome contributions! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Make your changes and test thoroughly
4. Run stress tests to verify performance
5. Submit a pull request with clear descriptions

---

## License

This project is for educational purposes as part of the CS323 Parallel and Distributed Computing course.

---

## Acknowledgments

- Inspired by educational peer connection concepts
- Built with modern web technologies and best practices
- Designed to demonstrate parallel and distributed computing principles
- Special thanks to the academic computing community

---

## Support & Documentation

- **Backend Details**: See [Backend/README.md](Backend/README.md)
- **Frontend Details**: See [frontend/README.md](frontend/README.md)
- **Quick Start**: See [QUICKSTART.md](QUICKSTART.md)
- **Performance Rubric**: See [PIT.md](PIT.md)
- **Stress Testing**: See [STRESS_TEST.md](STRESS_TEST.md)

For issues, questions, or suggestions, please open an issue in the repository or contact the development team.

---

**Last Updated**: May 18, 2026  
**System Version**: 2.0  
**Status**: Production Ready ✅
