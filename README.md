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

## License / branding

Node-RED is distributed by its upstream authors under **Apache-2.0**. The container image is the official image from Docker Hub (`nodered/node-red:5.0.9`) and is used unmodified; no upstream source code is redistributed here. This packaging is an independent community submission by Moechz and is not affiliated with the upstream project or TerraMaster.

## Limitations

- The editor has no authentication by default; enable adminAuth or keep it on the LAN.
- Installing extra nodes needs outbound access to the npm registry.
- The web UI is plain HTTP; use a TLS reverse proxy before exposing it to the internet.

---

Packaged for TOS 7 by Moechz. Upstream image used unmodified from Docker Hub.
