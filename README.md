# AnythingLLM + Ollama (lokaal)

Docker Compose-opzet voor een volledig lokale RAG-omgeving: [AnythingLLM](https://anythingllm.com) met [Ollama](https://ollama.com) als model- en embedding-server. Documenten verlaten het netwerk niet (telemetrie uit, lokale vectordatabase LanceDB).

## Gebruik

```powershell
docker compose up -d
docker compose exec ollama ollama pull qwen3:8b
docker compose exec ollama ollama pull gemma2:9b
docker compose exec ollama ollama pull nomic-embed-text
```

Open daarna <http://localhost:3001>, maak een workspace en een API-key aan.

## Ander model

Zet in een `.env` naast `docker-compose.yml`:

```
LLM_MODEL=qwen3:4b
```

Haal het model op met `ollama pull` en maak AnythingLLM opnieuw aan: `docker compose up -d --force-recreate anythingllm`.

## GPU

Het NVIDIA-blok in `docker-compose.yml` vereist de NVIDIA Container Toolkit (of WSL2-GPU-ondersteuning). Zonder GPU: verwijder het `deploy`-blok. Controleer met `docker compose exec ollama ollama ps` of `PROCESSOR` op `100% GPU` staat.

## Data

Modellen en AnythingLLM-data staan in de Docker-volumes `ollama` en `anythingllm`.
