# Dokbot — Projektplan

## Context

Dokbot ist ein KI-gestützter Wissensverwalter. Er nimmt Wissen entgegen, strukturiert es als Markdown in Git und macht es für Menschen und Maschinen zugänglich.

**Einsatzbereiche:**
- **Organisationen** (Teams, KMU, Abteilungen) — internes Know-how, Prozesse, Entscheidungen
- **Einzelpersonen** — persönliche Wissensdatenbank, Notizen, Lernmaterial
- **Fachgruppen** — Forschungsnotizen, technische Dokumentation, Community-Wissen

Dokbot ist **domänenagnostisch**: Welche Wissenstypen verwaltet werden, bestimmt ein konfigurierbares Template-System. Vordefinierte Template-Packs decken häufige Anwendungsfälle ab.

**Lizenz**: AGPL-3.0 (Community), proprietär (Pro). Open-Core-Modell.

---

## Vision & Kernfunktionen

1. **Informationsaufnahme** — Wissen aus verschiedenen Quellen (Menschen, KI-Agenten, Skripte), Erkennung von Lücken mit Rückfragen
2. **Strukturierung** — Automatisches Ordnen, Kategorisieren und Dokumentieren als Markdown
3. **Revisionskontrolle** — Wer hat wann was geändert, Versionierung aller Dokumente
4. **Freigabe-Workflows** — Kritische Änderungen mit Freigabe, unkritische ohne
5. **Template-System** — Konfigurierbare Wissenstypen mit Frontmatter-Schema, Ordnerstruktur und Validierung
6. **Menschliche Interfaces** — Markdown-Dokumente, (sekundär) Graph-Visualisierungen
7. **Maschinen-Interfaces** — MCP-Server, Dateizugriff, REST-API
8. **Auswertungen** — Dashboard, Änderungsübersicht, Wissensstatistiken

### Scope-Abgrenzung

| In Scope | Out of Scope |
|----------|-------------|
| Strukturiertes Wissen jeder Domäne | Echtzeit-Daten, Live-Dashboards |
| Versionierte Markdown-Dokumente in Git | Binärdateien, Medienverwaltung |
| Konfigurierbare Wissenstypen (Templates) | Projektmanagement, Ticketing |
| Semantische Suche und RAG | Volltextsuche über Nicht-Markdown-Formate |
| Zugriffstrennung via separate Instanzen | Zugriffskontrolle / RBAC innerhalb einer Instanz |

---

## MVP-Scope (Community Edition v0.1)

> "Sag mir was du weisst — ich schreibe es sauber auf, ordne es ein und mache es für dich, dein Team und eure KI-Tools abrufbar."

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
- `dokbot init --template <pack>` — Initialisiert mit vordefiniertem Template-Pack
- `.dokbot/config.yaml` — LLM-Provider, Template-Pack, Sprache

### Bewusst NICHT im MVP

| Feature | Geplant für |
|---------|-------------|
| Web-UI, SSO, Analytics, Graph-Viz | Pro Edition |
| Freigabe-Workflows (Vier-Augen-Prinzip) | v0.3 / Pro Edition |
| Elektronische Signaturen (21 CFR Part 11) | Pro Edition |
| Compliance-Dashboard | Pro Edition |
| Custom-Wissenstypen (eigene Schemas) | v0.2 |
| Template-Pack `research` | v0.2 |
| GitLab-Provider | v0.2 |
| Signierte Commits | v0.2 |
| MCP-Schreibzugriff | v0.2 |
| Automatische Lücken-Erkennung | v0.2 |
| CI/CD-Integration | v0.2 |

### User Journeys

**Organisation (Team/KMU):**
```
1. npm install -g dokbot
2. dokbot init --template organization
3. dokbot add
   → "Worum geht es?" → "Wie wir Rechnungen stellen..."
   → Rückfragen: Intervall? Tool? Verantwortlich?
   → Erstellt: prozesse/fakturierung.md
   → Git-Commit: "add: Fakturierungsprozess dokumentiert"
4. dokbot ask "Wie fakturieren wir?"
5. dokbot server start  →  MCP-Daemon läuft, Langdock/Claude/Cursor können zugreifen
```

