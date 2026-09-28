# Contributing a template

Thanks for adding to the catalogue. A few rules keep every template here safe to install sight-unseen on someone else's cluster.

## Where it goes

One file per template, at `templates/<slug>.yaml`. The file name (minus `.yaml`) must equal the template's own `slug` field, or its default (the lowercased, hyphenated form of `name`) when `slug` is omitted. `nebula template validate` checks this for you — see below.

For everything else about the file format itself — every field, the `${{ ... }}` expressions, limits, categories — read the [template file reference](https://docs.nebulactrl.dev/reference/templates/) first.

## Rules specific to this repository

- **Pin an exact tag.** `image:` must name a real, current release tag (or a digest). Never `latest`, never a floating tag like `stable` or a branch name — check the registry yourself and confirm the tag you pin is the one that tag currently resolves to.
- **Non-root unless the image genuinely cannot.** Omit `runAsUser` when the image already declares its own non-root user (most do) — the agent enforces `runAsNonRoot` and lets the image's own `USER` apply. Set `runAsUser: 0` only when the image's own entrypoint needs root (to fix ownership before dropping privileges, or to bind a port under 1024), and say so in a YAML comment next to the field, with a link or a one-line reason. A template that sets `runAsUser: 0` without a comment will be asked to add one.
- **Secrets are generated, never typed.** Every password, API key or signing secret is `${{ secret(N) }}`, generated fresh per install. A real credential — even a throwaway one you tested with — must never appear in a pull request, an image tag, a comment, or the `readme:` field. If your app needs a specific known-default account for first login (some apps ship one, like `admin`/`admin`), say so in the `readme:` field and tell people to change it — don't try to override it with a fake generated value that doesn't match what the app actually does.
- **Modest resources.** Set `cpu` and `memory` requests/limits on every process. These templates are aimed at homelab-sized clusters: pick values the app genuinely needs to start and idle, not a production sizing guess.
- **A volume for every data directory.** Anything the app itself documents as persistent (its database file, uploads, config) gets a `volumes:` entry at that exact path. An app with no genuine persistent state (a static tool, a stateless proxy) can have none — say so implicitly by leaving `volumes` out, not by guessing a path that doesn't matter.
- **A `readme:` with first-login steps.** At minimum: what to do the first time you open the app (register an account, log in with a documented default, run a setup wizard) and anything a new user would otherwise have to go looking for (a default port, a companion browser extension, where the admin panel lives).
- **Evidence in the pull request description.** For every image used, link the exact registry tag you pinned (a Docker Hub / GHCR tag page, or the `crane`/`docker manifest inspect` output you ran) and the official documentation you read for its port, data directory, non-root user and required environment variables. A reviewer should not have to re-derive what you already checked.

## Validating locally

Before opening a pull request, run the same check CI runs, over your new file:

```sh
docker run --rm -v "$PWD:/w:ro" ghcr.io/nebulactrl/nebula:latest template validate /w/templates/<slug>.yaml
```

It prints `<file>: ok (<name>, <n> services)` for a valid file, or every problem it found — one per line, each with the field path and, where it can, the line number — and exits non-zero if anything needs fixing. This is exactly what `.github/workflows/validate.yml` runs over every file in `templates/` on every pull request and every push to `main`.

## Review

A maintainer reviews every image reference and every variable in a pull request before merging — not just that the file parses. That means checking the image actually is what it claims to be, that the environment variables actually configure what the description says they do, and that nothing routes a secret or a credential anywhere it shouldn't.

A merged template is not permanent: if its image becomes unmaintained (no updates for its known vulnerabilities), gets pulled from its registry, or turns out to do something unsafe (phones home unexpectedly, ships a backdoored dependency, or similar), it may be removed from `templates/` without notice. Removing a template does not affect anyone who already installed it — it only stops appearing in the catalogue for new installs.
