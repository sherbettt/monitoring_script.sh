
**Шаг 1: Создаем скрипт мониторинга**

Создайте файл `/root/monitoring_script.sh` с содержимым:

```bash
#!/bin/bash

LOG_FILE="/var/log/monitoring.log"
PID_FILE="/var/run/monitoring_test.pid"
PROCESS_NAME="test"
MONITORING_URL="https://test.com/monitoring/test/api"

# Проверка наличия процесса
if pgrep -x "$PROCESS_NAME" >/dev/null; then
    current_pid=$(pgrep -x "$PROCESS_NAME")

    # Проверка изменения PID
    if [ -f "$PID_FILE" ]; then
        previous_pid=$(cat "$PID_FILE")
        if [ "$current_pid" != "$previous_pid" ]; then
            echo "$(date +"%Y-%m-%d %T") Процесс $PROCESS_NAME перезапущен. Новый PID: $current_pid" >> "$LOG_FILE"
        fi
    else
        echo "$current_pid" > "$PID_FILE"
    fi

    # Обновляем PID
    echo "$current_pid" > "$PID_FILE"

    # Проверка доступности сервера
    if ! curl -s -o /dev/null --fail --connect-timeout 10 "$MONITORING_URL"; then
        echo "$(date +"%Y-%m-%d %T") Ошибка: Сервер мониторинга недоступен" >> "$LOG_FILE"
    fi
else
    # Удаляем PID файл, если процесс не запущен
    [ -f "$PID_FILE" ] && rm "$PID_FILE"
fi
```

Установите права на выполнение:
```bash
sudo chmod +x /root/monitoring_script.sh
```

**Шаг 2: Настраиваем systemd сервис и таймер**

Создайте файл сервиса `/etc/systemd/system/monitoring.service`:
```ini
[Unit]
Description=Monitoring Service for Test Process

[Service]
ExecStart=/root/monitoring_script.sh
User=root
```

Создайте таймер `/etc/systemd/system/monitoring.timer`:
```ini
[Unit]
Description=Run Monitoring Script Every Minute

[Timer]
OnBootSec=1min
OnUnitActiveSec=1min

[Install]
WantedBy=timers.target
```

**Шаг 3: Активация и запуск**

Обновите конфигурацию systemd и активируйте таймер:
```bash
 systemctl daemon-reload
 systemctl enable monitoring.timer
 systemctl start monitoring.timer
```

**Шаг 4: Создание лог-файла**

Создайте файл логирования и установите права:
```bash
 touch /var/log/monitoring.log
 chmod 644 /var/log/monitoring.log
```

**Проверка работы:**
- Статус таймера: `systemctl status monitoring.timer`
- Логи скрипта: `tail -f /var/log/monitoring.log`
