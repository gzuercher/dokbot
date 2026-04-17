# Dokbot

> Der KI-gestützte Dokumentations-Bot für Organisationswissen.

Dokbot ist ein KI-Agent, der internes Know-how entgegennimmt, strukturiert, dokumentiert und für Menschen und Maschinen zugänglich macht.

**Status:** 🚧 In Planung — noch nicht produktionsreif.

---

## Was Dokbot tut

- **Wissen erfassen**: Sag dem Dokbot, was du weisst — er schreibt es sauber auf
- **Strukturieren**: Automatische Kategorisierung, Typerkennung und Markdown-Generierung
- **Versionieren**: Jede Änderung wird in Git protokolliert (wer, wann, was)
- **Abrufbar machen**: Semantische Suche, RAG-Antworten, MCP-Server für andere KI-Tools

## Für wen?

Organisationen mit 10–50 Personen, die ihr internes Wissen (Prozesse, Anleitungen, Know-how, Entscheidungen) strukturiert und für Mensch und Maschine zugänglich machen wollen.

## Vision

```
"Frag den Dokbot."
"Steht alles im Dokbot."
"Nutze den MCP von Dokbot."
```

## Dokumentation

- [Projektplan](docs/plan.md) — Vision, Geschäftsmodell, MVP-Scope, Use Cases, Architektur

## Tech Stack

- TypeScript / Node.js
- Vercel AI SDK (LLM-agnostisch: Claude, OpenAI, Ollama)
- Git-basierter Markdown-Storage
- LanceDB (embedded Vektor-Index)
- MCP-Server für Maschinen-Zugriff

## Lizenz

AGPL-3.0 — siehe [LICENSE](LICENSE).

## Team

Ein Produkt von [Raptus AG](https://raptus.ch), Lyss.
