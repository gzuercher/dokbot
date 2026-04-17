# Dokbot — Der Dokumentations-Bot für Organisationswissen

## Context

Es soll eine Anwendung entstehen, die als KI-gestützter Wissensverwalter für Organisationen dient. Der Kern ist ein KI-Agent, der internes Know-how entgegennimmt, strukturiert, dokumentiert und für Menschen und Maschinen zugänglich macht. Das Projekt befindet sich in der Ideationsphase — es existiert noch kein Code.

---

## Vision & Kernfunktionen

### 1. Informationsaufnahme
- Entgegennahme von Wissen aus verschiedenen Quellen: Menschen (natürliche Sprache), andere KI-Agenten, Skripte, Systeme
- Erkennung von Informationslücken beim Erfassen und aktives Nachfragen beim Informationsgebenden

### 2. Strukturierung & Dokumentation
- Automatisches Ordnen, Kategorisieren und sauberes Dokumentieren von eingehendem Wissen
- Erstellung von strukturierten Dokumenten (Markdown-basiert)
- Verknüpfung verwandter Wissensbereiche

### 3. Revisionskontrolle & Änderungshistorie
- Vollständige Protokollierung: Wer hat wann was geändert?
- Nachvollziehbarkeit aller Umstrukturierungen
- Versionierung aller Dokumente

### 4. Zugriffsrechte & Freigabe-Workflows
- **Mit Freigabe**: Änderungen an Prozessen (z.B. Fakturierungsabläufe, Organisationsstrukturen)
- **Ohne Freigabe**: Änderungen an Know-how-Dokumenten (technische/fachliche Umsetzungen), Korrekturen von Schreibfehlern
- Konfigurierbare Regeln, welche Änderungen eine Freigabe benötigen

### 5. Konsumierung — Menschliche Interfaces
- Markdown-Dokumente als primäres Leseformat
- (Sekundär) Graphen/Visualisierungen zur visuellen Darstellung von Zusammenhängen

### 6. Konsumierung — Maschinen-Interfaces
- **MCP-Server**: Andere KI-Agenten können via MCP auf das Wissen zugreifen
- **Dateizugriff**: Markdown-Files direkt lesbar
- **API**: Programmatischer Zugriff für Skripte und Systeme

### 7. Auswertungen & Übersicht
- Dashboard/Berichte über das verwaltete Wissen
- Änderungsübersicht (was ist passiert, was wurde hinzugefügt/geändert)
- Statistiken zum Wissensbestand

---

## Scope-Abgrenzung

| In Scope | Out of Scope |
|----------|-------------|
| Internes Organisationswissen | Kundenspezifische Projektdaten |
| Wie Leistungen erbracht werden | Externe Informationen |
| Fakturierungsprozesse | Kundenprojekt-Details |
| Organisationsabläufe (Meetings etc.) | |
| Technische/fachliche Umsetzungsdokumentation | |

> Kundenspezifische Daten sind explizit ausgeschlossen. Falls später benötigt, müssten diese in separaten, abgeschotteten Containern verwaltet werden.

---

## Geklärte Rahmenbedingungen

- **Organisationsgrösse**: Mittel (10–50 Personen)
- **Deployment**: Flexibel — Self-hosted und Cloud sollen möglich sein
- **LLM**: Flexibel — austauschbare LLM-Schicht (Claude, OpenAI, lokale Modelle)
- **Aktueller Fokus**: Machbarkeitsanalyse, bevor ein MVP geplant wird

---

## Bestehende Lösungen — Marktanalyse

Kein bestehendes Produkt deckt alle Anforderungen ab. Die Kombination aus KI-gestützter Lücken-Erkennung, Markdown-first, Git-Versionierung, selektiven Freigabe-Workflows UND MCP/Maschinen-Interface ist eine echte Marktlücke.

**Am nächsten dran:**

