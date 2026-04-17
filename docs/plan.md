# Dokbot — Projektplan

## Context

Dokbot ist ein KI-gestützter Wissensverwalter für Organisationen (10–50 Personen). Er nimmt internes Know-how entgegen, strukturiert es als Markdown in Git und macht es für Menschen und Maschinen zugänglich.

**Lizenz**: AGPL-3.0 (Community), proprietär (Pro). Open-Core-Modell.

---

## Vision & Kernfunktionen

1. **Informationsaufnahme** — Wissen aus verschiedenen Quellen (Menschen, KI-Agenten, Skripte), Erkennung von Lücken mit Rückfragen
2. **Strukturierung** — Automatisches Ordnen, Kategorisieren und Dokumentieren als Markdown
3. **Revisionskontrolle** — Wer hat wann was geändert, Versionierung aller Dokumente
4. **Freigabe-Workflows** — Prozessänderungen mit Freigabe, Know-how/Korrekturen ohne
5. **Menschliche Interfaces** — Markdown-Dokumente, (sekundär) Graph-Visualisierungen
6. **Maschinen-Interfaces** — MCP-Server, Dateizugriff, REST-API
7. **Auswertungen** — Dashboard, Änderungsübersicht, Wissensstatistiken

### Scope-Abgrenzung

| In Scope | Out of Scope |
|----------|-------------|
| Internes Organisationswissen | Kundenspezifische Projektdaten |
| Leistungserbringung, Fakturierung, Abläufe | Externe Informationen |
| Technische/fachliche Umsetzungsdokumentation | Kundenprojekt-Details |

---

## MVP-Scope (Community Edition v0.1)

> "Sag mir was du weisst — ich schreibe es sauber auf, ordne es ein und mache es für dein Team und eure KI-Tools abrufbar."

### Features

#### 1. Wissenserfassung via CLI
- `dokbot add` — Interaktiver Modus mit Rückfragen bei Lücken
- `dokbot add --file <path>` — Import aus bestehendem Dokument
- `dokbot add --stdin` — Piping von anderen Tools/Scripts

#### 2. Git-basierter Storage
- Jede Änderung = Git-Commit mit aussagekräftiger Message
- Automatische Ordnerstruktur nach Kategorien
- Frontmatter in Markdown-Files (Titel, Kategorie, Ersteller, Datum, Tags)

#### 3. Wissen abrufen
- `dokbot search <query>` — Semantische Suche
- `dokbot ask <frage>` — KI-Antwort basierend auf gespeichertem Wissen (RAG)
- `dokbot list` — Übersicht, filterbar nach Kategorie/Tags
- `dokbot show <doc>` — Dokument anzeigen

#### 4. MCP-Server (Daemon, Lesezugriff)
- Läuft als Hintergrund-Service (Daemon), nicht manuell starten
- `dokbot server start` — Startet den Daemon (systemd/launchd/Docker)
- `dokbot server stop` / `dokbot server status` — Verwaltung
- Transport: HTTP+SSE (StreamableHTTP), damit mehrere Clients gleichzeitig zugreifen können (z.B. Langdock, Claude Code, Cursor)
- Tools: `search_knowledge`, `ask_question`, `list_documents`, `get_document`
- Zusätzlich stdio-Modus (`dokbot serve --stdio`) für lokale MCP-Clients

#### 5. Basis-Konfiguration
- `dokbot init` — Initialisiert Knowledge Base (Git-Repo + Konfiguration)
- `.dokbot/config.yaml` — LLM-Provider, Kategorien, Sprache

### Bewusst NICHT im MVP

| Feature | Geplant für |
|---------|-------------|
| Web-UI, SSO, Analytics, Graph-Viz | Pro Edition |
| Freigabe-Workflows | Pro Edition |
| MCP-Schreibzugriff | v0.2 |
| Automatische Lücken-Erkennung | v0.2 |
| CI/CD-Integration | v0.2 |

### User Journey

