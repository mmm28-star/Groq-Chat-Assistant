# Groq Chat Assistant

A small web chatbot built with Flask, LangChain and the Groq API. It serves a single-page chat interface and keeps a separate conversation history for each browser session, so follow-up questions work the way you would expect.

The project is intentionally compact. The whole backend is one file of about 80 lines, which makes it a convenient starting point for a chatbot you want to understand and extend.

## Features

- Chat interface served by Flask, with no frontend build step
- Conversation memory kept per browser session
- Defaults to `llama-3.1-8b-instant` on Groq, configurable through an environment variable
- `GET /health` endpoint for uptime checks
- Automated tests that run offline with a fake model, so no API key is needed

## Quick start

You need Python 3.9 or newer and a Groq API key (free at https://console.groq.com/keys).

```bash
git clone https://github.com/mmm28-star/Groq-Chat-Assistant.git
cd Groq-Chat-Assistant

python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt

cp .env.example .env            # then set GROQ_API_KEY
python app.py
```

Open http://127.0.0.1:5000 and start chatting.

## Configuration

| Variable | Required | Description |
|---|---|---|
| `GROQ_API_KEY` | yes | Your Groq API key |
| `GROQ_MODEL` | no | Model name, default `llama-3.1-8b-instant` |
| `FLASK_DEBUG` | no | Set to `1` to run Flask in debug mode |

## API

`POST /data`

```json
{ "data": "What is retrieval-augmented generation?", "session_id": "any-string" }
```

Response:

```json
{ "response": true, "message": "..." }
```

Requests that share a `session_id` share conversation history. The web page generates a new id each time it loads, so reloading starts a new conversation. Errors return `{"response": false, "message": "..."}` with a 400 or 500 status.

`GET /health` returns `{"status": "ok"}`.

## How the memory works

Each session id maps to its own LangChain `ConversationChain` with a `ConversationBufferMemory`. The most recent 200 sessions are kept in process memory and the oldest is dropped when that limit is passed.

## Tests

```bash
pip install -r requirements-dev.txt
pytest
```

The tests replace the Groq client with LangChain's `FakeListLLM`, and cover input validation, replies, per-session history, session isolation and eviction.

## Project layout

```
app.py               Flask app, session handling, chat endpoint
templates/index.html chat page
static/              JavaScript and CSS for the page
tests/test_app.py    offline tests
requirements.txt     runtime dependencies
```

## Limitations

- History lives in process memory, so it is lost when the server restarts and is not shared between multiple workers.
- `ConversationChain` is deprecated in recent LangChain releases. It works with the pinned version, but a later upgrade should move to `RunnableWithMessageHistory`.
- There is no authentication or rate limiting, so do not expose it publicly as is.
- Replies arrive in one piece. The typing effect is drawn in the browser.