| Produkt | Art | Stärken | Fehlt | Deckung |
|---------|-----|---------|-------|---------|
| **Outline** | Open Source | Markdown-native, API, Versionierung, SSO | KI, Freigabe-Workflows, MCP | ~50% |
| **Guru** | Kommerziell | KI-Verifikation, Analytics, Experten-Review | Markdown, Git, MCP | ~55% |
| **Tettra** | Kommerziell | Fokus auf "wie die Org funktioniert", KI-Antworten, Lücken-Erkennung | Markdown, MCP, Git | ~50% |
| **Confluence** | Kommerziell | Enterprise-Wiki, Freigaben, API, Atlassian AI | Markdown, MCP, flexibles LLM | ~55% |
| **Wiki.js** | Open Source | Git-Backend, Markdown, API/GraphQL | KI, Freigabe-Workflows | ~45% |
| **Notion + AI** | Kommerziell | Wiki, KI-Suche, Versionierung, API | Freigabe-Workflows, MCP, Git | ~60% |
| **BookStack** | Open Source | Versionierung, API, Rollen | KI, Markdown-native, MCP | ~40% |

**Fazit**: Das Konzept besetzt eine echte Nische — besonders das MCP/Machine-Interface und die intelligente Freigabe-Steuerung existieren so nirgends.

---

## Umsetzungsmöglichkeiten — Übersicht

### 1. Claude Code + VPS (Agenten-Ansatz)
Claude Code CLI auf einem VPS mit spezialisierten Agenten. Git-Repo speichert Markdown. MCP-Server für Maschinenzugriff.
- **Vorteil**: Maximale Flexibilität, Claude versteht Dateien nativ, Git gibt Versionierung gratis
- **Nachteil**: Fragile Orchestrierung — Prozess-Supervision, Queues, Freigabe-UI, Fehlerbehandlung müssen selbst gebaut werden. Claude Code ist für Dev-Workflows designed, nicht für Produktionsdienste
- **Komplexität**: Hoch

### 2. Multi-Agent-Frameworks (CrewAI / LangGraph / AutoGen)
Rollenbasierte Agenten (Ingestion, Editor, Reviewer, Publisher) in einem Workflow-Graphen. Markdown in Git. API/MCP-Wrapper darüber.
- **Vorteil**: Purpose-built für Multi-Agent-Orchestrierung mit Retries, Memory, Kommunikation
- **Nachteil**: Frameworks ändern sich schnell (Breaking Changes); UI, Storage und API trotzdem selbst bauen
- **Komplexität**: Mittel-Hoch

### 3. Wiki-Plattform + KI-Erweiterung (Outline / Wiki.js / BookStack)
Bestehende Wiki deployen und KI via Webhooks oder Sidecar-Service anbauen. Outline ist der beste Kandidat (Markdown-native, API, SSO, Versionierung).
- **Vorteil**: UI, Auth, Suche, Versionierung, Freigaben out-of-the-box. Schnellste Time-to-Value
- **Nachteil**: KI ist aufgeschraubt; MCP-Server muss custom auf Platform-API gebaut werden
- **Komplexität**: Niedrig-Mittel

### 4. Custom Web App mit LLM-Integration
Full-Stack App (z.B. Next.js + Postgres oder FastAPI + React) mit Markdown-Editor, Git-Backend, Approval-Engine, LLM-Calls, REST-API + MCP.
- **Vorteil**: Totale Kontrolle über UX, Datenmodell, KI-Verhalten
- **Nachteil**: Höchster Entwicklungsaufwand — alles selbst bauen
- **Komplexität**: Hoch

### 5. Low-Code (n8n / Flowise / Langflow)
n8n-Workflows getriggert via Formular/API, durch LLM-Nodes für Strukturierung, Commit nach Git, Benachrichtigung an Approver.
- **Vorteil**: Schnelles Prototyping, visuelle Workflows, minimaler Code
- **Nachteil**: Schwer für komplexe Freigabe-Logik oder custom MCP-Server; fragil bei Skalierung
- **Komplexität**: Niedrig-Mittel

### 6. Git + Markdown + CI/CD + KI-Pipeline
Pures Git-Repo mit Markdown. Beiträge via PRs (oder Formular erstellt PRs). CI/CD führt KI-Verarbeitung aus (Linting, Tagging, Zusammenfassungen). Approver reviewen PRs. MCP-Server liest Content aus Repo.
- **Vorteil**: Git übernimmt Versionierung und Freigabe nativ. Einfach, robust, keine Datenbank. Markdown ist portabel
- **Nachteil**: Nicht freundlich für nicht-technische Mitwirkende. Echtzeit-Abfragen brauchen separate Schicht
- **Komplexität**: Mittel

