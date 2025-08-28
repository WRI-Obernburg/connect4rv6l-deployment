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


## 🎯 Überblick

Dieses Deployment-Repository stellt eine produktionsreife Docker-Umgebung für das Connect4 RV6L-System bereit. Es orchestriert alle notwendigen Services:

- **Backend Server** (Ports 3000, 4000)
- **Mobile Frontend** (Port 8080)
- **Control Panel** (integriert)
- **Local Display** (integriert)
- **Cloudflare Tunnel**

### Quicklinks (Production)
  - [Controlpanel](http://rv6l-application.local:4000/control)
  - [Mobilefrontend](https://connect4rv6l.vercel.app)
  - [Localfrontend](http://rv6l-application.local:4000/localfrontend)
  - [Localfrontend Indoor](http://rv6l-application.local:4000/localfrontend?indoor)

## 🔀 Betriebsmodi (Pfad A/B)

- **Pfad A – Nur Kiosk:** Das Local Frontend (Kiosk) läuft auf einem Displaygerät und verbindet sich mit einem externen Backend.

- **Pfad B – Backend auf dem Pi:** Backend + interne Web-UIs laufen auf einem Raspberry Pi (mDNS-Hostname `rv6l-application.local`).

> Hinweis: Der Backend Pi steuert den RV6L und stellt optional noch ein Display bereit (Pfad B), während bei Pfad A nur das Display angezeigt wird. Daraus ergibt sich, dass immer nur ein Pi mit Pfad B Konfiguration existieren darf, während beliebig viele Pfad A Geräte eingerichtet sein dürfen. Diese müssen dann nur einen anderen Hostname haben!

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

- Raspberry Pi Imager laden, Raspberry Pi OS (OHNE Desktop) auf SD-Karte schreiben.
- Im Imager „Erweiterte Optionen“ aktivieren: SSH einschalten, Benutzer/Passwort setzen, optional Hostname.
  Passwort ist auf dem Makerspace Laufwerk im RV6L-Ordner zu finden!
- Erststart: Mit LAN verbinden oder WLAN über `raspi-config` einrichten.
- Nach `sudo raspi-config` unter System -> S6 Auto Login aktivieren

```bash
sudo raspi-config
```



#### 2) Installation

Auf Raspberry Pi OS wird Chromium genutzt (Chrome-Äquivalent).

```bash
sudo apt update

# Desktopumgebung installieren
sudo apt-get install --no-install-recommends xserver-xorg x11-xserver-utils xinit openbox -y


# Chromeium installieren
sudo apt-get install --no-install-recommends chromium-browser -y


```

#### 3) Kiosk-Autostart einrichten

Ziel-URL des Kiosks:
- `http://rv6l-application.local:4000/localfrontend`

Autostart Datei bearbeiten `sudo nano /etc/xdg/openbox/autostart`


```bash
# Bildschirmschoner / Bildschirmabschaltung / Energieverwaltung deaktivieren

xset s off

xset s noblank

xset -dpms

# Beenden des X-Servers mit STRG-ALT-Rücktaste erlauben

setxkbmap -option terminate:ctrl_alt_bksp

# Chromium im Kiosk-Modus starten

sed -i 's/"exited_cleanly":false/"exited_cleanly":true/' ~/.config/chromium/'Local State'

sed -i 's/"exited_cleanly":false/"exited_cleanly":true/; s/"exit_type":"[^"]\+"/"exit_type":"Normal"/' ~/.config/chromium/Default/Preferences

chromium-browser --disable-infobars --noerrdialogs --incognito --check-for-update-interval=1 --simulate-critical-update --kiosk 'http://rv6l-application.local:4000/localfrontend'
```

Desktopumgebung starten


Am Ende der .profile Datei mit `sudo nano .profile` folgendes hinzufügen.

```bash

[[ -z $DISPLAY && $XDG_VTNR -eq 1 ]] && startx -- -nocursor

```

Handelt es sich bei dem Monitor um ein Indoor-Monitor, muss für die korrekte Spielfeld-Orientierung ein ?indoor an die URL angehängt werden.


### Pfad B: Vollständiges Backend (+ optionaler Kiosk Modus)

Dieser Modus betreibt Backend und lokale UIs direkt auf einem Raspberry Pi. Standard-Hostname und Zugangsdaten:

- Hostname: `rv6l-application.local`
- Username: `wri`
- Passwort: `xxxxxxxxxxx`

Passwort ist auf dem Makerspace Laufwerk im RV6L-Ordner zu finden.

#### 1) Neuen Raspberry Pi einrichten

- Raspberry Pi Imager herunterladen (mit Desktop Umgebung!) und SD-Karte schreiben.
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

> Hinweis (Pfad B): Stellen Sie in jedem Fall sicher, dass der Pi auf `eth0` eine statische IP hat, wenn der Roboter direkt per Ethernet verbunden ist.


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

sudo apt install -y chromium-browser

```

#### 5) Optional: Kiosk Autostart auf dem Pi

Erstelle `webbrowser.desktop` in `/etc/xdg/autostart/` mit dem Befehl `sudo nano /etc/xdg/autostart/webbrowser.desktop`

```
[Desktop Entry]
Type=Application
Name=Kiosk Autostart
Exec=sh -c "unclutter & xscreensaver -no-splash & xset s off & xset -dpms & xset s noblank & chromium-browser http://rv6l-application.local:4000/localfrontend --start-fullscreen --kiosk --incognito --noerrdialogs --no-first-run --disk-cache-dir=/dev/null"
X-GNOME-Autostart-enabled=true
```

Erteile die Ausführberechtigung: `sudo chmod +x /etc/xdg/autostart/webbrowser.desktop`

> Handelt es sich bei dem Monitor um ein Indoor-Monitor, muss für die korrekte Spielfeld-Orientierung ein ?indoor an die URL angehängt werden.

## ⚙️ Konfiguration
Nur bei Pfad B

### Umgebungsvariablen

Erstellen Sie eine `.env` Datei oder setzen Sie die Variablen direkt:

| Variable            | Beschreibung | Standard | Beispiel |
|---------------------|-------------|----------|----------|
| `FRONTEND_ADDRESS`  | Frontend-URL für QR-Codes | `https://connect4rv6l.vercel.app/` | `http://192.168.1.100:8080` |
| `CLOUDFLARED_TOKEN` | Token für Cloudflare Tunnel | - | `eyJhIjoiXXX...` |





## 🎮 Betrieb (Nur Pfad B)

### Container starten

```bash
cd connect4rv6l-deployment

# Alle Services starten
docker compose up -d

```

### Status überwachen

```bash
cd connect4rv6l-deployment

# Container-Status
docker compose ps

# Ausführlich mit Ports
docker compose ps --format "table {{.Name}}\t{{.Status}}\t{{.Ports}}"

# Resource-Verbrauch
docker stats
```

### Logs einsehen

```bash
cd connect4rv6l-deployment

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
# Ins richtige Verzeichnis wechseln
cd connect4rv6l-deployment

# Images neu ziehen
docker compose down --rmi 'all'
docker compose up -d
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

# Auf Raspberry Pi aufschalten:

```bash
ssh wri@<hostname>.local

# Für das Backend
ssh wri@rv6l-application.local

# Frontend
-> Hostname im Passwort Dokument im Makerspace Laufwerk nachschlagen
```

### Raspberry Pi Herunterfahren
```bash
# Nur Pfad B/Backend Pi
cd connect4rv6l-deployment

# Nur Pfad B/Backend Pi
docker compose down --rmi 'all'

sudo halt

```

### Raspberry Pi Hochfahren 
```bash

# Nur Pfad B/Backend Pi
cd connect4rv6l-deployment

# Nur Pfad B/Backend Pi
docker compose up

```


### Routine-Updates

```bash

cd connect4rv6l-deployment

# 1. Service stoppen
docker compose down --rmi 'all'

# 2. Services mit neuen Images neu starten
docker compose up -d

# 3. System-Updates
sudo apt update && sudo apt upgrade -y  # Ubuntu/Debian

```

### Komplett-Reset

```bash
cd connect4rv6l-deployment

# Container löschen
docker compose down --rmi 'all'

# Neustart von null
docker compose up -d
```


## 🔒 Sicherheit

### Netzwerk-Sicherheit

- **Interne Ports:** 3000, 4000 nur im lokalen Netzwerk verfügbar
- **Externe Erreichbarkeit:** Nur über konfigurierten Cloudflare Tunnel
- **Roboter-Netzwerk:** Isolierte Verbindung zu 192.168.2.1
- **Abgeschottete Containerarchitektur:** Externe Verbindungen kommen nur im Container an



## Support

1. **Container-Status prüfen:** `docker compose ps`
2. **Logs einsehen:** `docker compose logs -f`
3. **Services neu starten:** `docker compose restart`
4. **Control Panel aufrufen:** `http://localhost:4000/controlpanel`


### Weiterführende Dokumentation

- **Hauptprojekt:** [connect4rv6l](https://github.com/wri-obernburg/connect4rv6l)
- **Troubleshooting Guide:** Siehe Hauptrepo-README

---


## 👥 Autoren

- **Tim Arnold** - [GitHub Profil](https://github.com/timarnoldev)

---

