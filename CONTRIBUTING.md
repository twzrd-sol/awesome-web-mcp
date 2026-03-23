# Contributing to Awesome Web MCP

Thank you for helping build the most comprehensive curated list of web-accessible MCP servers and tools.

## How to Contribute

### Adding a New Entry

1. **Fork** this repository
2. **Add your entry** to the appropriate category in `README.md`
3. **Submit a Pull Request** with a clear title (e.g., `Add [Project Name] to [Category]`)

### Entry Format

Follow the existing table format:

```markdown
| [Project Name](https://project-url.com) | One-line description of what it does. Be specific. | Transport type | Auth method | License |
```

**Example:**
```markdown
| [Acme PDF](https://acme.dev) | Server-side PDF generation with template support, watermarking, and digital signatures | Streamable HTTP | API Key | MIT |
```

### Descriptions

- **Be specific.** "PDF tool" is too vague. "PDF merge, split, compress, and watermark with batch processing" is useful.
- **Lead with functionality**, not marketing language.
- **State the transport** — stdio, SSE, Streamable HTTP, or HTTP.
- **One sentence maximum.** If you need more, your scope might be too broad for a single entry.

---

## Quality Standards

Every entry must meet **all** of these criteria:

### Required

- [ ] **Public repository or documented API** — closed-source is fine, but documentation must be publicly accessible
- [ ] **Working implementation** — not a roadmap, not "coming soon," not a proof-of-concept
- [ ] **MCP compatibility** — implements the Model Context Protocol (tools, resources, or prompts)
- [ ] **Documentation** — README with installation instructions, usage examples, and at least one tool definition example
- [ ] **Active maintenance** — at least one commit in the last 6 months, or declared stable with no known issues
- [ ] **No critical vulnerabilities** — no exposed credentials, no SQL injection in tool handlers, no arbitrary code execution in resource endpoints

### Preferred (Not Required)

- Web transport support (Streamable HTTP or SSE)
- Authentication mechanism documented
- Test suite present
- Published to a package registry (npm, PyPI, crates.io)
- CI/CD pipeline visible

### Disqualifying

The following will result in immediate PR rejection:

- Abandoned projects (no commits in 12+ months, open critical issues unaddressed)
- Wrappers that add no value over the underlying API
- Projects with `TODO` as the only documentation
- Servers that expose raw shell access without sandboxing
- Duplicate entries (check existing list first)
- Self-promotional entries without genuine MCP integration

---

## Categories

Place your entry in the most specific applicable category:

| Category | What belongs here |
|----------|-------------------|
| Document & File Processing | PDF, Word, Excel, file conversion, document parsing |
| Image & Media Processing | Image manipulation, compression, format conversion, AI vision |
| Video & Audio | Transcription, TTS, video editing, audio processing |
| AI & Machine Learning | Model inference, embeddings, training, ML platforms |
| Browser & Web Automation | Scraping, headless browsers, testing, screenshots |
| Search & Information Retrieval | Search APIs, knowledge bases, indexing |
| Code & Developer Tools | Git, issue tracking, code analysis, package management |
| Database & Storage | SQL, NoSQL, vector databases, object storage |
| Communication & Collaboration | Email, chat, notifications, project management |
| Cloud Infrastructure & DevOps | Cloud providers, CI/CD, containers, IaC |
| Security & Authentication | Secrets management, vulnerability scanning, identity |
| Analytics & Monitoring | Metrics, logging, tracing, product analytics |
| Frameworks & SDKs | Libraries for building MCP servers and clients |
| Clients & Hosts | Applications that consume MCP servers |
| Registries & Discovery | Platforms for finding MCP servers |

If your project doesn't fit any category, propose a new one in your PR description.

---

## Pull Request Guidelines

### PR Title Format

```
Add [Project Name] to [Category]
```

Examples:
- `Add DocuSign MCP to Document & File Processing`
- `Add Datadog MCP to Analytics & Monitoring`
- `Fix broken link for Playwright MCP`

### PR Description

Include:

1. **What the project does** (1-2 sentences)
2. **Why it belongs on this list** (what makes it notable or production-grade)
3. **Your relationship to the project** (maintainer, user, or discovered it)
4. **Link to MCP integration** (if not obvious from the main repo)

### What Happens After You Submit

1. A maintainer reviews your PR (usually within 48 hours)
2. We verify the project meets quality standards
3. We may request changes to the description or categorization
4. Once approved, your entry is merged and live

---

## Updating Existing Entries

If a project has changed significantly (new transport support, renamed, deprecated), submit a PR with:

- Updated description reflecting current state
- Updated links (if repo moved)
- `[DEPRECATED]` tag if the project is no longer maintained (we'll remove it after 3 months)

---

## Reporting Issues

Found a broken link, outdated entry, or security concern with a listed project?

1. Open an issue with the `[Report]` prefix
2. Include the project name and specific problem
3. If it's a security issue with a listed project, contact the project maintainers directly — we are a directory, not a security response team

---

## Code of Conduct

- Be respectful in PR reviews and issue discussions
- Do not submit entries for projects you know to be malicious
- Do not remove competitor entries without justification
- Constructive criticism of listed projects is welcome; personal attacks are not

---

## License

By contributing, you agree that your contributions are released under [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/).
