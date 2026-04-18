# painscaler-demo

Public demo deployment for [painscaler](https://github.com/PainScaler/painscaler).
Hosts a scrubbed synthetic ZPA snapshot and the compose stack used at
[demo.painscaler.com](https://demo.painscaler.com).

The `painscaler` submodule is pinned to a release tag. Bump the pin to ship a
new demo version.

## Layout

```
painscaler/            # submodule, pinned to a painscaler release tag
snapshot.json          # scrubbed synthetic ZPA snapshot (not committed; see .example)
snapshot.example.json  # minimal placeholder
docker-compose.yml     # caddy + painscaler-api + painscaler-web
Caddyfile              # TLS for demo.painscaler.com
.env.example           # runtime config template
```

## Run locally

```bash
git clone --recurse-submodules git@github.com:PainScaler/demo.git
cd demo
cp .env.example .env
cp snapshot.example.json snapshot.json   # or drop in a real scrubbed snapshot
docker compose up --build
```

Visit `http://localhost` — demo mode banner should show in the UI.

## Bump painscaler version

```bash
git -C painscaler fetch --tags
git -C painscaler checkout vX.Y.Z
git -C painscaler submodule update --init --recursive
git add painscaler
git commit -m "bump painscaler to vX.Y.Z"
git push
```

CI rebuilds the image and redeploys.

## Snapshot data

The demo loads a JSON-encoded `fetcher.Snapshot` from `snapshot.json` when
`PAINSCALER_DEMO_SEED` is set. Data must be fully scrubbed of tenant-specific
names, IDs, and domains before committing.

Regenerate by running the scrub tool against a real tenant export (see
`painscaler/cmd/snapshot-dump` once published), then hand-review the output.

## Deploy

GitHub Actions workflow in `.github/workflows/deploy.yml` builds the image on
push to `main` and ships it to the demo host over SSH. Required secrets:

- `DEMO_SSH_HOST`, `DEMO_SSH_USER`, `DEMO_SSH_KEY`
