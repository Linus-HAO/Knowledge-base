# Knowledge-base
Linus
# LLM Wiki

A pattern for building personal knowledge bases using LLMs.

This is an idea file, it is designed to be copy pasted to your own LLM Agent (e.g. OpenAI Codex, Claude Code, OpenCode / Pi, or etc.). Its goal is to communicate the high level idea, but your agent will build out the specifics in collaboration with you.

## The core idea

Most people's experience with LLMs and documents looks like RAG: you upload a collection of files, the LLM retrieves relevant chunks at query time, and generates an answer. This works, but the LLM is rediscovering knowledge from scratch on every question. There's no accumulation. Ask a subtle question that requires synthesizing five documents, and the LLM has to find and piece together the relevant fragments every time. Nothing is built up. NotebookLM, ChatGPT file uploads, and most RAG systems work this way.

The idea here is different. Instead of just retrieving from raw documents at query time, the LLM **incrementally builds and maintains a persistent wiki** — a structured, interlinked collection of markdown files that sits between you and the raw sources. When you add a new source, the LLM doesn't just index it for later retrieval. It reads it, extracts the key information, and integrates it into the existing wiki — updating entity pages, revising topic summaries, noting where new data contradicts old claims, strengthening or challenging the evolving synthesis. The knowledge is compiled once and then *kept current*, not re-derived on every query.

This is the key difference: **the wiki is a persistent, compounding artifact.** The cross-references are already there. The contradictions have already been flagged. The synthesis already reflects everything you've read. The wiki keeps getting richer with every source you add and every question you ask.

You never (or rarely) write the wiki yourself — the LLM writes and maintains all of it. You're in charge of sourcing, exploration, and asking the right questions. The LLM does all the grunt work — the summarizing, cross-referencing, filing, and bookkeeping that makes a knowledge base actually useful over time. In practice, I have the LLM agent open on one side and Obsidian open on the other. The LLM makes edits based on our conversation, and I browse the results in real time — following links, checking the graph view, reading the updated pages. Obsidian is the IDE; the LLM is the programmer; the wiki is the codebase.

This can apply to a lot of different contexts. A few examples:

