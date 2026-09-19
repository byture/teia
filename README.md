# Teia

**Scaffold and grow production-ready AI agents with MCP, A2A and tracing built in — right from your terminal.**

Teia is an open source Python CLI and library for building AI agents. It creates a production-ready project in one command and, unlike most agent CLIs, keeps helping after day one: you can add agents, graph nodes, MCP servers and A2A endpoints to a project that already exists.

> **Status: early development.** Teia is being built in the open. Commands and APIs described here reflect the planned MVP and may change before the first release.

---

## Why Teia?

Starting an agent project is easy. Growing it without turning it into a mess is not. Most agent CLIs generate a starter project and then leave you alone: every new tool server, agent or protocol endpoint means copying boilerplate, wiring things by hand and hoping you did it the same way as last time.

Teia is built around four ideas:

**Generators that work on existing projects.** `teia add node`, `teia add mcp` and `teia expose a2a` extend the project you already have. Generators only create new files and never rewrite your code, so running them on a project you have customized is safe. Running the same command twice will not duplicate or overwrite anything without asking.

**MCP and A2A at the edges.** Your agent logic sits at the center. MCP is how the agent consumes tools; A2A is how it exposes itself to other agents. Agent code never imports protocol code, so a protocol upgrade does not force you to rewrite your agent, and the same graph can run behind an API, a CLI or an A2A endpoint.

**Production structure from the start.** Every project comes with typed settings, an LLM factory, a FastAPI app, a test layout and distributed tracing with OpenTelemetry. The boring parts are decided so you can focus on what the agent does.

**No lock-in.** Teia generates plain LangGraph, FastAPI and official protocol SDKs. There is no proprietary runtime, no custom protocol implementation and no telemetry format of our own. If you stop using Teia tomorrow, your project keeps working.

---

## Quickstart

A working A2A agent with MCP tools in four commands:

```bash
teia new my-agent && cd my-agent
teia add mcp fetch --command "uvx mcp-server-fetch"
teia expose a2a assistant
teia dev
```

Here is what happens:

1. `teia new` creates the project with a default agent called `assistant`.
2. `teia add mcp` registers an MCP server in `servers.yaml`. Its tools are loaded automatically and handed to your agents; no code changes needed.
3. `teia expose a2a` generates the agent card and the A2A executor for `assistant`.
4. `teia dev` starts the API and the A2A endpoint in a single process and prints the URLs of the API, each agent card and the trace viewer.

The only manual step is adding your model provider's API key to `.env`. That's configuration, not code.

---

## Growing your project

Once the project exists, Teia keeps working with it.

```bash
# Add a second agent
teia add agent researcher

# Add a node to an existing agent's graph
teia add node researcher summarize --after retrieve
```

Each node lives in its own file under `graphs/<agent>/nodes/` and is registered automatically with the `@node` decorator. The graph's edges stay explicit in `graph.py`, which belongs to you. Teia reads it (without modifying it) to validate names and prints the exact edge lines to add. This keeps conditional edges, cycles and subgraphs working exactly as they do in plain LangGraph.

---

## Commands

| Command | What it does |
|---|---|
| `teia new <project>` | Create a project with the standard structure and a default `assistant` agent |
| `teia add agent <name>` | Add a new agent (graph) to the project |
| `teia add node <agent> <name>` | Add a node to an existing agent |
| `teia add mcp <server>` | Register an MCP server (`--command` for stdio, `--url` for HTTP) |
| `teia expose a2a <agent>` | Expose an agent over A2A (agent card + executor) |
| `teia dev` | Run the API and A2A endpoints locally and print their URLs |

---

## What you get

```
my-agent/
├── pyproject.toml
├── .env.example
├── AGENTS.md              # instructions for coding agents
├── docker-compose.yml     # Jaeger for local tracing
├── src/my_agent/
│   ├── config/            # typed settings (pydantic-settings)
│   ├── llm/               # model factory driven by config
│   ├── tools/             # local tools, discovered automatically
│   ├── mcp/               # MCP client + servers.yaml
│   ├── graphs/
│   │   └── assistant/     # state.py, graph.py, nodes/
│   ├── a2a/               # agent cards and executors
│   ├── observability/     # OpenTelemetry setup
│   └── api/               # FastAPI app
└── tests/
    ├── unit/
    └── evals/             # kept separate: slow and non-deterministic
```

Every generated project also ships with an `AGENTS.md`. A predictable structure plus clear instructions makes coding agents like Claude Code, Cursor or Codex far more effective on your project.

---

## Tracing built in

Agents are distributed systems: a single request can cross your graph, several MCP servers and other agents over A2A. Teia projects come with OpenTelemetry tracing already wired up, using existing instrumentations and the OpenTelemetry GenAI conventions.

```bash
docker compose up -d   # starts Jaeger locally
```

Set `OTEL_EXPORTER_OTLP_ENDPOINT` in `.env` and every request shows up as a trace, from the incoming call through your graph's nodes to each MCP tool call. Tracing stays off until you set the endpoint, and capturing prompts and responses is disabled by default.

Because it's standard OTLP, the same traces work with Grafana, Honeycomb, Datadog or any compatible backend.

---

## Design principles

- **Convention over editing.** Components are discovered automatically through decorators and folders, so generators only need to create files.
- **Official SDKs only.** Teia uses `a2a-sdk`, the official MCP SDK and `langchain-mcp-adapters`. It never reimplements a protocol.
- **Dependency injection at the edges.** Graphs receive their tools as parameters; the composition layer loads MCP servers and local tools and passes them in.
- **Standards over formats.** Observability is OpenTelemetry. Configuration is YAML and environment variables.
- **No managed runtime.** Teia projects are regular Python apps that deploy wherever you already deploy.

---

## Installation

Teia will be published on PyPI with its first release. Once available:

```bash
uv tool install teia
# or
pipx install teia
```

Requires Python 3.11+.

---

## Roadmap

**MVP**
- `new`, `add agent`, `add node`, `add mcp`, `expose a2a` and `dev`
- LangGraph as the agent framework
- Basic distributed tracing with OpenTelemetry
- The four-command quickstart running end to end in CI

**0.2**
- Calling remote agents over A2A, declared in YAML and exposed to your graph as tools
- A local dev page listing every exposed agent, with its agent card and a chat to talk to it
- Traces that span multiple agents

**Later**
- `teia add tool`
- Versioned prompts, evals and cost metrics
- Integrations with existing deployment platforms
- Support for a second agent framework

---

## Contributing

Teia is in its early days, and feedback is the most valuable contribution right now. If you build agents and have opinions about what a CLI like this should do, open an issue or start a discussion.

Development setup:

```bash
uv sync
uv run pytest
uv run ruff check . && uv run ruff format .
uv run mypy src
```

Every new command needs a test that generates a project in a temporary directory and validates the created files. See `AGENTS.md` for architecture rules and conventions.

---

## License

Teia is licensed under the [Apache License 2.0](LICENSE).

Code generated by Teia in your projects belongs to you and can be used under any license you choose.

---

Teia is maintained by [Byture](https://byture.com).
