# specification-template

Template repository for specifications written in [AsciiDoc](https://asciidoc.org/), maintained by
Digitaliseringsdirektoratet. Create a new specification by using this repository as a template, then
edit the content under `docs/` and the RDF artifacts under `ontology/`.

## Repository layout

| Path | Purpose |
| --- | --- |
| `docs/main.adoc` | Document entry point — sets attributes and `include::`s every chapter |
| `docs/*.adoc` | One file per chapter (`forord.adoc`, `kapittel_1.adoc`, …) |
| `docs/locale/` | Translations for Asciidoctor's built-in labels (`attributes-nb`, `attributes-nn`) |
| `docs/shared/` | HTML-only fragments (PDF download link, analytics) |
| `docs/images/` | Images referenced via `:imagesdir: images` |
| `ontology/` | Machine-readable companion: `ontology.owl`, `validation_rules.ttl`, `example.ttl` |

Content is authored in Norwegian (Bokmål by default, `lang=nb`; Nynorsk supported, `lang=nn`).

## Building locally

Builds run through the `asciidoctor/docker-asciidoctor` image — no local toolchain required:

```sh
docker run -it -v $(pwd):/documents asciidoctor/docker-asciidoctor
# inside the container:
asciidoctor -a lang=nb -D docs -o index.html docs/main.adoc        # HTML
asciidoctor-pdf -a lang=nb -D docs -o document.pdf docs/main.adoc   # PDF
```

Generated artifacts (`*.html`, `*.pdf`, `*.epub`) are git-ignored — never commit them. For a live
preview, `pip install --user live-server && live-server docs`, then open <http://localhost:8888>.

See [CONTRIBUTING.md](CONTRIBUTING.md) for the full contributor guide.

## CI / publishing

CI uses the org's centralized reusable workflows from
[`Informasjonsforvaltning/workflows`](https://github.com/Informasjonsforvaltning/workflows) rather
than duplicating build logic here.

| Workflow | Trigger | Reusable workflow | What it does |
| --- | --- | --- | --- |
| `adocs-build-develop.yml` | push to `develop` (paths `docs/**`) | `specification-github-pages.yaml` | Builds HTML + PDF and publishes the site to the `gh-pages` branch (GitHub Pages) |
| `publish-docs-to-production.yml` | manual (`workflow_dispatch`) | `specification-upload-files.yaml` | Builds the docs and uploads them to the static-rdf-server at <https://data.norge.no> |

### Adapting the publish workflow for a real specification

`publish-docs-to-production.yml` ships with template placeholders. When using this template for an
actual specification, update its inputs:

- **`ontology`** — set to your specification's slug (replace `specification-template`).
- **`files`** — list each file to upload as `<path> <mime-type> <lang> [<dest-path>]`.
- **`version`** — uncomment and set when publishing a versioned release.
- The **`STATIC_RDF_SERVER_API_KEY`** secret must be available to the repository.

Publishing to production is intentionally manual. Spec repos that maintain release branches
(e.g. `v1`, `v2`) typically also trigger this workflow on push to those branches — see
[`dcat-ap-no`](https://github.com/Informasjonsforvaltning/dcat-ap-no) for a fully wired example.

> **GitHub Pages:** the site is served from the `gh-pages` branch. Set
> *Settings → Pages → Source* to `gh-pages` for the published output to appear.