### 7. Headless CMS + KI (Strapi / Directus)
Strukturierter Content mit eingebautem Versioning, Rollen, REST+GraphQL-API. KI via Webhooks/Plugins. MCP-Server darüber.
- **Vorteil**: Content-Modellierung, Versionierung, Berechtigungen, API built-in
- **Nachteil**: Nicht Markdown-native; KI und MCP-Integration sind custom
- **Komplexität**: Mittel

### 8. Anthropic API direkt (Tool-Use Agents)
Anthropic API mit Tool-Use für Agenten, die einen Knowledge Store verwalten. Git oder DB für Persistenz.
- **Vorteil**: Einfacher als Multi-Agent-Frameworks; eine API-Abhängigkeit
- **Nachteil**: Alle Orchestrierungslogik im eigenen Code; keine UI oder Workflows built-in
- **Komplexität**: Mittel

---

## Geschäftsmodell & Open-Source-Strategie

### Ist Open Source hier die richtige Wahl?

**Kurze Antwort**: Ja — aber als **Open Core**, nicht als reines Open Source.

### Das Problem mit "komplett Open Source"

Wenn alles Open Source ist und das Produkt gut genug, können Kunden es selbst deployen. Raptus verdient nur, wenn die Implementierung komplex genug ist oder der Kunde Support braucht. Das funktioniert, skaliert aber schlecht — du verkaufst Stunden, kein Produkt.

### Bewährte Modelle für diese Art Produkt

| Modell | Wie es funktioniert | Beispiele | Passt für Dokbot? |
|--------|-------------------|-----------|-----------------|
| **Open Core** | Kern ist Open Source, Premium-Features kostenpflichtig (z.B. SSO, erweiterte Approvals, Analytics-Dashboard, Enterprise-Support) | GitLab, Supabase, n8n, Outline | **Ja — empfohlen** |
| **Source Available** | Code ist sichtbar, kommerzielle Nutzung braucht Lizenz | Elastic (SSPL), MongoDB, Sentry (BSL) | Ja — Alternative |
| **Open Source + Managed Service** | Software gratis, gehosteter Service kostenpflichtig | WordPress → WP Engine, Grafana Cloud | Möglich als Zusatz |
| **Open Source + Consulting** | Alles gratis, Geld über Implementierung | Deine ursprüngliche Idee | Funktioniert, skaliert wenig |

### Empfehlung: Open Core mit klarer Trennung

**Community Edition (Open Source, MIT oder AGPL)**:
- KI-Agent für Wissenserfassung und -strukturierung
- Markdown-basierte Dokumentation in Git
- Basis-Versionierung
- MCP-Server (Lesezugriff)
- CLI-Interface
- Single-User oder kleines Team

**Pro/Enterprise Edition (kostenpflichtig)**:
- Erweiterte Freigabe-Workflows mit konfigurierbaren Regeln
- SSO / LDAP-Integration
- Analytics-Dashboard & Auswertungen
- Mehrmandantenfähigkeit
- Erweiterte Zugriffsrechte (Rollen, Abteilungen)
- MCP-Server (Schreibzugriff + erweiterte Tools)
- Graph-Visualisierungen
- Priority-Support durch Raptus
- Hosting als Managed Service

**Warum AGPL als Open-Source-Lizenz?**
AGPL zwingt jeden, der die Software als Service anbietet, den Quellcode offenzulegen. Das verhindert, dass ein Cloud-Anbieter Dokbot einfach hostet und verkauft, ohne beizutragen. GitLab, Grafana und andere nutzen genau dieses Modell.

### Vorteile von Open Source für Dokbot

1. **Vertrauen**: Organisationen sehen den Code — wichtig bei internem Wissen
2. **Community**: Andere Entwickler können beitragen (Plugins, Integrationen)
3. **Marketing**: GitHub-Stars und Community-Buzz sind kostenlose Werbung
4. **Talent**: Gute Entwickler arbeiten gern an Open-Source-Produkten
5. **Vendor Lock-in Argument**: "Euer Wissen gehört euch, nicht unserem SaaS"

---

## Technische Empfehlung als Produkt

Da Dokbot ein eigenständiges Produkt sein soll (keine Wiki-Erweiterung), fallen Ansätze 1–3 weg. Empfohlen wird eine Kombination:

### Architektur-Vorschlag

