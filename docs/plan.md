# Dokbot — Projektplan

## Context

Dokbot ist ein KI-gestützter Wissensverwalter für Organisationen (10–50 Personen). Er nimmt internes Know-how entgegen, strukturiert es als Markdown in Git und macht es für Menschen und Maschinen zugänglich.

**Geschäftsmodell**: Open Core (AGPL) + Managed Hosting durch Raptus AG.
**Lizenz**: AGPL-3.0 (Community), proprietär (Pro).

| Stufe | Inhalt | Preis |
|-------|--------|-------|
| Community (AGPL) | Core-Agent, CLI, Git-Storage, MCP-Server (read) | Gratis |
| Pro-Lizenz | Web-UI, Freigabe-Workflows, SSO, Analytics | CHF 15–30/User/Monat |
| Managed Hosting | Betrieb inkl. Updates, Backups, Monitoring | CHF 200–800/Monat |
| Consulting | Setup, Migration, Schulung | Stundenhonorar |

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

#### 4. MCP-Server (Lesezugriff)
- `dokbot serve` — Startet MCP-Server
- Tools: `search_knowledge`, `ask_question`, `list_documents`, `get_document`

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
5. dokbot serve  →  MCP-Server für andere KI-Tools
```

---

## Wissenstypen (fest im MVP, Custom-Typen in v0.2)

| Typ | Ordner | Felder im Frontmatter |
|-----|--------|-----------------------|
| Prozess | `prozesse/` | verantwortlich, beteiligte, trigger, schritte, tools, frequenz |
| Anleitung | `anleitungen/` | voraussetzungen, schritte, tipps |
| Know-how | `knowhow/` | kontext, kernaussage, quellen |
| Entscheidung | `entscheidungen/` | optionen, begruendung, datum, beteiligte |
| Sitzungsprotokoll | `protokolle/` | datum, teilnehmer, beschluesse, action_items |

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
└── protokolle/
```

**Markdown = Source of Truth** — LanceDB-Index ist abgeleitet, `dokbot reindex` baut ihn neu auf.

---

## Architektur

