# Bukas Service downloads

Public distribution repository for Bukas Service. Application source is maintained separately.

The distribution pipeline is being prepared; packages are available after each channel’s first successful release.

## Installation and operation

Apple Silicon and macOS 26 or later are supported. Xcode is not required on the
user's Mac. Install and authenticate Codex CLI separately (`codex login`) first.
Tailscale is optional for remote clients and is installed/configured separately.

Choose one channel:

```sh
brew install reirei-lab/tap/bukas-service       # stable
# OR
brew install reirei-lab/tap/bukas-service-edge  # latest successfully published main
bukas-service setup
brew services start bukas-service             # use bukas-service-edge for edge
bukas-service doctor
```

These commands become available after the first successful release of each channel.
Do not use sudo. The LaunchAgent starts on user login, not before login or after
logout. The client app can quit while the service keeps running. The Mac must
remain awake for remote access. A failed service is diagnosed using the logs;
the launch configuration intentionally does not repeatedly restart failed setup.

Both formulae install `bukas-service` and conflict with each other. Both read
`~/.config/bukas/service.json`. `setup` discovers Codex and records its absolute
path, verifies `codex login status`, and creates the file with mode 0600. It never
overwrites existing settings or starts/stops a service. Edit the JSON to change
settings, or supply `setup --host IP --port N --codex /absolute/path` on first use.
`doctor` checks configuration, storage compatibility, login status, and the
service's WebSocket hello and App Server connection; it does not run a model turn.

The default listener is `127.0.0.1:9753`. For iPhone access, either set `host` to
the Mac's Tailscale IP or retain localhost and configure Tailscale Serve. The
service has no separate client authentication: restrict Tailscale access to trusted
users/devices. See the service README for connection details. Do not expose it
directly to the public internet.

Logs: `$(brew --prefix)/var/log/bukas-service.log` and
`$(brew --prefix)/var/log/bukas-service-error.log`. Both channels share these logs.
Data remains at `~/.bukas` and `~/Library/Application Support/Bukas`, or the paths
selected in the configuration. Uninstall leaves configuration and data intact.

## Updating and changing channels

Stop only when interrupting current tasks is acceptable. Updating the package
does not automatically apply it to an already running process.

```sh
brew services stop bukas-service
brew update
brew upgrade bukas-service
brew services start bukas-service
bukas-service doctor
```

Switch stable to edge (reverse the names to switch back):

```sh
brew services stop bukas-service
brew uninstall bukas-service
brew install reirei-lab/tap/bukas-service-edge
brew services start bukas-service-edge
bukas-service doctor
```

Same-format data can be shared. A service refuses to open newer data formats.
Both stores carry `service-storage-version.json`; validation happens while holding
both service locks and before loading runtime data. Unmarked pre-distribution data
is format 1. This cannot protect against manually launching an old, pre-distribution
binary that has no format checks.

When introducing a storage format change, increment `ServiceStorageFormat.currentVersion`
and implement an explicit migration passed to `prepare`. It backs up both stores
under `~/Library/Application Support/Bukas Backups/<UUID>/` before migration, records
their original paths in `directories.json`, and marks both stores as migration-pending.
Pending/failed migrations block startup, including startup of the same version.
There are no format changes in the initial distribution release.

To roll back an incompatible format, first stop all Bukas services. Keep a copy
of the current stores, then restore **both** stores from a matching backup using
`directories.json` as the mapping. Restore only while no service owns either
directory. Work created after the backup is not present in the restored copy.
Backups include credentials and conversations; retain them privately.

