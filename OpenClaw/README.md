# OpenClaw – Installation und Betrieb

**Dokumentationsstand:** 12.11.2026  
**Status:** OpenClaw installiert und mit OpenRouter verbunden. Diese Dokumentation basiert auf den tatsächlich besprochenen Schritten; nicht belegte Schritte sind als Anleitung für eine Neuinstallation gekennzeichnet.

## 1. Installationsübersicht

| Merkmal | Stand |
| --- | --- |
| Plattform | Proxmox VE, eigene VM |
| Betriebssystem | Debian 13 |
| Hostname | `openclaw` |
| Ausführung | Direkt auf der VM, **ohne Docker** |
| Benutzer | SSH-/Administrationszugang: `markus`; OpenClaw-Betriebsbenutzer: `openclaw` |
| Nachgewiesene OpenClaw-Version | `2026.11.2 (b556fdf)`, Stand 12.11.2026 |
| KI-Anbieter | OpenRouter (API-Key eingerichtet) |
| Konfiguration | `~/.openclaw/openclaw.json` im Home-Verzeichnis des Betriebsbenutzers |
| Agent-Daten | `~/.openclaw/agents/main/agent/openclaw-agent.sqlite` |
| Konfigurationsbackup | `~/.openclaw/openclaw.json.bak` wurde beim Modellwechsel angelegt |
| Gateway-Standardport | TCP **18789** (tatsächliche Bind-Adresse/Port auf der VM prüfen) |

> Sicherheit: Dieses Repository ist öffentlich. IP-Adressen, API-Schlüssel, Tokens, Passwörter und vollständige produktive Konfigurationsdateien sind **nicht** Bestandteil dieser Dokumentation.

## 2. Installation einer neuen VM (Referenz, kein belegtes Installationsprotokoll)

Der ursprüngliche konkrete Installationsbefehl ist in den verfügbaren Gesprächsnotizen nicht nachgewiesen. Für eine Neuinstallation beschreibt die offizielle Anleitung aktuell:

1. Debian 13 als eigene VM bereitstellen und einen nichtprivilegierten Betriebsbenutzer vorsehen.
2. Voraussetzungen prüfen: `node --version`, `npm --version`. Laut aktueller Dokumentation: Node.js 24.16.0+ (24.x) oder 26.1.0+; die dort empfohlenen Security-Patchstände beachten.
3. Offiziellen Installer **vor Ausführung prüfen** und gemäß Herstelleranleitung einsetzen:

   ```bash
   curl -fsSL https://openclaw.ai/install.sh -o /tmp/openclaw-install.sh
   less /tmp/openclaw-install.sh
   bash /tmp/openclaw-install.sh
   ```

4. Onboarding durchführen, OpenRouter als Anbieter konfigurieren und API-Key **nur über die vorgesehene geheime Konfiguration** hinterlegen.
5. Wenn der Gateway noch nicht als Dienst eingerichtet wurde:

   ```bash
   openclaw gateway install
   openclaw gateway start
   ```

6. Status prüfen:

   ```bash
   openclaw --version
   openclaw doctor
   openclaw gateway status
   ```

**Wichtig:** Diese Referenzbefehle sind nicht als Nachweis zu verstehen, dass die vorhandene VM genauso installiert wurde.

Offizielle Dokumentation: https://docs.openclaw.ai/start/getting-started und https://docs.openclaw.ai/install

## 3. Nachgewiesene OpenRouter-Konfiguration

OpenRouter ist als KI-Provider angebunden. Am 12.11.2026 wurde die Nutzung kostenloser OpenRouter-Modelle und von Fallbacks behandelt.

Ein tatsächlich ausgeführter Befehl war:

```bash
openclaw models set openrouter/openrouter/free
```

Die CLI meldete, dass der Anbieter `openrouter` keinen lokalen Modellkatalog hat und das Modell deshalb offline nicht überprüft werden konnte. **Die Auswahl wurde dennoch gespeichert** (Anzeige `Default model: openrouter/free`).

Außerdem wurde folgender Fallback-Befehl aufgerufen beziehungsweise in der laufenden Sitzung dokumentiert:

```bash
openclaw models fallbacks add openrouter/nvidia/nemotron-3-ultra-550b-a55b:free
```

Der endgültige Erfolg dieses Fallback-Befehls ist aus dem vorliegenden Ausschnitt **nicht bestätigt**. Ein vorheriger Status zeigte `openrouter/auto` als Standard und null Fallbacks; für den aktuellen Zustand daher **nicht auf alte Werte verlassen**.