```
┌─────────────────────────────────────────┐
│            Dokbot Core (OSS)            │
│                                         │
│  ┌──────────┐  ┌──────────┐  ┌───────┐ │
│  │ KI-Agent │  │ Git-     │  │ MCP-  │ │
│  │ Engine   │  │ Storage  │  │Server │ │
│  │(LLM-     │  │(Markdown)│  │       │ │
│  │agnostisch)│ │          │  │       │ │
│  └──────────┘  └──────────┘  └───────┘ │
│  ┌──────────┐  ┌──────────┐            │
│  │ CLI      │  │ REST-API │            │
│  └──────────┘  └──────────┘            │
└─────────────────────────────────────────┘
         │              │
┌────────┴──────────────┴─────────────────┐
│           Dokbot Pro (Paid)             │
│  Web-UI · Approvals · SSO · Analytics  │
│  Graph-Viz · Multi-Tenant · Hosting    │
└─────────────────────────────────────────┘
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

| Akteur | Beschreibung |
|--------|-------------|
| **Wissensgebender** | Mensch, der Wissen eingibt oder aktualisiert |
| **Wissenskonsument** | Mensch, der Wissen abruft oder liest |
| **Reviewer/Admin** | Mensch, der Freigaben erteilt und Qualität sichert |
| **Dokbot-Agent** | Der KI-Agent selbst (agiert proaktiv) |
| **Externe Maschine** | Anderer KI-Agent, Skript oder System |

### A — Wissen erfassen

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| A1 | Wissen in natürlicher Sprache eingeben (interaktiv via CLI) | Wissensgebender | Idee |
| A2 | Bestehendes Dokument importieren (Markdown, Text, PDF) | Wissensgebender | Idee |
| A3 | Wissen via Pipe/stdin von anderem Tool empfangen | Externe Maschine | Idee |
| A4 | Sitzungsprotokoll aus Gesprächsnotizen erstellen | Wissensgebender | Idee |
| A5 | Entscheidung mit Begründung und Alternativen dokumentieren | Wissensgebender | Idee |
| A6 | Wissen via MCP-Schreibzugriff von anderer KI empfangen | Externe Maschine | Idee |
| A7 | Bulk-Import von bestehendem Wissen (Confluence-Export, Ordner) | Wissensgebender | Ergänzung |
| A8 | Voice-to-Knowledge: Transkript verarbeiten | Wissensgebender | Ergänzung |
| A9 | E-Mail oder Chat-Nachricht als Wissen erfassen | Externe Maschine | Ergänzung |

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
| D2 | Änderungshistorie eines Dokuments einsehen | Wissenskonsument | Idee |
| D3 | Zwei Versionen vergleichen (Diff) | Wissenskonsument | Idee |
| D4 | Änderung rückgängig machen (Rollback) | Reviewer/Admin | Idee |
| D5 | Gesamte Änderungsübersicht über Zeitraum anzeigen | Wissenskonsument | Idee |
| D6 | Umstrukturierungen nachvollziehbar machen | Dokbot-Agent | Idee |

### E — Zugriffsrechte & Freigabe-Workflows

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| E1 | Prozessänderung zur Freigabe einreichen | Wissensgebender | Idee |
| E2 | Freigabe erteilen oder ablehnen | Reviewer/Admin | Idee |
| E3 | Know-how-Änderung ohne Freigabe speichern | Wissensgebender | Idee |
| E4 | Schreibfehler-Korrektur ohne Freigabe speichern | Wissensgebender | Idee |
| E5 | Freigabe-Regeln pro Dokumenttyp definieren | Reviewer/Admin | Idee |
| E6 | Zugriff auf Wissensbereiche beantragen | Wissenskonsument | Idee |
| E7 | Rollen und Berechtigungen verwalten | Reviewer/Admin | Ergänzung |
| E8 | Benachrichtigung bei ausstehenden Freigaben | Reviewer/Admin | Ergänzung |

### F — Wissen konsumieren (Mensch)

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| F1 | Markdown-Dokumente direkt lesen | Wissenskonsument | Idee |
| F2 | Semantisch nach Wissen suchen | Wissenskonsument | Idee |
| F3 | Frage stellen und KI-Antwort erhalten (RAG) | Wissenskonsument | Ergänzung |
| F4 | Dokumente auflisten, filtern nach Typ/Kategorie/Tags | Wissenskonsument | Ergänzung |
| F5 | Zusammenhänge als Graph visualisieren | Wissenskonsument | Idee |
| F6 | Zusammenfassung eines Bereichs generieren | Wissenskonsument | Ergänzung |
| F7 | Onboarding-Paket generieren | Wissenskonsument | Ergänzung |
| F8 | Wissen exportieren (PDF, HTML, Markdown-Bundle) | Wissenskonsument | Ergänzung |

### G — Wissen konsumieren (Maschine)

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| G1 | Wissen via MCP-Server abfragen | Externe Maschine | Idee |
| G2 | Markdown-Files direkt aus Git-Repo lesen | Externe Maschine | Idee |
| G3 | Wissen via REST-API abfragen | Externe Maschine | Idee |
| G4 | Änderungsbenachrichtigungen (Webhook/Event) | Externe Maschine | Ergänzung |
| G5 | Strukturierte Daten extrahieren (JSON-Export) | Externe Maschine | Ergänzung |
| G6 | Wissenskontext als MCP-Ressource nutzen | Externe Maschine | Idee |

### H — Auswertungen & Analytics

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| H1 | Übersicht Wissensbestand (Anzahl, Typen, Abdeckung) | Wissenskonsument | Idee |
| H2 | Änderungsübersicht (kürzlich hinzugefügt/geändert) | Wissenskonsument | Idee |
| H3 | Wissens-Lücken-Report | Reviewer/Admin | Ergänzung |
| H4 | Nutzungsstatistik | Reviewer/Admin | Ergänzung |
| H5 | Aktivitätsübersicht (wer hat was beigetragen) | Reviewer/Admin | Ergänzung |
| H6 | Altersbericht (lange nicht aktualisiert) | Dokbot-Agent | Ergänzung |

### I — Administration & Setup

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| I1 | Knowledge Base initialisieren | Reviewer/Admin | Ergänzung |
| I2 | LLM-Provider konfigurieren | Reviewer/Admin | Ergänzung |
| I3 | Wissenstypen und Kategorien konfigurieren | Reviewer/Admin | Ergänzung |
| I4 | Vektor-Index neu aufbauen | Reviewer/Admin | Ergänzung |
| I5 | Backup und Restore | Reviewer/Admin | Ergänzung |
| I6 | MCP-Server starten und konfigurieren | Reviewer/Admin | Ergänzung |
| I7 | Sprache und Lokalisierung einstellen | Reviewer/Admin | Ergänzung |

### Use Case → Release Zuordnung

| Release | Use Cases |
|---------|-----------|
| **MVP (v0.1)** | A1, A2, A3, B1, B2, B3, B4, C1, C2, D1, D2, D5, F1, F2, F3, F4, G1, G2, I1, I2, I4, I6 |
| **v0.2** | A4, A5, A6, B5, B6, C3, D3, D6, F6, G3, G5, H1, H2, H6, I3 |
| **v0.3** | A7, B7, B8, C4, C5, D4, E1–E5, F8, G4, H3 |
| **Pro Edition** | A8, A9, C6, E6–E8, F5, F7, G6, H4, H5, I5, I7 |

---

## Nächste Schritte

- [x] Name festgelegt: **Dokbot**
- [x] GitHub-Repo erstellt: github.com/gzuercher/dokbot
- [x] Playbook-Basis vom Raptus Claude Playbook übernommen
- [ ] **Use Cases abnehmen** (blockiert weitere Umsetzung!)
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
