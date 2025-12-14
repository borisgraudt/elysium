# AI Routing Algorithm

## Overview

MeshNet uses AI-driven routing to select optimal paths for message delivery based on real-time network conditions.

## Metrics Collection

The `StatsCollector` tracks:
- **Latency**: Average round-trip time for ping/pong
- **Uptime**: How long a peer has been connected
- **Packet Loss**: Percentage of lost messages
- **Trust Score**: Reliability metric based on error rate

## Scoring Function

Routing score is calculated as:
```
score = 0.3 * latency_score + 0.2 * uptime_score + 0.5 * reliability_score
```

Where:
- `latency_score`: Normalized latency (lower = better)
- `uptime_score`: Normalized uptime (higher = better)
- `reliability_score`: Combination of trust, packet loss, and error rate

## Route Selection

Currently uses flooding-based routing with AI scoring for peer selection. Future versions will implement:
- Multi-hop routing with path scoring
- Federated learning for global optimization
- Predictive routing based on historical data

## Future: Federated Learning

Each node will:
1. Collect local routing statistics
2. Share anonymized metrics with peers
3. Learn optimal routing patterns
4. Adapt to network topology changes

