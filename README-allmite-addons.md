# allmite-addons — Server & model companions for Allmite Ai

> Turn any machine into an Allmite Ai backend: phone (Termux/proot),
> Windows PC, or Linux server — with automatic model catalog sync.
>
> **عربي:** إضافات تحوّل أي جهاز (هاتف/ويندوز/لينكس) لخادم يعمل مع تطبيق
> Allmite Ai، مع مزامنة تلقائية لقوائم النماذج. كل مجلد إضافة مستقلة.

---

## 1. What is in this repo (after extraction)

```
allmite-addons/
├── README.md                  ← this file (start here)
├── allmite-termux.tar.gz      ← stock Termux companion (phone, no proot)
├── allmite-proot.tar.gz       ← proot Ubuntu/Kali companion (phone servers)
├── allmite-windows.tar.gz     ← Windows PC server companion
└── allmite-linux.tar.gz       ← Linux PC/server companion
```

Each archive extracts to one folder with its own README + scripts.
Folders never import each other — use only the one(s) for your machine(s).

## 2. Extract first (per system)

**Termux / Linux / proot / macOS:**
```bash
cd ~   # or any working folder
tar -xzf allmite-termux.tar.gz
ls allmite-termux   # README.md install.sh bin/ jobs/ boot/
```

**Windows (PowerShell — `tar` is built in since Windows 10):**
```powershell
cd $HOME
tar -xzf allmite-windows.tar.gz
dir allmite-windows
```
If `tar` is missing (very old Windows): install from Microsoft Store
"App Installer", or open the `.tar.gz` with 7-Zip/WinRAR (extract twice:
first `.gz`, then `.tar`).

**Verify integrity (optional, all systems):**
```bash
tar -tzf <file>.tar.gz | head   # lists contents without extracting
```

## 3. Install per system (after extracting)

### A. Phone only — stock Termux (no proot)
```bash
pkg install git -y
cd ~/allmite-termux && ./install.sh
```
What it does: helper packages, pairing token (paste once in the app),
feed on `127.0.0.1:8099`, periodic job every ~6h, boot hook.
Needs: F-Droid Termux, Android 8+, `termux-setup-storage` accepted.

### B. Phone only — proot (where servers actually run)
```bash
# inside: proot-distro login ubuntu   (or your Kali env)
cd ~/allmite-proot
./provision-ubuntu.sh     # or ./provision-kali.sh for Kali
```
Installs: base tools, Node 22, opencode, CCR, Ollama (ARM64), cron.
Then start servers (`opencode serve --port 4096`, `ccr start`,
`ollama serve`) and wire `bin/models-sync.sh` + `jobs/setup-cron.sh`.

### C. Windows PC (phone connects to it)
In PowerShell **as Administrator**:
```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\allmite-windows\Setup-AllmiteServer.ps1
```
Installs: Node LTS, opencode, CCR, Ollama, firewall rules (4096/3456/
11434), OpenSSH server, starter CCR config. Check anytime with
`Check-AllmiteServer.ps1`. Companion sync: `Sync-AllmiteModels.ps1`
+ `Register-AllmiteTasks.ps1` (Task Scheduler, every 6h + feed at logon).

### D. Linux PC/server
```bash
cd ~/allmite-linux && ./setup-server.sh   # apt/dnf/pacman auto-detected
```
Then serve (`opencode serve --hostname 0.0.0.0 --port 4096` etc.) and
enable the systemd units + timer from `systemd/` (see its README).

## 4. Pair with the Allmite Ai app (all systems)

1. Each installer prints a **pairing token** — paste it once in the app
   (server settings). The same token gates the feed and the commands.
2. In the app use: `127.0.0.1:8099` (Termux/Linux/Windows companion) or
   `127.0.0.1:8098` (proot companion), or `http://PC_IP:8099` over LAN/SSH.
3. App buttons: "sync now" (inventory), "update" (pull missing + refresh),
   "ensure" (report gaps). Progress follows per job; failures never block.

## 5. How updates flow (so you never re-develop per update)

Companion (cron/job/task, ~6h or manual) → collects opencode models
(CLI/Zen API) + CCR gateway `/v1/models` + `ollama list` + package
versions → diffs vs last state → feed JSON (`/models.json`) → app merges:
new models appear, removed ones are marked stale (active defaults are
never auto-deleted).

## 6. Troubleshooting

| Symptom | Fix |
|---|---|
| `tar: command not found` (Windows) | update App Installer, or use 7-Zip |
| `permission denied` on `.sh` | `chmod +x *.sh` (or `bash file.sh`) |
| PowerShell blocks script | the `Set-ExecutionPolicy -Scope Process` line first |
| Feed unreachable from app | companion running? (`ps`/Task Manager), same loopback? token pasted? |
| `EACCES` in Termux | run `termux-setup-storage`, accept popup |
| Servers die on screen-off | `termux-wake-lock` + battery Unrestricted + Termux:Boot |

## 7. Security model

- Feeds bind `127.0.0.1` only (or SSH tunnel / trusted LAN with the token).
- The token gates **everything**; feeds carry **model metadata only —
  never API keys or secrets** (keys stay in CCR/opencode configs and the
  app's secure store).
- Scripts install from official sources only (opencode.ai, ollama.com,
  nodesource, npm, winget); review any script before running as usual.

## 8. Relationship to Allmite Ai

This repo holds the **server side**. The phone app (separate repo) holds
the client: device detection, server guide with copy buttons, catalog
merge, and update buttons.

## License

MIT — same as Allmite Ai. Third-party tools keep their own licenses
(Ollama, Node.js, llama.cpp, CCR, opencode).
