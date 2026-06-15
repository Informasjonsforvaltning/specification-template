# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A **template repository** for authoring specifications/standards in AsciiDoc, maintained by Digitaliseringsdirektoratet (Informasjonsforvaltning). New specifications are created by copying this template. There is no application code — the deliverables are an HTML and PDF document built from `.adoc` sources, plus an accompanying RDF ontology. Author-facing content is written in Norwegian (default `lang=nb`, Bokmål; `lang=nn`, Nynorsk also supported).

## Build & preview

All builds run through Asciidoctor in the `asciidoctor/docker-asciidoctor` Docker image (no local toolchain needed):

```sh
docker run -it -v $(pwd):/documents asciidoctor/docker-asciidoctor
# inside the container:
asciidoctor -a lang=nb -D docs -o index.html docs/main.adoc        # HTML
asciidoctor-pdf -a lang=nb -D docs -o document.pdf docs/main.adoc   # PDF
```

`docs/index.html`, `docs/document.pdf` and other generated artifacts (`*.html`, `*.pdf`, `*.epub`) are git-ignored — never commit them.

Local live preview (optional): `python -m pip install --user live-server && live-server docs`, then open http://localhost:8888.

## Document structure

`docs/main.adoc` is the single entry point. It sets document attributes (`:doctype: book`, `:toc: left`, etc.) and `include::`s everything else. The build pattern is **one file per chapter**, assembled in `main.adoc`:

- `docs/locale/attributes.adoc` — loaded first; conditionally pulls in `attributes-{lang}.adoc` (`-nb`, `-nn`) to translate built-in Asciidoctor labels. Translation files must contain **no blank lines**. To add a language, create `attributes-<iana-subtag>.adoc`.
- `docs/forord.adoc`, `docs/kapittel_1.adoc`, … — the actual content chapters. Add new chapters by creating an `.adoc` file and adding an `include::` line in `main.adoc`.
- `docs/shared/` — reusable fragments included by `main.adoc` (`download.adoc` for the PDF-download tip, `stats.adoc` for analytics). These are wrapped in `ifeval::["{backend}" == "html5"]` so they only render in HTML output.
- `docs/images/` — referenced via `:imagesdir: images`.

When editing content, work in the chapter `.adoc` files — do not hand-edit generated output. Follow Asciidoctor recommended practices (https://asciidoctor.org/docs/asciidoc-recommended-practices).

## Ontology

`ontology/` holds the machine-readable companion to the spec: `ontology.owl` (the ontology), `validation_rules.ttl` (SHACL/validation rules), and `example.ttl` (a worked example). These are RDF/Turtle files, independent of the AsciiDoc build.

## CI / publishing

GitHub Actions (`.github/workflows/`) build HTML + PDF on push and deploy via `peaceiris/actions-gh-pages`:

- `adocs-build-master.yml` — on push to `master`, deploys `./docs` to the `html` branch (the published site).
- `adocs-build-review.yml` — on push to `review`, deploys `./docs` (preview).

Both build with `-a lang=nb`. The PDF build step is `continue-on-error: true`, so a failing PDF will not block deployment.

## Contributing workflow

Branch off `master`, make changes in `docs/`, and open a PR (the default working branch in this checkout is `develop`; the repo's main branch is `master`). See `CONTRIBUTING.md` for the full Norwegian-language contributor guide.