---
# Standard ID: lowercase letters, digits and - only; must match the section name in rules.ini of the rules project
name: global
# Standard version, recorded in the scoring report
version: v0.4
---

# Global PR Quality Scoring Rules

<!-- markdownlint-disable MD028 -->
<!-- markdownlint-disable MD033 -->

The global standard has two parts:

- "Global rules": apply to every file of every report, including the files
  scored by a framework standard.
- "Check items": apply only to the files that do not belong to any framework
  standard (global files below):

  - Documentation: READMEs, Markdown and other documentation.
  - CI/CD: pipeline configs such as GitHub Actions, GitLab CI and Azure
    Pipelines.
  - Containers: Dockerfile, docker-compose.
  - Scripts: shell (sh, bash, zsh), PowerShell.
  - Other configs: JSON, YAML, `.env` templates, `.gitignore`,
    `.editorconfig`, `.gitattributes` and so on.

The check items are compiled from tools and guides commonly used in the
community: markdownlint, the GitHub Actions security hardening guide, Docker's
official Dockerfile best practices and hadolint, ShellCheck, PSScriptAnalyzer,
the OWASP Secrets Management Cheat Sheet and The Twelve-Factor App.

This document is set up as the global standard of code-review-scoring
(`role="global"`) by the `rule.ini` in the same directory, and registered
as `global` in `rules.ini` of the rules project. The automatic scan
rules are in `global-cr-scoring-rule.tsv` in the same directory:

```ini
[rule]
standard="global-cr-scoring-rule.md"
scan="global-cr-scoring-rule.tsv"
role="global"
```

When a PR also changes framework files, the deductions for global files are
merged into the framework report of the project they belong to, under the same
A to F dimensions. When a PR changes only global files, this standard produces a
report on its own. Workflow rules such as the scoring scope, deductions
requiring evidence and upstream factors are handled by the scoring workflow and
are not covered in this document.

## Global rules

The following rules apply to every file of every report, including the files
scored by a framework standard. Where a framework standard says otherwise, the
framework standard wins.

- Third-party workaround exemption: code written to work around an issue in a
  third-party core system is not deducted for check items in dimension F
  if a comment explains why **and** includes the vendor's issue ticket number.
  If either is missing, deduct as usual.
- No double counting with static analysis: issues already listed in a
  provided static analysis report (for example Checkstyle, SonarQube,
  ShellCheck, hadolint, markdownlint or actionlint) are not deducted again.
- No deduction for pure formatting: when no static analysis report is
  provided, pure formatting issues (indentation, whitespace, import order) are
  not deducted, except for GL-F9.

## Dimensions

All standards share the following six dimensions. The points and the grades
always come from `defaults.ini` of the rules project; the severity scale also
comes from `defaults.ini` and can be overridden in the `rule.ini` in the same
directory. The first letter after `GL-` in a check code indicates its
dimension.

| Code | Dimension | Focus |
| --- | --- | --- |
| A | Correctness | Scripts and configs work and stop on failure |
| B | Security | Secrets, permissions, supply chain, untrusted input |
| C | Test quality | Tests and checks still run; changes are verifiable |
| D | Architecture and design | Responsibilities, duplication, env separation |
| E | Performance and resource use | Build cache, image size, limits, timeouts |
| F | Readability and maintainability | Docs are complete and correct; scripts are clear; pinned versions, idioms, config docs, VCS settings |

## Check items

Each check item is written as a list item with fields separated by the
full-width `｜`:

- Fixed severity: `- code｜content｜default severity`
- Adjustable severity:
  `- code｜content｜default severity｜adjustable to｜condition`

The severity may be adjusted only when the stated condition holds. Any check
item may be lowered to `Nit`.

### A. Correctness

- GL-A1｜A shell script does not stop on failure: no `set -e` (`set -euo pipefail`
  is recommended) and no check of each command's exit code｜Major
- GL-A2｜Variable expansions in a shell script are not quoted, which breaks when
  the value contains spaces or wildcards, for example `rm -rf $DIR/*`｜Major｜
  Critical｜When the expansion is used to delete, overwrite or move files
- GL-A3｜A PowerShell script does not handle errors: no
  `$ErrorActionPreference = 'Stop'`, no check of `$LASTEXITCODE` after external
  programs, or all errors are set to be ignored｜Major
