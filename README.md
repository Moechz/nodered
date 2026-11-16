# nodered-for-tos

> TerraMaster TOS 7 App Center — **Docker application** package for
> [Node-RED](https://github.com/node-red/node-red).

## What this is

A four-file TOS Docker application archive (`nodereddocker.tar.gz`) that installs
Node-RED on TOS 7 through the App Center:

- `config.ini` — TOS application metadata (`application_type: docker`, id `nodereddocker`)
- `nodereddocker.lang` — 23-language superset (TOS requires 14)
- `nodereddocker.svg` — application icon (1252 bytes, 3 SVG elements)
- `docker-compose.yml` — one service, image `nodered/node-red:5.0.9`

## How it works

| | |
|---|---|
| Container image | `nodered/node-red:5.0.9` (Docker Hub, used unmodified) |
| Open | `http://<NAS-IP>:8043/` (container port 1880) |
| Runs as | non-root, `user: "1000:1000"` |
| Persistent data | `/data` → `/Volume1/DockerAppData/nodereddocker/data` |
| Logs | `docker logs nodereddocker` (stdout/stderr) |

`/data` (mounted from `DockerAppData/nodereddocker/data`) holds `flows.json`, `flows_cred.json`, `settings.js` and the `node_modules/` of the nodes you install from the palette.

## Privacy

This application runs entirely on your NAS. **The developer does not collect, receive or
have access to any data processed by the application.** There is no telemetry, no analytics
and no account.

| Topic | Detail |
|---|---|
| Data processed | The application's configuration, database, caches and logs, plus any files you create with it. |
| Where it is stored | The application data folder (`/Volume1/DockerAppData/nodereddocker/`) on your NAS. Nothing is uploaded to the developer. |
| Retention period | Until **you** delete it. Logs are rotated or size-limited by the application; uninstalling keeps the data unless you tick “delete data”. |
| Your rights | **Access / export**: copy the data folder. **Correct**: change the values in the web UI. **Delete**: uninstall with “delete data”, or remove the folder after stopping the container. |
| Deletion channel | Delete the folder yourself, or email **moechz@163.com** (answered within 30 days). |
| Security measures | Container runs as uid 1000 (never root), no `privileged`, no host network, no `cap_add`, no Docker socket, one published port; image tag pinned and refreshed on upstream fixes. |
| Children | Not directed at children. |

**Full privacy policy:** [PRIVACY.md](PRIVACY.md) —
<https://github.com/Moechz/nodered/blob/main/PRIVACY.md> — and the website version at
<https://www.moechz.com/privacy.html>.

## Third-party services (API disclosure)

The developer operates no server in this path. The application contacts only the services
**you configure**, plus (optionally) an update check and the image registry.

| Service | Endpoint | Data sent | Destination / region |
|---|---|---|---|
| Services you configure in the app | the address you enter | only the data that feature needs | normally your own LAN or the provider you chose |
| Update check (if enabled) | the upstream project's release feed | current version string | GitHub, US |
| Image pull | `registry-1.docker.io` or your registry mirror | anonymous pull | Docker Hub, US / your mirror |

See `PRIVACY.md` §6 for the full table. Disable any integration in the app's settings if you
do not want it.

## Licence and attributions

| Component | Licence | Notes |
|---|---|---|
| Node-RED | Apache-2.0 | upstream application, used unmodified inside the image |
| Container image `nodered/node-red:5.0.9` | see the image's own metadata | pulled from Docker Hub at install time |
| This packaging | GPL-3.0 (see `LICENSE`) | compose file, metadata, translations, icon |

“Node-RED”, “TerraMaster” and “TOS” are trademarks of their respective owners. This is an
independent community package by Moechz and is not affiliated with the upstream project or
TerraMaster. Full details: [NOTICE.md](NOTICE.md).

## Design highlights

- **Non-root.** `user: "1000:1000"`; verified on real hardware (the container reports uid 1000).
- **Fixed image tag, never `:latest`**, Docker Hub only, no `privileged`, no `network_mode: host`, and no Docker socket, no `cap_add`.
- **Health check** probing the service on container port 1880.
- **Deterministic build**: GNU tar with uid/gid 0 and zeroed timestamps, gzip without a timestamp, plus the review-standards self-check above.
- **Already non-root.** The upstream image runs as `node-red` (uid 1000); the package pins the same user and only `/data` is persistent.
- **No database.** Flows are plain JSON files, so a backup is a folder copy.

## Runtime file manifest

Everything the application writes on the NAS lives inside its own data
directory; the container-internal `/tmp` is discarded with the container. The
table lists what is created and how it behaves. Nothing else is written.

| Path | Purpose | Created when | Growth bound | Lifecycle |
|---|---|---|---|---|
| `/Volume1/DockerAppData/nodereddocker/data` | Node-RED application data (everything the container writes outside /tmp) | on first start / on demand | bounded by your own data | persistent; kept on uninstall unless you ask to delete it |

## Build

```bash
scripts/build.sh                  # -> out/nodereddocker.tar.gz + .sha256
scripts/build.sh 5.0.9-2   # packaging iteration bump
scripts/build.sh 5.0.9-1 aarch64
```

The build stages the four files, substitutes the version/platform placeholders,
removes CRLF/BOM/AppleDouble, then runs
`tools/verify.py`, a review-standards self-check (archive layout, JSON validity,
required fields, 14+ language coverage, version consistency, Docker-Hub-only
images, fixed tag, reserved/recommended host ports, non-root user, per-service
healthcheck/TZ/restart, data path below the application data root, never the
`/Volume*` wildcard, comment-free sequence blocks, `x-app-meta` position, no
literal secrets and the SVG size/element limits).

## Local verification

```bash
./TOSAppSelfTestingTool -c out/nodereddocker.tar.gz   # official spec check
bash test/install-and-verify.sh                  # on-device E2E (needs root)
```

> Docker applications cannot be side-loaded through the App Center's manual
> install page (it accepts `.deb` only); the official self-testing tool's `-i`
> mode or `docker compose` are the only ways to install one for testing.

## Store submission checklist

1. Public GitHub repo (code + README only).
2. Release **tag `v5.0.9-1`**, assets `nodereddocker.tar.gz` +
   `nodereddocker.tar.gz.sha256` (Docker asset naming: `<app_id>.tar.gz`, **no
   platform suffix**).
3. Developer platform → Add Application: ID `nodereddocker`, package type Docker,
   repository URL, architecture `x86_64`.
4. Version Management → Add Version `5.0.9-1` → automated validation → review.


## Limitations

- The editor has no authentication by default; enable adminAuth or keep it on the LAN.
- Installing extra nodes needs outbound access to the npm registry.
- The web UI is plain HTTP; use a TLS reverse proxy before exposing it to the internet.

---

Packaged for TOS 7 by Moechz. Upstream image used unmodified from Docker Hub.

## Support

- Issues and questions: <https://github.com/Moechz/nodered/issues>
- Email: moechz@163.com
- Privacy: [PRIVACY.md](PRIVACY.md) · Licence: [NOTICE.md](NOTICE.md)
