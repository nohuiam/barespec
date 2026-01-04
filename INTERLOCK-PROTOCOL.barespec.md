# InterLock Protocol Barespec

**Protocol:** BaNano Binary Format
**Version:** 1.0 (0x0100)
**Updated:** 2026-01-04
**Status:** Standardized across all Cognitive Architecture servers

---

## Overview

InterLock is the UDP mesh communication protocol for BOP/Imminence OS servers. All servers use the official BaNano 12-byte binary header format for cross-server signal exchange.

---

## Wire Format

### 12-Byte Header (Fixed)

```
Bytes 0-1:   Signal Type    (uint16, big-endian)
Bytes 2-3:   Protocol Version (uint16, big-endian) = 0x0100
Bytes 4-7:   Payload Length (uint32, big-endian)
Bytes 8-11:  Timestamp      (uint32, Unix seconds)
Bytes 12+:   Payload        (JSON, UTF-8)
```

### Payload Format

```json
{
  "sender": "server-name",
  "key1": "value1",
  "key2": "value2"
}
```

**Required field:** `sender` (string) - identifies the originating server

---

## TypeScript Interface

```typescript
interface Signal {
  signalType: number;    // uint16 signal code
  version: number;       // Protocol version (0x0100 = v1.0)
  timestamp: number;     // Unix seconds
  payload: {
    sender: string;      // Server ID (required)
    [key: string]: unknown;  // Signal-specific data
  };
}
```

---

## Encode/Decode Functions

### encode(signalType, sender, data?) → Buffer

```typescript
function encode(signalType: number, sender: string, data?: Record<string, unknown>): Buffer {
  const payload = JSON.stringify({ sender, ...data });
  const payloadBuffer = Buffer.from(payload, 'utf8');

  const header = Buffer.alloc(12);
  header.writeUInt16BE(signalType, 0);
  header.writeUInt16BE(0x0100, 2);  // Protocol version
  header.writeUInt32BE(payloadBuffer.length, 4);
  header.writeUInt32BE(Math.floor(Date.now() / 1000), 8);

  return Buffer.concat([header, payloadBuffer]);
}
```

### decode(buffer) → Signal | null

```typescript
function decode(buffer: Buffer): Signal | null {
  if (!buffer || buffer.length < 12) return null;

  try {
    const signalType = buffer.readUInt16BE(0);
    const version = buffer.readUInt16BE(2);
    const payloadLength = buffer.readUInt32BE(4);
    const timestamp = buffer.readUInt32BE(8);

    if (buffer.length < 12 + payloadLength) return null;

    const payload = JSON.parse(
      buffer.slice(12, 12 + payloadLength).toString('utf8')
    );

    if (!payload.sender) payload.sender = 'unknown';

    return { signalType, version, timestamp, payload };
  } catch {
    return null;
  }
}
```

**Error handling:** Returns `null` for invalid/incompatible signals (silent ignore)

---

## Standard Signal Types

### Core Signals (All Servers)

| Signal | Code | Description |
|--------|------|-------------|
| HEARTBEAT | 0x00 | Keep-alive ping |
| ACK | 0x01 | Acknowledgment |
| DISCOVERY | 0x04 | Peer discovery |
| DISCOVERY_ACK | 0x05 | Discovery response |
| SHUTDOWN | 0xFF | Server stopping |

### Verification Signals (verifier-mcp)

| Signal | Code | Description |
|--------|------|-------------|
| VERIFICATION_REQUEST | 0xD0 | Request claim verification |
| VERIFICATION_RESULT | 0xD1 | Verification complete |
| CLAIM_VERIFIED | 0xD2 | Claim supported |
| CLAIM_REFUTED | 0xD3 | Claim contradicted |

### Experience Signals (experience-layer)

| Signal | Code | Description |
|--------|------|-------------|
| EXPERIENCE_RECORDED | 0xF0 | Episode stored |
| PATTERN_EMERGED | 0xF1 | Pattern detected |
| LESSON_EXTRACTED | 0xF2 | Lesson created |
| LESSON_VALIDATED | 0xF3 | Lesson confidence updated |

### Consciousness Signals (consciousness-mcp)

| Signal | Code | Description |
|--------|------|-------------|
| DOCK_REQUEST | 0xC0 | Request to dock consciousness |
| DOCK_APPROVED | 0xC1 | Docking approved |
| DOCK_DENIED | 0xC2 | Docking denied |
| UNDOCK | 0xC3 | Release dock |

### Tenets Signals (tenets-server)

