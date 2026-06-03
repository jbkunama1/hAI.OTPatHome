# 🔐 hAI.OTPatHome – Self‑Hosted OTP @ Home

> **hAI.OTPatHome** ist ein schlankes Setup, um **TOTP‑Codes selbst zu hosten** – ideal als Ersatz für Handy‑Authenticator‑Apps.  
> Basis ist [2FAuth](https://github.com/Bubka/2FAuth), gepackt in einen Portainer‑Stack für dein Heimnetz. 🚀

![Logo](./logo_hAI.OTPat.png)

---

## 🛡️ Projekt-Status & Infos

![GitHub repo size](https://img.shields.io/github/repo-size/jbkunama1/hAI.OTPatHome?color=blue&label=Repo%20Size)
![GitHub license](https://img.shields.io/github/license/jbkunama1/hAI.OTPatHome?color=brightgreen)
![GitHub stars](https://img.shields.io/github/stars/jbkunama1/hAI.OTPatHome?style=social)
![Status](https://img.shields.io/badge/Status-✅%20läuft-brightgreen)

---

## 💡 Idee

- **Kein Handy nötig:** Zugriff auf deine TOTP‑Codes direkt im Browser (Desktop oder Mobile).
- **Self‑Hosted:** Läuft z. B. auf einem Debian/DietPi‑Server mit Docker & Portainer in deinem LAN.
- **Von außen erreichbar:** Optional über eigene Domain + Port‑Forwarding oder Reverse Proxy.
- **Einfacher Stack:** Ein Service, ein Volume, fertig.

Unter der Haube läuft [2FAuth](https://docs.2fauth.app/), eine spezialisierte Web‑App für TOTP‑Verwaltung.

---

## 🧱 Architektur

```text
+-------------------------+        +---------------------+
|   Client (Browser)      | <----> |   2FAuth Container  |
|  PC / Laptop / Tablet   |  HTTP  |   (hAI.OTPatHome)   |
+-------------------------+        +----------+----------+
                                            |
                                            v
                                  +------------------+
                                  |     Volume       |
                                  |  /opt/docker/    |
                                  |  2fauth/         |
                                  |  └── database/   |
                                  |      └── *.sqlite|
                                  +------------------+
```

- 2FAuth läuft im Container auf Port **8000**
- Extern erreichbar z. B. über Port **4444** oder via Reverse Proxy (443/HTTPS).

---

## ⚠️ Host-Verzeichnis vorbereiten (vor dem ersten Start)

> Docker würde das Verzeichnis sonst als `root` anlegen – der Container (UID/GID 1000) kann dann nicht schreiben.

```bash
# Unterordner anlegen – die SQLite-Datei liegt im Unterordner /database/!
mkdir -p /opt/docker/2fauth/database
chown -R 1000:1000 /opt/docker/2fauth
chmod -R 755 /opt/docker/2fauth
```

> ✅ Die Datei `database.sqlite` wird vom Container beim ersten Start **automatisch** angelegt –  
> nur der Ordner muss vorher existieren und beschreibbar sein.

---

## 🔑 APP_KEY – Pflichtformat `base64:`

> ⚠️ **Häufiger Fehler – führt zu HTTP 500 ohne Log-Eintrag!**

Der `APP_KEY` **muss** zwingend mit dem Prefix `base64:` beginnen. Fehlt dieses Prefix, crasht Laravel beim Start, bevor eine einzige Log-Zeile geschrieben wird.

```yaml
# ❌ FALSCH – kein Prefix, Laravel startet nicht
APP_KEY: "IrgendeinKey=="

# ✅ RICHTIG – mit base64:-Prefix
APP_KEY: "base64:DEIN_GENERIERTER_KEY_HIER=="
```

### Key generieren

```bash
# Methode 1: direkt im laufenden Container
docker exec -it 2fauth php artisan key:generate --show

# Methode 2: per openssl (Fallback)
echo "base64:$(openssl rand -base64 32)"
```

Den ausgegebenen Wert **inkl. `base64:`** in Portainer als `APP_KEY` eintragen, dann Stack neu deployen.

> 🛡️ **Sicherheit:** Den echten Key **niemals** ins Repository committen.  
> Trage ihn ausschließlich direkt in Portainer unter *Environment variables* ein.

---

## 🚀 Schnellstart (Portainer Stack / docker-compose)

```yaml
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
      # Beispiel LAN:    http://192.168.178.10:4444
      # Beispiel Domain: https://otp.deinedomain.de
      APP_URL: "http://DEINE-IP-ODER-DOMAIN:4444"
      APP_KEY: "base64:DEIN_GENERIERTER_KEY_HIER"
      APP_TIMEZONE: "Europe/Berlin"

      DB_CONNECTION: "sqlite"
      # ✅ Korrekter Pfad: Unterordner /database/ im Volume!
      DB_DATABASE: "/2fauth/database/database.sqlite"
```

### 🔧 Schritte

1. **Host-Verzeichnis anlegen** (siehe Abschnitt oben)
2. **APP_URL setzen**  
   - Nur LAN: `http://192.168.x.y:4444`  
   - Mit Domain + Reverse Proxy: `https://otp.deinedomain.de` (ohne Port)
3. **APP_KEY generieren** und mit `base64:`-Prefix in Portainer eintragen
4. Stack in **Portainer** deployen
5. 2FAuth im Browser aufrufen und Admin‑Konto anlegen

---

## 🧩 ENV‑Variablen (Übersicht)

| Variable        | Beispielwert                             | Beschreibung                                      |
|-----------------|------------------------------------------|---------------------------------------------------|
| `APP_NAME`      | `2FAuth`                                 | Name in der UI                                    |
| `APP_ENV`       | `production`                             | Laravel‑Umgebung                                  |
| `APP_DEBUG`     | `false`                                  | Debug‑Modus (nur zur Fehlersuche `true`)          |
| `APP_URL`       | `http://192.168.178.10:4444`             | Echte IP/Domain, kein Trailing-Slash!             |
| `APP_KEY`       | `base64:DEIN_KEY`                        | ⚠️ Pflicht mit `base64:`-Prefix! Nie ins Repo!    |
| `APP_TIMEZONE`  | `Europe/Berlin`                          | Zeitzone                                          |
| `DB_CONNECTION` | `sqlite`                                 | Datenbank‑Treiber                                 |
| `DB_DATABASE`   | `/2fauth/database/database.sqlite`       | ✅ Pfad inkl. Unterordner `/database/`            |

Weitere Optionen: [2FAuth‑Dokumentation](https://docs.2fauth.app/getting-started/config/)

---

## 🔐 Sicherheitshinweise

- **APP_KEY niemals ins Repository committen** – nur direkt in Portainer als ENV-Variable.
- **APP_KEY rotieren** nach dem Einrichten oder wenn er versehentlich sichtbar war:  
  `docker exec -it 2fauth php artisan key:generate --show`
- Stelle sicher, dass **APP_URL** korrekt gesetzt ist (inkl. `https` bei TLS).
- Nutze einen **Reverse Proxy** (z. B. Nginx Proxy Manager oder Traefik) für TLS-Termination.
- Regelmäßige Backups: `/opt/docker/2fauth/database/database.sqlite`

---

## 🧪 Roadmap / Ideen

- [ ] Beispiel‑Config für Nginx Proxy Manager / Traefik
- [ ] Optionaler Auth‑Proxy (z. B. Authelia/Authentik) vor 2FAuth
- [ ] Docker‑Healthcheck und einfache CI
- [ ] Tutorial speziell für Schul‑/Edu‑Setups 👨‍🏫

Pull Requests & Issues sind willkommen! 🙌

---

## 📜 Lizenz

Dieses Projekt steht unter der **MIT Lizenz** – siehe [`LICENSE`](./LICENSE) für Details.