- **Personal**: tracking your own goals, health, psychology, self-improvement — filing journal entries, articles, podcast notes, and building up a structured picture of yourself over time.
- **Research**: going deep on a topic over weeks or months — reading papers, articles, reports, and incrementally building a comprehensive wiki with an evolving thesis.
- **Reading a book**: filing each chapter as you go, building out pages for characters, themes, plot threads, and how they connect. By the end you have a rich companion wiki. Think of fan wikis like [Tolkien Gateway](https://tolkiengateway.net/wiki/Main_Page) — thousands of interlinked pages covering characters, places, events, languages, built by a community of volunteers over years. You could build something like that personally as you read, with the LLM doing all the cross-referencing and maintenance.
- **Business/team**: an internal wiki maintained by LLMs, fed by Slack threads, meeting transcripts, project documents, customer calls. Possibly with humans in the loop reviewing updates. The wiki stays current because the LLM does the maintenance that no one on the team wants to do.
- **Competitive analysis, due diligence, trip planning, course notes, hobby deep-dives** — anything where you're accumulating knowledge over time and want it organized rather than scattered.

## Architecture

There are three layers:

**Raw sources** — your curated collection of source documents. Articles, papers, images, data files. These are immutable — the LLM reads from them but never modifies them. This is your source of truth.

**The wiki** — a directory of LLM-generated markdown files. Summaries, entity pages, concept pages, comparisons, an overview, a synthesis. The LLM owns this layer entirely. It creates pages, updates them when new sources arrive, maintains cross-references, and keeps everything consistent. You read it; the LLM writes it.

**The schema** — a document (e.g. CLAUDE.md for Claude Code or AGENTS.md for Codex) that tells the LLM how the wiki is structured, what the conventions are, and what workflows to follow when ingesting sources, answering questions, or maintaining the wiki. This is the key configuration file — it's what makes the LLM a disciplined wiki maintainer rather than a generic chatbot. You and the LLM co-evolve this over time as you figure out what works for your domain.

## Operations

**Ingest.** You drop a new source into the raw collection and tell the LLM to process it. An example flow: the LLM reads the source, discusses key takeaways with you, writes a summary page in the wiki, updates the index, updates relevant entity and concept pages across the wiki, and appends an entry to the log. A single source might touch 10-15 wiki pages. Personally I prefer to ingest sources one at a time and stay involved — I read the summaries, check the updates, and guide the LLM on what to emphasize. But you could also batch-ingest many sources at once with less supervision. It's up to you to develop the workflow that fits your style and document it in the schema for future sessions.

**Query.** You ask questions against the wiki. The LLM searches for relevant pages, reads them, and synthesizes an answer with citations. Answers can take different forms depending on the question — a markdown page, a comparison table, a slide deck (Marp), a chart (matplotlib), a canvas. The important insight: **good answers can be filed back into the wiki as new pages.** A comparison you asked for, an analysis, a connection you discovered — these are valuable and shouldn't disappear into chat history. This way your explorations compound in the knowledge base just like ingested sources do.

**Lint.** Periodically, ask the LLM to health-check the wiki. Look for: contradictions between pages, stale claims that newer sources have superseded, orphan pages with no inbound links, important concepts mentioned but lacking their own page, missing cross-references, data gaps that could be filled with a web search. The LLM is good at suggesting new questions to investigate and new sources to look for. This keeps the wiki healthy as it grows.

## Indexing and logging

Two special files help the LLM (and you) navigate the wiki as it grows. They serve different purposes:

**index.md** is content-oriented. It's a catalog of everything in the wiki — each page listed with a link, a one-line summary, and optionally metadata like date or source count. Organized by category (entities, concepts, sources, etc.). The LLM updates it on every ingest. When answering a query, the LLM reads the index first to find relevant pages, then drills into them. This works surprisingly well at moderate scale (~100 sources, ~hundreds of pages) and avoids the need for embedding-based RAG infrastructure.

**log.md** is chronological. It's an append-only record of what happened and when — ingests, queries, lint passes. A useful tip: if each entry starts with a consistent prefix (e.g. `## [2026-04-02] ingest | Article Title`), the log becomes parseable with simple unix tools — `grep "^## \[" log.md | tail -5` gives you the last 5 entries. The log gives you a timeline of the wiki's evolution and helps the LLM understand what's been done recently.

## Optional: CLI tools

At some point you may want to build small tools that help the LLM operate on the wiki more efficiently. A search engine over the wiki pages is the most obvious one — at small scale the index file is enough, but as the wiki grows you want proper search. [qmd](https://github.com/tobi/qmd) is a good option: it's a local search engine for markdown files with hybrid BM25/vector search and LLM re-ranking, all on-device. It has both a CLI (so the LLM can shell out to it) and an MCP server (so the LLM can use it as a native tool). You could also build something simpler yourself — the LLM can help you vibe-code a naive search script as the need arises.

## Tips and tricks

- **Obsidian Web Clipper** is a browser extension that converts web articles to markdown. Very useful for quickly getting sources into your raw collection.
- **Download images locally.** In Obsidian Settings → Files and links, set "Attachment folder path" to a fixed directory (e.g. `raw/assets/`). Then in Settings → Hotkeys, search for "Download" to find "Download attachments for current file" and bind it to a hotkey (e.g. Ctrl+Shift+D). After clipping an article, hit the hotkey and all images get downloaded to local disk. This is optional but useful — it lets the LLM view and reference images directly instead of relying on URLs that may break. Note that LLMs can't natively read markdown with inline images in one pass — the workaround is to have the LLM read the text first, then view some or all of the referenced images separately to gain additional context. It's a bit clunky but works well enough.
- **Obsidian's graph view** is the best way to see the shape of your wiki — what's connected to what, which pages are hubs, which are orphans.
- **Marp** is a markdown-based slide deck format. Obsidian has a plugin for it. Useful for generating presentations directly from wiki content.
- **Dataview** is an Obsidian plugin that runs queries over page frontmatter. If your LLM adds YAML frontmatter to wiki pages (tags, dates, source counts), Dataview can generate dynamic tables and lists.
- The wiki is just a git repo of markdown files. You get version history, branching, and collaboration for free.

## Why this works

The tedious part of maintaining a knowledge base is not the reading or the thinking — it's the bookkeeping. Updating cross-references, keeping summaries current, noting when new data contradicts old claims, maintaining consistency across dozens of pages. Humans abandon wikis because the maintenance burden grows faster than the value. LLMs don't get bored, don't forget to update a cross-reference, and can touch 15 files in one pass. The wiki stays maintained because the cost of maintenance is near zero.

The human's job is to curate sources, direct the analysis, ask good questions, and think about what it all means. The LLM's job is everything else.

The idea is related in spirit to Vannevar Bush's Memex (1945) — a personal, curated knowledge store with associative trails between documents. Bush's vision was closer to this than to what the web became: private, actively curated, with the connections between documents as valuable as the documents themselves. The part he couldn't solve was who does the maintenance. The LLM handles that.


## Memory Rules (Linus's Customizations)

These rules evolved over ~800 source ingestions across two vaults and are recorded here as part of the methodology. Their canonical form lives in `/Users/hpc/.claude/shared-memory/` and is symlinked into every vault.

### Digestion Rule (linus-digestion-rule.md)

1. **Pre-ingestion scan**: Page count, OCR/text-extractability, duplicates — sample multiple pages, not just page 1
1a. **DOI-based PDF retrieval — two scenarios**: 
  - **"Digest"**: full pipeline (PDF → source-markdown → wiki update via index/log/cross-reference)
  - **"Download/get this paper"**: download PDF to `sources/download/New-YYYYMMDD/` → convert to source-markdown → stop (no wiki update)
  - Method: `curl -sL "sci-hub.su/{DOI}"` → extract `/storage/...pdf` → `curl -L -o "file.pdf"` with UA+Referer → validate `%PDF-` → PyPDF2 conversion. Mirror fallback: `.su` → `.ru` → `.st` → `.box` → `.red` → `.al` → `.mk` → `.ee`. Check `sci-hub.pub` for latest active mirrors if all fail. NEVER ask user to manually save — do it automatically.
2. **Page thresholds**: <100p → auto-digest. ≥100p → sparse extraction (first 8 + last 5 + every 25th page) for searchability; flag in log for user decision on full digestion
3. **Date-stamped download folder**: When downloading, create `New-YYYYMMDD/` under `Source-download/` for that day's batch
4. **Markdown conversion**: PyPDF2 preferred over pymupdf (pymupdf fails on PDFs PyPDF2 reads correctly). Sequential numbering (001, 002, ...) by creation time
5. **Quality check + cross-verification**: After conversion, scan for garbled content (<30% recognizable lines). **Cross-verify with PyPDF2** — pymupdf can silently fail on CJK/vertical-text PDFs. Spot-read at least 3 files per batch to confirm real regulatory content was extracted. Flag any fake-digestion files immediately.
6. **Wiki creation**: Every ingest MUST update `wiki/index.md`, `wiki/log.md`, `wiki/synthesis.md`, entity pages, concept pages. NEVER skip infrastructure updates. A single ingest touches 3-10+ pages.
7. **Literature-gap update**: After every ingest, mark newly found items as "Present" with source-markdown numbers. Only update the current vault's gap file — never another vault.
8. **Literature search pipeline**: When gaps found, attempt download first (in-memory, PDF never touches disk). If blocked (403, CAPTCHA), provide URLs for manual browser download. For Japanese `.go.jp` sites, use `curl` instead of Python (PMDA rejects Python's TLS).

### Paper Writing Rule (linus-rule-of-paper.md)

1. **References numbered by first appearance**: Reference numbers must be assigned in the order they first appear in the main text. A reference cited in §1 must have a lower number than a reference first cited in §3. When new references are added to earlier sections, ALL references must be renumbered. Vancouver/AMA citation style.

2. **Superscript citations**: Use `<sup>N</sup>` format. Multiple citations: `<sup>1,3,5</sup>`.

3. **Reference list**: Numbered under `## 参考文献`. ACS citation style. Verified refs marked with `[**Verified** — [[source-markdown NNN]]]`.

4. **Renumbering**: After any edit that adds/removes refs, run the Python renumbering script by first-appearance order. Never attempt manual renumbering.

5. **Word export — one-shot with superscript**: Combine pandoc + python-docx in a single Bash `&&` chain. Skip the `## 参考文献` section (detect by Heading style) to avoid superscripting volume numbers and years. Only match 1-2 digit numbers (refs 1-41), not 4-digit years. Settings.json must auto-approve: `Bash(pandoc *)`, `Bash(python3 *superscript*)`, `Bash(python3 *docx*)`.

### Shared Memory Across All Vaults

All vaults (current and future) share memory from `/Users/hpc/.claude/shared-memory/`:
- `user-profile.md` — Identity, background, collaboration preferences
- `linus-digestion-rule.md` — Standard ingest workflow (8 steps)
- `new-source-workflow.md` — Pre-ingestion scanning
- `sci-hub-download-workflow.md` — Paywalled paper access
- `federated-search-sync.md` — Cross-vault search index maintenance
- `CLAUDE-shared.md` — Cross-vault conventions, workflows, templates
- `Karpathy llm-wiki methodology.md` — This file — the canonical methodology shared to every vault

When a new vault is created, symlink all files from shared-memory/ into the vault's memory directory. Edit shared files ONLY at the shared source — never in project-specific directories.

### Shared Plugins

Obsidian plugins that are installed in one vault SHALL be shared across ALL vaults. The mechanism:

1. **Shared plugin store:** `/Users/hpc/.claude/shared-plugins/` — each plugin gets a subdirectory with its built files (main.js, manifest.json, styles.css, etc.)
2. **Per-vault copy (NOT symlink):** Each vault's `.obsidian/plugins/<plugin-id>/` contains a copy of files from shared-plugins/ → `/Users/hpc/.claude/shared-plugins/<plugin-id>/`
3. **Enable in community-plugins.json:** Add the plugin ID to each vault's `.obsidian/community-plugins.json` array
4. **Install once, copy everywhere:** When a new plugin is installed in any vault, move it to shared-plugins/, copy back, then copy into all other vaults and update their community-plugins.json

Currently installed shared plugins:
- `llm-wiki-audit` — LLM Wiki Audit plugin (from lewislulu/llm-wiki-skill): select text, file anchored feedback to `audit/`

### Federated Search Index

`/Users/hpc/Obsidian/source-markdown-shared/` is a federated search index combining all source-markdown from both vaults (~1,700+ files). After every ingest, run `sync-index.py` to rebuild. Open as an Obsidian vault to search all vaults simultaneously. Files prefixed by vault (CR_ = Chemical Regulation, CG_ = Change Regulation).

### Literature Gap Analysis

Each vault maintains a `wiki/literature-gap.md` (or similarly named) file tracking missing sources. After every ingest, update the gap file in the CURRENT vault — mark newly ingested "Missing" items as "Present" with source-markdown file numbers. The Change Regulation vault's gap items are marked with `CG-` prefix in Chemical Regulation's gap analysis, and vice versa, to enable cross-vault identification of where missing sources reside.

### New Vault Setup Convention

When the user asks to create a new wiki vault, automatically set up the complete structure:

1. **Create the vault directory structure:**
   ```
   sources/               # Raw immutable source documents
   source-markdown/        # Converted markdown versions
   audit/                  # Human feedback inbox (from Obsidian plugin)
     resolved/             # Processed feedback, archived with resolution notes
   scripts/                # Health check and audit review tools
     lint_wiki.py          # 7-pass wiki health check
     audit_review.py       # Group and list audits by target file
   wiki/
     index.md              # Full catalog by category
     log.md                # Append-only operations log
     synthesis.md          # Evolving synthesis
     summaries/            # One summary per source
     entities/             # Entity pages
     concepts/             # Concept pages
     comparisons/          # Comparison pages
     literature-gap.md     # Gap analysis for missing sources (MANDATORY)
   CLAUDE.md               # Vault-specific schema and conventions
   ```

2. **Set up shared memory:** Symlink all files from `/Users/hpc/.claude/shared-memory/` into the vault's `.claude/projects/<hash>/memory/` directory.

3. **Set up shared plugins:** Copy each plugin from `/Users/hpc/.claude/shared-plugins/<plugin-id>/` into the vault's `.obsidian/plugins/<plugin-id>/`. Add enabled plugin IDs to `.obsidian/community-plugins.json`. Currently the required plugin is `llm-wiki-audit` (LLM Wiki Audit — select text, file anchored feedback to `audit/`).

4. **Symlink Karpathy methodology:** In the vault root, create a symlink from `Karpathy llm-wiki methodology.md` → `/Users/hpc/.claude/shared-memory/Karpathy llm-wiki methodology.md`. This file IS the canonical methodology — it lives in shared memory and is symlinked into every vault's root, just like the memory files.

5. **Register in federated search:** Add the new vault's `source-markdown/` to the sync script at `/Users/hpc/Obsidian/source-markdown-shared/sync-index.py`.

6. **Create literature-gap.md:** Every vault SHALL have a `wiki/literature-gap.md` file tracking missing sources relevant to that vault's domain. Populate it with known gaps based on the vault's domain scope.

7. **Adapt CLAUDE.md:** Write vault-specific domain description, entity categories, and concept categories. Reference `CLAUDE-shared.md` for shared workflows and conventions. Include a cross-reference to the shared Karpathy methodology file.

8. **Post-setup recheck:** After every new vault setup — and periodically whenever re-entering an existing vault — run through this checklist to catch missing infrastructure:

   | # | Check | What to verify |
   |---|-------|-----------------|
   | 1 | **MEMORY.md exists** | Project memory directory has MEMORY.md index linking all shared-memory files |
   | 2 | **Shared-memory symlinks** | All files from `/Users/hpc/.claude/shared-memory/` are symlinked into the vault's memory directory |
   | 3 | **Vault root symlink** | `Karpathy llm-wiki methodology.md` symlink exists in vault root → shared-memory |
   | 4 | **source-markdown/ dir** | Exists and is writable |
   | 5 | **Wiki subdirs** | `wiki/` has `index.md`, `log.md`, `synthesis.md`, `summaries/`, `entities/`, `concepts/`, `comparisons/`, `literature-gap.md` |
   | 6 | **CLAUDE.md** | Vault root has CLAUDE.md with domain-specific schema |
   | 7 | **literature-gap.md** | Exists in `wiki/` and is populated with known gaps |
   | 8 | **sync-index.py registration** | The vault is listed in `/Users/hpc/Obsidian/source-markdown-shared/sync-index.py` with correct prefix |
   | 9 | **Search index rebuilt** | `sync-index.py` has been run at least once and produced expected file counts |
   | 10 | **Shared plugins copied** | All plugins in `/Users/hpc/.claude/shared-plugins/` are copied into `.obsidian/plugins/` |
   | 11 | **community-plugins.json** | All shared plugin IDs are listed in `.obsidian/community-plugins.json` |
   | 12 | **audit/ directory** | `audit/` and `audit/resolved/` exist at vault root |
   | 13 | **scripts/ directory** | `scripts/lint_wiki.py` and `scripts/audit_review.py` exist at vault root |

   **When to trigger:** Immediately after new vault creation; on first session entry into any vault; whenever something looks off (e.g., "MEMORY.md not found" or missing shared-memory context at session start). The recheck is lightweight — it's just verification, not recreation.

## Note

This document is intentionally abstract. It describes the idea, not a specific implementation. The exact directory structure, the schema conventions, the page formats, the tooling — all of that will depend on your domain, your preferences, and your LLM of choice. Everything mentioned above is optional and modular — pick what's useful, ignore what isn't. For example: your sources might be text-only, so you don't need image handling at all. Your wiki might be small enough that the index file is all you need, no search engine required. You might not care about slide decks and just want markdown pages. You might want a completely different set of output formats. The right way to use this is to share it with your LLM agent and work together to instantiate a version that fits your needs. The document's only job is to communicate the pattern. Your LLM can figure out the rest.