```
1. npm install -g dokbot
2. dokbot init
3. dokbot add
   → "Worum geht es?" → "Wie wir Rechnungen stellen..."
   → Rückfragen: Intervall? Tool? Verantwortlich?
   → Erstellt: prozesse/fakturierung.md
   → Git-Commit: "add: Fakturierungsprozess dokumentiert"
4. dokbot ask "Wie fakturieren wir?"
5. dokbot server start  →  MCP-Daemon läuft, Langdock/Claude/Cursor können zugreifen
```

---

## Wissenstypen (fest im MVP, Custom-Typen in v0.2)

| Typ | Ordner | Felder im Frontmatter |
|-----|--------|-----------------------|
| Prozess | `prozesse/` | verantwortlich, beteiligte, trigger, schritte, tools, frequenz, output |
| Anleitung | `anleitungen/` | voraussetzungen, schritte, tipps, geschaetzte_dauer |
| Know-how | `knowhow/` | kontext, kernaussage, quellen, level (basic/advanced) |
| Entscheidung | `entscheidungen/` | optionen, begruendung, datum, beteiligte, status (offen/entschieden/revidiert) |
| Sitzungsprotokoll | `protokolle/` | datum, teilnehmer, beschluesse, action_items, naechstes_meeting |
| Rolle | `rollen/` | verantwortlichkeiten, skills, personen, stellvertretung |
| Richtlinie | `richtlinien/` | geltungsbereich, regeln, ausnahmen, gueltig_ab, freigegeben_von |
| Glossar | `glossar/` | begriff, definition, kontext, synonyme |
| Tool | `tools/` | name, zweck, zugang, lizenz, verantwortlich, alternativen |
| Kontakt | `kontakte/` | name, organisation, rolle, erreichbarkeit, kontext |
| Vorlage | `vorlagen/` | zweck, anwendungsfall, format, letzte_aktualisierung |
| FAQ | `faq/` | frage, antwort, kategorie, verwandte_docs |
| Checkliste | `checklisten/` | anwendungsfall, schritte, verantwortlich, frequenz |
| KI-Skill | `ki-skills/` | name, beschreibung, ziel_agent (claude/langdock/allgemein), voraussetzungen, anweisungen, tools, beispiele |

---

## Datenarchitektur

```
Knowledge Base (Git-Repo)
├── .dokbot/
│   ├── config.yaml          # LLM-Provider, Sprache, etc.
│   ├── schemas/             # Wissenstyp-Definitionen
│   └── index/               # LanceDB Vektor-Index (abgeleitet)
├── prozesse/
├── anleitungen/
├── knowhow/
├── entscheidungen/
├── protokolle/
├── rollen/
├── richtlinien/
├── glossar/
├── tools/
├── kontakte/
├── vorlagen/
├── faq/
├── checklisten/
└── ki-skills/
```

### Zwei Speicherschichten

| Schicht | Technologie | Zweck | Source of Truth? |
|---------|------------|-------|-----------------|
| **Markdown in Git** | isomorphic-git | Dokumente, Versionierung, Änderungshistorie | **Ja** |
| **Vektor-Index** | LanceDB (embedded) | Semantische Suche, RAG-Abfragen | Nein — abgeleitet |

**Warum eine Vektor-Datenbank?** Markdown-Files können nur per Textsuche durchsucht werden (exakte Treffer). Für semantische Suche ("Wie onboarden wir neue Mitarbeiter?" findet auch ein Dokument mit dem Titel "Einarbeitung neuer Teammitglieder") werden die Dokumente in Vektoren umgewandelt (Embeddings) und in LanceDB gespeichert. Das ermöglicht:
- **Semantische Suche** (`dokbot search`) — findet inhaltlich passende Dokumente, nicht nur Keyword-Treffer
- **RAG-Antworten** (`dokbot ask`) — relevante Dokumente werden gefunden und dem LLM als Kontext übergeben
- **Ähnliche Dokumente** — Duplikaterkennung, Cross-Links, verwandte Themen

Der Vektor-Index ist **abgeleitet und wegwerfbar** — `dokbot reindex` baut ihn jederzeit aus den Markdown-Files neu auf. Geht er verloren, geht kein Wissen verloren.

---

## Architektur

