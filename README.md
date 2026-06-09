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

Example:

```text
https://<organization>.github.io/<repository>/0.1.0/
https://<organization>.github.io/<repository>/0.2.0/
https://<organization>.github.io/<repository>/1.0.0/
```

## Why not commit publisher.jar?

The FHIR IG Publisher is downloaded during each build.

Benefits:

* The repository remains small.
* No large binary files need to be stored in Git.
* Every build uses the latest published version of the IG Publisher. This means that build results may change over time
  as newer publisher versions become available.

This approach favors simplicity over strict build reproducibility.

Projects that require fully reproducible builds may prefer to pin a specific publisher version.

## Why cache FHIR packages?

The FHIR IG Publisher downloads templates, FHIR packages and dependencies into:

```text
~/.fhir/packages
```

Caching this directory significantly reduces build times while still allowing the publisher to download missing
dependencies when required.

## Why use .nojekyll?

The FHIR IG Publisher already generates a complete static website.

GitHub Pages normally supports an additional Jekyll processing step before publishing content. The `.nojekyll` marker
instructs GitHub Pages to serve the generated files exactly as produced by the FHIR IG Publisher.