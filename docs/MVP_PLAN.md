# MVP План - MeshLink до 1 января

## ✅ Выполнено

- [x] Core P2P протокол
- [x] Rust CLI
- [x] Python CLI (базовый)
- [x] AI-routing (базовый)
- [x] Тесты
- [x] Визуализация

## 🎯 Что нужно сделать для MVP

### 1. Красивый CLI ✅ (в процессе)

**Python CLI:**
- [x] Добавить rich для красивого вывода
- [x] Таблицы для peers
- [x] Панели для status
- [x] Красивый REPL

**Rust CLI:**
- [ ] Улучшить вывод (цвета, таблицы)

---

### 2. Post-Quantum Encryption (PQC)

**Файл:** `core/src/p2p/encryption_pqc.rs`

**Задачи:**
- [ ] Добавить зависимость `pqcrypto-kyber` или `pqcrypto-ntru`
- [ ] Реализовать PQC handshake (Kyber768 или NTRU)
- [ ] Fallback на RSA если PQC не поддерживается
- [ ] Обновить handshake протокол:
  1. RSA handshake (как сейчас)
  2. PQC shared key exchange
  3. AES session key (как сейчас)
- [ ] Логирование: `🔐 PQC handshake established with peerXYZ (Kyber768)`

**Зависимости:**
```toml
pqcrypto-kyber = "0.5"  # или pqcrypto-ntru
```

---

### 3. Улучшение AI-Routing

**Файл:** `core/src/ai/router.rs`

**Задачи:**
- [ ] История маршрутов (сохранять успешные маршруты)
- [ ] Адаптивное обучение: `score = α*old_score + β*new_score`
- [ ] Учет packet loss (добавить в PeerMetrics)
- [ ] CLI флаг `--ai-debug` для вывода оценок пиров
- [ ] Обновить документацию `docs/ai_routing.md` с формулами

**Формула scoring:**
```
score = 0.4 * latency_score + 0.2 * uptime_score + 0.3 * reliability_score + 0.1 * route_success_rate
```

---

### 4. Тесты

**Новые тесты:**
- [ ] `tests/test_handshake_pqc.rs` - тест PQC handshake
- [ ] `tests/test_routing_adaptive.rs` - тест адаптивного routing
- [ ] `tests/test_multi_node.rs` - тест на 5+ нодах

---

### 5. Demo скрипты

**Файл:** `scripts/demo_local.sh`

```bash
#!/bin/bash
# Запуск 3 нод
cargo run --bin core --release -- 8082 &
cargo run --bin core --release -- 8083 127.0.0.1:8082 &
cargo run --bin core --release -- 8084 127.0.0.1:8082 &
sleep 5

# Отправка сообщений
python3 python_cli/cli.py broadcast "MeshNet AI+PQC demo"
```

**Метрики:**
- [ ] Сохранение метрик в `docs/demo/metrics.log`
- [ ] Логирование latency, throughput, AI scores

---

### 6. Документация и Whitepaper

**Файл:** `docs/whitepaper_v1.md`

**Содержание:**
1. Проблема (цензура, централизация)
2. Решение (P2P mesh, PQC, AI-routing)
3. Архитектура
4. Протоколы (handshake, routing, encryption)
5. Результаты тестов
6. Потенциал (автономные коммуникации)

**PDF:**
- [ ] Генерация PDF из markdown (pandoc)

---

### 7. GitHub Actions CI/CD

**Файл:** `.github/workflows/ci.yml`

**Задачи:**
- [ ] `cargo fmt --check`
- [ ] `cargo clippy -- -D warnings`
- [ ] `cargo test`
- [ ] `cargo build --release`
- [ ] Создание релизных артефактов

---

### 8. Release упаковка

**Файлы:**
- [ ] `CHANGELOG.md`
- [ ] GitHub Release v1.0.0
- [ ] Бинарники (meshnet, meshcli.py)
- [ ] Whitepaper PDF
- [ ] Demo video

---

## Приоритеты

1. **Высокий:** PQC encryption, улучшение AI-routing
2. **Средний:** Тесты, demo скрипты
3. **Низкий:** Whitepaper, CI/CD, release

---

## Временные оценки

- PQC encryption: 4-6 часов
- AI-routing улучшения: 3-4 часа
- Тесты: 2-3 часа
- Demo: 2 часа
- Whitepaper: 4-6 часов
- CI/CD: 2 часа
- Release: 1 час

**Итого:** ~18-24 часа работы


