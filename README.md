# Server Control Bot — Полный гайд по развёртыванию

Пошаговая инструкция: от `git clone` до работающего бота на Ubuntu/Debian.

---

## 1. Подготовка сервера

```bash
sudo apt update && sudo apt install -y python3 python3-pip git
python3 --version   # нужен 3.10+
```

---

## 2. Клонирование репозитория

```bash
cd /opt
sudo git clone https://github.com/reposit0r/server_control.git server-control-bot
cd /opt/server-control-bot
```

---

## 3. Установка зависимостей

```bash
sudo pip install -r requirements.txt --break-system-packages
```

Проверить:

```bash
python3 -c "import aiogram, psutil, matplotlib, dotenv; print('OK')"
```

---

## 4. Настройка .env

```bash
sudo cp .env.example .env
sudo nano .env
```

Вписать:

```
BOT_TOKEN=ТВОЙ_ТОКЕН_ОТ_BOTFATHER
AUTHORIZED_USER_ID=ТВОЙ_TELEGRAM_ID
```

**BOT_TOKEN** — получить у [@BotFather](https://t.me/BotFather)
**AUTHORIZED_USER_ID** — узнать у [@userinfobot](https://t.me/userinfobot)

Закрыть доступ к файлу:

```bash
sudo chmod 600 /opt/server-control-bot/.env
```

---

## 5. Тестовый запуск

```bash
cd /opt/server-control-bot
sudo python3 bot.py
```

В Telegram написать боту `/start` — должен ответить:
**✅ Bot is active. 🤖 Version: 3.0.2**

Потыкать кнопки. Остановить: `Ctrl+C`.

---

## 6. Systemd-сервис

```bash
sudo cp /opt/server-control-bot/server-control-bot.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now server-control-bot
```

Проверить:

```bash
sudo systemctl status server-control-bot
```

---

## 7. Проверка всех фич

| Действие | Ожидание |
|---|---|
| `/start` | Ответ с версией 3.0.2 |
| 📊 System Load | CPU, RAM, Temp, Disk, Uptime |
| ℹ️ Device Info | Hostname, OS, версия бота |
| 📈 Graphs → CPU → 1 Hour | «Not enough data yet» (данные через ~1 мин) |
| 🔝 Top Processes → By CPU | Топ 10 процессов |
| 🔐 SSH Log | Лог SSH или «No events yet» |
| 📦 Updates | Список pending apt-пакетов |
| 📋 View Thresholds | TEMP 75 / CPU 90 / RAM 90 / Smoothing 30 |
| 🔄 Update Bot | «Bot is up to date — v3.0.2» |
| SSH-подключение к серверу | Моментальный алерт в Telegram |

---

## 8. Обновление бота

### Вариант А — через Telegram

Нажать **🔄 Update Bot** → бот покажет список новых коммитов → подтвердить → бот обновится и перезапустится.

### Вариант Б — через SSH

```bash
sudo /opt/server-control-bot/update.sh
```

### Workflow

1. Меняешь код локально
2. `git push`
3. На сервере: жмёшь «🔄 Update Bot» в Telegram или запускаешь `update.sh`
4. Бот скачивает новый код, показывает `v3.0.2 → v3.0.3`, перезапускается

---

## 9. (Опционально) Logrotate

```bash
sudo nano /etc/logrotate.d/server-control-bot
```

```
/opt/server-control-bot/ubuntu_bot.log {
    weekly
    rotate 4
    compress
    missingok
    notifempty
    copytruncate
}
```

---

## Troubleshooting

**Бот не отвечает:**
```bash
sudo systemctl status server-control-bot
sudo journalctl -u server-control-bot --no-pager -n 50
```

**«BOT_TOKEN is not set»:**
```bash
cat /opt/server-control-bot/.env
```

**SSH-лог пустой:**
```bash
ls -la /var/log/auth.log
# если нет auth.log:
journalctl -u ssh -n 5
```

**Графики пустые:**
Данные пишутся раз в 60 сек — подождать 2–3 минуты.

**Update Bot не работает:**
```bash
cd /opt/server-control-bot
git remote -v          # должен быть origin → github
git status             # не должно быть конфликтов
sudo git pull          # попробовать вручную
```