| Signal | Code | Description |
|--------|------|-------------|
| TENET_VIOLATION | 0xB0 | Ethical violation |
| COUNTERFEIT_DETECTED | 0xB1 | Counterfeit pattern |
| ETHICS_AFFIRMED | 0xB2 | Decision approved |
| BLIND_SPOT_ALERT | 0xB3 | Gap identified |

### Percolation Signals (percolation-server)

| Signal | Code | Description |
|--------|------|-------------|
| PERCOLATION_STARTED | 0x40 | Blueprint analysis started |
| HOLE_FOUND | 0x41 | Hole detected |
| HOLE_PATCHED | 0x42 | Hole fixed |
| PERCOLATION_COMPLETE | 0x43 | Analysis complete |
| PERCOLATION_FAILED | 0x44 | Analysis failed |

### Skill Signals (skill-builder)

| Signal | Code | Description |
|--------|------|-------------|
| SKILL_CREATED | 0xA0 | New skill created |
| SKILL_UPDATED | 0xA1 | Skill modified |
| SKILL_EXECUTED | 0xA2 | Skill run |
| SKILL_FAILED | 0xA3 | Execution failed |

### Build Signals (neurogenesis-engine)

| Signal | Code | Description |
|--------|------|-------------|
| BUILD_COMPLETED | 0xB0 | Build finished |
| BUILD_FAILED | 0xB5 | Build error |

### Lesson Signals (consciousness-mcp)

| Signal | Code | Description |
|--------|------|-------------|
| LESSON_LEARNED | 0xE5 | New lesson from consciousness |

---

## Tumbler (Whitelist Filter)

Each server has a tumbler that filters incoming signals:

```typescript
function isSignalAllowed(signal: Signal): boolean {
  const signalName = getSignalName(signal.signalType);
  return whitelist.includes(signalName);
}
```

**Behavior:**
- Accepted signals → routed to handlers
- Rejected signals → silently dropped
- Invalid signals → silently dropped (decode returns null)

---

## Server Implementation Files

```
src/interlock/
├── protocol.ts    # encode/decode/SignalTypes
├── socket.ts      # UDP socket, send/broadcast
├── handlers.ts    # Signal routing
├── tumbler.ts     # Whitelist filtering
└── index.ts       # Module exports
```

---

## Configuration (interlock.json)

```json
{
  "server": {
    "name": "server-name",
    "id": "server-name-001",
    "version": "1.0.0"
  },
  "ports": {
    "udp": 30XX,
    "http": 80XX,
    "websocket": 90XX
  },
  "peers": [
    { "name": "peer-name", "port": 30YY }
  ],
  "tumbler": {
    "mode": "whitelist",
    "whitelist": ["HEARTBEAT", "SIGNAL_NAME"]
  }
}
```

---

## Cognitive Architecture Servers

All 6 Cognitive Architecture servers use identical BaNano protocol:

| Server | UDP Port | Repository |
|--------|----------|------------|
| tenets-server | 3027 | nohuiam/tenets-server |
| consciousness-mcp | 3028 | nohuiam/consciousness-mcp |
| skill-builder | 3029 | nohuiam/skill-builder |
| percolation-server | 3030 | nohuiam/percolation-server |
| experience-layer | 3031 | nohuiam/experience-layer |
| verifier-mcp | 3021 | nohuiam/verifier-mcp |

---

## Mesh Communication

**Transport:** UDP broadcast
**Heartbeat:** Every 30 seconds
**Timeout:** 90 seconds (3x heartbeat)

**Statistics endpoint:** `GET /api/interlock/stats`

```json
{
  "socket": {
    "sent": 100,
    "received": 50,
    "dropped": 10,
    "peers": 6,
    "uptime": 3600
  },
  "tumbler": {
    "accepted": 45,
    "rejected": 5,
    "whitelist": ["HEARTBEAT", "..."]
  },
  "peers": [
    { "name": "server", "port": 3028, "status": "active", "lastSeen": 1704000000 }
  ]
}
```

---

## Protocol History

| Date | Change |
|------|--------|
| 2026-01-04 | Standardized all 6 Cognitive servers to BaNano 12-byte header |
| Pre-2026 | Servers used incompatible formats (JSON, various binary) |

---

## Why BaNano Over JSON

1. **Compact header** - 12 bytes fixed vs variable JSON envelope
2. **Protocol versioning** - Built-in version field for compatibility
3. **Fast parsing** - Binary header extraction before JSON parse
4. **Standard format** - All servers use identical encode/decode
