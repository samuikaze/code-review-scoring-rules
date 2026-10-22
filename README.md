# Code Review Scoring Rules

<!-- markdownlint-disable MD028 -->
<!-- markdownlint-disable MD033 -->

English | [繁體中文](docs/zh-tw/README.md)

A repository of PR code review scoring standards used by
[code-review-scoring](https://github.com/samuikaze/code-review-scoring).

The scoring standards are kept separate from the scoring workflow: the workflow
reads the standard documents and scan rule files in this repository only through
`configs/standards.ini`, so updating a standard requires no change to the
workflow itself.

## Directory Structure

```text
code-review-scoring-rules/
  common-rule/
    common-cr-scoring-rule.md          Common standard document
    common-cr-scoring-rule.tsv         Scan rules for the common standard
  springboot3-rule/
    springboot-3-cr-scoring-rule.md    Spring Boot 3 standard document
    springboot-3-cr-scoring-rule.tsv   Scan rules for Spring Boot 3
  docs/
    zh-tw/README.md                    Traditional Chinese README
```

## Available Standards

| Standard | Type | Applies to | Document |
| --- | --- | --- | --- |
| `common` | Common standard (`role="common"`) | Files not covered by any framework standard, such as READMEs, CI/CD configs, Dockerfiles, scripts and JSON/YAML configs | [common-cr-scoring-rule.md](common-rule/common-cr-scoring-rule.md) |
| `springboot-3` | Framework standard | Java/Spring Boot 3 projects | [springboot-3-cr-scoring-rule.md](springboot3-rule/springboot-3-cr-scoring-rule.md) |

Each standard consists of two files:

- **Standard document** (`.md`): defines the aspects, check items, items to
  confirm in advance and so on. The `name` field in the front matter is the
  standard ID, and `version` is the standard version, which is recorded in the
  scoring report.
- **Scan rule file** (`.tsv`): lists candidate hints found by regular
  expressions. These are hints only and never deduct points directly.

The standard documents themselves are written in Traditional Chinese.

## Usage

In the root of the code-review-scoring project, copy the config template:

```bash
cp configs/standards.example.ini configs/standards.ini
```

Add one section per standard to `configs/standards.ini`, pointing `path` and
`scan` to the files in this repository. Relative paths are resolved against
`configs/`. On Windows, use forward slashes:

```ini
[springboot-3]
path="/path/to/code-review-scoring-rules/springboot3-rule/springboot-3-cr-scoring-rule.md"
scan="/path/to/code-review-scoring-rules/springboot3-rule/springboot-3-cr-scoring-rule.tsv"
applies="*.java, *.kt, *.sql, *.xml, *.yml, *.yaml, *.properties, *.gradle, *.kts"

[common]
path="/path/to/code-review-scoring-rules/common-rule/common-cr-scoring-rule.md"
scan="/path/to/code-review-scoring-rules/common-rule/common-cr-scoring-rule.tsv"
role="common"
```

`standards.ini` is not under version control, so set up the paths on each
machine or for each user. See `configs/standards.example.ini` in
code-review-scoring for a full description of every field.

## Adding or Modifying Standards

1. Write the standard document based on
   `docs/templates/scoring-rule-template.md` in code-review-scoring. Put each
   new standard in its own `<standard-id>-rule/` directory.
2. The "aspects", "check items" and "items to confirm in advance" sections are
   parsed by scripts and must follow the fixed format. The "special rules" and
   "judgment hints" sections are for AI reference only and can be written
   freely.
3. Bump the `version` field in the front matter whenever you modify a standard
   document.
4. If a scan rule file is needed, generate or update it with the
   `code-review-scoring-standard` Skill in code-review-scoring instead of
   writing regular expressions by hand.
5. Run lint in the root of the code-review-scoring project to check the format,
   overlapping `applies` patterns, point totals and scan rules:

   ```bash
   bash skills/code-review-scoring/scripts/standard.sh lint
   ```

   ```powershell
   powershell -ExecutionPolicy Bypass `
     -File skills\code-review-scoring\scripts\standard.ps1 lint
   ```

## Notes

- Do not modify a standard while a scoring run is in progress. The workflow
  records the sha256 of each standard in step 1 and refuses to continue if the
  standard or its scan rules change afterwards.
- The first line of a scan rule file records the version and sha256 of its
  source standard document. After modifying a standard document, regenerate or
  review its scan rule file.
- Standard documents must pass markdownlint, with only MD028 and MD033
  disabled.
