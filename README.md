# nebula-templates

Community app templates for [NebulaCtrl](https://docs.nebulactrl.dev). A template is a single YAML file that describes one or more services — an image, a database, processes, volumes, a domain and the environment variables that wire it all together. Install one and NebulaCtrl creates the project, the services, the database, the domain and (optionally) deploys it in one step.

This repository is the **community catalogue**: anyone can contribute a template through a pull request, and every NebulaCtrl instance can read it, alongside its own **built-in** catalogue (official templates that ship with each release) and whatever **organization** templates a workspace has saved for itself. A community template whose slug matches a built-in template is hidden — the built-in one wins.

## How templates appear in NebulaCtrl

Every NebulaCtrl control plane refreshes this repository on an hourly timer (and once at startup), reading `templates/*.yaml` straight off the `main` branch. Nothing here is fetched per-user or per-request — an instance keeps the last snapshot it fetched and serves that, so a broken or temporarily unreachable repository never breaks anyone's install flow. The templates then show up in the "Community" section of the Templates gallery, alongside a link back to this repository and a note that a template's contents are worth reading before you install it — a template runs whatever image and command it names, with the environment variables it declares.

A self-hosted control plane can turn the community catalogue off (`--community-templates=false` / `NEBULA_COMMUNITY_TEMPLATES=false`) or point it at a fork (`--community-templates-repo=<owner>/<name>`).

## Installing a template by link

Every template here has a stable, public install link:

```
https://<your-control-plane>/<org>/templates?install=<raw-or-blob-url>
```

`<raw-or-blob-url>` is this template's URL in this repository — either form works:

```
https://github.com/RobinGasi/nebula-templates/blob/main/templates/mealie.yaml
https://raw.githubusercontent.com/RobinGasi/nebula-templates/main/templates/mealie.yaml
```

Opening that link takes you straight to the install dialog, pre-filled and previewed — nothing is saved to your organization until you click Install.

## The file format

The full field-by-field reference — every key, its constraints, the `${{ ... }}` expression syntax, the secret rule, and a complete worked example — lives in the [template file reference](https://docs.nebulactrl.dev/reference/templates/). Read that first; this repository's [CONTRIBUTING.md](./CONTRIBUTING.md) only covers the rules specific to a *community* contribution.

## A short example

A template needs a `schemaVersion`, a `name`, a `description`, a `category`, and 1–20 `services`. This one deploys a single stateless tool with no database, one process, and an optional domain built from an input answered at install time:

```yaml
schemaVersion: 1
name: Example Tool
description: A minimal one-service template, for illustration.
category: development
services:
  - name: app
    image: docker.io/library/example:1.4.2
    processes:
      - name: web
        kind: http
        port: 8080
        domain:
          hostname: ${{ inputs.HOSTNAME }}
          exposure: public
inputs:
  - key: HOSTNAME
    label: Hostname
    description: Leave empty to add a domain later.
```

Look at any file under [`templates/`](./templates/) for a complete, real example — several wire up a generated database, a generated secret, and a rendered public URL.

## License

[MIT](./LICENSE).
