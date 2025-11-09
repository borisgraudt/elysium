# Python CLI - Руководство пользователя

## Установка

Python CLI использует только стандартную библиотеку Python, дополнительные зависимости не требуются.

```bash
# Клонируйте репозиторий
git clone <repo-url>
cd meshlink

# CLI готов к использованию
python python_cli/cli.py --help
```

## Использование

### Командный режим

Выполнение одной команды:

```bash
# Показать статус ноды
python python_cli/cli.py status

# Список пиров
python python_cli/cli.py peers

# Отправить сообщение конкретному пиру
python python_cli/cli.py send <peer_id> "Hello, peer!"

# Отправить broadcast сообщение
python python_cli/cli.py broadcast "Hello, everyone!"
```

### Интерактивный режим (REPL)

Запуск интерактивного режима:

```bash
# Вариант 1: Без аргументов
python python_cli/cli.py

# Вариант 2: С флагом
python python_cli/cli.py -i
python python_cli/cli.py --interactive
python python_cli/cli.py repl
```

В интерактивном режиме:

```
MeshLink CLI - Interactive Mode
Type 'help' for commands, 'exit' to quit
------------------------------------------------------------
meshlink> status

Node Status:
------------------------------------------------------------
Node ID: f6e3047c-3885-4bd9-a06a-0572113c180a
Connected: 2/3 peers
API Port: 17082

meshlink> peers

Peers (2):
------------------------------------------------------------
8790c3fb... @ 127.0.0.1:8083 [Connected]
101265bb... @ 127.0.0.1:8084 [Connected]

meshlink> broadcast Hello from Python CLI!
Message sent: abc-123-def-456

meshlink> exit
Goodbye!
```

## Команды

### `status`

Показывает статус ноды:
- Node ID
- Количество подключенных пиров
- API порт

```bash
python python_cli/cli.py status
```

### `peers`

Показывает список всех известных пиров:
- Peer ID (первые 8 символов)
- Адрес
- Состояние подключения

```bash
python python_cli/cli.py peers
```

### `send <peer_id> <message>`

Отправляет сообщение конкретному пиру.

```bash
python python_cli/cli.py send 8790c3fb-4acb-4876-93d8-b67dc951636e "Hello!"
```

### `broadcast <message>`

Отправляет broadcast сообщение всем подключенным пирам.

```bash
python python_cli/cli.py broadcast "Hello, everyone!"
```

## Настройка

### Переменные окружения

**`MESHLINK_API_PORT`** - Порт API сервера ноды (по умолчанию: автоопределение)

```bash
export MESHLINK_API_PORT=17082
python python_cli/cli.py status
```

### Автоопределение порта

Если `MESHLINK_API_PORT` не установлен, CLI автоматически проверяет следующие порты:
- 17080
- 17081
- 17082
- 17083
- 17084
- 17085

Используется первый доступный порт.

## Примеры использования

### Базовое использование

```bash
# 1. Запустите ноду
cargo run --bin core -- 8082

# 2. В другом терминале - используйте Python CLI
python python_cli/cli.py status
python python_cli/cli.py peers
python python_cli/cli.py broadcast "Test message"
```

### Интерактивный режим

```bash
# Запустите интерактивный режим
python python_cli/cli.py -i

# Внутри REPL:
meshlink> status
meshlink> peers
meshlink> send <peer_id> "Hello"
meshlink> broadcast "Test"
meshlink> help
meshlink> exit
```

### С несколькими нодами

```bash
# Терминал 1: Нода 1
cargo run --bin core -- 8082

# Терминал 2: Нода 2
cargo run --bin core -- 8083 127.0.0.1:8082

# Терминал 3: Python CLI для ноды 1
export MESHLINK_API_PORT=17082
python python_cli/cli.py status

# Терминал 4: Python CLI для ноды 2
export MESHLINK_API_PORT=17083
python python_cli/cli.py status
```

## Обработка ошибок

CLI обрабатывает следующие ошибки:

- **Нода не найдена**: Если не удается подключиться к API серверу
- **Неверная команда**: Показывает справку
- **Ошибки API**: Выводит сообщение об ошибке от сервера

Примеры:

```bash
# Нода не запущена
$ python python_cli/cli.py status
Error: Could not find running node. Make sure a node is running.

# Неверная команда
$ python python_cli/cli.py invalid
Unknown command: invalid

# Ошибка API
$ python python_cli/cli.py send invalid-peer "Hello"
Error: Peer invalid-peer not connected
```

## Тестирование

Запуск тестов:

```bash
python -m pytest tests/cli_tests.py -v
# или
python tests/cli_tests.py
```

## Архитектура

### MeshLinkClient

Основной класс для взаимодействия с API ноды:

- `_discover_api_port()` - Автоопределение порта API
- `_test_connection(port)` - Проверка доступности порта
- `_send_request(request)` - Отправка запроса к API
- `send_message(to, message)` - Отправка сообщения
- `list_peers()` - Список пиров
- `show_status()` - Статус ноды

### Протокол API

CLI использует JSON-RPC протокол через TCP:

**Запрос:**
```json
{
  "command": "status",
  ...
}
```

**Ответ:**
```json
{
  "success": true,
  "data": {
    "node_id": "...",
    "connected_peers": 2,
    "total_peers": 3
  }
}
```

## Разработка

### Добавление новых команд

1. Добавьте метод в `MeshLinkClient`:
```python
def new_command(self, arg1, arg2):
    request = {"command": "new_command", "arg1": arg1, "arg2": arg2}
    response = self._send_request(request)
    # Обработка ответа
```

2. Добавьте обработку в `main()`:
```python
elif command == "new_command":
    # Парсинг аргументов
    client.new_command(arg1, arg2)
```

3. Добавьте обработку в `run_repl()`:
```python
elif command == "new_command":
    # Парсинг и вызов
    client.new_command(arg1, arg2)
```

### Тестирование

Добавьте тесты в `tests/cli_tests.py`:

```python
@patch('cli.MeshLinkClient._send_request')
def test_new_command(self, mock_send_request):
    mock_send_request.return_value = {"success": True, "data": {}}
    client = MeshLinkClient.__new__(MeshLinkClient)
    client.api_port = 17082
    client.new_command("arg1", "arg2")
    # Проверки
```

## FAQ

**Q: Почему CLI не находит ноду?**
A: Убедитесь, что нода запущена и API сервер слушает на порту. Проверьте порт через `MESHLINK_API_PORT`.

**Q: Можно ли использовать CLI без интерактивного режима?**
A: Да, все команды работают в командном режиме.

**Q: Нужны ли дополнительные зависимости?**
A: Нет, CLI использует только стандартную библиотеку Python.

**Q: Как отправить сообщение с пробелами?**
A: Используйте кавычки: `python python_cli/cli.py broadcast "Hello, world!"`

