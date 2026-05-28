# GitGet

A Linux desktop GitHub client built around six workflow modes — triage, investigation case management, pull-request review, org admin, repo browsing, and cross-repo search — with first-class support for live webhook delivery via self-hosted receiver + Cloudflare Tunnel.

Built on PySide6. Dark purple theme. Embedded PTY terminal. System tray with unread-count badge.

> Status: **0.1.0** — first release. See [CHANGELOG.md](CHANGELOG.md) for the full feature list.

---

## Why GitGet

Most desktop GitHub clients optimize for one workflow (PRs, or issues, or release notes). GitGet is built around the way investigative teams actually use GitHub — Discussions as case threads, Issues as evidence items, fast cross-repo triage, and live event delivery so you never miss a comment.

It is also a real, daily-driver GitHub client: PR review with inline comments, file browser with markdown/code/image/hex preview, org admin (members, secrets, Actions runs, webhooks), saved searches, and a built-in terminal so you don't have to leave the window.

---

## Features

### Workflow modes
- **Triage** — notification inbox with per-reason / per-type filters, mark-read, unsubscribe, live updates via webhooks.
- **Investigation** — GitHub Discussions as case threads with linked Issues as evidence items; markdown editor with live preview.
- **Pull Requests** — list, view files + colored unified diff, submit reviews (Approve / Request changes / Comment), inline comments on diff lines, squash / merge / rebase from the GUI.
- **Admin** — org/repo dashboard: members, secrets (libsodium sealed-box encryption), Actions runs, webhooks.
- **Contents** — lazy-loaded file tree with markdown / code / image / hex previews; empty-repo aware.
- **Search** — cross-repo search across issues, PRs, repositories, code, and users with saved-search persistence.

### Infrastructure
- **Webhook receiver** (`gitget-receiver`) — FastAPI service with HMAC-verified webhook ingestion, SQLite event log, async fan-out via WebSocket. Bundled systemd unit + Caddyfile + env template for self-hosted deployments.
- **Cloudflare Tunnel** — built-in supervisor for `cloudflared` (named or quick-tunnel modes); auto-restart and URL emission via Qt signal.
- **WebSocket bridge** — desktop client replays missed events on reconnect.

### UI
- PySide6, dark purple theme, custom QSS, app icon (SVG + 8 PNG sizes).
- Embedded PTY terminal (vim / htop / ssh-capable) via `pty.fork()` + pyte.
- Dockable terminal panel (Ctrl + `), View menu, About dialog.
- System tray icon with unread-count badge and quick actions.
- Markdown editor with toggleable live preview.

### Auth & state
- OAuth (web flow + PKCE; device flow); GitHub App JWT + installation tokens.
- libsecret keyring storage; tokens never written to disk in plaintext.
- Per-loop httpx clients so the polling engine and worker threads can share a `RestClient` without binding to a single asyncio loop.

---

## Install

### Pre-built packages

Download from the [Releases](../../releases) page:

| Format | File |
|---|---|
| Debian / Ubuntu | `gitget_0.1.0_amd64.deb` |
| AppImage | `gitget-0.1.0-x86_64.AppImage` |
| Flatpak | `org.obsidianwatch.GitGet.yaml` (build with `flatpak-builder`) |

```bash
sudo dpkg -i gitget_0.1.0_amd64.deb
# or
chmod +x gitget-0.1.0-x86_64.AppImage && ./gitget-0.1.0-x86_64.AppImage
```

### From source

Requires Python 3.12+ and [uv](https://github.com/astral-sh/uv).

```bash
git clone <this-repo>
cd GitGet_pub
uv sync
uv run gitget
```

---

## First-run setup

1. Launch GitGet — the login dialog appears.
2. Choose **OAuth** (default, simplest) or **GitHub App** (for org-level installation tokens).
3. Approve the auth flow in your browser; tokens are stored in libsecret.
4. Pick repos to track from the repo picker (you can change this anytime in Settings).

For live webhook delivery (optional), see [deploy/README.md](deploy/README.md) — covers the receiver service, Cloudflare Tunnel, and Caddy reverse-proxy setup.

---

## Project layout

```
src/gitget/
├── app.py              # QApplication bootstrap
├── __main__.py         # CLI entry
├── config.py           # Settings, paths, keyring service names
├── models.py           # Pydantic models for GitHub entities
├── auth/               # OAuth + GitHub App JWT + token storage
├── api/                # REST + GraphQL clients, rate limit, cache, services
├── polling/            # Background polling engine
├── receiver/           # Self-hosted webhook receiver (FastAPI)
├── tunnel/             # cloudflared supervisor
├── bridge/             # WebSocket client + reconnect replay
└── ui/                 # PySide6 views
    ├── main_window.py
    ├── login.py
    ├── settings.py
    ├── repo_picker.py
    ├── terminal.py     # Embedded PTY + pyte
    └── modes/
        ├── triage.py
        ├── investigation.py
        ├── pulls.py
        ├── admin.py
        ├── contents.py
        └── search.py
```

---

## Development

```bash
uv sync                          # install deps
uv run pytest                    # run tests (49 passing)
uv run ruff check                # lint
uv run ruff format               # format
uv run gitget                    # launch GUI
```

### Building packages

```bash
bash packaging/build-deb.sh      # → gitget_0.1.0_amd64.deb
bash packaging/build-appimage.sh # → gitget-0.1.0-x86_64.AppImage
```

---

## License

GPL-3.0. See [LICENSE](LICENSE) if present; otherwise the code is provided under the terms of the GNU General Public License v3.0.

---

© Obsidian Watch
