To set up **portable Grafana** in **`/opt/grafana`** and run it as a **Linux systemd service**, follow these steps:

---

## ✅ Step 1: Extract Grafana to `/opt/grafana`

Assuming you've downloaded a `.tar.gz` file, extract it like this:

```bash
sudo mkdir -p /opt/grafana
sudo tar -zxvf grafana-*.tar.gz -C /opt/grafana --strip-components=1
```

* `--strip-components=1` removes the top-level folder inside the archive, placing everything directly into `/opt/grafana`.

---

## ✅ Step 2: Create a Dedicated User (Optional but Recommended)

```bash
sudo useradd --no-create-home --shell /usr/sbin/nologin grafana
sudo chown -R grafana:grafana /opt/grafana
```

---

## ✅ Step 3: Create a Systemd Service File

Create the file `/etc/systemd/system/grafana.service`:

```bash
sudo nano /etc/systemd/system/grafana.service
```

Paste the following contents:

```ini
[Unit]
Description=Grafana instance
After=network.target

[Service]
User=grafana
Group=grafana
Type=simple
ExecStart=/opt/grafana/bin/grafana-server \
  --config=/opt/grafana/conf/defaults.ini \
  --homepath=/opt/grafana

Restart=on-failure
LimitNOFILE=10000

[Install]
WantedBy=multi-user.target
```

---

## ✅ Step 4: Reload systemd and Start Grafana

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable grafana
sudo systemctl start grafana
```

---

## ✅ Step 5: Check Grafana Service Status

```bash
sudo systemctl status grafana
```

If everything's OK, Grafana will be running on `http://localhost:3000`.

---

## ✅ Optional: Firewall

If using `ufw`, allow port 3000:

```bash
sudo ufw allow 3000/tcp
```

---

Let me know if you want to change the port or customize paths/logs.
