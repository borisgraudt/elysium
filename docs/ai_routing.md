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

Routing score is calculated using adaptive learning with exponential moving average:

### Base Score
```
base_score = 0.3 * latency_score + 
             0.15 * uptime_score + 
             0.3 * reliability_score + 
             0.25 * route_success_rate
```

Where:
- `latency_score`: Normalized latency (lower = better, max 1s)
- `uptime_score`: Normalized uptime (higher = better, normalized to 1 hour)
- `reliability_score`: Ping success rate (0.0 to 1.0)
- `route_success_rate`: Historical success rate for forwarded messages

### Adaptive Learning

The final score uses exponential moving average to blend historical performance with current metrics:

```
final_score = α * historical_score + β * base_score
```

Where:
- `α = 0.7` (weight for historical performance)
- `β = 0.3` (weight for current metrics)
- `historical_score`: Based on average latency from route history

This allows the routing algorithm to:
- **Learn from experience**: Peers with good historical performance get higher scores
- **Adapt to changes**: Recent performance updates the score gradually
- **Balance stability and responsiveness**: 70/30 split prevents rapid oscillations

## Route Selection

AI-routing selects the top N peers (default: 3) based on their scores:

1. **Filter eligible peers**: Exclude sender, exclude nodes in path (loop prevention), only connected peers
2. **Score all peers**: Calculate score using adaptive learning formula
3. **Sort by score**: Descending order (best peers first)
4. **Select top N**: Forward message to top 3 peers (or all if less than 3)

### Route History Tracking

Each peer maintains route statistics:
- `success_count`: Number of successful message deliveries
- `failure_count`: Number of failed deliveries
- `total_latency`: Cumulative latency for route measurements
- `sample_count`: Number of route samples
- `last_updated`: Timestamp of last update

These statistics are used to calculate `route_success_rate` and inform the adaptive learning algorithm.

### Self-Learning Impact

The adaptive learning mechanism ensures:
- **Better peers get selected more often**: High-scoring peers are preferred
- **Network adapts to conditions**: Latency spikes or connection issues reduce scores
- **Stable routing**: Gradual score updates prevent route flapping
- **Continuous improvement**: Network performance improves over time as routing learns optimal paths

## Future: Federated Learning

Each node will:
1. Collect local routing statistics
2. Share anonymized metrics with peers
3. Learn optimal routing patterns
4. Adapt to network topology changes