```
┌─────────────────────────────────────────┐
│              Dokbot Core (OSS)            │
│                                         │
│  ┌──────────┐  ┌──────────┐  ┌───────┐ │
│  │ KI-Agent │  │ Git-     │  │ MCP-  │ │
│  │ Engine   │  │ Storage  │  │Server │ │
│  │(LLM-     │  │(Markdown)│  │       │ │
│  │ agnostisch)│ │          │  │       │ │
│  └──────────┘  └──────────┘  └───────┘ │
│  ┌──────────┐  ┌──────────┐            │
│  │ CLI      │  │ REST-API │            │
│  └──────────┘  └──────────┘            │
└─────────────────────────────────────────┘
         │              │
┌────────┴──────────────┴─────────────────┐
│           Dokbot Pro (Paid)               │
│  Web-UI · Approvals · SSO · Analytics   │
│  Graph-Viz · Multi-Tenant · Hosting     │
└─────────────────────────────────────────┘
```

### Tech-Stack Vorschlag

- **Sprache**: TypeScript (breite Community, einfach für Beitragende)
- **KI-Engine**: Abstraktion über LiteLLM oder Vercel AI SDK (LLM-agnostisch)
- **Storage**: Git-basiert (libgit2/isomorphic-git) + Markdown
- **API**: REST (Hono oder Fastify)
- **MCP-Server**: @modelcontextprotocol/sdk
- **CLI**: Commander.js oder Oclif
- **Web-UI (Pro)**: Next.js oder SvelteKit
- **Auth (Pro)**: Lucia oder Auth.js

---

## Gewähltes Geschäftsmodell: Open Core + Managed Hosting (Variante A)

| Stufe | Inhalt | Preis |
|-------|--------|-------|
| **Community (AGPL)** | Core-Agent, CLI, Git-Storage, MCP-Server (read), Basis-Versionierung | Gratis |
| **Pro-Lizenz** | Web-UI, Freigabe-Workflows, SSO, Analytics, Graph-Viz, Priority Support | CHF 15–30/User/Monat |
| **Managed Hosting** | Raptus hostet und betreibt die Instanz inkl. Updates, Backups, Monitoring | CHF 200–800/Monat |
| **Consulting** | Setup, Migration, Schulung, Custom-Integrationen | Stundenhonorar |

**Go-to-Market**: Start mit Consulting (Cash-Flow ab Tag 1) → Community-Edition auf GitHub → Pro-Features schrittweise → Managed Hosting als skalierbares Angebot.

---

## MVP-Scope (Community Edition v0.1)

### Kernversprechen des MVP
> "Sag mir was du weisst — ich schreibe es sauber auf, ordne es ein und mache es für dein Team und eure KI-Tools abrufbar."

### MVP Features — Enthalten

#### 1. Wissenserfassung via CLI
- `dokbot add` — Interaktiver Modus: User gibt Wissen in natürlicher Sprache ein, Agent stellt Rückfragen bei Lücken
- `dokbot add --file <path>` — Import aus bestehendem Dokument
- `dokbot add --stdin` — Piping von anderen Tools/Scripts
- Der KI-Agent strukturiert den Input, erstellt/aktualisiert Markdown-Dokumente, wählt die richtige Kategorie

#### 2. Git-basierter Storage
- Jede Änderung = ein Git-Commit mit aussagekräftiger Message
- Automatische Ordnerstruktur nach Kategorien (z.B. `prozesse/`, `technik/`, `organisation/`)
- Frontmatter in Markdown-Files (Titel, Kategorie, Ersteller, Datum, Tags)

#### 3. Wissen abrufen
- `dokbot search <query>` — Semantische Suche über die Knowledge Base
- `dokbot ask <frage>` — KI beantwortet Fragen basierend auf dem gespeicherten Wissen (RAG)
- `dokbot list` — Übersicht aller Dokumente, filterbar nach Kategorie/Tags
- `dokbot show <doc>` — Dokument anzeigen

#### 4. MCP-Server (Lesezugriff)
- `dokbot serve` — Startet MCP-Server
- Tools: `search_knowledge`, `ask_question`, `list_documents`, `get_document`
- Andere KI-Agenten (Claude Code, Cursor, etc.) können die Knowledge Base abfragen

#### 5. Basis-Konfiguration
- `dokbot init` — Initialisiert eine neue Knowledge Base (Git-Repo + Konfiguration)
- Konfigurierbar: LLM-Provider (Claude, OpenAI, Ollama), Kategorien, Sprache
- `.dokbot/config.yaml` für Einstellungen