```
┌──────────────────────────────────────────────────┐
│                 Dokbot Core (OSS)                │
│                                                  │
│  ┌──────────┐  ┌──────────┐  ┌───────────────┐  │
│  │ KI-Agent │  │ Git-     │  │ Vektor-Index  │  │
│  │ Engine   │  │ Storage  │  │ (LanceDB)     │  │
│  │(LLM-     │  │(Markdown)│  │ Semantische   │  │
│  │agnostisch)│ │  = SoT   │──│ Suche + RAG   │  │
│  └──────────┘  └──────────┘  └───────────────┘  │
│  ┌──────────┐  ┌──────────┐  ┌───────────────┐  │
│  │ CLI      │  │ REST-API │  │ MCP-Server    │  │
│  │          │  │          │  │ (Daemon,      │  │
│  │          │  │          │  │  HTTP+SSE)    │  │
│  └──────────┘  └──────────┘  └───────────────┘  │
└──────────────────────────────────────────────────┘
         │              │               │
┌────────┴──────────────┴───────────────┴──────────┐
│              Dokbot Pro (Paid)                    │
│  Web-UI · Approvals · SSO · Analytics            │
│  Graph-Viz · Multi-Tenant · Hosting              │
└──────────────────────────────────────────────────┘
```

## Tech-Stack

- **Sprache**: TypeScript (Node.js)
- **Package Manager**: pnpm
- **Monorepo**: Turborepo (`core`, `cli`, `mcp-server`)
- **LLM-Abstraktion**: Vercel AI SDK (Claude, OpenAI, Ollama)
- **Vektor-Index**: LanceDB (embedded, datei-basiert)
- **Git**: isomorphic-git
- **CLI**: Commander.js
- **MCP**: @modelcontextprotocol/sdk

---

## Anwendungsfälle (Use Cases)

### Akteure

| Akteur | Beschreibung | Gibt Wissen ein | Konsumiert Wissen |
|--------|-------------|:---------------:|:-----------------:|
| **Mensch (Wissensgebender)** | Person, die Wissen eingibt oder aktualisiert (via CLI, Web-UI) | ✓ | |
| **Mensch (Wissenskonsument)** | Person, die Wissen abruft, liest oder Fragen stellt (via CLI, Web-UI, Markdown) | | ✓ |
| **Mensch (Reviewer/Admin)** | Person, die Freigaben erteilt, Qualität sichert und Dokbot konfiguriert | ✓ | ✓ |
| **Dokbot-Agent** | Der KI-Agent selbst — strukturiert, verknüpft, korrigiert, erkennt Lücken | ✓ | ✓ |
| **Externe KI** | Anderer KI-Agent (Langdock, Claude Code, Cursor) — fragt Wissen ab oder liefert neues | ✓ | ✓ |
| **Skript/System** | Automatisierungen, CI/CD-Pipelines, Webhooks — speist Daten ein oder liest sie aus | ✓ | ✓ |

### A — Wissen erfassen

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| A1 | Wissen in natürlicher Sprache eingeben (interaktiv via CLI) | Mensch (Wissensgebender) | Idee |
| A2 | Bestehendes Dokument importieren (Markdown, Text, PDF) | Mensch (Wissensgebender) | Idee |
| A3 | Wissen via Pipe/stdin von anderem Tool empfangen | Skript/System | Idee |
| A4 | Sitzungsprotokoll aus Gesprächsnotizen erstellen | Mensch (Wissensgebender) | Idee |
| A5 | Entscheidung mit Begründung und Alternativen dokumentieren | Mensch (Wissensgebender) | Idee |
| A6 | Wissen via MCP-Schreibzugriff von anderer KI empfangen | Externe KI | Idee |
| A7 | Bulk-Import von bestehendem Wissen (Confluence-Export, Ordner) | Mensch (Wissensgebender) | Ergänzung |
| A8 | Voice-to-Knowledge: Transkript verarbeiten | Mensch (Wissensgebender) | Ergänzung |
| A9 | E-Mail oder Chat-Nachricht als Wissen erfassen | Skript/System | Ergänzung |
| A10 | Wissen via REST-API einliefern | Externe KI, Skript/System | Ergänzung |

