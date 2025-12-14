# Как протестировать Python CLI

## Быстрый тест

### 1. Запустите ноду (Терминал 1)

```bash
cd /Users/boris/Desktop/meshlink
cargo run --bin core --release -- 8082
```

Дождитесь сообщения:
```
INFO: API server listening on 127.0.0.1:17082
```

**Оставьте этот терминал открытым!**

---

### 2. Протестируйте команды (Терминал 2)

#### Статус ноды

```bash
cd /Users/boris/Desktop/meshlink
python3 python_cli/cli.py status
```

**Ожидаемый результат:**
```
Node Status:
------------------------------------------------------------
Node ID: <uuid>
Connected: 0/0 peers
API Port: 17082
```

#### Список пиров

```bash
python3 python_cli/cli.py peers
```

**Ожидаемый результат:**
```
No peers connected
```

#### Отправка broadcast сообщения

```bash
python3 python_cli/cli.py broadcast "Hello from Python CLI!"
```

**Ожидаемый результат:**
```
Message sent: <message-id>
```

---

## Тест с несколькими нодами

### 1. Запустите первую ноду (Терминал 1)

```bash
cd /Users/boris/Desktop/meshlink
cargo run --bin core --release -- 8082 127.0.0.1:8083
```

### 2. Запустите вторую ноду (Терминал 2)

```bash
cd /Users/boris/Desktop/meshlink
cargo run --bin core --release -- 8083 127.0.0.1:8082
```

Подождите 10-15 секунд, пока ноды подключатся.

### 3. Протестируйте Python CLI (Терминал 3)

#### Статус ноды 1

```bash
cd /Users/boris/Desktop/meshlink
export MESHLINK_API_PORT=17082
python3 python_cli/cli.py status
```

**Ожидаемый результат:**
```
Node Status:
------------------------------------------------------------
Node ID: <uuid>
Connected: 1/1 peers
API Port: 17082
```

#### Список пиров ноды 1

```bash
python3 python_cli/cli.py peers
```

**Ожидаемый результат:**
```
Peers (1):
------------------------------------------------------------
<peer-id>... @ 127.0.0.1:8083 [Connected]
```

#### Отправка сообщения конкретному пиру

```bash
# Сначала получите peer_id из списка пиров
PEER_ID=$(python3 python_cli/cli.py peers 2>&1 | grep -E "^[a-f0-9]{8}" | head -1 | awk '{print $1}')

# Отправьте сообщение
python3 python_cli/cli.py send "$PEER_ID" "Hello, peer!"
```

**Ожидаемый результат:**
```
Message sent: <message-id>
```

#### Broadcast сообщение

```bash
python3 python_cli/cli.py broadcast "Hello, everyone!"
```

**Ожидаемый результат:**
```
Message sent: <message-id>
```

Проверьте логи в терминалах 1 и 2 - вы должны увидеть получение сообщения.

---

## Интерактивный режим (REPL)

### Запуск интерактивного режима

```bash
cd /Users/boris/Desktop/meshlink
export MESHLINK_API_PORT=17082
python3 python_cli/cli.py -i
```

Или просто:

```bash
python3 python_cli/cli.py
```

### Использование REPL

```
MeshLink CLI - Interactive Mode
Type 'help' for commands, 'exit' to quit
------------------------------------------------------------
meshlink> help

Commands:
  send <peer_id> <message>  - Send message to specific peer
  broadcast <message>       - Broadcast message to all peers
  peers                     - List all peers
  status                    - Show node status
  help                      - Show this help
  exit                      - Exit interactive mode

meshlink> status

Node Status:
------------------------------------------------------------
Node ID: f6e3047c-3885-4bd9-a06a-0572113c180a
Connected: 1/1 peers
API Port: 17082

meshlink> peers

Peers (1):
------------------------------------------------------------
8790c3fb... @ 127.0.0.1:8083 [Connected]

meshlink> broadcast Test message from REPL
Message sent: abc-123-def-456

meshlink> exit
Goodbye!
```