### MVP Features — Bewusst NICHT enthalten (v0.2+)

| Feature | Warum nicht im MVP | Geplant für |
|---------|-------------------|-------------|
| Web-UI | Kern-Wert liegt im Agent, nicht in der UI | Pro Edition |
| Freigabe-Workflows | Braucht Multi-User-Konzept | Pro Edition |
| SSO/Auth | Single-User oder Team-Trust reicht für v0.1 | Pro Edition |
| Analytics/Dashboard | Nice-to-have, nicht Kernwert | Pro Edition |
| Graph-Visualisierung | Sekundär | Pro Edition |
| MCP-Schreibzugriff | Lesezugriff reicht als Start | v0.2 |
| Automatische Lücken-Erkennung | Komplex, braucht genug Daten | v0.2 |
| CI/CD-Integration | Premature für v0.1 | v0.2 |

### User Journey MVP

```
1. npm install -g dokbot          # Installation
2. dokbot init                     # Neue Knowledge Base erstellen
3. dokbot add                      # Wissen eingeben (interaktiv)
   → Agent fragt: "Worum geht es?"
   → User: "Wie wir Rechnungen stellen..."
   → Agent fragt nach: Intervall? Tool? Verantwortlich?
   → Agent erstellt: prozesse/fakturierung.md
   → Git-Commit: "add: Fakturierungsprozess dokumentiert"

4. dokbot ask "Wie fakturieren wir?"  # Wissen abrufen
   → Antwort basierend auf gespeichertem Wissen

5. dokbot serve                    # MCP-Server starten
   → Andere KI-Tools können jetzt die KB abfragen
```

### Wissenstypen (fest im MVP, Custom-Typen in v0.2)

| Typ | Ordner | Spezifische Felder im Frontmatter |
|-----|--------|----------------------------------|
| **Prozess** | `prozesse/` | verantwortlich, beteiligte, trigger, schritte, tools, frequenz |
| **Anleitung** | `anleitungen/` | voraussetzungen, schritte, tipps |
| **Know-how** | `knowhow/` | kontext, kernaussage, quellen |
| **Entscheidung** | `entscheidungen/` | optionen, begruendung, datum, beteiligte |
| **Sitzungsprotokoll** | `protokolle/` | datum, teilnehmer, beschluesse, action_items |

Der KI-Agent erkennt automatisch den passenden Typ und füllt die Felder.

### Datenarchitektur

```
Knowledge Base (Git-Repo)
├── .dokbot/
│   ├── config.yaml          # LLM-Provider, Sprache, etc.
│   ├── schemas/             # Wissenstyp-Definitionen
│   └── index/               # LanceDB Vektor-Index (abgeleitet)
├── prozesse/
│   └── fakturierung.md
├── anleitungen/
│   └── deployment-guide.md
├── knowhow/
│   └── api-design-patterns.md
├── entscheidungen/
│   └── 2026-04-retro-einfuehrung.md
└── protokolle/
    └── 2026-04-17-weekly.md
```

**Markdown = Source of Truth** — der LanceDB-Index unter `.dokbot/index/` ist abgeleitet und kann jederzeit via `dokbot reindex` neu aufgebaut werden.

### Technische MVP-Entscheidungen

- **Sprache**: TypeScript (Node.js)
- **Package Manager**: pnpm
- **LLM-Abstraktion**: Vercel AI SDK (unterstützt Claude, OpenAI, Ollama out-of-the-box)
- **Vektor-Index**: LanceDB (embedded, datei-basiert, keine externe DB nötig)
- **Embeddings**: Via Vercel AI SDK (gleicher Provider wie LLM, oder dediziertes Embedding-Modell)
- **Git**: isomorphic-git (kein natives Git als Dependency)
- **CLI Framework**: Commander.js
- **MCP**: @modelcontextprotocol/sdk
- **Monorepo**: Turborepo mit Packages: `core`, `cli`, `mcp-server`

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

---

