# ShipScript

**An MCP server that documents and announces a repo for you.**

Point it at a GitHub URL. It reads the repository, works out the stack, screenshots
the live deployment, renders charts, and drafts a README, launch posts, slides and a
roadmap — then optionally opens a pull request with the README.

Exposed as [MCP](https://modelcontextprotocol.io) tools, so it plugs into any MCP
client (Claude Desktop, Claude Code, Cursor). A Gradio UI is included for running it
standalone.

---

## Why

The gap between "the project works" and "someone else can tell what it does" is
mostly mechanical: read the code, name the stack, take a screenshot, write it up.
That's automatable. ShipScript automates it and leaves the judgment calls — what to
emphasize, what to claim — to you in review.

## Tools

| Tool | What it does |
|---|---|
| `analyze_repo` | Fetches the repo, detects frameworks and stack, resolves the deployment URL, enriches with an LLM pass |
| `generate_readme` | Drafts a README from the enriched analysis |
| `capture_screenshot` | Screenshots the verified deployment (Playwright) |
| `generate_visuals` | Language donut, tech-stack chart, stats card as PNGs |
| `generate_social_content` | Posts for LinkedIn / Twitter / Dev.to |
| `generate_social_cards` | Per-platform card images |
| `generate_presentation` | Marp slide deck |
| `generate_roadmap` | `ROADMAP.md` |
| `create_pr` | Opens a PR adding the README |
| `package_output` | Bundles everything into a zip |

Most tools take the `context` object produced by `analyze_repo`, so call that first.

## Model setup

Generation goes through [litellm](https://github.com/BerriAI/litellm), so any
supported provider works — local Ollama, OpenAI, Anthropic, whatever — selected by
environment variable:

```bash
LLM_MODEL=ollama/qwen3-next:80b-cloud    # prose: README, roadmap, slides
JSON_MODEL=ollama/gemma4:31b-cloud       # structured output: analysis, social posts
```

Two models rather than one, deliberately: the prose model used for structured output
returned empty content on JSON calls, so the JSON path is pinned separately. The
response parser also handles reasoning models that put real output in
`reasoning_content` instead of `content`, and extracts JSON through several
fallback strategies rather than assuming clean output.

## Setup

```bash
git clone https://github.com/Zephyr4772/ShipScript.git
cd ShipScript
pip install -r requirements.txt
playwright install chromium        # needed for screenshots
```

Create `.env`:

```bash
LLM_MODEL=ollama/qwen3-next:80b-cloud
JSON_MODEL=ollama/gemma4:31b-cloud
GITHUB_TOKEN=                      # optional: raises API limits, enables create_pr
```

**Run the UI:**

```bash
python ui/app.py                   # http://localhost:7860
```

**Run as an MCP server** — add to your client's MCP config:

```json
{
  "mcpServers": {
    "shipscript": {
      "command": "python",
      "args": ["/absolute/path/to/ShipScript/src/server.py"]
    }
  }
}
```

**Smoke test:**

```bash
python test_golden_path.py
```

## Layout

```
src/
  server.py         FastMCP server — tool definitions
  github_tools.py   repo fetch, stack detection, deployment resolution
  llm_tools.py      litellm wrapper, JSON extraction, prose generation
  visual_tools.py   Playwright screenshots, matplotlib/plotly charts
ui/
  app.py            Gradio interface
test_golden_path.py
```

## Limitations

- **Review the output.** It drafts from static analysis plus an LLM pass; it can
  describe intent the code doesn't implement. That failure mode is exactly what a
  reviewer is for — don't merge a generated README unread.
- **Screenshots need a reachable public URL.** Local or auth-gated deployments won't capture.
- **`create_pr` needs a PAT with `repo` scope**, passed per call rather than stored.
- **Large repos** get truncated before the LLM pass, so analysis skews toward what
  the detector surfaces first.
- **No tests beyond the golden-path script.**

## License

Not yet licensed — all rights reserved. Open an issue if you'd like to reuse it.