**Persönliche Knowledge Base:**
```
1. npm install -g dokbot
2. dokbot init --template personal
3. dokbot add
   → "Worum geht es?" → "Ich habe gerade das Buch 'Thinking Fast and Slow' gelesen..."
   → Rückfragen: Kernaussagen? Bewertung? Zitate?
   → Erstellt: rezensionen/thinking-fast-and-slow.md
   → Git-Commit: "add: Rezension Thinking Fast and Slow"
4. dokbot ask "Was weiss ich über kognitive Verzerrungen?"
5. dokbot server start  →  Claude Code / Cursor können auf persönliches Wissen zugreifen
```

---

## Wissenstypen & Template-System

Wissenstypen sind **konfigurierbar**. Jeder Typ definiert einen Ordner, Frontmatter-Felder und Validierungsregeln. Dokbot liefert vordefinierte **Template-Packs** für häufige Anwendungsfälle mit. Nutzer können Packs kombinieren, einzelne Typen hinzufügen/entfernen oder eigene Typen definieren.

```
dokbot init                          # Interaktiv: Pack wählen
dokbot init --template organization  # Organisations-Pack
dokbot init --template personal      # Persönliches Wissensmanagement
dokbot init --template organization,personal  # Kombiniert
```

### Template-Architektur

Jeder Wissenstyp ist eine YAML-Datei in `.dokbot/schemas/`:

```yaml
# .dokbot/schemas/process.yaml
name: process
label: Prozess
folder: prozesse
fields:
  - name: verantwortlich
    type: string
    required: true
  - name: beteiligte
    type: string[]
  - name: trigger
    type: string
  - name: schritte
    type: string[]
    required: true
  - name: tools
    type: string[]
  - name: frequenz
    type: string
  - name: output
    type: string
requires_approval: true
```

**MVP**: Packs werden mitgeliefert (built-in), Schema-Dateien werden bei `dokbot init` generiert. Custom-Typen ab v0.2.

### Template-Pack: `organization`

Für Teams und Unternehmen — internes Know-how, Prozesse, Entscheidungen, Rollen.

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

### Template-Pack: `personal`

Für persönliches Wissensmanagement — Lernen, Notizen, Lesezeichen, Ideen.

| Typ | Ordner | Felder im Frontmatter |
|-----|--------|-----------------------|
| Notiz | `notizen/` | kontext, tags |
| Idee | `ideen/` | bereich, status (offen/in_arbeit/umgesetzt/verworfen), naechster_schritt |
| Lesezeichen | `lesezeichen/` | url, zusammenfassung, tags, gelesen (boolean) |
| Lernnotiz | `lernen/` | thema, quelle, kernpunkte, level (beginner/intermediate/advanced), naechste_schritte |
| Journal | `journal/` | datum, stimmung, highlights, erkenntnisse |
| Ziel | `ziele/` | bereich, messbar, deadline, fortschritt, meilensteine |
| Rezension | `rezensionen/` | werk, autor, typ (buch/artikel/kurs/talk), bewertung, kernaussagen, zitate |
| Projekt | `projekte/` | status (idee/aktiv/pausiert/abgeschlossen), ziel, naechste_schritte, ressourcen |
| Referenz | `referenzen/` | thema, inhalt, quelle, zuletzt_verifiziert |
| Kontakt | `kontakte/` | name, kontext, erreichbarkeit, notizen |

### Template-Pack: `research` (v0.2)

Für Forschung und technische Dokumentation.

| Typ | Ordner | Felder im Frontmatter |
|-----|--------|-----------------------|
| Experiment | `experimente/` | hypothese, methode, ergebnis, status (geplant/laufend/abgeschlossen), datum |
| Literatur | `literatur/` | titel, autoren, jahr, doi, zusammenfassung, relevanz |
| Konzept | `konzepte/` | definition, verwandte_konzepte, quellen, beispiele |
| Datensatz | `datensaetze/` | name, quelle, format, groesse, beschreibung, lizenz |
| Protokoll | `protokolle/` | experiment, datum, beobachtungen, abweichungen, naechste_schritte |

### Gemeinsame Typen (in allen Packs enthalten)

Einige Wissenstypen sind universell und in jedem Pack verfügbar:

