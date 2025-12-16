# Установка и настройка Patroni (PostgreSQL HA)

Пошаговый гайд по установке и запуску **Patroni** для управления кластером PostgreSQL.

---

## 1. Подготовка каталога данных PostgreSQL

```bash
mkdir -p /var/lib/pgsql/17/data/
chown -R postgres:postgres /var/lib/pgsql/17/data/
chmod 700 /var/lib/pgsql/17/data/
```

**Комментарий:**  
Создаётся каталог данных PostgreSQL 17 с корректными правами доступа.

---

## 2. Создание каталогов Patroni

```bash
mkdir -p /etc/patroni
mkdir -p /var/log/patroni
mkdir -p /opt/patroni
chown -R postgres:postgres /opt/patroni /etc/patroni /var/log/patroni
```

**Комментарий:**  
- `/etc/patroni` — конфигурационные файлы  
- `/var/log/patroni` — логи  
- `/opt/patroni` — виртуальное окружение Python  

---

## 3. Создание Python virtualenv

```bash
sudo -iu postgres
python3 -m venv /opt/patroni/venv
source /opt/patroni/venv/bin/activate
```

**Комментарий:**  
Patroni устанавливается в отдельное виртуальное окружение Python.

---

## 4. Установка Patroni и зависимостей

```bash
pip install --upgrade pip
pip install patroni[etcd] psycopg2-binary
```

**Комментарий:**  
- `patroni[etcd]` — Patroni с поддержкой DCS  
- `psycopg2-binary` — драйвер PostgreSQL  

---

## 5. Проверка установки

```bash
patroni --version
```

**Комментарий:**  
Проверяем, что Patroni установлен и доступен.

---

## 6. Тестовый запуск Patroni

```bash
patroni /etc/patroni/patroni.yml
```

**Комментарий:**  
Ручной запуск для проверки корректности конфигурации.

---

## 7. Проверка состояния кластера

```bash
patronictl -c /etc/patroni/patroni.yml list
```

**Комментарий:**  
Отображает текущее состояние кластера Patroni.

---

## 8. systemd-сервис Patroni

Файл: `/etc/systemd/system/patroni.service`

```ini
[Unit]
Description=Patroni PostgreSQL HA
After=network.target
Wants=network.target

[Service]
Type=simple

User=postgres
Group=postgres

ExecStart=/opt/patroni/venv/bin/patroni /etc/patroni/patroni.yml

RuntimeDirectory=patroni
RuntimeDirectoryMode=0755

Restart=always
RestartSec=5

LimitNOFILE=10240

[Install]
WantedBy=multi-user.target
```

---

## 9. Активация и запуск сервиса

```bash
systemctl daemon-reload
systemctl enable patroni
systemctl start patroni
```

---

## 10. Проверка статуса сервиса

```bash
systemctl status patroni
```