Aktuellen Ist-Zustand direkt auf der VM abfragen:

```bash
sudo -iu openclaw
openclaw models status
openclaw models fallbacks list
openclaw --version
```

Weitere damals diskutierte freie Modelle (nicht als installiert bestätigt):

- `openrouter/cohere/north-mini-code:free`
- `openrouter/google/gemma-4-31b-it:free`

### Kostenabsicherung

`:free` bezeichnet eine Modellvariante bei OpenRouter, **keine dauerhafte Preisgarantie**. Modellverfügbarkeit, Limits und Tarife können sich ändern. Für einen möglichst kostenfreien Betrieb regelmäßig das tatsächliche Modell und die OpenRouter-Abrechnungs-/Nutzungseinstellungen prüfen. API-Schlüssel niemals in Git committen.

## 4. Betrieb, Zugang und Fehlersuche

Auf der VM als Benutzer `openclaw`:

```bash
openclaw --version
openclaw gateway status
openclaw models status
openclaw doctor
```

Falls eine systemd-User-Unit verwendet wird, kann der Dienstzustand ergänzend mit `systemctl --user` kontrolliert werden. Ein konkreter Unit-Name ist hier bewusst nicht behauptet, weil er noch nicht verifiziert wurde.

Das Gateway verwendet standardmäßig **Port 18789**. Vor einem Netzwerkzugriff zuerst die effektive Konfiguration und Bind-Adresse prüfen. Für Fernzugriff bevorzugt SSH-Tunnel/VPN verwenden; **keinen ungeschützten Gateway-Port ins Internet freigeben**.

Die Konfiguration liegt relativ zum Home-Verzeichnis des ausführenden Benutzers:

```bash
echo "$HOME"
ls -la ~/.openclaw/
```

Vor dem Teilen von Diagnoseausgaben Authentifizierungsdaten, Tokens, URLs mit Zugangsdaten und private Kundendaten entfernen.

## 5. Integration und geplanter Ausbau

- **OpenHands:** Ein Multiprojekt-Setup und SSH-basierte Tests wurden im Oktober 2026 bearbeitet. Ein späterer End-to-End-Test der OpenHands-API scheiterte an einer zurückgesetzten Verbindung bei `POST /api/conversations`. Die Integration gilt damit **nicht** als erfolgreich abgeschlossen.
- **Gitea:** Eine spätere Anbindung wurde geplant; hier ist keine vollständige produktive Integration belegt.
- **Trading-Simulator:** OpenClaw soll für Analysen eingesetzt werden; ein separater lokaler Simulator wurde geplant/eingerichtet. Diese Funktion ist **nicht Teil der Grundinstallation** und kein Nachweis für einen produktiven automatischen Handel.
- **Desktop-Client:** Erprobung wurde besprochen; der produktive Status ist nicht abschließend nachgewiesen.

## 6. Noch zu verifizieren

- [ ] Ursprüngliche Installationsmethode (Installer, npm oder andere) dokumentieren
- [ ] Node.js- und npm-Version erfassen
- [ ] Tatsächlichen Gateway-Listen-Port und Bind-Adresse prüfen
- [ ] Aktuellen OpenRouter-Standard und die Fallback-Liste dokumentieren
- [ ] Gateway-Autostart, Benutzerrechte und Erreichbarkeit prüfen
- [ ] Backup- und Wiederherstellungstest für OpenClaw-Konfiguration und Agent-Daten durchführen
- [ ] OpenHands-/Gitea-Integration getrennt dokumentieren und testen

## 7. Versions- und Änderungsprotokoll

| Datum | Festgestellter Stand |
| --- | --- |
| 19.10.2026 | Debian-13-VM und OpenClaw/OpenHands-Multiprojektarbeiten; SSH-Tests teils erfolgreich, API-End-to-End-Test fehlgeschlagen |
| 10.11.2026 | OpenClaw installiert, OpenRouter angebunden |
| 12.11.2026 | OpenClaw `2026.11.2 (b556fdf)`; kostenlose OpenRouter-Modellwahl bearbeitet |
| 12.11.2026 | Erste strukturierte Installations- und Betriebsdokumentation erstellt |

---

**Hinweis:** Diese Datei enthält bewusst keine produktiven Secrets und ist zugleich Bestandsaufnahme und Reinstallationshilfe. Für eine exakte 1:1-Wiederherstellung sind zusätzlich gesicherte Konfiguration und verifizierte Laufzeitdetails nötig.