### B — Wissen strukturieren & dokumentieren

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| B1 | Eingegebenes Wissen in strukturiertes Markdown umwandeln | Dokbot-Agent | Idee |
| B2 | Passenden Wissenstyp automatisch erkennen und zuweisen | Dokbot-Agent | Idee |
| B3 | Wissen in die richtige Kategorie/Ordner einordnen | Dokbot-Agent | Idee |
| B4 | Frontmatter automatisch befüllen | Dokbot-Agent | Idee |
| B5 | Verwandte Dokumente verknüpfen (Cross-Links) | Dokbot-Agent | Idee |
| B6 | Duplikate erkennen und Zusammenführung vorschlagen | Dokbot-Agent | Ergänzung |
| B7 | Glossarbegriffe erkennen und verlinken | Dokbot-Agent | Ergänzung |
| B8 | Inhaltsverzeichnis / Index pflegen | Dokbot-Agent | Ergänzung |

### C — Informationsqualität sichern

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| C1 | Informationslücken erkennen und Rückfragen stellen | Dokbot-Agent | Idee |
| C2 | Schreibfehler und Formatierung automatisch korrigieren | Dokbot-Agent | Idee |
| C3 | Veraltetes Wissen identifizieren | Dokbot-Agent | Ergänzung |
| C4 | Widersprüche zwischen Dokumenten erkennen | Dokbot-Agent | Ergänzung |
| C5 | Qualitätsprüfung (vollständig, verständlich?) | Dokbot-Agent | Ergänzung |
| C6 | Regelmässige Review-Zyklen auslösen | Dokbot-Agent | Ergänzung |

### D — Revisionskontrolle & Änderungshistorie

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| D1 | Jede Änderung mit Autor, Datum, Beschreibung protokollieren | Dokbot-Agent | Idee |
| D2 | Änderungshistorie eines Dokuments einsehen | Mensch (Wissenskonsument), Externe KI | Idee |
| D3 | Zwei Versionen vergleichen (Diff) | Mensch (Wissenskonsument) | Idee |
| D4 | Änderung rückgängig machen (Rollback) | Mensch (Reviewer/Admin) | Idee |
| D5 | Gesamte Änderungsübersicht über Zeitraum anzeigen | Mensch (Wissenskonsument), Externe KI | Idee |
| D6 | Umstrukturierungen nachvollziehbar machen | Dokbot-Agent | Idee |

### E — Zugriffsrechte & Freigabe-Workflows

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| E1 | Prozessänderung zur Freigabe einreichen | Mensch (Wissensgebender), Externe KI | Idee |
| E2 | Freigabe erteilen oder ablehnen | Mensch (Reviewer/Admin) | Idee |
| E3 | Know-how-Änderung ohne Freigabe speichern | Mensch (Wissensgebender), Externe KI | Idee |
| E4 | Schreibfehler-Korrektur ohne Freigabe speichern | Mensch (Wissensgebender), Dokbot-Agent | Idee |
| E5 | Freigabe-Regeln pro Dokumenttyp definieren | Mensch (Reviewer/Admin) | Idee |
| E6 | Zugriff auf Wissensbereiche beantragen | Mensch (Wissenskonsument), Externe KI | Idee |
| E7 | Rollen und Berechtigungen verwalten | Mensch (Reviewer/Admin) | Ergänzung |
| E8 | Benachrichtigung bei ausstehenden Freigaben | Mensch (Reviewer/Admin) | Ergänzung |

### F — Wissen konsumieren (Mensch)

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| F1 | Markdown-Dokumente direkt lesen | Mensch (Wissenskonsument) | Idee |
| F2 | Semantisch nach Wissen suchen | Mensch (Wissenskonsument) | Idee |
| F3 | Frage stellen und KI-Antwort erhalten (RAG) | Mensch (Wissenskonsument) | Ergänzung |
| F4 | Dokumente auflisten, filtern nach Typ/Kategorie/Tags | Mensch (Wissenskonsument) | Ergänzung |
| F5 | Zusammenhänge als Graph visualisieren | Mensch (Wissenskonsument) | Idee |
| F6 | Zusammenfassung eines Bereichs generieren | Mensch (Wissenskonsument) | Ergänzung |
| F7 | Onboarding-Paket generieren | Mensch (Wissenskonsument) | Ergänzung |
| F8 | Wissen exportieren (PDF, HTML, Markdown-Bundle) | Mensch (Wissenskonsument) | Ergänzung |

