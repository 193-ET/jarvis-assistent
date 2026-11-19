# Projekt Jarvis – Sprachassistent mit lokaler KI

*Stand: November 2026*

Von Oktober bis November 2026 habe ich auf meinem Heim-Server einen eigenen Sprachassistenten („Jarvis“) mit einem lokal laufenden Sprachmodell aufgebaut und rund um die Uhr betrieben – ohne Cloud-KI, auf einer Consumer-Grafikkarte (AMD RX 580, 8 GB).

## Ablauf einer Anfrage

Eine gesprochene Frage war nach wenigen Sekunden beantwortet, ohne dass Daten das Haus verlassen mussten (außer für die Sprachausgabe).

1. **Wake-Word:** Zwei Mikrofone (Server-PC und Tablet) hören dauerhaft auf „Hey Jarvis“ (openWakeWord).
2. **Zusammenführen:** Hören beide Mikrofone denselben Satz, wählt ein eigener Dienst die wahrscheinlich richtige Version aus.
3. **Spracherkennung:** Whisper (lokal, Deutsch) macht daraus Text.
4. **Home Assistant** übergibt den Text an mein „Gehirn“ – einen eigenen Zwischendienst, der sich für Home Assistant wie ein normaler Ollama-Server ausgibt.
5. **Gehirn:** sucht passende Erinnerungen (Vektorsuche), fügt Profil, Datum/Uhrzeit, Schul- und Kalenderdaten hinzu und lässt exakte Dinge (Rechnen, Uhrzeiten) regelbasiert lösen statt raten.
6. **Lokales Sprachmodell** (Ollama, Qwen3 4B) antwortet oder ruft Werkzeuge auf (Licht, Heizung, Fernseher, Schule, Kalender).
7. **Kontrolle:** Eine Regel-Prüfung korrigiert Rechenfehler, Englisch oder Floskeln, bevor die Antwort rausgeht.
8. **Sprachausgabe** über den Google Nest Mini (Google Cloud TTS, Stimme Chirp 3 HD).

## Hardware und Software

Alles lief auf einem einzigen Windows-PC, der gleichzeitig die Vereins-Webseite hostet.

| Bereich | Eingesetzt |
| --- | --- |
| Server | Intel i5-9600K, 16 GB RAM, AMD Radeon RX 580 (8 GB), Windows 11 |
| Sprachmodell | Qwen3 4B Instruct über Ollama (Vulkan-Backend), 12.288 Token Kontext |
| Erinnerungen | nomic-embed-text (Embeddings) + SQLite |
| Smart Home | Home Assistant in Docker (Bosch-Heizung, Ledvance, Lepro, BSEED, Google TV) |
| Sprache | openWakeWord, Whisper (STT), Google Cloud Chirp 3 HD (TTS) |
| Wissen offline | Kiwix mit kompletter Wikipedia (Englisch 52,7 GB, Deutsch 18,6 GB) |
| Telefonie | eigener SIP-Client über die FRITZ!Box (echte Anrufe aufs Handy) |
| Apps | eigene Android-Apps (HUD und Smartring-Anbindung), Tablet als Steuertafel |
| Zugriff von außen | nur LAN + Tailscale, Home Assistant nie offen im Internet |

## Bausteine

Das „Gehirn“ umfasste rund 6.000 Zeilen Python (asyncio/aiohttp) in einem eigenen Docker-Container.

| Baustein | Aufgabe |
| --- | --- |
| Ollama-kompatibler Proxy | fängt jede Anfrage von Home Assistant ab, reichert sie an und kontrolliert die Antwort |
| Gedächtnis | speichert jede Frage und Antwort mit Embedding; Vektorsuche nach ähnlichen früheren Gesprächen |
| „Pinnwand“ | Wissensgraph über den Nutzer (Kategorien, Personen, Orte, Ereignisse), im Browser als Netz darstellbar |
| Rechner | exakte Mathematik und Uhrzeitrechnung mit SymPy statt Raten durch das Modell |
| Antwort-Prüfung | Regeln erkennen Rechenfehler, Englisch, Ablehnungen ohne Werkzeug; Antwort wird einmal neu geschrieben |
| Lernen | merkt Korrekturen und Wiederholungen, arbeitet daraus Lektionen aus |
| Nachtschicht | 01–06 Uhr: Tages-, Wochen- und Monatsrückblicke, Konsistenzprüfung, Selbststudium aus der Offline-Wikipedia |
| Schul-Melder | liest WebUntis und itslearning (nur lesen) und meldet nur Wichtiges |
| Nachrichten | sammelt lokale/regionale Meldungen und ordnet ihre Wichtigkeit ein |
| Selbst-Update | übernimmt neuen Code im laufenden Betrieb, mit Absturzschutz und automatischem Zurücksetzen |
| Wächter | überwachen Webseite, Home Assistant und KI und reparieren sie selbstständig, mit Push-/Mail-Meldung |

