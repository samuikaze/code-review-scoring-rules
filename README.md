# Code Review Scoring Rules

<!-- markdownlint-disable MD028 -->
<!-- markdownlint-disable MD033 -->

English | [繁體中文](docs/zh-tw/README.md)

A rules project of PR code review scoring rules for
[code-review-scoring](https://github.com/samuikaze/code-review-scoring).

The scoring rules are kept separate from the scoring workflow: the workflow
finds this project through `root` in the `[rules]` section of its
`configs/settings.ini`, and reads the rules registered in `rules.ini` here.
Updating a rule requires no change to the workflow itself.

> [!IMPORTANT]
> Rules do not need versions in several languages. The workflow reads only
> the one rule folder `rules.ini` registers for a rule, and never switches by
> language. The `zh-tw/` and `en/` folders here
> are only a demonstration; write your own rules in one language. See
> [Language of rule files](#language-of-rule-files).

## Directory structure

```text
code-review-scoring-rules/
  rules.ini                              Registered rules: folder, enable
  defaults.ini                           Default points, scale and grades
  springboot3-rule/
    zh-tw/
      rule.ini                           Rule settings: document, scan rules, applies
      springboot-3-cr-scoring-rule.md    Spring Boot 3 standard (Traditional Chinese)
      springboot-3-cr-scoring-rule.tsv   Scan rules for Spring Boot 3
    en/                                  The same rule in English (same files)
  global-rule/
    zh-tw/
      rule.ini                           Rule settings: document, scan rules, role
      global-cr-scoring-rule.md          Global standard (Traditional Chinese)
      global-cr-scoring-rule.tsv         Scan rules for the global standard
    en/                                  The same rule in English (same files)
  docs/
    zh-tw/README.md                      Traditional Chinese README
```

## Available rules

| Rule | Type | Applies to | Traditional Chinese | English |
| --- | --- | --- | --- | --- |
| `springboot-3` | Framework rule | Java/Spring Boot 3 projects | [springboot-3-cr-scoring-rule.md](springboot3-rule/zh-tw/springboot-3-cr-scoring-rule.md) | [springboot-3-cr-scoring-rule.md](springboot3-rule/en/springboot-3-cr-scoring-rule.md) |
| `global` | Global rule (`role="global"`) | Global rules: every file of every report. Check items: files not covered by any framework rule, such as READMEs, CI/CD configs, Dockerfiles, scripts and JSON/YAML configs | [global-cr-scoring-rule.md](global-rule/zh-tw/global-cr-scoring-rule.md) | [global-cr-scoring-rule.md](global-rule/en/global-cr-scoring-rule.md) |

`rules.ini` registers the Traditional Chinese folders. Each language folder
of a rule is a rule folder of its own, written in that language, and holds:

- **`rule.ini`**: the rule settings, with comments in the language of its
  folder. `[rule]` names the standard document
  (`standard`), the scan rule file (`scan`) and the files a framework rule
  applies to (`applies`), or `role="global"` for the global rule; the paths
  are relative to the rule folder. A framework rule may also override the
  points, deduction merging, severity scale and grades of `defaults.ini`. The
  global rule may override only the deduction merging and the severity scale
  (without `cap`): its points and grades always come from `defaults.ini`.
- **Standard document** (`.md`): the dimensions, check items, confirmations
  and so on. The `name` field in the front matter is the rule name and must
  equal its section in `rules.ini`; `version` is recorded in the scoring
  report. The global standard also has a section Global rules
  (`## 全域規則` or `## Global rules`) with the rules that apply to every
  file of every report; where a framework standard says otherwise, the
  framework standard wins.
- **Scan rule file** (`.tsv`): candidate hints found by regular expressions.
  These are hints only and never deduct points directly.

## Settings

All settings files are INI files; values are in double quotes, and lines
starting with `;` or `#` are comments.

- **`rules.ini`**: registers the rules, one section per rule. The section
  name is the rule name and must equal `name` in the front matter of its
  standard document. Both keys are required:
  - `path`: the rule folder (the folder of its `rule.ini`), as a full path or
    relative to the folder of `rules.ini`. For the rules of this project it
    also picks the language: `zh-tw` or `en` (see
    [Language of rule files](#language-of-rule-files)).
  - `enable`: `"true"` or `"false"`. Disabled rules are not used for scoring,
    but `standard lint` still checks them. At most one global rule
    (`role="global"`) can be enabled.

  ```ini
  [springboot-3]
  path="springboot3-rule/zh-tw"
  enable="true"
  ```

- **`defaults.ini`**: `[scoring]`, `[points]`, `[severity:CODE]` and
  `[grade:CODE]`, used by every rule whose `rule.ini` does not set them. A
  group in `rule.ini` replaces the whole group of `defaults.ini`: any
  `[severity:*]` section replaces the whole scale, any `[grade:*]` section
  replaces all grades.
- **`rule.ini`**: see the rule settings template of code-review-scoring
  ([English](https://github.com/samuikaze/code-review-scoring/blob/main/templates/en/rule-template.ini),
  [Traditional Chinese](https://github.com/samuikaze/code-review-scoring/blob/main/templates/zh-tw/rule-template.ini))
  for every section and key.

The default severity scale uses the classic SonarQube severities Critical,
Major and Minor for problems that must be fixed, and Nit from the Google code
review guide for small style issues. The default grades are A (95), B (85),
C (70), D (60) and F (0); a Critical deduction caps the grade at B.

Override the scale of a rule only when there is a clear reason, and write the
reason in its standard document. Severity and grade codes stay as they are in
every language; give them a display name with `name` when the report should
show another one.

## Language of rule files

Rule files do not need i18n. The scoring workflow reads only the one rule
folder registered in `rules.ini` and never switches by language, so write a
single rule in whatever language you or your team prefer. There is no need to
maintain versions in several languages.

The `zh-tw/` and `en/` folders in this project are two language versions of
the same rules, with identical check codes, severities and confirmations, to
show both templates. Each has its own `rule.ini`, and everything in a folder,
including the comments of `rule.ini`, is written in the language of the
folder. To switch a rule to English, change `zh-tw` to `en` in its `path` in
`rules.ini`:

```ini
[springboot-3]
path="springboot3-rule/en"
enable="true"
```

When writing rule files in another language, note that:

- The section and dimension names parsed by the scripts are currently
  recognized only in Traditional Chinese or English, for example `面向` or
  `Dimensions`, `檢查項` or `Check items`, `事先確認事項` or `Confirmations`,
  and `正確性` or `Correctness`. Everything else (check item descriptions,
  special rules, judgment hints and so on) can be in any language.
- Check codes, severity codes (by default `Critical`, `Major`, `Minor` and
  `Nit`) and the full-width separator `｜` stay as they are and are not
  translated.
- The `# source:` line of a scan rule file records the sha256 of its standard
  document, so each language version of a rule needs its own scan rule file
  stamped against it.

## Usage

1. Clone this project anywhere on your machine:

   ```bash
   git clone https://github.com/samuikaze/code-review-scoring-rules.git
   ```

2. Point `root` in the `[rules]` section of `configs/settings.ini` of
   code-review-scoring to the cloned folder; see
   [the `[rules]` section](https://github.com/samuikaze/code-review-scoring/blob/main/docs/en/configurations.md#the-rules-section)
   of its configuration guide.

3. To use the English rules, change `zh-tw` to `en` in the `path` of each
   rule in `rules.ini` (see
   [Language of rule files](#language-of-rule-files)).
4. Run `standard lint` in the root of code-review-scoring to check the rules
   (see below).

## Adding or modifying rules

Change the rules from the code-review-scoring project, not from here: this
project holds only the rules and their settings, and the templates, scripts
and skills stay there.

- **With an agent**: ask an agent in code-review-scoring to add or change a
  rule. Its
  [`code-review-scoring-rule-authoring`](https://github.com/samuikaze/code-review-scoring/blob/main/skills/code-review-scoring-rule-authoring/SKILL.md)
  skill shows every change, including the other language folders of the rule,
  writes it here only after you confirm, updates the scan rules and runs lint.
  See
  [Changing the rules](https://github.com/samuikaze/code-review-scoring#changing-the-rules).
- **By hand**: write the files from the templates of code-review-scoring (see
  [Scoring standards](https://github.com/samuikaze/code-review-scoring#scoring-standards)),
  bump `version` in the front matter of every changed document, generate
  the scan rules with its `code-review-scoring-standard` skill, and run
  `standard lint` there.

Neither way commits: review and commit the changes in this project yourself.

## Notes

- Do not modify the rules while a scoring run is in progress. The workflow
  records the sha256 of each standard document, scan rule file, `rule.ini`
  and `defaults.ini` in step 1 and refuses to continue if they change
  afterwards.
- The `# source:` line at the top of a scan rule file records the version and
  sha256 of its source standard document. After modifying a standard document,
  regenerate or review its scan rule file.
- Standard documents must pass markdownlint, with only MD028 and MD033
  disabled.
