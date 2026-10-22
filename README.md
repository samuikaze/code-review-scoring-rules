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
    zh-tw/
      common-cr-scoring-rule.md          Common standard (Traditional Chinese)
      common-cr-scoring-rule.tsv         Scan rules for the common standard
    en/
      common-cr-scoring-rule.md          Common standard (English)
      common-cr-scoring-rule.tsv         Scan rules for the common standard
  springboot3-rule/
    zh-tw/
      springboot-3-cr-scoring-rule.md    Spring Boot 3 standard (Traditional Chinese)
      springboot-3-cr-scoring-rule.tsv   Scan rules for Spring Boot 3
    en/
      springboot-3-cr-scoring-rule.md    Spring Boot 3 standard (English)
      springboot-3-cr-scoring-rule.tsv   Scan rules for Spring Boot 3
  docs/
    zh-tw/README.md                      Traditional Chinese README
```

## Available Standards

| Standard | Type | Applies to | Traditional Chinese | English |
| --- | --- | --- | --- | --- |
| `common` | Common standard (`role="common"`) | Files not covered by any framework standard, such as READMEs, CI/CD configs, Dockerfiles, scripts and JSON/YAML configs | [common-cr-scoring-rule.md](common-rule/zh-tw/common-cr-scoring-rule.md) | [common-cr-scoring-rule.md](common-rule/en/common-cr-scoring-rule.md) |
| `springboot-3` | Framework standard | Java/Spring Boot 3 projects | [springboot-3-cr-scoring-rule.md](springboot3-rule/zh-tw/springboot-3-cr-scoring-rule.md) | [springboot-3-cr-scoring-rule.md](springboot3-rule/en/springboot-3-cr-scoring-rule.md) |

Each standard consists of two files:

- **Standard document** (`.md`): defines the dimensions, check items,
  confirmations and so on. The `name` field in the front matter is the standard
  ID, and `version` is the standard version, which is recorded in the scoring
  report.
- **Scan rule file** (`.tsv`): lists candidate hints found by regular
  expressions. These are hints only and never deduct points directly.

## Language of Rule Files

Rule files do not need i18n. The scoring workflow reads only the one standard
document registered in `configs/standards.ini` and never switches by language,
so just write a single rule file in whatever language you or your team prefer.
There is no need to maintain versions in several languages.

The `zh-tw/` and `en/` directories in this repository are simply two language
versions of the same standards, with identical check codes, severities and
confirmations. Register either one.

When writing rule files in another language, note that:

- The section and dimension names parsed by the scripts are currently
  recognized only in Traditional Chinese or English, for example `面向` or
  `Dimensions`, `檢查項` or `Check items`, `事先確認事項` or `Confirmations`,
  and `正確性` or `Correctness`. Everything else (check item descriptions,
  special rules, judgment hints and so on) can be in any language.
- Check codes, severities (`Critical`, `Major`, `Minor`, `Nit`) and the
  full-width separator `｜` stay as they are and are not translated.
- The `# source:` line of a scan rule file records the sha256 of its standard
  document, so each language version of a standard needs its own scan rule file
  stamped against it.

## Usage

In the root of the code-review-scoring project, copy the config template:

```bash
cp configs/standards.example.ini configs/standards.ini
```

Add one section per standard to `configs/standards.ini`, pointing `path` and
`scan` to the files in the language directory of your choice in this
repository. Relative paths are resolved against `configs/`. On Windows, use
forward slashes:

```ini
[springboot-3]
path="/path/to/code-review-scoring-rules/springboot3-rule/en/springboot-3-cr-scoring-rule.md"
scan="/path/to/code-review-scoring-rules/springboot3-rule/en/springboot-3-cr-scoring-rule.tsv"
applies="*.java, *.kt, *.sql, *.xml, *.yml, *.yaml, *.properties, *.gradle, *.kts"

[common]
path="/path/to/code-review-scoring-rules/common-rule/en/common-cr-scoring-rule.md"
scan="/path/to/code-review-scoring-rules/common-rule/en/common-cr-scoring-rule.tsv"
role="common"
```

To use the Traditional Chinese version, replace `en` with `zh-tw` in the paths.

`standards.ini` is not under version control, so set up the paths on each
machine or for each user. See `configs/standards.example.ini` in
code-review-scoring for a full description of every field.

## Adding or Modifying Standards

1. Write the standard document based on
   `docs/templates/scoring-rule-template.md` in code-review-scoring. Put each
   new standard in its own `<standard-id>-rule/<language>/` directory. A single
   language is enough; see [Language of Rule Files](#language-of-rule-files).
2. The "Dimensions", "Check items" and "Confirmations" sections are parsed by
   scripts and must follow the fixed format. The "Special rules" and "Judgment
   hints" sections are for AI reference only and can be written freely.
3. Bump the `version` field in the front matter whenever you modify a standard
   document. For standards this repository provides in several languages,
   update the other language versions as well.
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
- The `# source:` line at the top of a scan rule file records the version and
  sha256 of its source standard document. After modifying a standard document,
  regenerate or review its scan rule file.
- Standard documents must pass markdownlint, with only MD028 and MD033
  disabled.
