# Установка и настройка etcd и Patroni

Пошаговый гайд с комментариями, пояснениями и готовыми командами  
для развёртывания **etcd** и **Patroni (PostgreSQL HA)**.

---

# Часть 1. Установка и настройка etcd

---

## 1. Клонирование репозитория и сборка etcd

```bash
git clone -b v3.6.0 https://github.com/etcd-io/etcd.git
cd etcd/
./scripts/build.sh
```

**Комментарий:**  
Скрипт `build.sh` собирает бинарные файлы `etcd` и `etcdctl` в каталоге `./bin`.

---

## 2. Копирование бинарников

```bash
cp ./bin/* /usr/local/bin/
```

**Комментарий:**  
Размещаем бинарные файлы в стандартном системном каталоге, чтобы они были доступны из любого места.

---

## 3. Создание системного пользователя etcd

```bash
useradd --system --home /var/lib/etcd --shell /sbin/nologin etcd
```

**Комментарий:**  
Пользователь не имеет shell и предназначен только для запуска сервиса.

---

## 4. Подготовка каталогов данных etcd

```bash
mkdir -p /var/lib/etcd
chown -R etcd:etcd /var/lib/etcd
chmod 700 /var/lib/etcd
```

---

# Часть 2. Установка и настройка Patroni

---

## 5. Подготовка каталогов PostgreSQL

```bash
mkdir -p /var/lib/pgsql/17/data/
chown -R postgres:postgres /var/lib/pgsql/17/data/
chmod 700 /var/lib/pgsql/17/data/
```

---

## 6. Каталоги Patroni

```bash
mkdir -p /etc/patroni
mkdir -p /var/log/patroni
mkdir -p /opt/patroni
chown -R postgres:postgres /opt/patroni /etc/patroni /var/log/patroni
```

---

## 7. Python virtualenv

```bash
sudo -iu postgres
python3 -m venv /opt/patroni/venv
source /opt/patroni/venv/bin/activate
```

---

## 8. Установка Patroni

```bash
pip install --upgrade pip
pip install patroni[etcd] psycopg2-binary
```

---

## 9. systemd-сервис Patroni

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
Restart=always
RestartSec=5
LimitNOFILE=10240

[Install]
WantedBy=multi-user.target
```
