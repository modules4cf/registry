<p align="center"><img src="https://raw.githubusercontent.com/modules4cf/.github/main/assets/modules4cf-logo.png" alt="modules4cf" width="260"></p>

# modules4cf registry

**A self-hostable, ForgeBox-compatible CFML package registry, built by the community, for the community.**
*It will power [modules4cf.org](https://modules4cf.org), and anyone can run their own copy with Docker.*

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](#-get-involved)
[![Status: planning](https://img.shields.io/badge/status-planning-orange.svg)](https://github.com/modules4cf/registry/milestones)

> **Status:** this project is at the planning stage. There is no code yet. The work is broken down into [issues and milestones](https://github.com/modules4cf/registry/milestones), and every one of them is open for you to pick up.

---

## 👋 Vision

Today the modules of the [modules4cf](https://github.com/modules4cf) organization are published to [ForgeBox](https://www.forgebox.io). ForgeBox is a great service and will stay an important part of the CFML ecosystem.

In the long run we want an **open registry run by the community**:

* **A public registry at modules4cf.org.** Anyone can register a CFML project there, link it to its GitHub repository and have it served to every CommandBox user.
* **A drop-in endpoint for CommandBox.** A ColdBox or CommandBox user registers modules4cf.org as an endpoint once, then installs packages from it, either instead of ForgeBox or alongside it.
* **Run your own.** The whole registry ships as a Docker project. Clone it, run `docker compose up`, and you have your own registry for your company, your team or your community.
* **Open source, from the people, for the people.** Apache 2.0 licensed, developed in the open, governed by the people who use it.

## ✨ Planned features

* **ForgeBox API compatibility**, so the existing CommandBox CLI works without plugins or changes (see [the compatibility notes](docs/forgebox-api-compatibility.md)).
* **Install, show, search and publish** with the commands you already know: `box install`, `forgebox show`, `forgebox search`, `box publish`.
* **Link a GitHub project:** register a package from its `box.json` and GitHub releases, and keep it up to date automatically through a webhook when you publish a release.
* **Artifact storage** on the local disk or any S3-compatible storage, with checksums on every artifact.
* **Accounts with GitHub sign-in** and personal API tokens for publishing.
* **Optional upstream proxy/mirror to ForgeBox**, so one endpoint can serve both local packages and packages from ForgeBox.
* **A web UI** to browse, search and read package pages, including rendered readmes and changelogs.
* **An MCP server** ([Model Context Protocol](https://modelcontextprotocol.io)), so AI assistants and tools can search packages, read package details, versions and readmes, and get install instructions. Read-only at first.
* **Federation:** link projects hosted elsewhere and let registries reference each other.
* **Security and moderation:** package ownership rules, protection against name squatting, rate limiting and abuse reporting.

## 🚀 How you will use it

CommandBox (4.5 or later) already supports custom ForgeBox-based endpoints through the `forgebox endpoint` namespace ([docs](https://commandbox.ortusbooks.com/forgebox-enterprise/commands), [settings](https://commandbox.ortusbooks.com/config-settings/endpoint-settings)). Once the registry is live, using it will look like this:

```bash
# Register the registry once, under a name of your choice (here: "m4cf")
box forgebox endpoint register m4cf https://modules4cf.org/api/v1

# Install a package from it by prefixing the slug with the endpoint name
box install m4cf:orm4cf

# Or pin it in box.json
#   "dependencies": { "orm4cf": "m4cf:orm4cf@^5.2.0" }

# Optional: make it your default endpoint, so "box install orm4cf" uses it
box forgebox endpoint set-default m4cf

# See and manage your endpoints
box forgebox endpoint list
box forgebox endpoint remove m4cf
```

To publish, log in to the endpoint once and publish as usual:

```bash
box forgebox login endpointName=m4cf
box publish endpointName=m4cf
```

> **Note:** the API URL `https://modules4cf.org/api/v1` is the planned address. Today, modules4cf.org simply redirects to the GitHub organization. The `box.json` dependency form with the endpoint prefix is still to be verified against a running registry (see [open questions](#-open-questions)).

## 🏗️ Architecture proposal

This is a **proposal**. Everything here is up for discussion in the issues.

```
                  ┌───────────────────────────────────────────┐
 CommandBox  ───► │  registry app                              │
 Browser     ───► │   • ForgeBox-compatible REST API /api/v1   │──► database (packages, versions,
 AI tools    ───► │   • web UI                                 │     users, tokens, orgs)
 (MCP)            │   • MCP server (read-only)                 │
 GitHub      ───► │   • GitHub OAuth + release webhooks        │──► artifact storage
 (webhooks)       │   • optional ForgeBox proxy/mirror         │     (local disk or S3-compatible)
                  └───────────────────────────────────────────┘
```

* **Runtime:** we propose **CFML on Lucee**, to dogfood the ecosystem the registry serves and make it easy for CFML developers to contribute. This is an [open question](#-open-questions).
* **Database:** we propose PostgreSQL for production, with a simple option for small installs. To be decided in the data model issue.
* **Storage:** pluggable, local filesystem by default and any S3-compatible service (AWS S3, MinIO, Cloudflare R2, ...) for larger installs.
* **Packaging:** one Docker image, published to the GitHub Container Registry, plus a `docker-compose.yml` that brings up the app, the database and optionally MinIO.
* **Configuration:** environment variables only, so the same image works for modules4cf.org and for your own registry.

## 🐳 Self-hosting

Self-hosting is a first-class goal, not an afterthought. modules4cf.org will run exactly the image you can run yourself. The planned quick start:

```bash
git clone https://github.com/modules4cf/registry.git
cd registry
cp .env.example .env      # set your hostname, database and storage
docker compose up -d
```

Then point CommandBox at it with `box forgebox endpoint register myregistry https://registry.example.com/api/v1`.

## 🔁 Mirror and proxy to ForgeBox

As an option, a registry can act as a proxy for ForgeBox. Packages that aren't hosted locally are looked up upstream, and their metadata (and optionally their artifacts) can be cached. This lets a team use one single endpoint, and keep working when the upstream is slow or unreachable. Local packages always take precedence. The details, including respecting upstream terms of use, are tracked in the issues.

## 🔗 Federation and linking projects

You shouldn't have to re-upload what already lives on GitHub. A maintainer will be able to:

1. Sign in with GitHub.
2. Link a repository. The registry reads its `box.json` (slug, version, metadata) and its GitHub releases.
3. Install a webhook, so every new GitHub release shows up in the registry automatically.

Later, registries will be able to reference each other, so a community or company registry can link to packages on modules4cf.org and the other way round.

## 🏷️ Naming

Packages of the modules4cf organization keep the **`*4cf`** naming convention (for example `orm4cf`), as explained on the [organization page](https://github.com/modules4cf). The registry itself is open to any CFML project, under its own name, with clear ownership rules.

## 🙌 Get involved

**This registry will only be as good as the community behind it. We would love your help, whoever you are and whatever your experience.**

* **Pick an issue.** Look for [`good first issue`](https://github.com/modules4cf/registry/labels/good%20first%20issue) and [`help wanted`](https://github.com/modules4cf/registry/labels/help%20wanted). Leave a comment so others know you're on it.
* **Join the design discussions.** The architecture is still open. Opinions on runtime, database, API details and governance are very welcome in the issues.
* **Test with your CommandBox.** Once the first milestone is out, try installing and publishing, and tell us what breaks.
* **Write docs**, for users and for people running their own registry.
* **Spread the word** on the [CFML Slack](https://cfml.slack.com) and the [Lucee dev forum](https://dev.lucee.org).

Read [CONTRIBUTING.md](CONTRIBUTING.md) before you open your first pull request.

## 🗺️ Roadmap

| Milestone | Goal |
|---|---|
| [M1 MVP read-only registry](https://github.com/modules4cf/registry/milestone/1) | CommandBox can `install`, `show` and `search` packages from a self-hosted registry started with Docker |
| [M2 publishing](https://github.com/modules4cf/registry/milestone/2) | Accounts, API tokens, `box publish`, linking GitHub projects |
| [M3 web UI and community](https://github.com/modules4cf/registry/milestone/3) | Web UI, MCP server, moderation, security, modules4cf.org goes live |
| [M4 federation and mirror](https://github.com/modules4cf/registry/milestone/4) | ForgeBox proxy/mirror, federation between registries |

## ❓ Open questions

* Which runtime do we build on? CFML on Lucee is the proposal.
* Which database do we support first, and do we support a lightweight single-container mode?
* Which parts of the ForgeBox API are actually needed beyond what CommandBox calls (see [the compatibility notes](docs/forgebox-api-compatibility.md))?
* How exactly should endpoint-prefixed dependencies look in `box.json`? This needs to be verified against a running registry.
* What are the rules for proxying or mirroring ForgeBox content, and what do the ForgeBox terms of use allow?
* Who can claim a package name, and how are disputes resolved?

## 📜 License and credits

Licensed under the [Apache License 2.0](LICENSE).

This project is not affiliated with or endorsed by Ortus Solutions. ForgeBox and CommandBox are products of [Ortus Solutions](https://www.ortussolutions.com). We implement a compatible API so the CommandBox CLI, which is open source, works with community registries, and we are grateful for everything ForgeBox and CommandBox have given the CFML community.