---

## Автоопределение порта

Если не указан `MESHLINK_API_PORT`, CLI автоматически найдет доступный порт:

```bash
# Убедитесь, что нода запущена
# CLI автоматически проверит порты: 17080, 17081, 17082, 17083, 17084, 17085

python3 python_cli/cli.py status
```

---

## Тестирование ошибок

### Нода не запущена

```bash
# Убейте все ноды
pkill -f "target/.*/core"

# Попробуйте использовать CLI
python3 python_cli/cli.py status
```

**Ожидаемый результат:**
```
Error: Could not find running node. Make sure a node is running.
```

### Неверная команда

```bash
python3 python_cli/cli.py invalid_command
```

**Ожидаемый результат:**
```
Unknown command: invalid_command
```

### Неверный peer_id

```bash
python3 python_cli/cli.py send invalid-peer-id "Hello"
```

**Ожидаемый результат:**
```
Error: Peer invalid-peer-id not connected
```

---

## Запуск unit тестов

```bash
cd /Users/boris/Desktop/meshlink
python3 tests/cli_tests.py -v
```

**Ожидаемый результат:**
```
test_broadcast_message_success ... ok
test_list_peers_success ... ok
test_send_message_success ... ok
test_show_status_success ... ok
test_discover_api_port_from_defaults ... ok
test_discover_api_port_from_env ... ok
test_send_request_connection_error ... ok
test_send_request_success ... ok
test_test_connection_failure ... ok
test_test_connection_success ... ok

----------------------------------------------------------------------
Ran 10 tests in 0.003s

OK
```

---

## Полный тест-сценарий

### Сценарий: Отправка сообщений между 3 нодами

**Терминал 1:**
```bash
cargo run --bin core --release -- 8082 127.0.0.1:8083 127.0.0.1:8084
```

**Терминал 2:**
```bash
cargo run --bin core --release -- 8083 127.0.0.1:8082 127.0.0.1:8084
```

**Терминал 3:**
```bash
cargo run --bin core --release -- 8084 127.0.0.1:8082 127.0.0.1:8083
```

Подождите 15-20 секунд для подключения всех нод.

**Терминал 4:**
```bash
# Тест ноды 1
export MESHLINK_API_PORT=17082
python3 python_cli/cli.py status
python3 python_cli/cli.py peers

# Отправка broadcast
python3 python_cli/cli.py broadcast "Test from Python CLI to 3 nodes"

# Интерактивный режим
python3 python_cli/cli.py -i
# В REPL:
# > status
# > peers
# > broadcast Hello from REPL
# > exit
```

**Проверьте логи в терминалах 1, 2, 3:**
- Все ноды должны получить broadcast сообщение
- В логах должно быть видно получение сообщений

---

## Проверка работы

### ✅ Успешный тест

1. CLI подключается к ноде
2. Команды выполняются без ошибок
3. Сообщения отправляются и получаются
4. REPL режим работает корректно
5. Все тесты проходят

### ❌ Проблемы

**CLI не находит ноду:**
- Проверьте, что нода запущена
- Проверьте порт: `lsof -i :17082`
- Укажите порт явно: `export MESHLINK_API_PORT=17082`

**Ошибка подключения:**
- Убедитесь, что API сервер запущен (должно быть в логах ноды)
- Проверьте, что порт правильный (9000 + listen_port)

**Сообщения не доставляются:**
- Проверьте, что пиры подключены (`peers` команда)
- Проверьте логи нод на наличие ошибок

---

## Полезные команды

```bash
# Проверить, какие порты заняты
lsof -i :17082,17083,17084

# Убить все ноды
pkill -f "target/.*/core"

# Запустить CLI с отладкой (если нужно)
python3 -u python_cli/cli.py status

# Проверить версию Python
python3 --version  # Должно быть 3.7+
```

---

**Готово! Теперь вы можете протестировать Python CLI во всех режимах.**


