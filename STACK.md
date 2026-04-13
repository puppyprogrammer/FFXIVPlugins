# FFXIVPlugins — Stack Architecture & Status

This document inventories the goatcorp stack, maps every lockout
chokepoint, and tracks what we've built, forked, and hardened.

---

## 1. Repos (all forked under `puppyprogrammer/`)

| Layer | Fork | Upstream | Status | Custom branch |
|---|---|---|---|---|
| **Windows launcher** | [`FFXIVQuickLauncher`](https://github.com/puppyprogrammer/FFXIVQuickLauncher) | `goatcorp/FFXIVQuickLauncher` | Rebranded, URL-repointed, themed, security-hardened | `ai-friendly`, `security/temp-file-hardening` |
| **In-game framework** | [`Dalamud`](https://github.com/puppyprogrammer/Dalamud) | `goatcorp/Dalamud` | URL-repointed, SHA-256 plugin verification, temp file hardening, IPC access control | `ai-friendly` |
| **Backend API** | [`XLWebServices`](https://github.com/puppyprogrammer/XLWebServices) | `goatcorp/XLWebServices` | HTTPS enforced, security headers added | `security/https-enforcement` |
| **Plugin index** | [`DalamudPluginsD17`](https://github.com/puppyprogrammer/DalamudPluginsD17) | `goatcorp/DalamudPluginsD17` | AI-friendly plugin manifests | `add-ffxivoices` |
| **Built plugin CDN** | [`PluginDistD17`](https://github.com/puppyprogrammer/PluginDistD17) | `goatcorp/PluginDistD17` | In sync with upstream | — |
| **Game struct defs** | [`FFXIVClientStructs`](https://github.com/puppyprogrammer/FFXIVClientStructs) | `aers/FFXIVClientStructs` | Workflows added | `ai-friendly` |
| **Dalamud release CDN** | [`dalamud-distrib`](https://github.com/puppyprogrammer/dalamud-distrib) | `goatcorp/dalamud-distrib` | In sync with upstream | — |
| **Static assets** | [`DalamudAssets`](https://github.com/puppyprogrammer/DalamudAssets) | `goatcorp/DalamudAssets` | In sync with upstream | — |
| **Linux launcher** | [`XIVLauncher.Core`](https://github.com/puppyprogrammer/XIVLauncher.Core) | `goatcorp/XIVLauncher.Core` | In sync with upstream (not yet rebranded) | — |

---

## 2. Lockout chokepoints

Every URL the launcher and Dalamud reach out to. All critical ones have
been repointed at our infrastructure (`ffxivplugins.commslink.net`).

### 2A. `kamori.goats.dev` → `ffxivplugins.commslink.net` ✅

| Endpoint | Caller | Status |
|---|---|---|
| `/Plugin/PluginMaster` | Dalamud plugin installer | **Repointed** |
| `/Plugin/History/{name}` | Dalamud changelog UI | **Repointed** |
| `/Dalamud/Release/Meta` | Launcher + Dalamud | **Repointed** |
| `/Dalamud/Release/VersionInfo` | Launcher + Dalamud self-update | **Repointed** |
| `/Dalamud/Release/Changelog` | Dalamud UI | **Repointed** |
| `/Dalamud/Asset/Meta` | Launcher | **Repointed** |
| `/Dalamud/Release/Runtime/Hashes/{v}` | Launcher | **Repointed** |
| `/Dalamud/Release/Runtime/DotNet/{v}` | Launcher | **Repointed** |
| `/Dalamud/Release/Runtime/WindowsDesktop/{v}` | Launcher | **Repointed** |
| `/Launcher/GetLease` | Launcher self-update | **Bypassed** (no kill switch) |
| `/Launcher/GetFile` | Launcher self-update | **Bypassed** |
| `/Launcher/GetLauncherClientConfig` | Launcher feature flags | **Repointed** |
| `/Proxy/Meta` | Launcher news | **Repointed** |

### 2B. `goatcorp.github.io` — static GitHub Pages

| URL | Status |
|---|---|
| `goatcorp.github.io/dalamud-distrib/` | Mirrorable via our fork's Pages |
| `goatcorp.github.io/integrity/` | Mirrorable |
| `goatcorp.github.io/faq/` | Not critical |

### 2C. `kiko.goats.dev` — bug feedback

Telemetry/feedback only. Safe to ignore or redirect to commslink.net.

### 2D. SE-owned (never touch)

`ffxiv-login.square-enix.com`, `patch-gamever.ffxiv.com`, `frontier.ffxiv.com` — these are Square Enix game servers. We pass through exactly like upstream does.

---

## 3. Security improvements

Vulnerabilities found in upstream code, fixed in our fork:

| # | Vulnerability | Component | Our fix | PR |
|---|---|---|---|---|
| 1 | No plugin download integrity verification | Dalamud | SHA-256 hash field + verification before extraction | [Dalamud #1](https://github.com/puppyprogrammer/Dalamud/pull/1) |
| 2 | API server accepts plaintext HTTP, no security headers | XLWebServices | HTTPS enforced, HSTS + full header suite | [XLWebServices #1](https://github.com/puppyprogrammer/XLWebServices/pull/1) |
| 3 | Temp file symlink-race (local privilege escalation) | Dalamud + Launcher | Atomic creation in app-specific subdir + reparse point check | [Dalamud #1](https://github.com/puppyprogrammer/Dalamud/pull/1), [Launcher #1](https://github.com/puppyprogrammer/FFXIVQuickLauncher/pull/1) |
| 4 | No IPC access control between plugins | Dalamud | `SetAllowedCallers()` opt-in allowlist on CallGate channels | [Dalamud #1](https://github.com/puppyprogrammer/Dalamud/pull/1) |

All fixes are build-verified and backwards-compatible with upstream plugins.

---

## 4. Launcher customizations (FFXIVQuickLauncher `ai-friendly` branch)

13 commits on top of upstream `master`:

1. **Phase 4:** Repoint all `kamori.goats.dev` URLs → `ffxivplugins.commslink.net`
2. **Phase 4 hotfixes:** Bypass Velopack update check, hardcode WINDOWS define
3. **Phase 6:** upstream-sync + smoke-test + lkg-tag workflows
4. **Rebrand:** XIVLauncher → FFXIVPlugins (exe name, window title, AppData path, shortcuts)
5. **Polish:** Remove unsupported warning, update About credits, Discord/FAQ/repo URLs
6. **Build:** Single-file publish, strip PDBs
7. **Theme:** CommsLink color palette via MaterialDesign overrides
8. **Settings:** Simplified UI — removed In-Game/Backup tabs, enlarged window
9. **Security:** Temp file hardening (fix #3)

---

## 5. Risk register

| Risk | Likelihood | Mitigation |
|---|---|---|
| Goatcorp hard-codes signed URL check in Dalamud | low–med | We ship our own Dalamud build. Our launcher only pulls from our CDN. |
| Game patches break Dalamud sigs | **high** | upstream-sync workflow pulls `Dalamud` + `FFXIVClientStructs` commits daily. |
| Goatcorp changes Kamori response schema | low | We control both server and client. Schema drift only matters if we stay wire-compatible, which we don't need to. |
| SE legal action / detection | low | Same risk surface as goatcorp. Nothing we do increases it. |
| Loss of FFXIVClientStructs upstream | med | Vendor it; auto-merge while it lasts; budget for taking over if forced. |
| Plugin authors won't dual-list | med | Manifest format is byte-compatible with goatcorp's. Dual-listing is a one-line PR. |

---

## 6. What's next

- [ ] Merge security PRs to default branches
- [ ] Rebuild Dalamud binary with security fixes and publish as new release
- [ ] Set up upstream-sync GitHub Actions (daily auto-PR from goatcorp, human review before merge)
- [ ] Plugin signing (Ed25519 — provider signs approved plugins, Dalamud verifies before load)
- [ ] AI plugin submission flow (PR-based, AI review + human approval)
- [ ] Linux launcher rebrand (XIVLauncher.Core)
