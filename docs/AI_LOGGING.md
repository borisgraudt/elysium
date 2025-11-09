# AI Routing Logs - Обучение модели

## Где хранятся логи

Логи AI-routing сохраняются в файл **`logs/ai_routing_logs.jsonl`** (JSON Lines формат - одна JSON запись на строку).

Файл создается автоматически при запуске ноды в директории, откуда запускается нода.

## Формат логов

Каждая строка в файле - это JSON объект с информацией о решении маршрутизации:

```json
{
  "timestamp": "2025-11-09T12:34:56.789Z",
  "message_id": "abc-123-def",
  "node_id": "node-uuid",
  "from_peer": "peer-uuid",
  "selected_peers": [
    {
      "peer_id": "peer-1",
      "score": 0.85,
      "metrics": {
        "peer_id": "peer-1",
        "latency_ms": 12.5,
        "uptime_secs": 3600.0,
        "ping_count": 100,
        "ping_failures": 2,
        "reliability_score": 0.98,
        "is_connected": true
      }
    }
  ],
  "available_peers": [
    {
      "peer_id": "peer-1",
      "latency_ms": 12.5,
      "uptime_secs": 3600.0,
      "ping_count": 100,
      "ping_failures": 2,
      "reliability_score": 0.98,
      "is_connected": true
    },
    {
      "peer_id": "peer-2",
      "latency_ms": 25.0,
      "uptime_secs": 1800.0,
      "ping_count": 50,
      "ping_failures": 5,
      "reliability_score": 0.90,
      "is_connected": true
    }
  ],
  "message_context": {
    "ttl": 5,
    "path_length": 2,
    "is_broadcast": true,
    "target_peer": null
  }
}
```

## Что логируется

### selected_peers
Пиры, выбранные AI-routing для форвардинга сообщения, с их scores и метриками.

### available_peers
Все доступные пиры на момент принятия решения с их метриками. Это позволяет понять контекст решения.

### message_context
Контекст сообщения:
- `ttl` - оставшееся время жизни (hop count)
- `path_length` - длина пути (количество пройденных нод)
- `is_broadcast` - является ли сообщение broadcast
- `target_peer` - целевой пир (если не broadcast)

## Использование для обучения

### 1. Сбор данных

Запустите ноды и отправляйте сообщения:

```bash
# Запустить ноды
cargo run --bin core -- 8082
cargo run --bin core -- 8083 127.0.0.1:8082
cargo run --bin core -- 8084 127.0.0.1:8082

# Отправлять сообщения
MESHLINK_API_PORT=17082 cargo run --bin cli -- broadcast "Test message"
```

Логи будут сохраняться в `logs/ai_routing_logs.jsonl` для каждой ноды.

### 2. Парсинг логов

```python
import json

logs = []
with open('logs/ai_routing_logs.jsonl', 'r') as f:
    for line in f:
        logs.append(json.loads(line))
```

### 3. Подготовка данных для обучения

Каждая запись содержит:
- **Входные данные (features):**
  - Метрики всех доступных пиров (latency, uptime, reliability)
  - Контекст сообщения (TTL, path_length, is_broadcast)
  
- **Выходные данные (labels):**
  - Выбранные пиры и их scores
  - Можно использовать для supervised learning

### 4. Обучение модели

Пример структуры данных для обучения:

```python
# Features
X = []
for log in logs:
    features = []
    # Метрики всех пиров
    for peer in log['available_peers']:
        features.extend([
            peer['latency_ms'] or 0,
            peer['uptime_secs'],
            peer['reliability_score'],
            peer['ping_count'],
            peer['ping_failures']
        ])
    # Контекст сообщения
    features.extend([
        log['message_context']['ttl'],
        log['message_context']['path_length'],
        1.0 if log['message_context']['is_broadcast'] else 0.0
    ])
    X.append(features)

# Labels (scores выбранных пиров)
y = []
for log in logs:
    scores = [peer['score'] for peer in log['selected_peers']]
    y.append(scores)
```

## Анализ логов

### Статистика по решениям

```python
import json
from collections import defaultdict

logs = []
with open('logs/ai_routing_logs.jsonl', 'r') as f:
    for line in f:
        logs.append(json.loads(line))

# Средний score выбранных пиров
avg_scores = []
for log in logs:
    if log['selected_peers']:
        avg_scores.append(
            sum(p['score'] for p in log['selected_peers']) / len(log['selected_peers'])
        )

print(f"Average selected peer score: {sum(avg_scores) / len(avg_scores):.2f}")

# Распределение по количеству выбранных пиров
peer_counts = defaultdict(int)
for log in logs:
    peer_counts[len(log['selected_peers'])] += 1

print("Peer selection distribution:")
for count, freq in sorted(peer_counts.items()):
    print(f"  {count} peers: {freq} times")
```

## Рекомендации

1. **Собирайте данные в разных условиях:**
   - Разное количество пиров
   - Разная нагрузка сети
   - Разные метрики пиров

2. **Логируйте достаточно данных:**
   - Минимум 1000+ решений для базовой модели
   - 10000+ для более точной модели

3. **Валидация:**
   - Разделите данные на train/test (80/20)
   - Используйте cross-validation

4. **Метрики качества:**
   - Точность выбора лучших пиров
   - Средняя latency выбранных маршрутов
   - Успешность доставки сообщений

## Отключение логирования

Если логирование не нужно, можно закомментировать вызов `log_routing_decision` в `node.rs` (строка ~1162).

