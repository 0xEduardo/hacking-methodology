# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Obsidian-based knowledge base documenting security testing methodologies, tools, and techniques for penetration testing. All content is written in Markdown with Obsidian-style internal links (`[[link]]` and `[text](path.md)`). There is no executable code, build system, or package manager — it is purely documentation.

The content is open and freely shareable, compiled from articles, books, videos, and talks.

## Architecture

### Original Content (`infra/` and `web/`)

The repository's own content follows a two-domain structure, each with a consistent **recon → exploitation → helpers** pipeline:

- **`infra/`** — Infrastructure and network-level security (host discovery, service enumeration, privilege escalation)
- **`web/`** — Web application security (subdomain enumeration, vulnerability exploitation, tool usage)

Within `web/exploitation/`, content is further categorized by attack surface:
- `authentication/` — JWT, SAML, OAuth, 2FA bypass
- `bypass/` — 403 and WAF bypass techniques
- `cloud/` — AWS, Azure, GCP specific attacks
- `cms/` — WordPress and other CMS exploits
- `vulns/` — Individual vulnerability classes (XSS, SQLi, SSRF, etc. — ~30 files)

`README.md` serves as the main navigation index linking to all pages.

### HackTricks Reference (`hacktricks/`)

A local clone of the official [HackTricks](https://github.com/HackTricks-wiki/hacktricks) knowledge base (~950 markdown files, ~140 MB of images). It is **git-ignored** and used only as offline reference material — not part of the project's own content.

Key content areas inside `hacktricks/src/`:
- `pentesting-web/` — Web vulnerability guides (75+ topics)
- `network-services-pentesting/` — Port-specific service exploitation (97+ topics)
- `binary-exploitation/` — ROP, heap, stack overflow, format strings
- `generic-methodologies-and-resources/` — General pentesting methodology, forensics, phishing
- `windows-hardening/`, `linux-hardening/`, `macos-hardening/` — OS hardening
- `crypto/`, `stego/`, `reversing/`, `mobile-pentesting/`, `blockchain/` — Specialized domains

HackTricks uses mdBook as its build system (configured via `book.toml`), which is separate from this project's Obsidian-based approach.

## Conventions

- All documentation files use `.md` extension
- Files are named with lowercase kebab-case (e.g., `host-header-injection.md`, `linux-privesc.md`)
- Each file documents a specific topic with tool commands, payloads, and methodology notes
- Internal cross-references use relative Markdown links (`[text](relative/path.md)`)
- The project is designed to be opened in Obsidian for graph view and backlink navigation

## Working with This Repository

- When adding new content, place it in the appropriate domain (`infra/` or `web/`) and phase (`recon/`, `exploitation/`, or `helpers/`)
- Update `README.md` when adding new pages to maintain the navigation index
- Keep filenames consistent with existing kebab-case convention
- Content language is English
- The `hacktricks/` directory is git-ignored — do not commit changes to it or reference it in `README.md`