| Typ | Ordner | Felder im Frontmatter |
|-----|--------|-----------------------|
| Glossar | `glossar/` | begriff, definition, kontext, synonyme |
| FAQ | `faq/` | frage, antwort, kategorie, verwandte_docs |
| Checkliste | `checklisten/` | anwendungsfall, schritte, verantwortlich, frequenz |

### Custom-Typen (ab v0.2)

Nutzer können eigene Wissenstypen definieren:

```bash
dokbot type add           # Interaktiv: Name, Ordner, Felder definieren
dokbot type list          # Alle konfigurierten Typen anzeigen
dokbot type show <typ>    # Schema eines Typs anzeigen
```

Eigene Typen werden als YAML in `.dokbot/schemas/` gespeichert und sind sofort nutzbar.

### Compliance-Felder im Frontmatter (alle Wissenstypen)

Zusätzlich zu den typspezifischen Feldern unterstützen alle Dokumente optionale Compliance-Felder:

| Feld | Typ | Ab Version | Beschreibung |
|------|-----|-----------|-------------|
| `status` | enum | MVP | `draft` · `review` · `approved` · `effective` · `archived` |
| `approved_by` | string | v0.2 | Wer hat freigegeben (Name oder ID) |
| `approved_at` | date | v0.2 | Zeitpunkt der Freigabe |
| `review_comment` | string | v0.2 | Begründung der Freigabe/Ablehnung |
| `contains_personal_data` | boolean | v0.2 | DSG/DSGVO-Kennzeichnung |
| `retention_until` | date | v0.3 | Aufbewahrungsfrist |
| `classification` | enum | v0.3 | `public` · `internal` · `confidential` · `restricted` |

---

## Datenarchitektur

```
Knowledge Base (Git-Repo)
├── .dokbot/
│   ├── config.yaml          # LLM-Provider, Template-Pack, Sprache
│   ├── schemas/             # Wissenstyp-Definitionen (YAML pro Typ)
│   └── index/               # LanceDB Vektor-Index (abgeleitet)
├── <ordner>/                # Ordner pro Wissenstyp (aus Schema)
├── ...                      # Struktur hängt vom gewählten Template-Pack ab
└── ...
```

**Beispiel mit Pack `organization`:**
```
├── prozesse/  anleitungen/  knowhow/  entscheidungen/  protokolle/
├── rollen/  richtlinien/  glossar/  tools/  kontakte/
└── vorlagen/  faq/  checklisten/  ki-skills/
```

**Beispiel mit Pack `personal`:**
```
├── notizen/  ideen/  lesezeichen/  lernen/  journal/
├── ziele/  rezensionen/  projekte/  referenzen/  kontakte/
└── glossar/  faq/  checklisten/
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
┌───────────────────────────────────────────────────────────────┐
│                      Dokbot Core (OSS)                        │
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                   Agentensystem                         │  │
│  │                                                         │  │
│  │  ┌───────────────────────────────────────────────────┐  │  │
│  │  │              Koordinator-Agent                    │  │  │
│  │  │     (Empfang, Routing, Orchestrierung)            │  │  │
│  │  └──────────┬──────────┬──────────┬──────────────────┘  │  │
│  │             ▼          ▼          ▼                      │  │
│  │  ┌──────────────┐ ┌────────┐ ┌────────────────────┐     │  │
│  │  │ Funktionale  │ │ Domäne │ │ Hintergrund-       │     │  │
│  │  │ Agenten      │ │ Agenten│ │ Agenten            │     │  │
│  │  │ (Struktur,   │ │ (pro   │ │ (Qualität,         │     │  │
│  │  │  Suche, Git) │ │  Typ)  │ │  Verknüpfungen)    │     │  │
│  │  └──────────────┘ └────────┘ └────────────────────┘     │  │
│  └─────────────────────────────────────────────────────────┘  │
│                                                               │
│  ┌──────────┐  ┌──────────┐  ┌───────────────┐               │
│  │ Git-     │  │ Vektor-  │  │ Template-     │               │
│  │ Storage  │  │ Index    │  │ Engine        │               │
│  │(Markdown)│  │(LanceDB) │  │ (Schemas)     │               │
│  │  = SoT   │  │          │  │               │               │
│  └──────────┘  └──────────┘  └───────────────┘               │
│                                                               │
│  ┌──────────┐  ┌──────────┐  ┌───────────────┐               │
│  │ CLI      │  │ REST-API │  │ MCP-Server    │               │
│  └──────────┘  └──────────┘  └───────────────┘               │
└───────────────────────────────────────────────────────────────┘
         │              │               │
┌────────┴──────────────┴───────────────┴──────────┐
│              Dokbot Pro (Paid)                    │
│  Web-UI · Approvals · SSO · Analytics            │
│  Graph-Viz · Multi-Tenant · Hosting              │
│  Compliance · e-Signaturen · Audit                │
└──────────────────────────────────────────────────┘
```

