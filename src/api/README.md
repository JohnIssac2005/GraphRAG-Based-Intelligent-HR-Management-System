# HR Intelligence Assistant: web front end

A single-page website on top of `src.pipeline.answer()`.

## Run

From the project root (the folder that contains `src/` and `.env`):

    pip install flask python-dotenv
    python -m src.api.server        (python -m api.server if the folder is next to src)

Open http://127.0.0.1:5000

## Files

- `api/server.py`        Flask server: the page, `POST /api/ask`, and read-only `/api/db/*`, `/api/docs/*`, `/api/graph/*`
- `api/static/index.html` the whole interface (HTML, CSS, JavaScript; no build step)
- `api/real_data.py`     read-only access to Postgres (tables), Neo4j (documents, graph) for the Database, Documents and Knowledge Graph pages
- `api/local_db.py`      local SQLite copy used by the Database page (`python -m src.api.local_db` creates or refreshes it)
- `api/guard.py`         screens questions: unauthorised requests and off-topic questions get a fixed custom reply, with no LLM call
- `api/mock_data.py`     saved sample answers and sample tables/documents/graph, used only in mock mode
- `api/__init__.py`      empty, so `python -m api.server` works

## Settings (environment variables or `.env`)

| Name | Meaning |
|---|---|
| `APP_PASSWORD` | if set, users must sign in with this password |
| `SECRET_KEY`   | fixed session key (set it if you use APP_PASSWORD) |
| `HR_MOCK=1`    | answer from two saved samples, no databases or LLM needed (for checking the layout) |
| `PORT`         | port to listen on, default 5000 |

## What the page shows for each answer

Route, confidence, grounding score, time and LLM calls; the answer with coloured
[SQL] / [Graph] / [Doc] tags; the SQL query and rows, the Cypher query, rows and related
facts, and the document passages used; unsupported names or numbers if the grounding check
finds any; follow-up question buttons. Daily-limit and start-up errors appear as messages.

## Pages

A collapsible sidebar switches between Assistant, Database (tables, rows, sorting, search, paging, structure, CSV export), Documents (type filters, search, reader, links to the graph) and Knowledge Graph (interactive: pan, zoom, drag, click for properties, double-click to expand). All three data pages are read-only and use the read-only Postgres role and Neo4j read routing.

## Database page and the local copy

The Database page reads `data/local_snapshot.sqlite` on this computer, never the online server. The first time, press **Create local copy** (or run `python -m src.api.local_db`); this reads PostgreSQL once with the read-only role. **Update local copy** refreshes it. Set `HR_LOCAL_DB` to store the file elsewhere.

## Custom replies

`guard.py` answers without calling the pipeline when a question asks to change or delete data, asks for passwords, keys or other credentials, tries to override the assistant's rules, tries to get around access controls, asks for sensitive personal details (unauthorised), or is unrelated to HR such as weather, jokes, general knowledge or coding (off-topic). Edit the lists and `REPLIES` in that file to change what is caught or said.