### G — Wissen konsumieren (Maschine)

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| G1 | Wissen via MCP-Server abfragen (search, get, ask) | Externe KI | Idee |
| G2 | Markdown-Files direkt aus Git-Repo lesen | Externe KI, Skript/System | Idee |
| G3 | Wissen via REST-API abfragen | Externe KI, Skript/System | Idee |
| G4 | Änderungsbenachrichtigungen empfangen (Webhook/Event) | Skript/System | Ergänzung |
| G5 | Strukturierte Daten extrahieren (JSON-Export) | Skript/System | Ergänzung |
| G6 | Wissenskontext als MCP-Ressource nutzen | Externe KI | Idee |
| G7 | KI-Skill abrufen und als Anweisung verwenden | Externe KI | Ergänzung |

### H — Auswertungen & Analytics

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| H1 | Übersicht Wissensbestand (Anzahl, Typen, Abdeckung) | Mensch (Reviewer/Admin), Externe KI | Idee |
| H2 | Änderungsübersicht (kürzlich hinzugefügt/geändert) | Mensch (Wissenskonsument), Externe KI | Idee |
| H3 | Wissens-Lücken-Report | Mensch (Reviewer/Admin) | Ergänzung |
| H4 | Nutzungsstatistik | Mensch (Reviewer/Admin) | Ergänzung |
| H5 | Aktivitätsübersicht (wer hat was beigetragen) | Mensch (Reviewer/Admin) | Ergänzung |
| H6 | Altersbericht (lange nicht aktualisiert) | Dokbot-Agent | Ergänzung |

### I — Administration & Setup

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| I1 | Knowledge Base initialisieren | Mensch (Reviewer/Admin) | Ergänzung |
| I2 | LLM-Provider konfigurieren | Mensch (Reviewer/Admin) | Ergänzung |
| I3 | Wissenstypen und Kategorien konfigurieren | Mensch (Reviewer/Admin) | Ergänzung |
| I4 | Vektor-Index neu aufbauen | Mensch (Reviewer/Admin), Skript/System | Ergänzung |
| I5 | Backup und Restore | Mensch (Reviewer/Admin), Skript/System | Ergänzung |
| I6 | MCP-Server (Daemon) starten und konfigurieren | Mensch (Reviewer/Admin) | Ergänzung |
| I7 | Sprache und Lokalisierung einstellen | Mensch (Reviewer/Admin) | Ergänzung |

### Use Case → Release Zuordnung

| Release | Use Cases |
|---------|-----------|
| **MVP (v0.1)** | A1, A2, A3, B1, B2, B3, B4, C1, C2, D1, D2, D5, F1, F2, F3, F4, G1, G2, I1, I2, I4, I6 |
| **v0.2** | A4, A5, A6, A10, B5, B6, C3, D3, D6, F6, G3, G5, G7, H1, H2, H6, I3 |
| **v0.3** | A7, B7, B8, C4, C5, D4, E1–E5, F8, G4, H3 |
| **Pro Edition** | A8, A9, C6, E6–E8, F5, F7, G6, H4, H5, I5, I7 |

---

## Nächste Schritte

- [x] Name festgelegt: **Dokbot**
- [x] GitHub-Repo erstellt: github.com/gzuercher/dokbot
- [x] Playbook-Basis vom Raptus Claude Playbook übernommen
- [x] Use Cases abgenommen (2026-04-17, können später noch angepasst werden)
- [ ] Detaillierte Architektur ausarbeiten
- [ ] Prompt-Design für den KI-Agent
- [ ] Datenmodell im Detail (Frontmatter-Schema, Config-Format)
- [ ] MCP-Server Spezifikation
- [ ] CLI UX-Design
- [ ] LLM-Abstraktionsschicht
- [ ] Monorepo-Setup
- [ ] AGPL-Lizenz korrekt einrichten
- [ ] Prototyp: Core + CLI + MCP-Server
- [ ] Dogfooding bei Raptus