---

## Agentensystem

Dokbot verwendet ein **Multi-Agent-System** mit klarer Arbeitsteilung. Ein Koordinator-Agent empfängt alle Anfragen und delegiert an spezialisierte Agenten. Kein Agent arbeitet direkt mit dem Benutzer — die Kommunikation läuft immer über den Koordinator.

### Architektur-Prinzipien

- **Koordinator ist nicht operativ** — er analysiert, routet und orchestriert, schreibt aber selbst keine Dokumente
- **Agenten sind zustandslos** — jeder Aufruf enthält den vollständigen Kontext
- **Agenten sind zusammensetzbar** — der Koordinator kann mehrere Agenten sequenziell oder parallel einsetzen
- **Domänen-Agenten werden dynamisch geladen** — basierend auf den konfigurierten Template-Packs
- **Implementierung via Vercel AI SDK** — Multi-Agent-Orchestrierung mit Tool-Calling

### Koordinator-Agent

Der zentrale Einstiegspunkt für alle Operationen. Er selbst erzeugt keine Dokumente und verändert keine Daten.

| Aufgabe | Beschreibung |
|---------|-------------|
| **Analyse** | Erkennt die Absicht des Benutzers (neues Wissen, Suche, Frage, Update) |
| **Routing** | Wählt den passenden Agenten oder die Agenten-Kette |
| **Orchestrierung** | Koordiniert mehrstufige Abläufe (z.B. Erfassung → Strukturierung → Verknüpfung) |
| **Qualitätskontrolle** | Prüft Agenten-Ergebnisse vor der Rückgabe an den Benutzer |
| **Kontext-Anreicherung** | Ergänzt Anfragen mit relevantem Kontext aus der Knowledge Base |

```
Benutzer: "Wir haben gestern entschieden, auf Hono statt Express zu wechseln"
    │
    ▼
Koordinator
    ├── 1. Analyse: Entscheidung erkannt
    ├── 2. Delegiert an: Entscheidungs-Agent (Domäne)
    ├── 3. Delegiert an: Verknüpfungs-Agent → findet tools/express.md, tools/hono.md
    ├── 4. Delegiert an: Strukturierungs-Agent → erstellt Dokument
    ├── 5. Prüft Ergebnis
    └── 6. Rückgabe an Benutzer: "Entscheidung dokumentiert, 2 Verknüpfungen erstellt"
```

### Funktionale Agenten (horizontal)

Arbeiten aufgabenübergreifend, unabhängig vom Wissenstyp.

| Agent | Aufgabe | MVP? |
|-------|---------|:----:|
| **Strukturierungs-Agent** | Wandelt unstrukturierten Input in Markdown mit Frontmatter um. Wählt den passenden Wissenstyp, befüllt Felder, generiert Dateinamen. | ✓ |
| **Such-Agent** | Führt semantische Suchen durch, findet relevante Dokumente, bereitet Kontext für RAG auf. | ✓ |
| **Git-Agent** | Erstellt Commits, verwaltet Branches, führt Diffs durch. Einziger Agent mit Git-Schreibzugriff. | ✓ |
| **Rückfrage-Agent** | Erkennt fehlende Informationen anhand des Schemas und stellt gezielte Rückfragen. | ✓ |
| **Verknüpfungs-Agent** | Findet verwandte Dokumente, erstellt Cross-Links, erkennt Duplikate. | v0.2 |
| **Qualitäts-Agent** | Prüft Dokumente auf Vollständigkeit, Konsistenz, Aktualität. Erkennt Widersprüche. | v0.2 |
| **Audit-Agent** | Erstellt Änderungsberichte, exportiert Audit-Trails, überwacht Compliance-Felder. | v0.2 |
| **Import-Agent** | Verarbeitet externe Formate (PDF, Confluence, Markdown-Bundles) und wandelt sie in Dokbot-Dokumente um. | v0.3 |