- GL-A4｜A CI/CD or deployment config is wrong: it refers to a file, variable,
  job or stage that does not exist, or its triggers or execution order differ
  from what is intended｜Major｜Critical｜
  When it affects deployment to or operation of production
- GL-A5｜Documentation is inconsistent with the code or config changes in this
  PR: commands, parameters, paths, environment variables or versions are out of
  date｜Minor｜Major｜
  When the documentation describes installation, deployment or operations steps
- GL-A6｜A script depends on a specific environment without checking or
  documenting it: hard-coded absolute paths, assumptions about the current
  directory, or bash-only syntax run with `sh`｜Minor

### B. Security

- GL-B1｜A password, token, key or connection string containing a password is
  hard-coded in a file, or a file containing secrets (for example `.env` or a
  private key) is added to version control｜Critical
- GL-B2｜CI inserts untrusted input directly into a command, for example a
  GitHub Actions `run` that uses `${{ github.event.issue.title }}` or
  `${{ github.head_ref }}` directly｜Critical
- GL-B3｜CI references a third-party action, orb or template without pinning it
  to a full commit SHA｜Minor｜Major｜
  When it references a branch name (for example `@main`), or the step can access
  secrets or has write permissions
- GL-B4｜CI permissions are too broad: no minimal `permissions`, `write-all`,
  checking out and running PR code in `pull_request_target`, or passing secrets
  to steps that do not need them｜Major｜Critical｜
  When untrusted code can access secrets or write permissions
- GL-B5｜A container runs as root: the Dockerfile does not end with `USER`
  switching to a non-root user, or compose sets `privileged: true`｜Major
- GL-B6｜Secrets end up in an image or logs: secrets passed in with `ENV` or
  `ARG`, files containing secrets copied with `COPY`, or secrets printed in
  scripts or CI｜Critical
- GL-B7｜Downloaded programs are executed without verification: `curl … | sh`,
  `iex (iwr …)`, or downloaded files without a checksum or signature
  check｜Major
- GL-B8｜A script runs a composed string with `eval` or
  `Invoke-Expression`｜Major｜Critical｜
  When the string contains external or user input
- GL-B9｜A newly added or updated base image, tool or package has known
  vulnerabilities｜Major

### C. Test quality

- GL-C1｜CI removes, skips or relaxes test and check steps: deleting a test
  job, adding `continue-on-error: true` or `|| true`, without explaining
  why｜Major
- GL-C2｜A new or modified script has no way to be verified: no tests, no
  dry-run option, and the PR description does not explain how to verify
  it｜Minor

### D. Architecture and design

- GL-D1｜CI config is heavily duplicated: several jobs or workflows copy the
  same steps instead of using reusable workflows, composite actions, templates
  or YAML anchors｜Minor
- GL-D2｜A Dockerfile does not separate the build and runtime environments:
  build tools, source code or caches are left in the final image (a multi-stage
  build is recommended)｜Minor
- GL-D3｜Environment-specific settings are hard-coded in an image, script or
  config file instead of being provided by environment variables or external
  config｜Major
- GL-D4｜A script mixes responsibilities: a single script combines unrelated
  work such as build, deployment and data processing, or duplicates the
  function of an existing script｜Minor

### E. Performance and resource use

- GL-E1｜The instruction order of a Dockerfile invalidates the cache: all source
  code is copied before dependencies are installed, or frequently changing
  steps come first｜Minor
- GL-E2｜An image contains unnecessary content: package caches are not cleaned
  in the same layer after installation (for example `apt-get` without
  `--no-install-recommends`, or `/var/lib/apt/lists` not removed), or
  `.dockerignore` is missing｜Minor
- GL-E3｜CI does not cache dependencies, downloads or builds the same content
  repeatedly, or its triggers are too broad (the whole pipeline runs on every
  push)｜Minor
- GL-E4｜A CI job or container has no timeout or resource limit (for example
  `timeout-minutes`, memory and CPU limits), so it can hold resources for a
  long time when it runs away｜Minor

### F. Readability and maintainability

- GL-F1｜A README lacks essential content: purpose, installation or usage,
  configuration, or the usage instructions are not updated after adding
  features, scripts or configs｜Minor
