# mytravelpartner

A multi-agent AI travel planner. Describe a trip in plain English and get flights, hotels, weather and a day-by-day itinerary in about a minute.

**Live demo:** https://mytravelpartner.vercel.app/

> Based on [TravelBrain](https://github.com/KalyanM45/TravelBrain-Multi-Agent-AI-Travel-Planner) by KalyanM45, licensed under GPL-3.0. This repository is my deployed copy, set up and configured for Vercel + Render.

## How it works

A LangGraph pipeline runs five agents in sequence, and each one writes its result into shared state:

`Flight agent → Hotel agent → Weather agent → Itinerary agent → Final agent`

Agents get live data through MCP (Model Context Protocol) servers:

| Source | Used for |
|---|---|
| Tavily (remote MCP, HTTP) | Web search |
| Aviationstack (local MCP, stdio) | Airports, airlines, flights |
| OpenWeather (local MCP, stdio) | Current weather and forecast |

The LLM is served by Groq. Conversation state is saved in PostgreSQL through the LangGraph checkpointer, so a trip can be reopened and extended with follow-up questions.

## Tech stack

Python, FastAPI, LangGraph, LangChain, Groq LLM, MCP, PostgreSQL, Docker, HTML/CSS/JavaScript, Vercel, Render

## Architecture

```
Browser ──► Vercel (static frontend, dist/)
               │  rewrites /api/* and /health
               ▼
            Render (Docker, FastAPI + LangGraph) ──► Render PostgreSQL
               │
               └──► Groq, Tavily, Aviationstack, OpenWeather
```

## What I did in this repository

- Deployed the backend on Render using a Docker build and a `render.yaml` Blueprint (web service plus managed PostgreSQL)
- Deployed the static frontend on Vercel using `scripts/build-frontend.sh` and `vercel.json` rewrites that proxy API calls to Render
- Configured environment variables and secrets, and debugged the build and deployment issues

## Run locally

Requirements: Python 3.11, [uv](https://docs.astral.sh/uv/), a PostgreSQL database, and free API keys for Groq, Tavily, Aviationstack and OpenWeather.

```bash
git clone https://github.com/shreyaaas17-web/mytravelpartner.git
cd mytravelpartner
uv sync
```

Create a `.env` file (never commit it):

```dotenv
GROQ_API_KEY=
TAVILY_API_KEY=
AVIATIONSTACK_API_KEY=
OPENWEATHER_API_KEY=
DATABASE_URL=postgresql://user:password@host:5432/dbname
```

Start the server:

```bash
uv run uvicorn app:app --reload
```

Open http://localhost:8000.

## Deploy

**Backend (Render):** New → Blueprint → select this repo. Render reads `render.yaml`. Enter the four API keys when prompted.

**Frontend (Vercel):** Import the repo. Vercel runs `bash scripts/build-frontend.sh` and serves `dist/`. In `vercel.json`, set the rewrite destinations to your Render service URL.

## License

GNU General Public License v3.0. See [LICENSE](LICENSE). Original work by KalyanM45.