### Domänen-Agenten (vertikal)

Spezialisiert auf einen Wissenstyp. Kennen die Besonderheiten ihres Typs: welche Felder kritisch sind, welche Rückfragen typisch sind, wie ein gutes Dokument dieses Typs aussieht.

**Domänen-Agenten werden dynamisch geladen** — nur für die im Template-Pack konfigurierten Wissenstypen. Jeder Domänen-Agent erhält einen typspezifischen System-Prompt.

#### Pack `organization`

| Agent | Spezialisierung |
|-------|----------------|
| **Prozess-Agent** | Erfragt Trigger, Schritte, Verantwortliche. Erkennt fehlende Schritte, schlägt Verbesserungen vor. |
| **Protokoll-Agent** | Extrahiert Beschlüsse und Action Items aus Gesprächsnotizen. Verknüpft mit vorherigen Sitzungen. |
| **Entscheidungs-Agent** | Strukturiert Optionen, Begründung, Beteiligte. Verknüpft mit betroffenen Prozessen/Tools. |
| **Richtlinien-Agent** | Prüft auf Geltungsbereich und Freigabe. Erkennt Konflikte mit bestehenden Richtlinien. |
| **FAQ-Agent** | Wandelt Fragen in FAQ-Einträge um. Erkennt ob eine Frage bereits beantwortet ist und schlägt Updates vor. |
| **Checklisten-Agent** | Erstellt prüfbare Schrittfolgen. Verknüpft mit zugrundeliegenden Prozessen. |
| **Anleitungs-Agent** | Strukturiert Schritt-für-Schritt. Erfragt Voraussetzungen und geschätzte Dauer. |
| **KI-Skill-Agent** | Formuliert Anweisungen für externe KI-Agenten. Validiert Prompt-Qualität. |

#### Pack `personal`

| Agent | Spezialisierung |
|-------|----------------|
| **Journal-Agent** | Strukturiert tägliche Einträge. Erkennt wiederkehrende Themen über Zeit. |
| **Lern-Agent** | Extrahiert Kernpunkte aus Lernmaterial. Verknüpft mit bestehendem Wissen, schlägt nächste Schritte vor. |
| **Rezensions-Agent** | Strukturiert Bewertungen. Extrahiert Kernaussagen und Zitate. |
| **Ziel-Agent** | Formuliert messbare Ziele. Verfolgt Fortschritt, erinnert an Meilensteine. |
| **Projekt-Agent** | Verwaltet Projektstatus. Verknüpft mit Zielen, Ideen und Referenzen. |

#### Pack `research` (v0.2)

| Agent | Spezialisierung |
|-------|----------------|
| **Experiment-Agent** | Strukturiert Hypothese → Methode → Ergebnis. Verknüpft mit Literatur. |
| **Literatur-Agent** | Extrahiert bibliographische Daten. Erstellt Zusammenfassungen, verknüpft mit Konzepten. |
| **Konzept-Agent** | Definiert Begriffe im Forschungskontext. Baut Begriffsnetze auf. |

### Hintergrund-Agenten

Laufen periodisch oder event-gesteuert, nicht als Reaktion auf Benutzeranfragen.

| Agent | Trigger | Aufgabe | MVP? |
|-------|---------|---------|:----:|
| **Verknüpfungs-Scanner** | Nach jedem Commit, periodisch | Findet neue Zusammenhänge zwischen Dokumenten, schlägt Cross-Links vor | v0.2 |
| **Aktualitäts-Prüfer** | Periodisch (konfigurierbar) | Identifiziert veraltete Dokumente, schlägt Reviews vor | v0.2 |
| **Konsistenz-Prüfer** | Nach jedem Commit | Erkennt Widersprüche zwischen Dokumenten | v0.3 |
| **Index-Updater** | Nach jedem Commit | Aktualisiert den Vektor-Index für geänderte Dokumente | ✓ |

### Agenten-Komposition