### A — Wissen erfassen

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| A1 | Wissen in natürlicher Sprache eingeben (interaktiv via CLI) | Wissensgebender | Idee |
| A2 | Bestehendes Dokument importieren (Markdown, Text, PDF) | Wissensgebender | Idee |
| A3 | Wissen via Pipe/stdin von anderem Tool empfangen | Externe Maschine | Idee |
| A4 | Sitzungsprotokoll aus Gesprächsnotizen erstellen | Wissensgebender | Idee |
| A5 | Entscheidung mit Begründung und Alternativen dokumentieren | Wissensgebender | Idee |
| A6 | Wissen via MCP-Schreibzugriff von anderer KI empfangen | Externe Maschine | Idee |
| A7 | Bulk-Import von bestehendem Wissen (z.B. Confluence-Export, Ordner mit Docs) | Wissensgebender | Ergänzung |
| A8 | Voice-to-Knowledge: Transkript von Meeting/Sprachnotiz verarbeiten | Wissensgebender | Ergänzung |
| A9 | E-Mail oder Chat-Nachricht als Wissen erfassen | Externe Maschine | Ergänzung |

### B — Wissen strukturieren & dokumentieren

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| B1 | Eingegebenes Wissen automatisch in strukturiertes Markdown umwandeln | Dokbot-Agent | Idee |
| B2 | Passenden Wissenstyp automatisch erkennen und zuweisen | Dokbot-Agent | Idee |
| B3 | Wissen in die richtige Kategorie/Ordner einordnen | Dokbot-Agent | Idee |
| B4 | Frontmatter (Titel, Tags, Datum, Verantwortlich) automatisch befüllen | Dokbot-Agent | Idee |
| B5 | Verwandte Dokumente automatisch verknüpfen (Cross-Links) | Dokbot-Agent | Idee |
| B6 | Duplikate erkennen und Zusammenführung vorschlagen | Dokbot-Agent | Ergänzung |
| B7 | Glossarbegriffe automatisch erkennen und verlinken | Dokbot-Agent | Ergänzung |
| B8 | Inhaltsverzeichnis / Index über alle Dokumente pflegen | Dokbot-Agent | Ergänzung |

### C — Informationsqualität sichern

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| C1 | Informationslücken beim Erfassen erkennen und Rückfragen stellen | Dokbot-Agent | Idee |
| C2 | Schreibfehler und Formatierungsprobleme automatisch korrigieren | Dokbot-Agent | Idee |
| C3 | Veraltetes Wissen identifizieren und Aktualisierung anfordern | Dokbot-Agent | Ergänzung |
| C4 | Widersprüche zwischen Dokumenten erkennen und melden | Dokbot-Agent | Ergänzung |
| C5 | Qualitätsprüfung: Ist ein Dokument vollständig und verständlich? | Dokbot-Agent | Ergänzung |
| C6 | Regelmässige Review-Zyklen für Prozessdokumente auslösen | Dokbot-Agent | Ergänzung |

### D — Revisionskontrolle & Änderungshistorie

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| D1 | Jede Änderung mit Autor, Datum und Beschreibung protokollieren | Dokbot-Agent | Idee |
| D2 | Änderungshistorie eines Dokuments einsehen | Wissenskonsument | Idee |
| D3 | Zwei Versionen eines Dokuments vergleichen (Diff) | Wissenskonsument | Idee |
| D4 | Änderung rückgängig machen (Rollback) | Reviewer/Admin | Idee |
| D5 | Gesamte Änderungsübersicht über einen Zeitraum anzeigen | Wissenskonsument | Idee |
| D6 | Umstrukturierungen nachvollziehbar machen (Verschiebungen, Umbenennungen) | Dokbot-Agent | Idee |

### E — Zugriffsrechte & Freigabe-Workflows

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| E1 | Änderung an einem Prozessdokument zur Freigabe einreichen | Wissensgebender | Idee |
| E2 | Freigabe für eine Prozessänderung erteilen oder ablehnen | Reviewer/Admin | Idee |
| E3 | Änderung an Know-how-Dokument ohne Freigabe direkt speichern | Wissensgebender | Idee |
| E4 | Schreibfehler-Korrektur ohne Freigabe direkt speichern | Wissensgebender | Idee |
| E5 | Regeln definieren, welche Dokumenttypen Freigabe benötigen | Reviewer/Admin | Idee |
| E6 | Zugriff auf bestimmte Wissensbereiche beantragen | Wissenskonsument | Idee |
| E7 | Rollen und Berechtigungen pro Wissensbereich verwalten | Reviewer/Admin | Ergänzung |
| E8 | Benachrichtigung bei ausstehenden Freigaben erhalten | Reviewer/Admin | Ergänzung |

