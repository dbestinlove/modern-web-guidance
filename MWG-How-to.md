# Modern Web Guidance — How to Use This Repository

**Modern Web Guidance (MWG)** embeds web-platform expertise, best practices, and browser compatibility data directly into your AI coding agents. It steers agents away from legacy patterns learned in training data and toward native, high-performance modern web APIs.

This repository (`GoogleChrome/modern-web-guidance`) is the **publish/distribution target** for the skill packs and CLI. It is stewarded by the Google Chrome, Microsoft Edge, Mozilla Firefox, and MDN teams.

> [!NOTE]
> Contributions (guides, bug fixes, eval improvements) happen in the **source repository**: [GoogleChrome/modern-web-guidance-src](https://github.com/GoogleChrome/modern-web-guidance-src). See [`CONTRIBUTING.md`](CONTRIBUTING.md).

---

## 1. Repository Layout

```
modern-web-guidance/
├── README.md                      # Project overview, quickstart, install methods, evals
├── CONTRIBUTING.md                # Points to the -src repo for contributions
├── package.json                   # npm package: exposes the `modern-web-guidance` CLI bin
├── plugin.json                    # agent-plugins.org manifest (generic plugin installs)
├── gemini-extension.json          # Gemini CLI extension manifest
├── kimi.plugin.json               # Kimi Code plugin manifest (lists both skill packs)
├── policies/
│   └── modern-web-guidance.toml   # Shell-command approval policy (asks before running the CLI)
├── .claude-plugin/ .codex-plugin/ .cursor-plugin/ .grok-plugin/ .agents/
│                                  # Per-agent marketplace/plugin manifests
└── skills/
    ├── modern-web-guidance/       # Core skill pack (~205 tokens)
    │   ├── SKILL.md               # Agent instructions: when/how to search & retrieve guides
    │   └── guides/                # The guidance corpus, organized by discipline:
    │       ├── accessibility/ built-in-ai/ css/ forms/ html/ js/
    │       ├── performance/ privacy/ pwa/ security/ ui-atoms/
    │       ├── ui-behaviors/ ui-components/ user-experience/
    │       └── visual-design/ wasm/ webmcp/
    └── chrome-extensions/         # Optional skill pack (~175 tokens)
        ├── SKILL.md               # Manifest V3 + Chrome Web Store publishing guidance
        └── references/
```

**What's inside the guides:** ~262 modern web features across ~204 real-world developer use cases (see the full lists in [`README.md`](README.md)). Guides are token-efficient and validated with automated evals showing **+27 to +36 percentage-point uplift** in agent task pass rates.

---

## 2. Quickstart (Recommended)

Run the interactive installation wizard, which detects your coding agent(s) and wires up the skill:

```shell
npx modern-web-guidance@latest install
```

To pick only specific skill packs:

```shell
npx modern-web-guidance@latest install --choose
```

Available packs:
* **`modern-web-guidance`** — modern browser APIs, CSS layout, performance, accessibility, forms, built-in AI, etc.
* **`chrome-extensions`** — Manifest V3, background workers, extension APIs, Chrome Web Store publishing.

Later, update everything with:

```shell
npx modern-web-guidance@latest update
```

---

## 3. Try It Without Installing

The CLI works standalone, offline, with zero dependencies and no API keys:

```shell
# Search for relevant guides (returns JSON: [{ "id", "description" }, ...])
npx modern-web-guidance@latest search "animate a dialog modal backdrop"

# Retrieve a full guide by ID (multiple IDs: comma-separated)
npx modern-web-guidance@latest retrieve "animate-to-from-top-layer"

# Scan a codebase for legacy patterns to modernize
npx modern-web-guidance@latest legacy-scan .
npx modern-web-guidance@latest legacy-scan . --json   # machine-readable findings
```

Tips for good searches:
* Use a **2–5 word action-oriented query** describing a single capability (`"light dismiss dialog"`, `"responsive data table"`).
* Omit framework names (`React`, `Tailwind`) and app-specific nouns; split multi-feature tasks into separate searches.
* If results are weak or empty, rephrase once with a concrete HTML/CSS/UI component noun before falling back to `list`.

---

## 4. How It Works With Your Coding Agent

1. **Activation** — When you ask your agent to do a web UI/CSS/client-side JS task, it activates the `modern-web-guidance` skill (per `skills/modern-web-guidance/SKILL.md`).
2. **Local semantic search** — The agent runs `modern-web-guidance search "<query>"`, matched via an offline embedding model + BM25 (no network calls at query time; `npx` fetch keeps content fresh).
3. **Guide fetch** — The agent runs `retrieve <guide-id>`, injecting targeted code patterns, gotchas, and fallback strategies into its context window.
4. **Compliance & disclosure** — The agent cross-checks its output against the retrieved guide and ends with a `### Modern Web Guidance` section citing which guides shaped the code.

Because guides are vanilla HTML/CSS/JS, the skill instructs agents to adapt them to your framework without dropping the modern feature.

---

## 5. Alternative Installation Methods

| Agent / Tool | Command(s) |
| :--- | :--- |
| **Vercel Skills CLI** | `npx skills add GoogleChrome/modern-web-guidance` |
| **GitHub CLI** | `gh skill install GoogleChrome/modern-web-guidance` |
| **Google Antigravity** | `agy plugin install https://github.com/GoogleChrome/modern-web-guidance` |
| **GitHub Copilot CLI** | `/plugin marketplace add GoogleChrome/modern-web-guidance` then `/plugin install modern-web-guidance@googlechrome` |
| **Claude Code** | `/plugin marketplace add GoogleChrome/modern-web-guidance` → `/plugin install modern-web-guidance@googlechrome` → `/plugin` (enable AutoUpdate) → `/reload-plugins` |
| **Grok Build CLI** | `grok plugin install https://github.com/GoogleChrome/modern-web-guidance --trust` |
| **Kimi Code** | `/plugins install https://github.com/GoogleChrome/modern-web-guidance` |
| **OpenAI Codex** | `codex plugin marketplace add GoogleChrome/modern-web-guidance` then `codex plugin add modern-web-guidance@googlechrome` |

> [!WARNING]
> Claude Code installs from Anthropic's built-in `claude-plugins-official` marketplace are stranded at old versions. Uninstall from that marketplace and reinstall from the `googlechrome` marketplace (commands in [`README.md`](README.md), “Updating”).

---

## 6. Project Configuration (`.mwgrc`)

Opt in to experimental/volatile feature guides with an optional `.mwgrc` JSON file in your project root (or `~/.mwgrc` user-level):

| CLI Command | `.mwgrc` Key | Effect |
| :--- | :--- | :--- |
| `npx modern-web-guidance@latest set-allow-origin-trials` | `"allowOriginTrials": true` | Unlocks guides for experimental Origin Trial APIs. |
| `npx modern-web-guidance@latest set-allow-volatile-features` | `"allowVolatileFeatures": true` | Unlocks guides for volatile single-engine features. |

By default, guides for Origin Trial and volatile features remain discoverable via `search` but are gated on `retrieve` so agents don't adopt unstable APIs without your consent.

You can also document a **browser support policy** in `AGENTS.md` / `CLAUDE.md` (e.g., *"Never implement polyfills; use lightweight custom fallbacks"*). All guides otherwise assume **Baseline Widely available** features need no fallbacks.

---

## 7. Modernizing an Existing Codebase

Use `legacy-scan` only for broad, open-ended modernization requests (not for specific feature work):

1. **Scan**: `npx -y modern-web-guidance@latest legacy-scan <path> --json`
2. **Verify**: Check each finding against its `keepWhen` / `dropWhen` rubric; drop false positives (fallbacks, vendor code, lookalikes).
3. **Report**: Summarize verified findings (note: the scan covers ~20% of the catalog with deterministic legacy rules).
4. **Modernize**: `retrieve` each finding's guide, apply the replacement, and re-run `legacy-scan` to confirm.

---

## 8. Browsing Guides Directly

You can read any guide as plain Markdown without the CLI:

```
skills/modern-web-guidance/guides/<category>/<guide-id>.md
```

Example: [`skills/modern-web-guidance/guides/ui-components/navigation-drawer.md`](skills/modern-web-guidance/guides/ui-components/navigation-drawer.md). Each category directory (`css`, `forms`, `performance`, `accessibility`, `security`, `ui-components`, …) contains both broad discipline guides and narrow use-case guides.

---

## 9. Privacy & Telemetry

Google collects anonymous usage data (install counts, agent name, retrieved guide IDs, and CLI search queries). Raw user prompts are **not** collected. To opt out:

```bash
export DISABLE_TELEMETRY=1   # add to .bashrc / .zshrc
```

The CLI itself runs fully local and offline after `npx` fetches the package.

---

## 10. Troubleshooting

* **Command hangs (offline/sandboxed)**: retry with `npx --offline …`; in approval-gated environments (Codex, Claude Code), allowlist `npx -y modern-web-guidance@latest *` **with network access**. Prefer `pnpx` if pnpm is available; on Windows use `npx.cmd`.
* **Read-only `~/.npm`**: set `NPM_CONFIG_CACHE=/tmp/npm-cache`.
* **Large `retrieve` output (> 9.5 KB)**: it's saved to a temp file — read that file in full rather than using `cat`/`head`.
* **Stale content**: always invoke via `npx … @latest` (or run `update` after an `install`).

---

## 11. Contributing & Licensing

* **Contributions**: open PRs/issues in [GoogleChrome/modern-web-guidance-src](https://github.com/GoogleChrome/modern-web-guidance-src) — this repo is a publish target only.
* **Evals**: results and harness details on the [Eval Dashboard](https://googlechrome.github.io/modern-web-guidance-src/).
* **License**: code under Apache 2.0 ([LICENSE](LICENSE)); guide content under CC-BY 4.0. Portions derived from MDN, W3C, WHATWG, and IETF specs.

**Docs**: <https://developer.chrome.com/docs/modern-web-guidance>
