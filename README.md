# FFXIVPlugins

**Plugins for FFXIV. One click. No catch.**

FFXIVPlugins is a community-run alternative distribution of plugins for
Final Fantasy XIV. We mirror the open-source Dalamud framework and host
plugins that the official repository can't or won't approve — including
AI-assisted tools, experimental utilities, and quality-of-life mods built
by independent developers.

> **Operated by [CommsLink](https://commslink.net).** Independent project,
> not affiliated with goatcorp, XIVLauncher, or Dalamud.

This is the meta / hub repository for `puppyprogrammer/FFXIVPlugins` — a
ground-up fork of the entire goatcorp Dalamud plugin ecosystem.

We mirror every layer of the upstream stack so that the project cannot be
shut off by upstream changes to URLs, signing, or approval policy.

> **Status:** Phase 1 complete — static CDN mirrors live. Phases 2–6 in progress.
> See [`STACK.md`](./STACK.md) for the full plan.

---

## What this fork actually is

The official goatcorp ecosystem is *open source* but **not AI-friendly** —
they reject AI plugins on policy grounds. The technical infrastructure is
fine; the gatekeeping is purely social, applied at the plugin-approval and
distribution layers.

This project replaces every goatcorp-controlled component with a fork we
control, so AI plugins (and any other plugins the upstream project would
reject) can be distributed to users normally — installed in-game, auto-
updated, the whole experience — without depending on goatcorp infra.

## The sibling repos

Each upstream goatcorp repo has a 1:1 fork under `puppyprogrammer/`. Names
are kept identical to the upstream so we can auto-merge upstream fixes
without rename pain.

| Layer | Fork | Upstream | Role |
|---|---|---|---|
| **Plugin index** | [`DalamudPluginsD17`](https://github.com/puppyprogrammer/DalamudPluginsD17) | `goatcorp/DalamudPluginsD17` | TOML manifests pointing at plugin Git commits |
| **Built plugin CDN** | [`PluginDistD17`](https://github.com/puppyprogrammer/PluginDistD17) | `goatcorp/PluginDistD17` | Built plugin zips, served via Pages |
| **In-game framework** | [`Dalamud`](https://github.com/puppyprogrammer/Dalamud) | `goatcorp/Dalamud` | Injected into ffxiv_dx11.exe; loads plugins |
| **Game struct defs** | [`FFXIVClientStructs`](https://github.com/puppyprogrammer/FFXIVClientStructs) | `aers/FFXIVClientStructs` | Memory layouts every plugin uses |
| **Dalamud release CDN** | [`dalamud-distrib`](https://github.com/puppyprogrammer/dalamud-distrib) | `goatcorp/dalamud-distrib` | Built Dalamud zips, served via Pages |
| **Static assets** | [`DalamudAssets`](https://github.com/puppyprogrammer/DalamudAssets) | `goatcorp/DalamudAssets` | Fonts/icons Dalamud loads at startup |
| **Backend API** | [`XLWebServices`](https://github.com/puppyprogrammer/XLWebServices) | `goatcorp/XLWebServices` | The `kamori.goats.dev` server. ASP.NET. |
| **Windows launcher** | [`FFXIVQuickLauncher`](https://github.com/puppyprogrammer/FFXIVQuickLauncher) | `goatcorp/FFXIVQuickLauncher` | WPF launcher, patches game, injects Dalamud |
| **Linux/Steam Deck launcher** | [`XIVLauncher.Core`](https://github.com/puppyprogrammer/XIVLauncher.Core) | `goatcorp/XIVLauncher.Core` | Cross-platform launcher |
| **CI build pipeline** | [`Plogon`](https://github.com/puppyprogrammer/Plogon) | `goatcorp/Plogon` | Docker plugin builder |
| **Plugin packager** | [`DalamudPackager`](https://github.com/puppyprogrammer/DalamudPackager) | `goatcorp/DalamudPackager` | MSBuild task plugins use to package |
| **Plugin SDK** | [`Dalamud.NET.Sdk`](https://github.com/puppyprogrammer/Dalamud.NET.Sdk) | `goatcorp/Dalamud.NET.Sdk` | MSBuild SDK most modern plugins target |

## Live infrastructure

| Component | URL |
|---|---|
| Dalamud release CDN | https://puppyprogrammer.github.io/dalamud-distrib/ |
| Static assets CDN | https://puppyprogrammer.github.io/DalamudAssets/ |
| Built plugin CDN | https://puppyprogrammer.github.io/PluginDistD17/ |
| Backend API | _Phase 2 — coming soon, will live on commslink.net infra_ |

## Why a fork instead of a third-party repo?

Dalamud already supports user-added "third-party plugin repositories" — but
that path leaves the entire **runtime** (Dalamud itself, the launcher, the
auth/version API) under goatcorp's control. If goatcorp ever adds a signing
check, an allowlist, or a kill switch to the official Dalamud build, every
third-party repo dies overnight.

A full ecosystem fork is the only configuration that is structurally
immune to upstream lockout.

## Roadmap

- **Phase 0** ✅ — Inventory the goatcorp stack and map every endpoint. See [`STACK.md`](./STACK.md).
- **Phase 1** ✅ — Fork all upstream repos, stand up static CDN mirrors via GitHub Pages.
- **Phase 2** 🟡 — Run our own XLWebServices instance on commslink.net, pointing at our forks.
- **Phase 3** ⬜ — Build and publish our own Dalamud release with `MainRepoUrl` repointed at our XLWebServices.
- **Phase 4** ⬜ — Rebrand and ship `FFXIVPlugins Launcher` — a fork of XIVLauncher with all goatcorp URLs replaced.
- **Phase 5** ⬜ — Replace DIP17 with a no-gatekeeping plugin pipeline. AI plugins are first-class.
- **Phase 6** ⬜ — Sustainability: auto-merge bot for upstream Dalamud + ClientStructs commits, sig-update process for game patches.

## License

Each sibling repo retains its upstream license (AGPLv3 for Dalamud, GPLv3
for the launchers, MIT for ClientStructs, etc.). This meta repo is MIT.