### F — Wissen konsumieren (Mensch)

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| F1 | Markdown-Dokumente direkt lesen | Wissenskonsument | Idee |
| F2 | Semantisch nach Wissen suchen (Freitext-Suche) | Wissenskonsument | Idee |
| F3 | Frage an die Knowledge Base stellen und KI-Antwort erhalten (RAG) | Wissenskonsument | Ergänzung |
| F4 | Alle Dokumente auflisten, filtern nach Typ/Kategorie/Tags | Wissenskonsument | Ergänzung |
| F5 | Zusammenhänge zwischen Wissensbereichen als Graph visualisieren | Wissenskonsument | Idee |
| F6 | Zusammenfassung eines Wissensbereichs generieren lassen | Wissenskonsument | Ergänzung |
| F7 | Onboarding-Paket: "Was muss ein neuer Mitarbeiter wissen?" generieren | Wissenskonsument | Ergänzung |
| F8 | Wissen exportieren (PDF, HTML, gebündeltes Markdown) | Wissenskonsument | Ergänzung |

### G — Wissen konsumieren (Maschine)

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| G1 | Wissen via MCP-Server abfragen (search, get, ask) | Externe Maschine | Idee |
| G2 | Markdown-Files direkt aus dem Git-Repo lesen | Externe Maschine | Idee |
| G3 | Wissen via REST-API abfragen | Externe Maschine | Idee |
| G4 | Änderungsbenachrichtigungen empfangen (Webhook/Event) | Externe Maschine | Ergänzung |
| G5 | Strukturierte Daten aus der KB extrahieren (JSON-Export) | Externe Maschine | Ergänzung |
| G6 | Wissenskontext in anderen KI-Agenten nutzen (als MCP-Ressource) | Externe Maschine | Idee |

### H — Auswertungen & Analytics

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| H1 | Übersicht über den Wissensbestand (Anzahl Docs, Typen, Abdeckung) | Wissenskonsument | Idee |
| H2 | Änderungsübersicht: Was wurde kürzlich hinzugefügt/geändert? | Wissenskonsument | Idee |
| H3 | Wissens-Lücken-Report: Welche Bereiche sind schlecht dokumentiert? | Reviewer/Admin | Ergänzung |
| H4 | Nutzungsstatistik: Welche Dokumente werden am meisten abgefragt? | Reviewer/Admin | Ergänzung |
| H5 | Aktivitätsübersicht: Wer hat was beigetragen? | Reviewer/Admin | Ergänzung |
| H6 | Altersbericht: Welche Dokumente wurden lange nicht aktualisiert? | Dokbot-Agent | Ergänzung |

### I — Administration & Setup

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| I1 | Knowledge Base initialisieren (Git-Repo + Konfiguration) | Reviewer/Admin | Ergänzung |
| I2 | LLM-Provider konfigurieren (Claude, OpenAI, Ollama) | Reviewer/Admin | Ergänzung |
| I3 | Wissenstypen und Kategorien konfigurieren | Reviewer/Admin | Ergänzung |
| I4 | Vektor-Index neu aufbauen | Reviewer/Admin | Ergänzung |
| I5 | Backup und Restore der Knowledge Base | Reviewer/Admin | Ergänzung |
| I6 | MCP-Server starten und konfigurieren | Reviewer/Admin | Ergänzung |
| I7 | Sprache und Lokalisierung einstellen | Reviewer/Admin | Ergänzung |

---

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
- [ ] Detaillierte Architektur ausarbeiten (Package-Struktur, Datenfluss, Schnittstellen)
- [ ] Prompt-Design für den KI-Agent (System-Prompts, Rückfrage-Logik, Typerkennung)
- [ ] Datenmodell im Detail (Frontmatter-Schema, Config-Format)
- [ ] MCP-Server Spezifikation (Tools, Resources, Input/Output)
- [ ] CLI UX-Design (Interaktionsflüsse, Output-Format)
- [ ] LLM-Abstraktionsschicht (Provider-Wechsel, Embedding-Strategie)
- [ ] Monorepo-Setup (Turborepo, Package-Struktur, CI/CD)
- [ ] AGPL-Lizenz korrekt einrichten
- [ ] Prototyp: Core + CLI + MCP-Server
- [ ] Dogfooding bei Raptus
