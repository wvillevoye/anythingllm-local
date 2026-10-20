# AnythingLLM + Ollama (lokaal)

Docker Compose-opzet voor een volledig lokale RAG-omgeving: [AnythingLLM](https://anythingllm.com) met [Ollama](https://ollama.com) als model- en embedding-server. Documenten verlaten het netwerk niet (telemetrie uit, lokale vectordatabase LanceDB). Bedoeld voor MarkFluence, dat AnythingLLM via de API gebruikt.

## Gebruik

```powershell
docker compose up -d
docker compose exec ollama ollama pull qwen3:8b
docker compose exec ollama ollama pull gemma2:9b-instruct-q4_K_M
docker compose exec ollama ollama pull nomic-embed-text
```

Open daarna <http://localhost:3001>, maak een workspace en een API-key aan.

De modelnaam moet exact overeenkomen met de naam in `ollama list`. MarkFluence kent bijvoorbeeld `gemma2:9b-instruct-q4_K_M`, niet `gemma2:9b`. Wil je ook `gemma3:4b` (kleiner en sneller) kiezen, haal dat dan ook op met `ollama pull`.

## Ander model

Zet in een `.env` naast `docker-compose.yml`:

```
LLM_MODEL=qwen3:4b
```

Haal het model op met `ollama pull` en maak AnythingLLM opnieuw aan: `docker compose up -d --force-recreate anythingllm`.

`LLM_MODEL` is alleen de standaard van AnythingLLM zelf. In MarkFluence kies je het antwoordmodel in het beheerscherm (tabblad Antwoordmodel), en alleen uit de lijst `AnythingLlm:ToegestaneModellen` in `appsettings.json`.

## GPU

Het NVIDIA-blok in `docker-compose.yml` vereist de NVIDIA Container Toolkit (of WSL2-GPU-ondersteuning). Zonder GPU: verwijder het `deploy`-blok. Controleer met `docker compose exec ollama ollama ps` of `PROCESSOR` op `100% GPU` staat.

### Wat er is ingesteld

De opzet is afgestemd op een laptop met een **NVIDIA RTX 4060 Laptop (8 GB videogeheugen)**. Een model is het snelst als het helemaal in het videogeheugen past; zet Ollama delen op de CPU, dan wordt het trager.

| Instelling | Waarde | Waarom |
|---|---|---|
| `OLLAMA_FLASH_ATTENTION` (ollama) | `1` | Zuiniger met geheugen bij lange context |
| `OLLAMA_KV_CACHE_TYPE` (ollama) | `q8_0` | Kleinere cache voor de context, zodat een 9B-model beter in 8 GB past |
| `OLLAMA_NUM_PARALLEL` (ollama) | `1` | Eén context-slot in plaats van meerdere; spaart videogeheugen |
| `OLLAMA_KEEP_ALIVE` (ollama) | `24h` | Het model blijft geladen; standaard haalt Ollama het na 5 minuten weg |
| `OLLAMA_KEEP_ALIVE_TIMEOUT` (anythingllm) | `86400` | AnythingLLM stuurt bij elke chatvraag zelf een keep-alive mee (standaard 300 s). Die gaat boven `OLLAMA_KEEP_ALIVE`; zonder deze instelling wordt het taalmodel tóch na ongeveer 5 minuten ontladen |
| `OLLAMA_MODEL_TOKEN_LIMIT` (anythingllm) | `8192` | Contextgrootte die AnythingLLM aan Ollama vraagt |

Haal `OLLAMA_KEEP_ALIVE_TIMEOUT` niet weg zonder reden: de uitleg hierboven is de reden dat hij er staat.

### Controleren

Stel een vraag en draai:

```powershell
docker compose exec ollama ollama ps
```

In de kolom `UNTIL` hoort `24 hours from now` te staan. Gemeten op 20 oktober 2026 op de RTX 4060 Laptop: `qwen3:8b` 5,6 GB op `100% GPU` met context 8192, en `nomic-embed-text` (335 MB) ook op `100% GPU`. Samen passen ze in het videogeheugen.

### Snelheid

Gemiddeld van drie runs van 250 tokens, via de Ollama-API met vaste contextgrootte, met de instellingen hierboven:

| Model | Context | Processor | Tokens per seconde |
|---|---|---|---|
| `qwen3:8b` | 8192 | 100% GPU | 29-34 |
| `gemma2:9b-instruct-q4_K_M` | 4096 | 7% CPU / 93% GPU | 18-21 |
| `gemma2:9b-instruct-q4_K_M` | 8192 | 12% CPU / 88% GPU | 17-19 |

`gemma2` past op deze GPU niet helemaal in het videogeheugen (Windows en andere programma's gebruiken al ruim 1 GB). Sluit programma's die videogeheugen gebruiken of zet `OLLAMA_MODEL_TOKEN_LIMIT` lager als je Gemma volledig op de GPU wilt, met als prijs minder bronnen in één antwoord. De winst van flash attention en `q8_0` zelf is op deze computer niet gemeten.

### Na een wijziging

```powershell
docker compose up -d ollama        # na een wijziging bij ollama
docker compose up -d anythingllm   # na een wijziging bij anythingllm
```

De modellen en de index blijven staan (ze zitten in volumes), maar AI-zoeken is tijdens de herstart kort niet beschikbaar.

## Data

Modellen en AnythingLLM-data staan in de Docker-volumes `ollama` en `anythingllm`.