- GL-F2｜Markdown structure errors: skipped heading levels, code blocks
  without a language, or broken link or image paths｜Nit｜Minor｜
  When a broken link keeps readers from finding a required document
- GL-F3｜Non-obvious commands, flags or special handling in a script or config
  have no comment explaining why｜Minor
- GL-F4｜A PowerShell script uses aliases (for example `ls`, `%`, `?`) or
  positional parameters instead of full cmdlet and parameter names｜Nit
- GL-F5｜A base image, tool or package version is not pinned: the `latest`
  tag, no tag, or no version specified at installation｜Major
- GL-F6｜A Dockerfile does not follow conventions: `ADD` used to copy ordinary
  files or download remote files (use `COPY`, or `curl` with verification), or
  `CMD` or `ENTRYPOINT` in shell form, so the container does not receive stop
  signals｜Minor
- GL-F7｜A new setting or environment variable has no default value and is not
  documented in a template or documentation (for example `.env.example` or the
  README)｜Minor
- GL-F8｜New generated files, build artifacts or local settings are not added
  to `.gitignore`, or `.gitignore` excludes files that should be under version
  control｜Minor
- GL-F9｜A file's line endings, encoding or indentation do not match the
  project settings (`.editorconfig`, `.gitattributes`), for example a shell
  script using CRLF｜Minor｜Major｜
  When it keeps the script from running on the target platform

## Confirmations

At the start, the scoring workflow asks about the items listed here together
with any missing documents. Items that the diff or documents already answer are
not asked again.
Format: `- related codes｜what to confirm`, with multiple codes separated by
half-width commas.

- GL-A4,GL-B4｜Whether the modified CI/CD or deployment pipeline deploys to
  production and whether it can access production secrets (asked only when the
  diff modifies CI/CD or deployment configs)
- GL-B9｜Whether vulnerability scan results for the images or dependencies are
  available (asked only when the diff modifies a Dockerfile, base image or
  dependencies)

## Special rules

- GL-B9: Scored only when vulnerability scan results are provided; otherwise
  recorded as not applicable.
- GL-A1, GL-A3: Apply only to scripts that are executed directly. For library
  files loaded by other scripts with `source`, `.` or dot-sourcing, judge by
  whether the loading script already handles errors.
- GL-B1: No deduction for obvious fake values in test data or examples (for
  example `changeme`, `<token>`, `example`), or for references to environment
  variables or secret management services (for example `${DB_PASSWORD}`,
  `${{ secrets.X }}`).
- GL-B5: No deduction when the base image runs as a non-root user by default
  (for example distroless `nonroot`), or for a multi-stage build that uses root
  only in the build stage and switches users in the final stage.
- GL-F5: May be lowered to `Nit` for compose or example files that are for
  development and clearly marked as not for production.

## Judgment hints

- Find the applicable check items by file type:
  - Documentation: GL-A5, GL-F1, GL-F2.
  - CI/CD: GL-A4, GL-B2 to GL-B4, GL-B6, GL-C1, GL-D1, GL-E3, GL-E4.
  - Containers: GL-B5 to GL-B7, GL-B9, GL-D2, GL-E1, GL-E2, GL-F5, GL-F6.
  - Scripts: GL-A1 to GL-A3, GL-A6, GL-B7, GL-B8, GL-C2, GL-D4, GL-F3, GL-F4.
  - Other configs: GL-B1, GL-D3, GL-F7 to GL-F9.
- GL-B2: In GitHub Actions, untrusted input should first be put in `env:` and
  referenced in commands as an environment variable; use in `if:` conditions
  does not count.
- GL-B3: Pinning GitHub's official `actions/*` to a commit SHA is also
  recommended; when pinning a SHA, marking the corresponding version number in a
  comment is common practice.
- GL-A5, GL-A6, GL-A4, GL-C2, GL-D1, GL-D2, GL-D3, GL-D4, GL-E3, GL-E4, GL-F1,
  GL-F3, GL-F7, GL-F8 and GL-F9 require reading the whole file or its context,
  do not appear in the automatic scan candidates, and must be checked manually.
- Automatic scan candidates are only hints: for example, `FROM builder` may
  refer to an earlier build stage rather than an untagged image. Confirm each
  one against the file.
