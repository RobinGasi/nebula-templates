## Template

- Name / slug:
- What it does (one sentence, for the reviewer — not the template's own `description:`):

## Checklist

- [ ] One file at `templates/<slug>.yaml`; the file name matches the template's slug (`nebula template validate` confirms this).
- [ ] `nebula template validate templates/<slug>.yaml` passes locally (paste its output below).
- [ ] Every `image:` is pinned to an exact release tag or digest — never `latest` or a floating/branch tag.
- [ ] `runAsUser` is omitted wherever the image already runs non-root by default; where it's set to `0`, there's a YAML comment next to it explaining why the image needs root.
- [ ] Every password, API key or signing secret is `${{ secret(N) }}` — no real or fake-but-real-looking credential appears anywhere in the file.
- [ ] Every process that needs one has `cpu`/`memory` requests and limits sized for a small homelab deployment, not a production guess.
- [ ] Every documented persistent data path has a `volumes:` entry.
- [ ] The template has a `readme:` with first-login steps (default account, setup wizard, where the admin panel is, anything non-obvious).

## Evidence

For each image used, link what you checked:

- Registry tag (Docker Hub / GHCR page, or `docker manifest inspect` / `crane config` output) confirming the pinned tag, its `User`, `ExposedPorts` and any declared `Volumes`:
- Official documentation for the port, data directory, non-root user and required environment variables:

## Validation output

```
(paste the output of `nebula template validate templates/<slug>.yaml` here)
```
