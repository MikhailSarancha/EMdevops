
# Скрипт для мониторинга процесса `test` в Linux

## Шаг 1: Создание Bash-скрипта

`/usr/local/bin/monitor_test.sh`:

```bash
#!/bin/bash

# здесь будем хранить логи
LOG_FILE="/var/log/monitoring.log"

# URL для мониторинга
MONITORING_URL="https://test.com/monitoring/test/api"

# имя процесса
PROCESS_NAME="test"

# проверим, запущен ли процесс
if pgrep -x "$PROCESS_NAME" > /dev/null; then
    # процесс запущен, отправляем запрос на сервер мониторинга
    if curl -s -o /dev/null -w "%{http_code}" "$MONITORING_URL" | grep -q "200"; then
        # сервер доступен, запрос успешен
        echo "$(date): Процесс $PROCESS_NAME запущен, сервер мониторинга доступен." >> "$LOG_FILE"
    else
        # сервер недоступен
        echo "$(date): Процесс $PROCESS_NAME запущен, но сервер мониторинга недоступен." >> "$LOG_FILE"
    fi
else
    #  не запущен
    echo "$(date): Процесс $PROCESS_NAME не запущен." >> "$LOG_FILE"
fi
```

Сделаем скрипт исполняемым:

```bash
chmod +x /usr/local/bin/monitor_test.sh
```

## Шаг 2: Создание Systemd юнита

Создадим файл юнита, например, `/etc/systemd/system/monitor_test.service`:

```ini
[Unit]
Description=Monitor test process
After=network.target

[Service]
ExecStart=/usr/local/bin/monitor_test.sh
Restart=on-failure
RestartSec=60

[Install]
WantedBy=multi-user.target
```

## Шаг 3: Включение и запуск сервиса


```bash
sudo systemctl daemon-reload
sudo systemctl enable monitor_test.service
sudo systemctl start monitor_test.service
```


