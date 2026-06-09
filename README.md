# Sample FHIR Implementation Guide

A minimal example showing how to:

* Build a FHIR Implementation Guide (IG) using the HL7 FHIR IG Publisher.
* Generate HTML output locally.
* Publish versioned HTML snapshots to GitHub Pages using GitHub Actions.
* Create a simple versioning strategy based on Semantic Versioning (SemVer).

The goal of this repository is educational. It intentionally keeps the Implementation Guide as small as possible so that
the build and deployment process can be understood without introducing additional tooling such as SUSHI or FHIR
Shorthand (FSH).

## Project structure

```text
.
├── .github/
│   └── workflows/
│       └── publish-ig.yml
├── input/
│   ├── includes/
│   │   └── menu.xml
│   ├── pagecontent/
│   │   └── index.md
│   └── resources/
│       └── ImplementationGuide-example.json
├── .gitignore
├── LICENSE
├── README.md
└── ig.ini
```

## Local development

### Prerequisites

* Java 17 or later
* Ruby
* Git

### Download the FHIR IG Publisher

The publisher is intentionally not committed to the repository and is downloaded on demand, both locally and in GitHub
Actions. Download the latest version:

```bash
curl -L https://github.com/HL7/fhir-ig-publisher/releases/latest/download/publisher.jar -o publisher.jar
```

### Build the Implementation Guide

```bash
java -Dfile.encoding=UTF-8 -jar publisher.jar -ig ig.ini
```

The generated HTML output is written to:

```text
output/
```

Open:

```text
output/index.html
```

in a browser to view the generated site.

If the build completes successfully, the generated site will also contain supporting pages such as `artifacts.html`,
`toc.html` and `qa.html`.

## GitHub Pages publishing

This repository uses GitHub Actions to publish versioned snapshots of the generated Implementation Guide.

A new deployment is triggered whenever a version-like Git tag is pushed. The workflow then validates the tag against the
Semantic Versioning specification before continuing.

Examples:

```text
0.1.0
1.0.0
1.2.3
1.0.0-alpha.1
```

Examples that are intentionally rejected:

```text
v1.0.0
01.0.0
1.0
```

Create and publish a version:

```bash
git tag 0.1.0
git push origin 0.1.0
```

The workflow:

1. Validates the tag against the Semantic Versioning specification.
2. Downloads the latest FHIR IG Publisher.
3. Builds the Implementation Guide.
4. Publishes the generated output to the `gh-pages` branch.
5. Stores each version in its own directory.
6. Updates the `latest` alias to point to the most recently published version.
7. Updates the root page to redirect to `latest`.

Example:

```text
https://sample-ig.xtracked.io/
https://sample-ig.xtracked.io/latest/
https://sample-ig.xtracked.io/0.1.2/
```

### Versioned publishing

Each release is published to its own version folder.

Example:

```text
/
├── latest/
├── 0.1.0/
├── 0.1.1/
└── 0.1.2/
```

The root URL redirects to `latest`, and `latest` redirects to the most recently published version.

Example:

```text
https://sample-ig.xtracked.io/
→ https://sample-ig.xtracked.io/latest/
→ https://sample-ig.xtracked.io/0.1.2/
```

This approach keeps all previously published versions available while providing a stable URL for the latest release.

A `versions.json` file is maintained in the root of the GitHub Pages site. It contains metadata about all published
versions and identifies the current `latest` version.

The `versions.json` file is generated automatically during deployment.

Example:

```json
[
  {
    "version": "0.1.3",
    "title": "0.1.3",
    "aliases": ["latest"]
  },
  {
    "version": "0.1.2",
    "title": "0.1.2",
    "aliases": []
  }
]
```

This metadata can be used by client-side JavaScript to implement features such as version switchers or warnings when
viewing an outdated version.

### Custom domain support

The GitHub Actions workflow automatically publishes a CNAME file to the `gh-pages` branch. This allows GitHub Pages to
serve the generated site through a custom domain.

Example:

```text
https://sample-ig.xtracked.io/
```

After the first successful deployment:

1. Open Settings > Pages.
2. Select the `gh-pages` branch and save the configuration.
3. Verify that the custom domain has been detected from the published CNAME file.
4. Enable "Enforce HTTPS" after GitHub has provisioned the TLS certificate.

GitHub automatically provisions and renews the TLS certificate for the custom domain.

## Design decisions

The repository intentionally favors simplicity and maintainability over feature completeness. The following sections
explain some of the key implementation choices.

### Why not commit publisher.jar?

The FHIR IG Publisher is downloaded during each build.

Benefits:

* The repository remains small.
* No large binary files need to be stored in Git.
* Every build uses the latest published version of the IG Publisher. This means that build results may change over time
  as newer publisher versions become available.

This approach favors simplicity over strict build reproducibility.

Projects that require fully reproducible builds may prefer to pin a specific publisher version.

### Why cache FHIR packages?

The FHIR IG Publisher downloads templates, FHIR packages and dependencies into:

```text
~/.fhir/packages
```

Caching this directory significantly reduces build times while still allowing the publisher to download missing
dependencies when required.

### Why use .nojekyll?

The FHIR IG Publisher already generates a complete static website.

GitHub Pages normally supports an additional Jekyll processing step before publishing content. The `.nojekyll` marker
instructs GitHub Pages to serve the generated files exactly as produced by the FHIR IG Publisher.

### Why use redirects instead of copying the latest version?

Each version is published only once.

The `latest` location contains a lightweight redirect rather than a copy of the generated HTML output. This avoids
storing duplicate content and ensures that all version-specific URLs remain the canonical locations of published
releases.

The version metadata stored in `versions.json` makes it possible to build version-aware features without modifying
previously published releases.