Der Koordinator kann Agenten zu **Ketten** zusammensetzen. Typische Abläufe:

**Wissen erfassen (einfach):**
```
Koordinator → Rückfrage-Agent → Strukturierungs-Agent → Git-Agent
```

**Wissen erfassen (mit Domäne):**
```
Koordinator → Rückfrage-Agent → Domänen-Agent → Strukturierungs-Agent
           → Verknüpfungs-Agent → Git-Agent
```

**Frage beantworten (RAG):**
```
Koordinator → Such-Agent → Koordinator (Antwort-Synthese)
```

**Qualitätsprüfung (Hintergrund):**
```
Aktualitäts-Prüfer → Qualitäts-Agent → Koordinator (Report)
```

### Agenten-Konfiguration

Jeder Agent wird durch einen **System-Prompt** und eine **Tool-Liste** definiert. Domänen-Agenten erhalten zusätzlich das Schema ihres Wissenstyps.

```yaml
# .dokbot/agents/process-agent.yaml (generiert aus Schema)
name: process-agent
description: Spezialisiert auf Prozess-Dokumentation
base_agent: domain           # Erbt vom Domain-Agent-Template
schema: process              # Referenz auf .dokbot/schemas/process.yaml
tools:
  - read_document
  - search_knowledge
  - suggest_fields
system_prompt_extras: |
  Du bist spezialisiert auf die Dokumentation von Geschäftsprozessen.
  Frage immer nach: Trigger, Schritte, Verantwortliche, Output.
  Achte auf vollständige Schrittfolgen ohne Lücken.
```

**MVP**: Agenten-Konfigurationen werden aus den Template-Packs generiert. Custom-Agent-Prompts ab v0.2.

## Tech-Stack

- **Sprache**: TypeScript (Node.js)
- **Package Manager**: pnpm
- **Monorepo**: Turborepo (`core`, `cli`, `mcp-server`)
- **LLM-Abstraktion**: Vercel AI SDK (Claude, OpenAI, Ollama)
- **Vektor-Index**: LanceDB (embedded, datei-basiert)
- **Git**: isomorphic-git
- **CLI**: Commander.js
- **MCP**: @modelcontextprotocol/sdk

### Git-Provider-Abstraktion

Dokbot verwendet `isomorphic-git` für alle Git-Operationen — das ist bereits transport-agnostisch. Für provider-spezifische Funktionen (Authentifizierung, Webhooks, APIs) wird eine Abstraktionsschicht eingeführt:

```
GitProvider (Interface)
├── GitHubProvider    # MVP — Personal Access Token, OAuth
├── GitLabProvider    # v0.2 — Token, OAuth, Self-Managed Support
└── LocalOnlyProvider # MVP — Kein Remote, nur lokales Git
```

**Warum?** Regulierte Branchen (MedTech, Pharma, Finanz) setzen oft auf GitLab Self-Managed für Datenhoheit und Compliance. Dokbot muss unabhängig vom Git-Hosting funktionieren.

**MVP-Scope**: `LocalOnlyProvider` + `GitHubProvider` (Remote optional). Das Interface wird so definiert, dass weitere Provider ohne Core-Änderungen ergänzt werden können.

---

## Compliance & regulierte Umgebungen

Dokbot wird in Organisationen eingesetzt, die regulatorischen Anforderungen unterliegen können (MedTech, Pharma, Finanz, öffentliche Verwaltung). Das Compliance-Konzept ist in Schichten aufgebaut: Grundlagen im MVP, erweiterte Features in Pro.

### Relevante Standards

| Branche | Standards | Anforderungen an Dokbot |
|---------|-----------|------------------------|
| MedTech | IEC 62304, ISO 13485, MDR/IVDR | Dokumentenlenkung, Audit Trail, Validierung |
| Pharma | GxP, 21 CFR Part 11 | Elektronische Signaturen, Audit Trail, Unveränderbarkeit |
| Finanz | FINMA, DORA, MaRisk | Datenhoheit, Nachvollziehbarkeit, Auslagerungskontrolle |
| Allgemein | DSG (CH), DSGVO (EU) | Personendaten in Wissensdatenbank, Löschpflichten |

### Compliance-Features nach Release

#### Im MVP angelegt (Grundlagen)

