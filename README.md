# 🔐 hAI.OTPatHome – Self‑Hosted OTP @ Home

> **hAI.OTPatHome** ist ein schlankes Setup, um **TOTP‑Codes selbst zu hosten** – ideal als Ersatz für Handy‑Authenticator‑Apps.  
> Basis ist [2FAuth](https://github.com/Bubka/2FAuth), gepackt in einen Portainer‑Stack für dein Heimnetz. 🚀

![Logo](./logo-hai-otpathome.svg)

---

## 🛡️ Projekt-Status & Infos

![GitHub repo size](https://img.shields.io/github/repo-size/jbkunama1/hAI.OTPatHome?color=blue&label=Repo%20Size)
![GitHub license](https://img.shields.io/github/license/jbkunama1/hAI.OTPatHome?color=brightgreen)
![GitHub stars](https://img.shields.io/github/stars/jbkunama1/hAI.OTPatHome?style=social)
![CI Status](https://img.shields.io/badge/CI-coming%20soon-orange)

> **Hinweis:** Ersetze den GitHub‑User `jbkunama1` in den Badge‑URLs durch deinen eigenen, falls du das Repo forken/umbenennen möchtest.

---

## 💡 Idee

- **Kein Handy nötig:** Zugriff auf deine TOTP‑Codes direkt im Browser (Desktop oder Mobile).
- **Self‑Hosted:** Läuft z. B. auf einem Debian/DietPi‑Server mit Docker & Portainer in deinem LAN.
- **Von außen erreichbar:** Optional über eigene Domain + Port‑Forwarding oder Reverse Proxy.
- **Einfacher Stack:** Ein Service, ein Volume, fertig.

Unter der Haube läuft [2FAuth](https://docs.2fauth.app/), eine spezialisierte Web‑App für TOTP‑Verwaltung.[web:33][web:44]

---

## 🧱 Architektur

```text
+-------------------------+        +---------------------+
|   Client (Browser)      | <----> |   2FAuth Container  |
|  PC / Laptop / Tablet   |  HTTPS |   (hAI.OTPatHome)   |
+-------------------------+        +----------+----------+
                                            |
                                            v
                                      +-----------+
                                      |  Volume   |
                                      |  /2fauth  |
                                      +-----------+
```

- 2FAuth läuft im Container auf Port **8000**
- Extern erreichbar z. B. über Port **4444** oder via Reverse Proxy (443/HTTPS).[web:33][web:35]

---

## 🚀 Schnellstart (Portainer Stack / docker-compose)

```yaml
version: "3.8"

services:
  2fauth:
    image: 2fauth/2fauth:latest
    container_name: 2fauth
    restart: unless-stopped

    ports:
      - "4444:8000"

    volumes:
      - /opt/docker/2fauth:/2fauth

    environment:
      APP_NAME: "2FAuth"
      APP_ENV: "production"
      APP_DEBUG: "false"
      APP_URL: "https://otp.deinedomain.de:4444"
      APP_KEY: "base64:DEIN_APP_KEY_HIER"
      APP_TIMEZONE: "Europe/Berlin"

      DB_CONNECTION: "sqlite"
      DB_DATABASE: "/2fauth/database/database.sqlite"
```

### 🔧 Schritte

1. **Volume‑Pfad anpassen**  
   `/opt/docker/2fauth` auf deinen Wunschpfad ändern.
2. **APP_URL setzen**  
   - Nur LAN: `http://192.168.x.y:4444`  
   - Mit Domain: `https://otp.deinedomain.de:4444` oder hinter Reverse Proxy ohne Port.[web:33][web:45]
3. **APP_KEY generieren**  
   Mit einem Laravel‑Key‑Generator (z. B. lokal mit `php artisan key:generate --show`) einen `base64:`‑Key erzeugen und einsetzen.[web:33][web:37]
4. Stack in **Portainer** als neuen Stack anlegen, `docker-compose.yml` einfügen oder Datei referenzieren.
5. 2FAuth im Browser aufrufen und ein Admin‑Konto anlegen.

---

## 🌐 GitHub Pages – Projektseite

Dieses Repository enthält eine einfache `index.html`, die direkt als GitHub Pages Startseite dienen kann.  
Aktiviere GitHub Pages z. B. unter **Settings → Pages → Source: `main` / `/root`**.

Die Seite zeigt:

- Projektlogo
- Kurze Erklärung & Features
- Code‑Snippet für den Stack
- Links zur 2FAuth‑Doku und zu deinem Heim‑Setup

---

## 🧩 ENV‑Variablen (Übersicht für Portainer)

| Variable        | Beispielwert                          | Beschreibung                          |
|-----------------|---------------------------------------|---------------------------------------|
| APP_NAME        | `2FAuth`                              | Name in der UI                        |
| APP_ENV         | `production`                          | Laravel‑Umgebung                      |
| APP_DEBUG       | `false`                               | Debug‑Modus                           |
| APP_URL         | `https://otp.deinedomain.de:4444`     | Öffentliche URL von 2FAuth            |
| APP_KEY         | `base64:…`                            | Verschlüsselungs‑Key                  |
| APP_TIMEZONE    | `Europe/Berlin`                       | Zeitzone                              |
| DB_CONNECTION   | `sqlite`                              | Datenbank‑Treiber                     |
| DB_DATABASE     | `/2fauth/database/database.sqlite`    | DB‑Dateipfad im Container             |

Weitere Optionen findest du in der offiziellen 2FAuth‑Dokumentation zu Environment‑Variablen.[web:34][web:37]

---

## 🔐 Sicherheitshinweise

- Stelle sicher, dass **APP_URL** korrekt gesetzt ist (inkl. `https` bei TLS).  
- Nutze einen **Reverse Proxy** (z. B. Nginx Proxy Manager oder Traefik) für TLS‑Termination und zusätzliche Access‑Control.[web:42]
- Erstelle regelmäßige Backups des Volumes `/2fauth` (insb. Datenbank & Konfiguration).
- Verwende starke Passwörter und, wenn möglich, separate Accounts für Familie/Team.

---

## 🧪 Roadmap / Ideen

- [ ] Beispiel‑Config für Nginx Proxy Manager / Traefik  
- [ ] Optionaler Auth‑Proxy (z. B. Authelia/Authentik) vor 2FAuth  
- [ ] Docker‑Healthcheck und einfache CI  
- [ ] Tutorial speziell für Schul‑/Edu‑Setups 👨‍🏫

Pull Requests & Issues sind willkommen! 🙌

---

## 📜 Lizenz

Dieses Projekt steht unter der **MIT Lizenz** – siehe [`LICENSE`](./LICENSE) für Details.
