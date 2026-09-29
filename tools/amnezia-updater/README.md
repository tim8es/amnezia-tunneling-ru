# Amnezia List Updater

A small companion utility that keeps Amnezia VPN's **"all sites through VPN except this list"**
split-tunneling list synchronized with the canonical
[lib4u/amnezia-tunneling-ru](https://github.com/lib4u/amnezia-tunneling-ru) release.

It does **not** modify Amnezia Client, VPN server settings, protocols, or the selected route mode.

## UX

1. Download the build for your operating system.
2. Run it.
3. Click **«Включить автообновление»**.
4. Done. The operating system checks for updates every six hours; no background daemon stays resident.

The updater installs its packaged runtime into the current user's application-data directory
before registering the schedule, so moving or deleting the downloaded installer afterwards does
not break scheduled updates.

## Safety model

- Source is hard-coded to the original repository:
  `https://github.com/lib4u/amnezia-tunneling-ru/releases/download/latest/amnezia.json`.
- JSON is validated before any write.
- SHA-256 prevents unnecessary settings writes.
- Existing manually added domains are preserved.
- Domains previously managed by the upstream list are removed if upstream removes them.
- Existing resolved IPs are retained for managed domains when upstream provides only a hostname.
- A backup of `Conf/ExceptSites` is written before every applied update.
- If Amnezia VPN is running, settings are **not** touched. The update is stored as pending and is
  applied after Amnezia exits.
- Unknown/uninitialized Amnezia settings fail closed: nothing is written.
- `Conf/routeMode` and `Conf/sitesSplitTunnelingEnabled` are never changed automatically.

## Commands

```text
amnezia-list-updater --install
amnezia-list-updater --update --silent
amnezia-list-updater --status
amnezia-list-updater --uninstall
amnezia-list-updater --validate-file amnezia.json
```

Running without arguments opens the one-screen GUI.

## Scheduling

| OS | Mechanism |
|---|---|
| Windows | Task Scheduler, every 6 hours |
| macOS | per-user LaunchAgent, every 6 hours |
| Linux | per-user systemd timer, every 6 hours |

No administrator/root privileges are required.

## Build

Requires CMake 3.21+ and Qt 6.5+ (Core, Network, Widgets, Test).

```bash
cmake -S tools/amnezia-updater -B build -DBUILD_TESTING=ON
cmake --build build --parallel
ctest --test-dir build --output-on-failure
```

CI builds and tests Windows, macOS, and Linux. Linux CI also downloads the current upstream
`amnezia.json` and validates it with the built utility.

## Managed-set merge

The updater records only the domain names that it manages. On the next update:

```text
user entries = current Amnezia list - previous managed set
result       = user entries + new managed set
```

This makes upstream deletions work without deleting user-created exceptions.