| Feature | Beschreibung |
|---------|-------------|
| **Audit Trail via Git** | Jeder Commit = Änderungsdatensatz mit Autor, Datum, Beschreibung. Unveränderbar durch Git-Integrität. |
| **Dokumentenstatus im Frontmatter** | Feld `status` mit Werten: `draft`, `review`, `approved`, `effective`, `archived` |
| **Git-Provider-Abstraktion** | Interface für GitHub, GitLab, lokale Repos — kein Vendor-Lock-in |
| **Lokaler Betrieb** | `LocalOnlyProvider` ermöglicht Betrieb ohne Cloud-Dienste |
| **LLM-Konfiguration** | Wahl des LLM-Providers in `.dokbot/config.yaml` — auch lokale Modelle (Ollama) möglich |

#### v0.2 — Erweiterte Compliance

| Feature | Beschreibung |
|---------|-------------|
| **GitLab-Provider** | Volle Unterstützung für GitLab (Cloud + Self-Managed) |
| **Signierte Commits** | GPG/SSH-Signaturen für kryptographische Autorenverifizierung |
| **Freigabe-Metadaten** | Frontmatter-Felder `approved_by`, `approved_at`, `review_comment` |
| **Audit-Export** | `dokbot audit <doc>` — exportiert vollständige Änderungshistorie als Report |
| **Personendaten-Tagging** | Frontmatter-Feld `contains_personal_data: true` für DSG/DSGVO-Kennzeichnung |

#### v0.3 / Pro — Volle Compliance

| Feature | Beschreibung |
|---------|-------------|
| **Dokumentenlenkung** | Vier-Augen-Prinzip, formale Freigabe-Workflows mit Eskalation |
| **Elektronische Signaturen** | 21 CFR Part 11 -konforme Signaturen (Identität + Zeitstempel + Bedeutung) |
| **Aufbewahrungsfristen** | Automatische Archivierung nach konfigurierbaren Fristen |
| **Lösch-Workflows** | DSG/DSGVO-konforme Löschung mit Nachweis |
| **Compliance-Dashboard** | Übersicht: offene Reviews, abgelaufene Dokumente, fehlende Freigaben |
| ~~Zugriffskontrolle~~ | Bewusst nicht implementiert — Zugriffstrennung über separate Instanzen (siehe Architekturentscheidung Abschnitt E) |
| **On-Premises LLM** | Dokumentierte Konfiguration für self-hosted LLMs (kein Datenabfluss) |
| **Datenresidenz-Konfiguration** | Sicherstellung, dass keine Daten die definierte Region verlassen |

### LLM-Compliance

Besondere Anforderungen an die LLM-Anbindung in regulierten Umgebungen:

| Anforderung | Lösung |
|-------------|--------|
| Kein Datenabfluss an LLM-Training | Vertraglich via API-Provider (Anthropic, OpenAI) oder lokale Modelle |
| Erklärbarkeit | Quellenangabe bei RAG-Antworten (welches Dokument, welche Passage) |
| Auditierbarkeit | Logging von LLM-Anfragen und -Antworten (optional, konfigurierbar) |
| Datenhoheit | Lokale Modelle via Ollama, kein Zwang zu Cloud-LLMs |

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

### E — Freigabe-Workflows

