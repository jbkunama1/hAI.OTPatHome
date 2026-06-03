# 🔐 hAI.OTPatHome – Self‑Hosted OTP @ Home

> **hAI.OTPatHome** ist ein schlankes Setup, um **TOTP‑Codes selbst zu hosten** – ideal als Ersatz für Handy‑Authenticator‑Apps.  
> Basis ist [2FAuth](https://github.com/Bubka/2FAuth), gepackt in einen Portainer‑Stack für dein Heimnetz. 🚀

![Logo](./logo_hAI.OTPat.png)

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
- **Self‑Hosted:** Läuft z. B. auf einem Debian/DietPi‑Server mit Docker & Portainer in deinem LAN.
- **Von außen erreichbar:** Optional über eigene Domain + Port‑Forwarding oder Reverse Proxy.
- **Einfacher Stack:** Ein Service, ein Volume, fertig.

Unter der Haube läuft [2FAuth](https://docs.2fauth.app/), eine spezialisierte Web‑App für TOTP‑Verwaltung.

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
- Extern erreichbar z. B. über Port **4444** oder via Reverse Proxy (443/HTTPS).

---

## ⚠️ Wichtiger Hinweis: Host-Verzeichnis vorbereiten

> **Vor dem ersten Start** muss das Volume-Verzeichnis auf dem Host manuell angelegt werden –  
> sonst wird es von Docker als `root` erstellt und der Container (läuft als UID/GID 1000) kann nicht schreiben.

```bash
mkdir -p /opt/docker/2fauth
touch /opt/docker/2fauth/database.sqlite
chown -R 1000:1000 /opt/docker/2fauth
chmod 755 /opt/docker/2fauth
```

**Warum `/2fauth/database.sqlite` und nicht `/2fauth/database/database.sqlite`?**  
Der Unterordner `database/` wird vom Container nicht automatisch angelegt. Existiert er nicht,  
schlägt `touch /2fauth/database/database.sqlite` beim Start mit einem Fehler fehl.  
Der korrekte, robuste Pfad legt die Datei direkt im Volume-Root ab: `/2fauth/database.sqlite`.

---

## 🔑 APP_KEY – Pflichtformat `base64:`

> ⚠️ **Häufiger Fehler – führt zu HTTP 500 ohne Log-Eintrag!**

Der `APP_KEY` **muss** zwingend mit dem Prefix `base64:` beginnen. Fehlt dieses Prefix, crasht Laravel beim Start, bevor eine einzige Log-Zeile geschrieben wird.

```yaml
# ❌ FALSCH – kein Prefix, Laravel startet nicht
APP_KEY: "tBulrEZGeXq+garhNc6HJImRXY4SXeeni3yIFm97t84="

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
      APP_URL: "https://otp.deinedomain.de"
      APP_KEY: "base64:DEIN_GENERIERTER_KEY_HIER=="
      APP_TIMEZONE: "Europe/Berlin"

      DB_CONNECTION: "sqlite"
      DB_DATABASE: "/2fauth/database.sqlite"
```

### 🔧 Schritte

1. **Host-Verzeichnis anlegen** (siehe Hinweis oben)
2. **Volume‑Pfad anpassen**  
   `/opt/docker/2fauth` auf deinen Wunschpfad ändern.
3. **APP_URL setzen**  
   - Nur LAN: `http://192.168.x.y:4444`  
   - Mit Domain + Reverse Proxy: `https://otp.deinedomain.de` (ohne Port!)
   - Direktzugriff ohne Proxy: `https://otp.deinedomain.de:4444`
4. **APP_KEY generieren und mit `base64:`-Prefix eintragen** (siehe Abschnitt oben)
5. Stack in **Portainer** als neuen Stack anlegen, `docker-compose.yml` einfügen oder Datei referenzieren.
6. 2FAuth im Browser aufrufen und ein Admin‑Konto anlegen.

---

## 🧩 ENV‑Variablen (Übersicht für Portainer)

| Variable        | Beispielwert                          | Beschreibung                                        |
|-----------------|---------------------------------------|-----------------------------------------------------|
| APP_NAME        | `2FAuth`                              | Name in der UI                                      |
| APP_ENV         | `production`                          | Laravel‑Umgebung                                    |
| APP_DEBUG       | `false`                               | Debug‑Modus                                         |
| APP_URL         | `https://otp.deinedomain.de`          | Öffentliche URL (ohne Port bei Proxy)               |
| APP_KEY         | `base64:DEIN_KEY==`                   | ⚠️ Pflicht mit `base64:`-Prefix! Nie ins Repo!      |
| APP_TIMEZONE    | `Europe/Berlin`                       | Zeitzone                                            |
| DB_CONNECTION   | `sqlite`                              | Datenbank‑Treiber                                   |
| DB_DATABASE     | `/2fauth/database.sqlite`             | DB‑Dateipfad direkt im Volume-Root                  |

Weitere Optionen findest du in der offiziellen [2FAuth‑Dokumentation zu Environment‑Variablen](https://docs.2fauth.app/getting-started/config/).

---

## 🔐 Sicherheitshinweise

- **APP_KEY niemals ins Repository committen** – nur direkt in Portainer als ENV-Variable.
- Stelle sicher, dass **APP_URL** korrekt gesetzt ist (inkl. `https` bei TLS).
- Nutze einen **Reverse Proxy** (z. B. Nginx Proxy Manager oder Traefik) für TLS‑Termination und zusätzliche Access‑Control.
- Erstelle regelmäßige Backups des Volumes `/opt/docker/2fauth` (insb. `database.sqlite` & Konfiguration).
- Verwende starke Passwörter und, wenn möglich, separate Accounts für Familie/Team.

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
