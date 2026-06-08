# Awesome Web MCP [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> A curated list of production-grade MCP (Model Context Protocol) servers, clients, frameworks, and resources for web-based tool integration.

The **Model Context Protocol** is an open standard developed by [Anthropic](https://anthropic.com) that enables AI models to interact with external tools, data sources, and services through a unified interface. **Web MCP** refers to MCP implementations accessible over network transports (HTTP+SSE, Streamable HTTP) rather than local stdio — enabling browser-native, cloud-hosted, and multi-tenant deployments.

This list focuses on **web-accessible** MCP implementations: servers exposed over HTTP, browser-compatible clients, cloud-hosted registries, and the infrastructure required to operate MCP at scale.

**Why this list exists:** MCP is fragmenting fast. Hundreds of servers exist, but most lack authentication, use stdio-only transport, and were never designed for production web deployment. This list filters for quality — servers that handle auth, support web transports, and solve real problems.

---

## Contents

- [Protocol Fundamentals](#protocol-fundamentals)
- [Transport Mechanisms](#transport-mechanisms)
- [Document & File Processing](#document--file-processing)
- [Image & Media Processing](#image--media-processing)
- [Video & Audio](#video--audio)
- [AI & Machine Learning](#ai--machine-learning)
- [Browser & Web Automation](#browser--web-automation)
- [Search & Information Retrieval](#search--information-retrieval)
- [Code & Developer Tools](#code--developer-tools)
- [Database & Storage](#database--storage)
- [Communication & Collaboration](#communication--collaboration)
- [Cloud Infrastructure & DevOps](#cloud-infrastructure--devops)
- [Security & Authentication](#security--authentication)
- [Analytics & Monitoring](#analytics--monitoring)
- [Frameworks & SDKs](#frameworks--sdks)
- [Clients & Hosts](#clients--hosts)
- [Registries & Discovery](#registries--discovery)
- [Architecture Patterns](#architecture-patterns)
- [Security Considerations](#security-considerations)
- [Performance & Scaling](#performance--scaling)
- [Specification & Standards](#specification--standards)
- [Learning Resources](#learning-resources)
- [Contributing](#contributing)

---

## Protocol Fundamentals

The MCP architecture follows a client-host-server pattern:

```
┌─────────────────────────────────────────────┐
│  Host (Claude Desktop, IDE, Web App)        │
│                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │ Client A │  │ Client B │  │ Client C │  │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  │
│       │              │              │        │
└───────┼──────────────┼──────────────┼────────┘
        │              │              │
   ┌────▼─────┐  ┌─────▼────┐  ┌─────▼────┐
   │ Server A │  │ Server B │  │ Server C │
   │ (Local)  │  │ (Remote) │  │ (Remote) │
   └──────────┘  └──────────┘  └──────────┘
```

**Core primitives:**

| Primitive | Direction | Description |
|-----------|-----------|-------------|
| **Tools** | Server → Client | Executable functions the model can invoke (e.g., `convert_pdf`, `resize_image`) |
| **Resources** | Server → Client | Read-only data the model can access (e.g., file contents, database records) |
| **Prompts** | Server → Client | Reusable prompt templates with parameters |
| **Sampling** | Client → Server | Allows servers to request LLM completions through the client |
| **Roots** | Client → Server | Filesystem boundaries the client exposes to the server |

**Capability negotiation** occurs during the `initialize` handshake — both client and server declare supported features, preventing runtime failures from unsupported operations.

---

## Transport Mechanisms

MCP defines three transport layers. Web deployments require HTTP-based transports:

### stdio (Local Only)
- Process-level communication via stdin/stdout
- Zero network overhead, lowest latency (~μs)
- Single-tenant by design — one client per server process
- **Not suitable for web deployment**

### HTTP + Server-Sent Events (SSE)
- Client → Server: HTTP POST requests
- Server → Client: SSE stream for async notifications
- Requires persistent connection for the SSE channel
- Session management via unique endpoint URIs
- **Legacy transport** — supported but being superseded

### Streamable HTTP (Current Standard)
- Single HTTP endpoint handles all communication
- Optional SSE upgrade for streaming responses
- Stateless by default — sessions opt-in via `Mcp-Session-Id` header
- DNS-compatible, CDN-cacheable, load-balancer friendly
- **Recommended for all new web deployments**

```
Streamable HTTP Flow:

Client                          Server
  │                               │
  │── POST /mcp (initialize) ──►  │
  │◄── 200 + Mcp-Session-Id ────  │
  │                               │
  │── POST /mcp (tools/list) ──►  │
  │◄── 200 (JSON-RPC response) ─  │
  │                               │
  │── POST /mcp (tools/call) ──►  │
  │◄── 200 text/event-stream ───  │  ← SSE upgrade for streaming
  │    data: {progress: 50%}      │
  │    data: {result: ...}        │
  │                               │
```

**Key difference from REST:** MCP uses JSON-RPC 2.0 over HTTP, not resource-oriented URLs. All requests hit a single endpoint. The protocol handles routing internally via method names (`tools/call`, `resources/read`, etc.).

---

## Document & File Processing

Production-grade MCP servers for document manipulation, conversion, and analysis.

| Server | Description | Transport | Auth | License |
|--------|-------------|-----------|------|---------|
| [**MiOffice**](https://mioffice.ai) | 85+ file processing tools (PDF merge/split/compress, Word/Excel conversion, image processing, video tools, AI features). 100% client-side WASM processing — files never leave the browser. MCP-enabled for AI agent integration. | Streamable HTTP | API Key | Proprietary |
| [Cloudflare Document AI](https://developers.cloudflare.com/workers-ai/) | OCR, document parsing, and text extraction via Workers AI | HTTP | Bearer Token | Proprietary |
| [Marker](https://github.com/VikParuchuri/marker) | PDF/EPUB/MOBI to Markdown conversion with high accuracy. Table and equation support. | stdio | None | GPL-3.0 |
| [Pandoc MCP](https://github.com/vivekVells/mcp-pandoc) | Universal document converter wrapping Pandoc. Supports 40+ formats. | stdio | None | MIT |
| [Docling](https://github.com/DS4SD/docling) | IBM Research document understanding — layout analysis, table structure recognition, PDF parsing with ML | stdio | None | MIT |
| [Unstructured](https://github.com/Unstructured-IO/unstructured) | Pre-processing pipeline for unstructured documents (PDF, DOCX, PPTX, HTML, images) | HTTP | API Key | Apache-2.0 |
| [pdf-tools-mcp](https://github.com/alexchenzl/pdf-tools-mcp) | PDF text extraction, page splitting, metadata reading | stdio | None | MIT |
| [Google Drive MCP](https://github.com/anthropics/mcp-servers) | Read/search/export Google Drive documents | stdio | OAuth 2.0 | MIT |

### Selection Criteria

When evaluating document processing MCP servers for production:

- **Privacy model**: Does the server upload files to a remote API, or process locally/client-side? For regulated industries (HIPAA, GDPR), client-side processing is mandatory.
- **Format coverage**: Most servers handle PDF. Few handle Office formats (DOCX, XLSX, PPTX) well. Verify edge cases: password-protected files, large files (100MB+), CJK text encoding.
- **Streaming support**: Large file processing should stream progress events, not block for minutes with no feedback.

---

## Image & Media Processing

| Server | Description | Transport | Auth | License |
|--------|-------------|-----------|------|---------|
| [**MiOffice**](https://mioffice.ai) | Image compress, resize, rotate, format conversion, HEIC→JPG, AI background removal, AI upscaling. WASM-powered, fully client-side. | Streamable HTTP | API Key | Proprietary |
| [Sharp MCP](https://github.com/nicholasgriffintn/mcp-sharp) | High-performance Node.js image processing via libvips — resize, crop, rotate, format conversion | stdio | None | MIT |
| [Cloudinary MCP](https://cloudinary.com/documentation/mcp_server) | Cloud image/video management — upload, transform, optimize, deliver via CDN | HTTP | API Key + Secret | Proprietary |
| [Replicate MCP](https://github.com/deepfates/mcp-replicate) | Run any ML model on Replicate (Stable Diffusion, SDXL, ControlNet, etc.) | HTTP | API Token | MIT |
| [Remove.bg MCP](https://www.remove.bg/api) | AI background removal via dedicated API | HTTP | API Key | Proprietary |
| [TinyPNG MCP](https://tinypng.com/developers) | Lossy PNG/JPEG/WebP compression with excellent visual quality preservation | HTTP | API Key | Proprietary |

---

## Video & Audio

| Server | Description | Transport | Auth | License |
|--------|-------------|-----------|------|---------|
| [FFmpeg MCP](https://github.com/mcp-sh/mcp-ffmpeg) | Video/audio processing via FFmpeg — transcode, trim, merge, extract audio, generate thumbnails | stdio | None | MIT |
| [Whisper MCP](https://github.com/openai/whisper) | Speech-to-text transcription supporting 99 languages | stdio / HTTP | API Key (cloud) | MIT |
| [ElevenLabs MCP](https://elevenlabs.io/docs/api-reference) | Neural text-to-speech with voice cloning | HTTP | API Key | Proprietary |
| [AssemblyAI MCP](https://www.assemblyai.com/docs) | Audio intelligence — transcription, speaker diarization, sentiment analysis, topic detection | HTTP | API Key | Proprietary |
| [Deepgram MCP](https://developers.deepgram.com/) | Real-time and pre-recorded speech recognition with WebSocket streaming | HTTP / WebSocket | API Key | Proprietary |

---

## AI & Machine Learning

| Server | Description | Transport | Auth | License |
|--------|-------------|-----------|------|---------|
| [OpenAI MCP](https://github.com/anthropics/mcp-servers) | GPT-4, DALL-E, embeddings, and assistants via MCP interface | HTTP | API Key | MIT |
| [HuggingFace MCP](https://huggingface.co/docs/mcp) | Access 400K+ models — inference, dataset search, space management | HTTP | Bearer Token | Apache-2.0 |
| [Ollama MCP](https://github.com/ollama/ollama) | Local LLM inference (Llama, Mistral, Phi, Gemma) with OpenAI-compatible API | HTTP | None (local) | MIT |
| [LangChain MCP](https://github.com/langchain-ai/langchain) | Chain MCP tools into multi-step agent workflows | HTTP | Varies | MIT |
| [Weights & Biases MCP](https://wandb.ai/site) | Experiment tracking, model registry, dataset versioning | HTTP | API Key | Proprietary |

---

## Browser & Web Automation

| Server | Description | Transport | Auth | License |
|--------|-------------|-----------|------|---------|
| [Playwright MCP](https://github.com/anthropics/mcp-servers/tree/main/src/playwright) | Browser automation — navigation, screenshots, form filling, scraping. Chromium/Firefox/WebKit. | stdio | None | Apache-2.0 |
| [Puppeteer MCP](https://github.com/anthropics/mcp-servers/tree/main/src/puppeteer) | Chromium automation with CDP (Chrome DevTools Protocol) integration | stdio | None | MIT |
| [Browserbase MCP](https://www.browserbase.com/) | Cloud browser infrastructure — headless browsers with anti-detection, proxies, CAPTCHA solving | HTTP | API Key | Proprietary |
| [Firecrawl MCP](https://github.com/mendableai/firecrawl) | Web scraping with LLM-ready output — handles JavaScript rendering, pagination, rate limiting | HTTP | API Key | AGPL-3.0 |
| [Crawl4AI](https://github.com/unclecode/crawl4ai) | LLM-friendly web crawler — async, respects robots.txt, outputs clean Markdown | stdio | None | Apache-2.0 |
| [Stagehand](https://github.com/browserbase/stagehand) | AI-native browser automation using natural language instructions | HTTP | API Key | MIT |

---

## Search & Information Retrieval

| Server | Description | Transport | Auth | License |
|--------|-------------|-----------|------|---------|
| [Brave Search MCP](https://github.com/anthropics/mcp-servers/tree/main/src/brave-search) | Web search via Brave Search API — privacy-focused, no tracking | HTTP | API Key | MIT |
| [Exa MCP](https://exa.ai/) | Neural search — semantic search over the web with embedding-based retrieval | HTTP | API Key | Proprietary |
| [Tavily MCP](https://tavily.com/) | Search API optimized for AI agents — returns structured, LLM-ready results | HTTP | API Key | Proprietary |
| [SearXNG](https://github.com/searxng/searxng) | Self-hosted metasearch engine aggregating 70+ search engines. Free, private, no API keys needed. | HTTP | None (self-hosted) | AGPL-3.0 |
| [Google Search MCP](https://github.com/anthropics/mcp-servers/tree/main/src/google-search) | Google Custom Search JSON API wrapper | HTTP | API Key | MIT |
| [Wikipedia MCP](https://github.com/anthropics/mcp-servers) | Wikipedia article search, summaries, and full content retrieval | HTTP | None | MIT |

---

## Code & Developer Tools

| Server | Description | Transport | Auth | License |
|--------|-------------|-----------|------|---------|
| [GitHub MCP](https://github.com/anthropics/mcp-servers/tree/main/src/github) | Repository management — issues, PRs, code search, file operations, Actions | HTTP | PAT / GitHub App | MIT |
| [GitLab MCP](https://gitlab.com/) | GitLab API integration — merge requests, pipelines, container registry | HTTP | PAT / OAuth | MIT |
| [Linear MCP](https://github.com/anthropics/mcp-servers/tree/main/src/linear) | Issue tracking — create, update, search issues and projects | HTTP | API Key | MIT |
| [Sentry MCP](https://github.com/getsentry/sentry-mcp) | Error monitoring — query issues, view stack traces, manage releases | HTTP | Auth Token | Apache-2.0 |
| [Sourcegraph MCP](https://sourcegraph.com/) | Code intelligence — cross-repo search, code navigation, batch changes | HTTP | Access Token | Apache-2.0 |
| [npm MCP](https://www.npmjs.com/) | Package registry search, version info, dependency analysis | HTTP | None (public) | N/A |

---

## Database & Storage

| Server | Description | Transport | Auth | License |
|--------|-------------|-----------|------|---------|
| [PostgreSQL MCP](https://github.com/anthropics/mcp-servers/tree/main/src/postgres) | Read-only SQL queries against PostgreSQL databases with schema inspection | stdio | Connection string | MIT |
| [SQLite MCP](https://github.com/anthropics/mcp-servers/tree/main/src/sqlite) | Local SQLite database operations — query, create tables, analyze schemas | stdio | None (local) | MIT |
| [Supabase MCP](https://supabase.com/docs/guides/ai/mcp) | Postgres + Auth + Storage + Realtime — full backend via MCP | HTTP | Service Key | Apache-2.0 |
| [MongoDB MCP](https://github.com/mongodb/mongodb-mcp-server) | Document database operations — CRUD, aggregation pipelines, index management | stdio / HTTP | Connection string | Apache-2.0 |
| [Redis MCP](https://redis.io/) | In-memory data structure store — caching, pub/sub, streams | stdio | Password | BSD-3 |
| [Pinecone MCP](https://www.pinecone.io/) | Vector database for semantic search and RAG applications | HTTP | API Key | Proprietary |
| [Qdrant MCP](https://qdrant.tech/) | Open-source vector similarity search engine with filtering | HTTP / gRPC | API Key (optional) | Apache-2.0 |
| [Neon MCP](https://neon.tech/docs/ai/mcp) | Serverless Postgres — branching, autoscaling, connection pooling | HTTP | API Key | Apache-2.0 |

---

## Communication & Collaboration

| Server | Description | Transport | Auth | License |
|--------|-------------|-----------|------|---------|
| [Slack MCP](https://github.com/anthropics/mcp-servers/tree/main/src/slack) | Channel messaging, user lookup, thread management | HTTP | Bot Token | MIT |
| [Discord MCP](https://discord.com/developers/docs) | Server management, messaging, moderation via Discord API | HTTP | Bot Token | Proprietary |
| [Notion MCP](https://github.com/anthropics/mcp-servers/tree/main/src/notion) | Pages, databases, blocks — full Notion workspace manipulation | HTTP | Integration Token | MIT |
| [Gmail MCP](https://github.com/anthropics/mcp-servers) | Email send/read/search with Gmail API | HTTP | OAuth 2.0 | MIT |
| [Resend MCP](https://resend.com/) | Transactional email API — send, track, manage domains | HTTP | API Key | Proprietary |

---

## Cloud Infrastructure & DevOps

| Server | Description | Transport | Auth | License |
|--------|-------------|-----------|------|---------|
| [AWS MCP](https://github.com/aws/aws-mcp-servers) | S3, Lambda, DynamoDB, CloudFormation — AWS service operations | HTTP | IAM Credentials | Apache-2.0 |
| [Cloudflare MCP](https://developers.cloudflare.com/mcp/) | Workers, KV, R2, D1, DNS management | HTTP | API Token | BSD-3 |
| [Vercel MCP](https://vercel.com/) | Deployment management, environment variables, domain configuration | HTTP | Bearer Token | Proprietary |
| [Docker MCP](https://github.com/docker/mcp-server) | Container lifecycle — build, run, inspect, logs, compose operations | stdio | Docker socket | Apache-2.0 |
| [Kubernetes MCP](https://github.com/strowk/mcp-k8s) | Cluster management — pods, deployments, services, logs | stdio / HTTP | Kubeconfig | Apache-2.0 |
| [Terraform MCP](https://www.terraform.io/) | Infrastructure as Code — plan, apply, state management | stdio | Provider-specific | MPL-2.0 |

---

## Security & Authentication

| Server | Description | Transport | Auth | License |
|--------|-------------|-----------|------|---------|
| [Snyk MCP](https://snyk.io/) | Vulnerability scanning for dependencies, containers, and IaC | HTTP | API Token | Proprietary |
| [Vault MCP](https://www.vaultproject.io/) | Secrets management — dynamic credentials, encryption as a service | HTTP | Vault Token | MPL-2.0 |
| [1Password MCP](https://developer.1password.com/) | Secrets retrieval from 1Password vaults via Connect Server | HTTP | Service Account | Proprietary |
| [TWZRD Agent Intel](https://intel.twzrd.xyz) | Solana on-chain trust scoring for AI agents. Verify wallet reputation before x402 micropayments | Streamable HTTP | None (free tools) | Proprietary |

---

## Analytics & Monitoring

| Server | Description | Transport | Auth | License |
|--------|-------------|-----------|------|---------|
| [Datadog MCP](https://www.datadoghq.com/) | Metrics, traces, logs — full observability stack | HTTP | API + App Key | Proprietary |
| [PostHog MCP](https://posthog.com/) | Product analytics — events, funnels, feature flags, session replay | HTTP | API Key | MIT |
| [Grafana MCP](https://grafana.com/) | Dashboard queries, alerting, data source management | HTTP | API Key / Service Account | AGPL-3.0 |
| [Umami MCP](https://umami.is/) | Privacy-focused web analytics — pageviews, events, custom data | HTTP | API Key | MIT |

---

## Frameworks & SDKs

Tools for building MCP servers and clients.

| Framework | Language | Description | Transport Support |
|-----------|----------|-------------|-------------------|
| [**@modelcontextprotocol/sdk**](https://github.com/modelcontextprotocol/typescript-sdk) | TypeScript | Official reference SDK. Full protocol implementation. | stdio, SSE, Streamable HTTP |
| [**mcp-python-sdk**](https://github.com/modelcontextprotocol/python-sdk) | Python | Official Python SDK with async support (anyio). | stdio, SSE, Streamable HTTP |
| [**mcp-go**](https://github.com/mark3labs/mcp-go) | Go | Community Go SDK with middleware support. | stdio, SSE, Streamable HTTP |
| [**mcp-rs**](https://github.com/modelcontextprotocol/rust-sdk) | Rust | Official Rust SDK — async with Tokio runtime. | stdio, SSE, Streamable HTTP |
| [**mcp-kotlin**](https://github.com/modelcontextprotocol/kotlin-sdk) | Kotlin/JVM | Official Kotlin SDK for JVM-based servers. | stdio, SSE, Streamable HTTP |
| [**mcp-csharp**](https://github.com/modelcontextprotocol/csharp-sdk) | C# | Official .NET SDK. | stdio, SSE, Streamable HTTP |
| [FastMCP](https://github.com/jlowin/fastmcp) | Python | High-level Pythonic API — decorator-based tool definitions, automatic schema generation. | stdio, SSE, Streamable HTTP |
| [Foxy Contexts](https://github.com/strowk/foxy-contexts) | Rust | Declarative MCP server framework with built-in testing harness. | stdio, SSE |

### Building a Web MCP Server (Minimal Example)

A Streamable HTTP server requires handling JSON-RPC 2.0 over a single POST endpoint:

```
POST /mcp HTTP/1.1
Content-Type: application/json

{"jsonrpc": "2.0", "method": "tools/list", "id": 1}
```

Response:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "tools": [
      {
        "name": "convert_pdf",
        "description": "Convert PDF to another format",
        "inputSchema": {
          "type": "object",
          "properties": {
            "format": {"type": "string", "enum": ["docx", "png", "txt"]},
            "url": {"type": "string", "format": "uri"}
          },
          "required": ["format", "url"]
        }
      }
    ]
  }
}
```

**Critical implementation details:**
- Always validate `Content-Type: application/json` on incoming requests
- Return `405 Method Not Allowed` for non-POST requests (except optional GET for SSE)
- Implement `Mcp-Session-Id` header for stateful sessions
- Set appropriate CORS headers for browser clients

---

## Clients & Hosts

Applications that consume MCP servers.

| Client | Type | Description | Web Support |
|--------|------|-------------|-------------|
| [Claude Desktop](https://claude.ai/download) | Desktop App | Anthropic's desktop client with native MCP support | stdio only |
| [Claude Code](https://github.com/anthropics/claude-code) | CLI | Terminal-based coding agent with MCP integration | stdio + Streamable HTTP |
| [Cursor](https://cursor.sh/) | IDE | AI code editor with MCP tool support | stdio |
| [Windsurf](https://codeium.com/windsurf) | IDE | AI-native code editor by Codeium | stdio |
| [Zed](https://zed.dev/) | IDE | High-performance editor with MCP context servers | stdio |
| [Continue](https://continue.dev/) | IDE Extension | Open-source AI coding assistant (VS Code, JetBrains) | stdio + SSE |
| [Cline](https://github.com/cline/cline) | VS Code Extension | Autonomous coding agent with MCP support | stdio |
| [LibreChat](https://github.com/danny-avila/LibreChat) | Web App | Open-source ChatGPT clone with MCP integration | SSE + Streamable HTTP |
| [Open WebUI](https://github.com/open-webui/open-webui) | Web App | Self-hosted LLM frontend with MCP tool calling | SSE |
| [mcp-client-cli](https://github.com/chrishayuk/mcp-client-cli) | CLI | Lightweight CLI for testing MCP servers | stdio + SSE |

---

## Registries & Discovery

Platforms for finding and sharing MCP servers.

| Registry | Description | URL |
|----------|-------------|-----|
| [MCP Hub](https://mcphub.voidai.wtf) | Curated MCP server registry with reviews and security audits | [mcphub.voidai.wtf](https://mcphub.voidai.wtf) |
| [Smithery](https://smithery.ai/) | MCP server marketplace with one-click installation | [smithery.ai](https://smithery.ai) |
| [Glama](https://glama.ai/mcp/servers) | MCP server directory with categorization and search | [glama.ai](https://glama.ai/mcp/servers) |
| [mcp.run](https://mcp.run) | WebAssembly-based MCP server hosting and discovery | [mcp.run](https://mcp.run) |
| [PulseMCP](https://pulsemcp.com/) | MCP server directory with trending and activity tracking | [pulsemcp.com](https://pulsemcp.com) |
| [MCP Market](https://mcpmarket.com/) | Community-driven MCP server listing | [mcpmarket.com](https://mcpmarket.com) |

---

## Architecture Patterns

### Gateway Pattern

Route multiple MCP servers through a single HTTP endpoint. Handles auth, rate limiting, and routing centrally.

```
                    ┌──────────────────┐
                    │   MCP Gateway    │
                    │  (Auth + Route)  │
Client ──HTTPS──►  │                  │
                    │  /mcp?server=A   │──► Server A (PDF tools)
                    │  /mcp?server=B   │──► Server B (Image tools)
                    │  /mcp?server=C   │──► Server C (Search)
                    └──────────────────┘
```

**When to use:** Multi-tenant SaaS, enterprise deployments, API marketplaces.

### Sidecar Pattern

Deploy MCP server alongside your application in the same container/pod. Server accesses application internals directly.

```
┌─────────────────────────┐
│ Pod / Container          │
│                          │
│  ┌──────────┐ ┌────────┐│
│  │   App    │◄│  MCP   ││──► External clients
│  │ (DB,API) │ │ Server ││
│  └──────────┘ └────────┘│
│         shared memory    │
└─────────────────────────┘
```

**When to use:** Database access, internal API exposure, application-specific tools.

### Edge Pattern

Deploy MCP servers at the network edge (Cloudflare Workers, Deno Deploy, Vercel Edge) for sub-50ms latency globally.

```
Client (Tokyo) ──► Edge Node (Tokyo) ──► MCP Server (V8 isolate)
                                              │
Client (NYC)   ──► Edge Node (NYC)   ──► MCP Server (V8 isolate)
                                              │
                                    Shared KV / D1 / R2
```

**When to use:** Latency-sensitive tools, global user bases, lightweight transformations.

### Federated Pattern

Multiple independent MCP servers register with a discovery service. Clients query the registry, then connect directly.

```
                 ┌──────────┐
            ┌───►│ Registry │◄───┐
            │    └──────────┘    │
         register            register
            │                    │
     ┌──────┴──────┐     ┌──────┴──────┐
     │  Server A   │     │  Server B   │
     │  (Org 1)    │     │  (Org 2)    │
     └──────▲──────┘     └──────▲──────┘
            │                    │
            └──── Client ───────┘
              (direct connect)
```

**When to use:** Decentralized ecosystems, cross-organization tool sharing, marketplace models.

---

## Security Considerations

> **Critical finding:** An audit of 2,000+ public MCP servers found that **100% lacked authentication**. Most stdio servers were never designed for network exposure. Before deploying any MCP server over HTTP, implement the following.

### Authentication

MCP itself does not define an auth mechanism. Web deployments must implement their own:

| Method | Use Case | Considerations |
|--------|----------|----------------|
| **OAuth 2.0 + PKCE** | Multi-tenant, user-delegated access | Recommended by MCP spec. Requires auth server infrastructure. |
| **API Keys** | Server-to-server, developer tools | Simple but lacks granularity. Rotate regularly. |
| **mTLS** | Zero-trust, service mesh | Certificate management overhead. Strongest server identity guarantee. |
| **JWT Bearer** | Stateless, microservices | Validate `iss`, `aud`, `exp`. Use asymmetric signing (RS256/ES256). |

### Transport Security

- **Always use HTTPS** in production. MCP over plain HTTP exposes tool calls, parameters, and results to network observers.
- **Validate the `Origin` header** for browser-based clients to prevent CSRF.
- **Set CORS headers explicitly** — never use `Access-Control-Allow-Origin: *` with credentialed requests.
- **Rate limit per-session** — MCP's persistent connection model makes per-IP rate limiting insufficient.

### Input Validation

MCP tool inputs are defined by JSON Schema, but schemas are **advisory, not enforced** by the protocol. Servers must:

- Validate all `inputSchema` constraints server-side (never trust the client)
- Sanitize file paths to prevent directory traversal (`../../../etc/passwd`)
- Bound resource consumption (file sizes, query complexity, timeout limits)
- Treat all tool arguments as untrusted user input — the model may relay prompt injection attacks through tool calls

### Prompt Injection via Tools

MCP creates a new attack surface: **tool results can contain adversarial content** that influences the model's next actions. Defenses:

- Mark tool outputs with clear boundaries in the prompt
- Implement output sanitization for tool results returned to the model
- Use allowlists for sensitive operations (file writes, API calls, shell commands)
- Log all tool invocations for audit trails

---

## Performance & Scaling

### Latency Budgets

Typical MCP tool call latency breakdown:

```
Total: ~200-500ms (simple tool)

  Network RTT:        10-50ms   (varies by region)
  TLS handshake:      20-40ms   (first request only, amortized with keep-alive)
  JSON-RPC parsing:    1-2ms    (negligible)
  Tool execution:    100-400ms  (depends on tool complexity)
  Response serial.:    1-5ms    (negligible)
```

### Optimization Strategies

1. **Connection reuse**: HTTP/2 multiplexing eliminates head-of-line blocking across concurrent tool calls
2. **Tool batching**: Some clients support sending multiple `tools/call` requests in a single HTTP POST (JSON-RPC batch)
3. **Streaming responses**: Use SSE upgrade for tools that produce incremental output (file processing, search results)
4. **Edge caching**: Cache `tools/list` and `resources/list` responses — tool schemas rarely change
5. **Warm pools**: Pre-initialize heavy tool dependencies (database connections, ML models) at server startup

### Scaling Patterns

| Pattern | Throughput | Complexity | Best For |
|---------|-----------|------------|----------|
| Single process | ~100 req/s | Low | Development, small teams |
| Horizontal (load balancer) | ~10K req/s | Medium | SaaS products |
| Serverless (Workers/Lambda) | ~100K req/s | Medium | Spiky traffic, global distribution |
| Event-driven (queue + workers) | ~1M+ req/s | High | High-throughput pipelines |

---

## Specification & Standards

| Resource | Description |
|----------|-------------|
| [MCP Specification](https://spec.modelcontextprotocol.io/) | Full protocol specification (JSON-RPC 2.0 based) |
| [MCP GitHub](https://github.com/modelcontextprotocol) | Protocol organization — SDKs, reference implementations, spec |
| [JSON-RPC 2.0](https://www.jsonrpc.org/specification) | Underlying wire protocol used by MCP |
| [Server-Sent Events](https://html.spec.whatwg.org/multipage/server-sent-events.html) | W3C spec for SSE (used in streaming transport) |
| [MCP Authorization Spec](https://spec.modelcontextprotocol.io/specification/2025-03-26/basic/authorization/) | OAuth 2.0 + PKCE authorization flow for MCP |

---

## Learning Resources

### Official Documentation

- [MCP Introduction](https://modelcontextprotocol.io/introduction) — Concept overview and getting started
- [MCP Quickstart](https://modelcontextprotocol.io/quickstart) — Build your first MCP server in 15 minutes
- [MCP TypeScript SDK Docs](https://github.com/modelcontextprotocol/typescript-sdk#readme) — TypeScript SDK reference
- [MCP Python SDK Docs](https://github.com/modelcontextprotocol/python-sdk#readme) — Python SDK reference
- [Anthropic MCP Guide](https://docs.anthropic.com/en/docs/agents-and-tools/mcp) — Using MCP with Claude

### Tutorials & Articles

- [Building MCP Servers with Cloudflare Workers](https://developers.cloudflare.com/agents/guides/remote-mcp-server/) — Deploy MCP to the edge
- [MCP Transport Deep Dive](https://spec.modelcontextprotocol.io/specification/2025-03-26/basic/transports/) — Transport layer internals
- [Building a Multi-Tool MCP Server](https://modelcontextprotocol.io/tutorials/building-mcp-with-llms) — Step-by-step server construction

### Videos

- [MCP Explained in 5 Minutes](https://www.youtube.com/results?search_query=model+context+protocol+explained) — Quick visual overview
- [Anthropic MCP Launch](https://www.youtube.com/results?search_query=anthropic+model+context+protocol) — Original announcement and demo

---

## Contributing

Contributions are welcome. Please read the [Contributing Guidelines](CONTRIBUTING.md) before submitting a pull request.

**Quality bar:** This list prioritizes production-grade implementations. We do not list toy projects, unmaintained repos, or servers with zero documentation.

### Quick Submission Checklist

- [ ] Project has a public repository or documented API
- [ ] README includes installation, usage, and at least one example
- [ ] Actively maintained (commit within last 6 months)
- [ ] Supports at least one web transport (HTTP+SSE or Streamable HTTP) OR is a foundational tool commonly used in web MCP deployments
- [ ] No known critical security vulnerabilities

---

## Maintainers

This list is maintained by the [MiOffice](https://mioffice.ai) team — builders of 85+ privacy-first file processing tools with MCP integration.

**Sponsored by [mioffice.ai](https://mioffice.ai)** — AI Office Suite. 85+ tools. 100% client-side. Your files never leave your browser.

---

## License

[![CC0](https://licensebuttons.net/p/zero/1.0/88x31.png)](https://creativecommons.org/publicdomain/zero/1.0/)

This list is released under [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/). You may copy, modify, and distribute this work, even for commercial purposes, without asking permission.