> **Architekturentscheidung: Keine Zugriffsrechte in Dokbot.**
> Dokbot implementiert bewusst **keine Zugriffskontrolle** (kein RBAC, keine Rollen, keine Bereichs-Berechtigungen). Gründe:
> - **Sicherheitsrisiko**: LLM-basierte Systeme sind anfällig für Prompt Injection. Zugriffsrechte, die durch einen KI-Agenten durchgesetzt werden, bieten keine verlässliche Sicherheitsgrenze.
> - **Komplexität**: Zugriffskontrolle in einem Git-basierten System mit LLM-Zugriff ist schwer korrekt umzusetzen und noch schwerer zu auditieren.
> - **Alternative**: Wer Zugriffstrennung braucht, betreibt **mehrere Dokbot-Instanzen** mit getrennten Git-Repos (z.B. eine für HR, eine für Engineering). Das ist einfacher, sicherer und auditierbar.

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| E1 | Prozessänderung zur Freigabe einreichen | Mensch (Wissensgebender), Externe KI | Idee |
| E2 | Freigabe erteilen oder ablehnen | Mensch (Reviewer/Admin) | Idee |
| E3 | Know-how-Änderung ohne Freigabe speichern | Mensch (Wissensgebender), Externe KI | Idee |
| E4 | Schreibfehler-Korrektur ohne Freigabe speichern | Mensch (Wissensgebender), Dokbot-Agent | Idee |
| E5 | Freigabe-Regeln pro Dokumenttyp definieren | Mensch (Reviewer/Admin) | Idee |
| ~~E6~~ | ~~Zugriff auf Wissensbereiche beantragen~~ | — | Entfällt (keine Zugriffsrechte) |
| ~~E7~~ | ~~Rollen und Berechtigungen verwalten~~ | — | Entfällt (keine Zugriffsrechte) |
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
| I1 | Knowledge Base initialisieren (mit Template-Pack-Auswahl) | Mensch (Reviewer/Admin) | Ergänzung |
| I2 | LLM-Provider konfigurieren | Mensch (Reviewer/Admin) | Ergänzung |
| I3 | Custom-Wissenstyp definieren (Name, Ordner, Felder) | Mensch (Reviewer/Admin) | Ergänzung |
| I4 | Vektor-Index neu aufbauen | Mensch (Reviewer/Admin), Skript/System | Ergänzung |
| I5 | Backup und Restore | Mensch (Reviewer/Admin), Skript/System | Ergänzung |
| I6 | MCP-Server (Daemon) starten und konfigurieren | Mensch (Reviewer/Admin) | Ergänzung |
| I7 | Sprache und Lokalisierung einstellen | Mensch (Reviewer/Admin) | Ergänzung |
| I8 | Template-Pack hinzufügen oder entfernen | Mensch (Reviewer/Admin) | Neu |
| I9 | Konfigurierte Wissenstypen auflisten und anzeigen | Mensch (Reviewer/Admin) | Neu |

### J — Compliance & Regulatorik

| # | Use Case | Akteur | Quelle |
|---|----------|--------|--------|
| J1 | Dokumentenstatus setzen (draft/review/approved/effective/archived) | Mensch (Reviewer/Admin), Dokbot-Agent | Neu |
| J2 | Git-Provider wählen (GitHub, GitLab, lokal) | Mensch (Reviewer/Admin) | Neu |
| J3 | Signierte Commits aktivieren (GPG/SSH) | Mensch (Reviewer/Admin) | Neu |
| J4 | Audit-Report für ein Dokument exportieren | Mensch (Reviewer/Admin), Skript/System | Neu |
| J5 | Personendaten-Kennzeichnung setzen (DSG/DSGVO) | Mensch (Wissensgebender), Dokbot-Agent | Neu |
| J6 | Aufbewahrungsfrist definieren und überwachen | Mensch (Reviewer/Admin) | Neu |
| J7 | Klassifizierung vergeben (public/internal/confidential/restricted) | Mensch (Reviewer/Admin) | Neu |
| J8 | LLM-Anfragen und -Antworten auditierbar loggen | Mensch (Reviewer/Admin), Skript/System | Neu |
| J9 | Compliance-Dashboard anzeigen (offene Reviews, abgelaufene Docs) | Mensch (Reviewer/Admin) | Neu |
| J10 | On-Premises LLM konfigurieren (kein Datenabfluss) | Mensch (Reviewer/Admin) | Neu |

### Use Case → Release Zuordnung

| Release | Use Cases |
|---------|-----------|
| **MVP (v0.1)** | A1, A2, A3, B1, B2, B3, B4, C1, C2, D1, D2, D5, F1, F2, F3, F4, G1, G2, I1, I2, I4, I6, I9, **J1, J2** |
| **v0.2** | A4, A5, A6, A10, B5, B6, C3, D3, D6, F6, G3, G5, G7, H1, H2, H6, I3, I8, **J3, J4, J5** |
| **v0.3** | A7, B7, B8, C4, C5, D4, E1–E5, F8, G4, H3, **J6, J7** |
| **Pro Edition** | A8, A9, C6, E8, F5, F7, G6, H4, H5, I5, I7, **J8, J9, J10** |

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