## Probleme und Lösungen

Die meiste Arbeit steckte nicht im ersten Aufbau, sondern im stabilen Dauerbetrieb.

| Problem | Ursache | Lösung |
| --- | --- | --- |
| Antworten brachen ab | Gespräch größer als der Kontext (9.092 > 8.192 Token) | Kontext auf 12.288 erhöht, überlange Anfragen automatisch gekürzt |
| Falsche Uhrzeit-Rechnungen | Sprachmodelle rechnen unzuverlässig | eigener Rechner + automatische Nachkontrolle jeder Antwort |
| Server fror nachts ein | RX 580 hängte unter Dauerlast der Nachtschicht (Kernel-Power 41 im Ereignisprotokoll) | Last zwischen GPU und CPU aufgeteilt (18 von 36 Schichten auf der GPU), später nachts nur CPU |
| KI nach Update weg | Ollama startete sich nach seinem Auto-Update nicht neu | Wächter startet es nach 3 Minuten automatisch und meldet es |
| Arbeitsspeicher dauerhaft ausgelastet | KI-Modell, Docker-Dienste und Webseite teilten sich 16 GB RAM | Modell wird nach 10 Minuten entladen und beim Wake-Word vorgewärmt; Docker-RAM begrenzt |
| Ungeplante Neustarts | Windows-Updates starteten den Server neu | Updates nur noch am Wochenende nach Zustimmung, Container-Updates samstags |

## Ergebnis und Entscheidung

Die lokale KI lief, musste aber aus Kosten- und Ressourcengründen weichen: Am 18.11.2026 habe ich sie durch Claude im Hintergrund ersetzt.

- **Gemessen:** Mit Teilauslagerung auf die CPU lieferte das Modell rund 11,8 Token pro Sekunde bei 2,4 GB Grafikspeicher.
- **Gegen lokal:** Für einen stabilen Betrieb hätte der Server mehr Arbeitsspeicher und eine stärkere Grafikkarte gebraucht. Allein mehr RAM hätte gebraucht etwa 50 € gekostet, die Grafikkarte wäre trotzdem an ihrer Grenze geblieben. Claude lief dagegen über mein bestehendes Abo ohne Zusatzkosten – schneller und klüger als das kleine lokale Modell. Dazu kamen Abstürze der Grafikkarte unter Dauerlast und über 75 GB Speicherplatz für Modelle und Offline-Wikipedia.
- **Vereins-Webseite in Gefahr:** Derselbe Server hostet die Webseite des gemeinnützigen Vereins Russlingo Kids e.V. (Russischunterricht für Kinder in Bremen, [russlingo-kids.de](https://russlingo-kids.de)), die ich ehrenamtlich aufgebaut habe und betreibe. Die Abstürze und Lastspitzen der lokalen KI haben ihre Stabilität stark gefährdet – und die Webseite hat Vorrang vor einem Hobbyprojekt.
- **Umstieg:** Das Gehirn spricht jetzt mit Claude statt mit Ollama, gibt sich gegenüber Home Assistant aber weiter als Ollama aus. Geräte, Apps und Automationen liefen dadurch ohne Änderung weiter, Antworten kamen in etwa 2–4 Sekunden.

## Was ich gelernt habe

Das Projekt hat mir gezeigt, wie man ein System aus vielen Diensten dauerhaft stabil hält.

- Lokale Sprachmodelle betreiben und anpassen (Kontextgröße, GPU/CPU-Aufteilung, Laden und Entladen)
- Docker, Home Assistant, Netzwerk und VPN (Tailscale) auf einem Windows-Server
- Fehlersuche über Logs und das Windows-Ereignisprotokoll, Ursachen statt Symptome beheben
- Selbstheilung: Wächter, automatische Neustarts, Updates im laufenden Betrieb
- Abwägen von Kosten, Hardware und Nutzen – und eine Lösung bewusst wieder abzulösen

**Wie es entstanden ist:** Idee, Anforderungen, Aufbau der Hardware, Tests und Betrieb lagen bei mir. Den Großteil des Codes habe ich gemeinsam mit einer KI (Claude) als Programmier-Assistent geschrieben.
