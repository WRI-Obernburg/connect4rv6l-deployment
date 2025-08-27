# Connect4 RV6L – Deployment 🚀

[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](https://choosealicense.com/licenses/mit/)
[![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![Docker Compose](https://img.shields.io/badge/Docker%20Compose-2496ED?logo=docker&logoColor=white)](https://docs.docker.com/compose/)

> **Production-ready Deployment-Konfiguration für das Connect4 RV6L Roboterarm-Spiel**

Dieses Repository enthält die Docker-Compose-Konfiguration und Deployment-Skripte für das [Connect4 RV6L Hauptprojekt](https://github.com/wri-obernburg/connect4rv6l).

## 📋 Inhaltsverzeichnis

- [🎯 Überblick](#-überblick)
- [🔀 Betriebsmodi (Pfad A/B)](#-betriebsmodi-pfad-ab)
- [🔧 Voraussetzungen](#-voraussetzungen)
- [🚀 Installation](#-installation)
  - [Pfad A: Nur Kiosk](#pfad-a-nur-kiosk)
  - [Pfad B: Vollständiges Backend (+ optionaler Kiosk Modus)](#pfad-b-vollständiges-backend--optionaler-kiosk-modus)
- [⚙️ Konfiguration](#-konfiguration)
- [🎮 Betrieb (Nur Pfad B)](#-betrieb-nur-pfad-b)
- [🔧 Troubleshooting](#-troubleshooting)
- [🔄 Wartung & Updates](#-wartung--updates)
- [🔒 Sicherheit](#-sicherheit)
- [Support](#support)
- [📄 Lizenz](#-lizenz)
- [👥 Autoren](#-autoren)

## 🎯 Überblick

Dieses Deployment-Repository stellt eine produktionsreife Docker-Umgebung für das Connect4 RV6L-System bereit. Es orchestriert alle notwendigen Services:

- **Backend Server** (Ports 3000, 4000)
- **Mobile Frontend** (Port 8080)
- **Control Panel** (integriert)
- **Local Display** (integriert)
- **Cloudflare Tunnel**

## 🔀 Betriebsmodi (Pfad A/B)

- **Pfad A – Nur Kiosk:** Das Local Frontend (Kiosk) läuft auf einem Displaygerät und verbindet sich mit einem externen Backend.
  - Aufruf: `http://<backend-host>:4000/localfrontend`
- **Pfad B – Backend auf dem Pi:** Backend + interne Web-UIs laufen auf einem Raspberry Pi (mDNS-Hostname `rv6l-application.local`).
  - Aufruf: `http://rv6l-application.local:4000/localfrontend`

> Hinweis: Ersetzen Sie `<backend-host>` durch Hostname oder IP des Backend-Hosts. Im Regelfall sollte dies `rv6l-application.local` sein.

## 🔧 Voraussetzungen


### Netzwerk-Anforderungen

- Zugang zum Internet (Über WLan oder mit USB Lan Adapter am Raspberry Pi)
- Netzwerkverbindung zum RV6L-Roboter (`192.168.2.1`)

### Empfohlene Hardware

- **CPU:** 2+ Kerne
- **RAM:** 4GB+ 
- **Netzwerk:** Ethernet-Verbindung zum Roboter

## 🚀 Installation

### Pfad A: Nur Kiosk

Dieser Modus zeigt nur das lokale Frontend (Kiosk) an und verbindet sich mit einem externen Backend. Stellen Sie sicher, dass der Pi Netzwerkzugang zum Backend-Host hat (`<backend-host>`).

#### 1) Raspberry Pi einrichten (Kiosk-Gerät)

- Raspberry Pi Imager laden, Raspberry Pi OS (mit Desktop) auf SD-Karte schreiben.
- Im Imager „Erweiterte Optionen“ aktivieren: SSH einschalten, Benutzer/Passwort setzen, optional Hostname.
- Erststart: Mit LAN verbinden oder WLAN über `raspi-config` einrichten.

```bash
sudo raspi-config
```

Empfehlungen: Autologin in Desktop aktivieren (System → Boot/Auto Login), Bildschirm-Standby deaktivieren (optional per Autostart unten).

#### 2) Browser installieren (Chromium/Chrome)

Auf Raspberry Pi OS wird Chromium genutzt (Chrome-Äquivalent). Je nach Distribution ist der Paketname unterschiedlich:

```bash
sudo apt update
# Variante 1 (häufig auf Raspberry Pi OS):
sudo apt install -y chromium-browser
# Variante 2 (Debian Bookworm/ähnlich):
sudo apt install -y chromium
```

#### 3) Kiosk-Autostart einrichten

Ziel-URL des Kiosks:
- `http://<backend-host>:4000/localfrontend`

Autostart für LXDE konfigurieren. Passen Sie ggf. den Browser-Binärnamen von `chromium-browser` auf `chromium` an.

```bash
mkdir -p ~/.config/lxsession/LXDE-pi
cat >> ~/.config/lxsession/LXDE-pi/autostart << 'EOF'
@xset s off
@xset -dpms
@xset s noblank
@chromium-browser --noerrdialogs --disable-infobars --kiosk http://<backend-host>:4000/localfrontend
EOF
```

> Ersetzen Sie `<backend-host>` durch `rv6l-application.local` oder die IP/den Host des Backends.

### Pfad B: Vollständiges Backend (+ optionaler Kiosk Modus)

Dieser Modus betreibt Backend und lokale UIs direkt auf einem Raspberry Pi. Standard-Hostname und Zugangsdaten:

- Hostname: `rv6l-application.local`
- Username: `wri`
- Passwort: `xxxxxxxxxxx`

Passwort ist auf dem Makerspace Laufwerk im RV6L-Ordner zu finden.

#### 1) Neuen Raspberry Pi einrichten

- Raspberry Pi Imager herunterladen und SD-Karte schreiben.
- Im Imager „Erweiterte Optionen“ setzen: SSH aktivieren, Benutzername/Passwort setzen, Hostname `rv6l-application.local`.
- Mit LAN verbinden, WLAN später über `raspi-config` einrichten (optional).
- Statische IP für die Roboter-Verbindung an `eth0` setzen (Roboter: `192.168.2.1`, Pi: `192.168.2.2`).
  - ifupdown installieren und Schnittstelle hochfahren:

```bash
sudo apt update
sudo apt install -y ifupdown
```

  - Datei `/etc/network/interfaces` bearbeiten (Beispiel):

```text
auto eth0
allow-hotplug eth0
iface eth0 inet static
address 192.168.2.2
netmask 255.255.255.0
```

```bash
# Schnittstelle anwenden
sudo ifup eth0
```

> Hinweis: Der Roboter ist unter `192.168.2.1` erreichbar; der Pi sollte daher eine freie IP im gleichen Subnetz erhalten, z. B. `192.168.2.2`.

#### 2) Docker installieren

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker
sudo apt-get install -y docker-compose-plugin
```

#### 3) Repository klonen und starten

```bash
git clone https://github.com/WRI-Obernburg/connect4rv6l-deployment
cd connect4rv6l-deployment

# Optional: .env für Cloudflare Tunnel anlegen
# echo "CLOUDFLARED_TOKEN=ey..." > .env

# Services im Hintergrund starten
docker compose up -d

# Status prüfen
docker compose ps
```

- Backend erreichbar über: `http://rv6l-application.local:4000`
- Lokales Frontend: `http://rv6l-application.local:4000/localfrontend`
- Control Panel: `http://rv6l-application.local:4000/controlpanel`

#### 4) Browser installieren (Chromium/Chrome)

Auf Raspberry Pi OS wird in der Regel Chromium genutzt (Chrome-Äquivalent). Installieren Sie je nach Distribution den passenden Paketnamen:

```bash
sudo apt update
# Variante 1 (häufig auf Raspberry Pi OS):
sudo apt install -y chromium-browser
# Variante 2 (Debian Bookworm/ähnlich):
sudo apt install -y chromium
```

#### 5) Optional: Kiosk Autostart auf dem Pi

Aktivieren Sie den Autostart im Kiosk-Modus auf die lokale Frontend-URL. Ersetzen Sie den Browser-Binärnamen ggf. durch `chromium` statt `chromium-browser`.

```bash
mkdir -p ~/.config/lxsession/LXDE-pi
cat >> ~/.config/lxsession/LXDE-pi/autostart << 'EOF'
@xset s off
@xset -dpms
@xset s noblank
@chromium-browser --noerrdialogs --disable-infobars --kiosk http://rv6l-application.local:4000/localfrontend
EOF
```

## ⚙️ Konfiguration

### Umgebungsvariablen

Erstellen Sie eine `.env` Datei oder setzen Sie die Variablen direkt:

| Variable            | Beschreibung | Standard | Beispiel |
|---------------------|-------------|----------|----------|
| `FRONTEND_ADDRESS`  | Frontend-URL für QR-Codes | `https://connect4rv6l.vercel.app/` | `http://192.168.1.100:8080` |
| `CLOUDFLARED_TOKEN` | Token für Cloudflare Tunnel | - | `eyJhIjoiXXX...` |


### Netzwerk-Setup

#### Lokales Netzwerk

```bash
# Host-IP ermitteln
ip addr show | grep "inet " | grep -v 127.0.0.1

# Oder auf Windows
ipconfig | findstr IPv4
```

> Hinweis (Pfad B): Stellen Sie sicher, dass der Pi auf `eth0` eine statische IP hat, wenn der Roboter direkt per Ethernet verbunden ist.

#### Firewall-Konfiguration

```bash
# Ubuntu/Debian - Ports öffnen
sudo ufw allow 3000
sudo ufw allow 4000  
sudo ufw allow 8080

# CentOS/RHEL
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --permanent --add-port=4000/tcp
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

## 🎮 Betrieb (Nur Pfad B)

### Container starten

```bash
# Alle Services starten
docker compose up -d

```

### Status überwachen

```bash
# Container-Status
docker compose ps

# Ausführlich mit Ports
docker compose ps --format "table {{.Name}}\t{{.Status}}\t{{.Ports}}"

# Resource-Verbrauch
docker stats
```

### Logs einsehen

```bash
# Alle Logs
docker compose logs -f

# Nur Backend-Logs
docker compose logs -f connect4

# Letzte 200 Zeilen
docker compose logs connect4 --tail 200

```


## 🔧 Troubleshooting

### Schnellchecks

- Läuft der Backend-Container? `docker ps` bzw. `docker compose ps` sollte `connect4` als `running` anzeigen.
- Ist der RV6L erreichbar? `ping 192.168.2.1`.

### Häufige Probleme

#### ❌ Container startet nicht

```bash
# Logs prüfen
docker compose logs [service-name]

# Ports prüfen
ss -tulpn | grep -E ':3000|:4000|:8080'

# Images neu ziehen
docker compose pull
docker compose up -d --force-recreate
```

#### ❌ WebSocket-Verbindung schlägt fehl

- Läuft das Backend und ist Port 3000 offen?
- Netzwerk/Kabel prüfen

```bash
# Backend-Status prüfen
curl -I http://localhost:3000/state

```

### Hardware/Netzwerk

```bash
# Ist der RV6L erreichbar? (Pfad B)
ping 192.168.2.1

# Statische IP aktiv? IP von eth0 prüfen (Pfad B)
ip addr show eth0 | grep inet
```

- Kiosk lädt nicht (Pfad A/B):
  - URL korrekt? `<backend-host>` durch tatsächlichen Host ersetzen.
  - Netzwerk/DNS prüfen (Ping auf Hostname/IP testen).



## 🔄 Wartung & Updates

### Routine-Updates

```bash
# 1. Neueste Images ziehen
docker compose pull

# 2. Services mit neuen Images neu starten
docker compose up -d

# 3. Ungenutzte Images aufräumen
docker image prune -f

# 4. System-Updates
sudo apt update && sudo apt upgrade -y  # Ubuntu/Debian

```

### Komplett-Reset

```bash
# ⚠️ ACHTUNG: Löscht alle Container und Volumes
docker compose down -v

# Neustart von null
docker compose up -d
```


## 🔒 Sicherheit

### Netzwerk-Sicherheit

- **Interne Ports:** 3000, 4000 nur im lokalen Netzwerk verfügbar
- **Externe Erreichbarkeit:** Nur über konfigurierten Cloudflare Tunnel
- **Roboter-Netzwerk:** Isolierte Verbindung zu 192.168.2.1



## Support

1. **Container-Status prüfen:** `docker compose ps`
2. **Logs einsehen:** `docker compose logs -f`
3. **Services neu starten:** `docker compose restart`
4. **Control Panel aufrufen:** `http://localhost:4000/controlpanel`


### Weiterführende Dokumentation

- **Hauptprojekt:** [connect4rv6l](https://github.com/wri-obernburg/connect4rv6l)
- **Troubleshooting Guide:** Siehe Hauptrepo-README

---

## 📄 Lizenz

Dieses Projekt steht unter der [MIT-Lizenz](https://choosealicense.com/licenses/mit/).

## 👥 Autoren

- **Tim Arnold** - [GitHub Profil](https://github.com/timarnoldev)

---

