# Candidate Enrichment Pipeline (n8n + Ollama)

🇳🇱 [Nederlands](#nederlands) · 🇬🇧 [English](#english)

---

## Nederlands

Een automatisering die binnenkomende sollicitaties verrijkt met een lokaal
AI-model **vóórdat een recruiter ze beoordeelt** — van een ruwe sollicitatie
naar een beknopte, gestructureerde recruiter-briefing.

Gebouwd met [n8n](https://n8n.io) voor de orkestratie en een **lokaal
[Ollama](https://ollama.com)-model** voor de AI-stap, zodat er geen
kandidaatgegevens de machine verlaten.

### Wat het doet

Een sollicitatie (naam, functie, motivatie, cv-tekst) komt binnen via een webhook.
De workflow:

1. **Normaliseert** de binnenkomende payload naar een schoon kandidaatobject.
2. **Verrijkt** die met een lokaal LLM (`mistral` via Ollama), dat een
   gestructureerde JSON-beoordeling teruggeeft: samenvatting, jaren ervaring,
   skills, seniority en een fit-score met toelichting.
3. **Parset & valideert** de model-output (met een vangnet als het model geen
   geldige JSON teruggeeft).
4. **Bouwt een recruiter-briefing** — een leesbare samenvatting die de recruiter ziet.

### De pipeline 

```
Sollicitatie  ( webhook) → Normaliseren → AI-verrijking (Ollama) → Response parsen → Recruiter-briefing

```

### Waarom deze opzet

- **Lokaal & privacyvriendelijk** — de AI draait op een self-hosted Ollama, dus
  er gaan geen kandidaatgegevens naar een externe API.
- **Eén verantwoordelijkheid per node** — elke stap is klein en inspecteerbaar,
  waardoor de flow makkelijk te lezen, testen en uitbreiden is.
- **Defensief parsen** — LLM-output is niet gegarandeerd geldige JSON, dus de
  parse-stap heeft een vangnet in plaats van de flow te laten crashen.
- **Verwisselbare sink** — de briefing kan met één extra node naar e-mail, Slack
  of een ATS worden gestuurd.

### Techniek

- **n8n** (self-hosted, Docker) — workflow-orkestratie
- **Ollama** + `mistral` — lokaal LLM voor de verrijking
- **Docker Compose** — reproduceerbare lokale opzet

### Zelf draaien

Vereist: [Docker](https://docs.docker.com/get-docker/) en
[Ollama](https://ollama.com) met een model gepulld (`ollama pull mistral`).

```bash
# 1. Start Ollama (op de host) en pull het model
ollama pull mistral

# 2. Start n8n
docker compose up -d

# 3. Open n8n en importeer de workflow
#    http://localhost:5678  →  workflow.json importeren

# 4. Publiceer de workflow en stuur een test-sollicitatie:
curl -X POST http://localhost:5678/webhook/sollicitatie \
  -H "Content-Type: application/json" \
  -d @examples/sample-candidate.json
```

> Let op: omdat n8n in Docker draait en Ollama op de host, bereikt de workflow
> Ollama via `http://host.docker.internal:11434`.

### Projectstructuur

```

├── README.md
├── workflow.json # de n8n-workflow (importeer deze)
├── docker-compose.yml # n8n-service
├── examples/
│ └── sample-candidate.json
└── docs/
└── flow-screenshot.png

```


### Mogelijke uitbreidingen

- Briefing routeren naar e-mail / Slack / een ATS.
- Deterministische checks toevoegen (telefoon/e-mail normaliseren, dedupe).
- Het model met een strakkere prompt naar één vaste seniority-label dwingen.

---

## English

An automation that enriches incoming job applications with a local AI model
**before a recruiter reviews them** — turning a raw application into a concise,
structured recruiter briefing.

Built with [n8n](https://n8n.io) for orchestration and a **local
[Ollama](https://ollama.com) model** for the AI step, so no candidate data ever
leaves the machine.

### What it does

A candidate application (name, role, motivation, CV text) comes in via a webhook.
The workflow then:

1. **Normalises** the incoming payload into a clean candidate object.
2. **Enriches** it with a local LLM (`mistral` via Ollama), which returns a
   structured JSON assessment: summary, years of experience, skills, seniority,
   and a fit score with justification.
3. **Parses & validates** the model output (with a fallback if the model returns
   invalid JSON).
4. **Builds a recruiter briefing** — a readable summary card the recruiter sees.

### Pipeline

```

Application (webhook) → Normalise → AI enrichment (Ollama) → Parse response → Recruiter briefing

```

### Why this design

- **Local-first / privacy-friendly** — the AI runs on a self-hosted Ollama
  instance, so no candidate data is sent to an external API.
- **One responsibility per node** — each step is small and inspectable, making
  the flow easy to read, test, and extend.
- **Defensive parsing** — LLM output isn't guaranteed to be valid JSON, so the
  parse step has a fallback instead of crashing the flow.
- **Swappable sink** — the final briefing can be routed to email, Slack, or an
  ATS by adding a single node.

### Tech stack

- **n8n** (self-hosted, Docker) — workflow orchestration
- **Ollama** + `mistral` — local LLM for enrichment
- **Docker Compose** — reproducible local setup

### Run it yourself

Prerequisites: [Docker](https://docs.docker.com/get-docker/) and
[Ollama](https://ollama.com) with a model pulled (`ollama pull mistral`).

```bash
# 1. Start Ollama (on the host) and pull the model
ollama pull mistral

# 2. Start n8n
docker compose up -d

# 3. Open n8n and import the workflow
#    http://localhost:5678  →  import workflow.json

# 4. Publish the workflow, then send a test application:
curl -X POST http://localhost:5678/webhook/sollicitatie \
  -H "Content-Type: application/json" \
  -d @examples/sample-candidate.json
```

> Note: because n8n runs in Docker and Ollama runs on the host, the workflow
> reaches Ollama via `http://host.docker.internal:11434`.

### Project structure

```

├── README.md
├── workflow.json # the n8n workflow (import this)
├── docker-compose.yml # n8n service
├── examples/
│ └── sample-candidate.json
└── docs/
└── flow-screenshot.png

```


### Possible extensions

- Route the briefing to email / Slack / an ATS.
- Add deterministic checks (phone/email normalisation, deduplication).
- Constrain the model to a fixed seniority label with a stricter prompt.