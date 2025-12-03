# Petra

An AI-powered news briefing system that transforms RSS feeds into personalized, daily email digests.

## What It Does

Petra ingests articles from 20 AI news sources, processes them through a multi-agent LLM pipeline, and delivers personalized briefings tailored to each subscriber's interests.

Each briefing contains three sections:
- **The Landscape** — An executive overview of the day's AI news
- **Your Top 5** — Personalized article picks based on subscriber topics
- **Deep Dives** — Substantive analysis that synthesizes insights across sources

## Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Article        │     │  Briefing       │     │  Subscribe      │
│  Service        │────▶│  Generator      │     │  Service        │
│  (RSS Fetcher)  │     │  (LLM Pipeline) │     │  (User Intake)  │
└─────────────────┘     └────────┬────────┘     └─────────────────┘
                                 │
                                 ▼
                        ┌─────────────────┐
                        │  Email Delivery │
                        │  (Resend API)   │
                        └─────────────────┘
```

### Services

| Service | Port | Description |
|---------|------|-------------|
| `petra-articles` | 8002 | RSS feed aggregator (20 sources) |
| `petra-generator` | 8003 | LLM pipeline + email sender |
| `petra-subscribe` | 8001 | User signup, preferences, unsubscribe |

## Quick Start

### Prerequisites
- Python 3.11+
- OpenAI API key
- Resend API key (for email delivery)

### Local Development

```bash
# Install dependencies
pip install -r node1_requirements.txt
pip install -r node2_requirements.txt
pip install -r article_service_requirements.txt

# Start Article Service
PYTHONPATH=src python3 src/article_service.py

# Start Subscribe Service
python src/node1_backend.py

# Generate a briefing (requires API keys)
OPENAI_API_KEY="sk-..." RESEND_API_KEY="re_..." PYTHONPATH=src python3 src/node2_briefing_generator.py
```

### Environment Variables

| Variable | Description |
|----------|-------------|
| `OPENAI_API_KEY` | OpenAI API key for LLM processing |
| `RESEND_API_KEY` | Resend API key for email delivery |
| `FROM_EMAIL` | Sender email address |
| `ARTICLE_SERVICE_URL` | URL of the article service |

## Deployment (Render)

The `render.yaml` is configured for deployment. Services:

1. **petra-articles** — Article aggregation service
2. **petra-generator** — Briefing generation API with `/trigger` endpoint
3. **petra-subscribe** — Subscription management pages

### Scheduling Daily Emails

Use a free external cron service (e.g., [cron-job.org](https://cron-job.org)) to hit:
```
GET https://petra-generator.onrender.com/trigger
```

## User Flow

1. **Subscribe** — `petra-subscribe.onrender.com/`
2. **Success** — `petra-subscribe.onrender.com/success`
3. **Daily Email** — Delivered to inbox
4. **Preferences** — `petra-subscribe.onrender.com/preferences?email=...`
5. **Unsubscribe** — `petra-subscribe.onrender.com/unsubscribe?email=...`

## API Endpoints

### Article Service (`/`)
- `GET /articles` — Fetch articles (query: `limit`, `since`)
- `GET /health` — Health check

### Generator Service (`/`)
- `GET /trigger` — Trigger briefing generation for all users
- `POST /generate` — Generate briefings (JSON body)
- `GET /preview/{email}` — Preview briefing HTML for a user
- `GET /users` — List all subscribers
- `GET /health` — Health check

### Subscribe Service (`/`)
- `GET /` — Signup form
- `POST /api/intake` — Create subscription
- `GET /preferences` — Preferences page
- `POST /api/preferences` — Update preferences
- `GET /unsubscribe` — Unsubscribe page
- `POST /api/unsubscribe` — Process unsubscribe
- `GET /health` — Health check

## News Sources

Petra aggregates from 20 AI-focused RSS feeds including:
- The Verge AI
- Ars Technica AI
- TechCrunch AI
- MIT Technology Review AI
- VentureBeat AI
- OpenAI Blog
- Anthropic News
- Google AI Blog
- NVIDIA AI Blog
- Hugging Face Blog
- And more...

## Tech Stack

- **Backend**: FastAPI, Python
- **LLM**: OpenAI GPT
- **Email**: Resend API
- **Templating**: Jinja2
- **Hosting**: Render

## License

MIT
