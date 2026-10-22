# Code Review Scoring Rules

<!-- markdownlint-disable MD028 -->
<!-- markdownlint-disable MD033 -->

[English](../../README.md) | 繁體中文

[code-review-scoring](https://github.com/samuikaze/code-review-scoring)
使用的 PR Code Review 評分標準文件庫。

評分標準與評分流程分開存放：評分流程只透過 `configs/standards.ini`
讀取本儲存庫的標準文件與掃描規則檔，標準的更新不需要修改評分流程。

## 目錄結構

```text
code-review-scoring-rules/
  common-rule/
    common-cr-scoring-rule.md          通用標準文件
    common-cr-scoring-rule.tsv         通用標準的掃描規則
  springboot3-rule/
    springboot-3-cr-scoring-rule.md    Spring Boot 3 標準文件
    springboot-3-cr-scoring-rule.tsv   Spring Boot 3 的掃描規則
  docs/
    zh-tw/README.md                    繁體中文 README
```

## 評分標準一覽

| 標準代號 | 類型 | 適用範圍 | 文件 |
| --- | --- | --- | --- |
| `common` | 通用標準（`role="common"`） | 不屬於任何框架標準的檔案，例如 README、CI／CD 設定、Dockerfile、腳本、JSON／YAML 設定 | [common-cr-scoring-rule.md](../../common-rule/common-cr-scoring-rule.md) |
| `springboot-3` | 框架標準 | Java／Spring Boot 3 專案 | [springboot-3-cr-scoring-rule.md](../../springboot3-rule/springboot-3-cr-scoring-rule.md) |

每套標準由兩個檔案組成：

- **標準文件**（`.md`）：定義面向、檢查項、事先確認事項等內容，
  前置資料的 `name` 為標準代號，`version` 為標準版本，版本會記錄在評分報告中。
- **掃描規則檔**（`.tsv`）：以正規表示式列出候選線索，只作為提示，不會直接扣分。

## 使用方式

在 code-review-scoring 專案根目錄複製設定檔範本：

```bash
cp configs/standards.example.ini configs/standards.ini
```

在 `configs/standards.ini` 中為每套標準建立一個區段，
`path` 與 `scan` 指向本儲存庫的檔案。相對路徑以 `configs/` 為準，
Windows 請使用斜線：

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

`standards.ini` 不進版控，換電腦或換人時請各自設定路徑。
各欄位的完整說明見 code-review-scoring 的 `configs/standards.example.ini`。

## 新增或修改評分標準

1. 依 code-review-scoring 的 `docs/templates/scoring-rule-template.md`
   撰寫標準文件，新標準請建立 `<標準代號>-rule/` 目錄存放。
2. 「面向」「檢查項」「事先確認事項」三個章節由腳本解析，格式固定；
   「特例規則」「判斷提示」只給 AI 參考，可以自由撰寫。
3. 修改標準文件後，請同步更新前置資料的 `version`。
4. 需要掃描規則檔時，可在 code-review-scoring 中透過
   `code-review-scoring-standard` Skill 產生或更新，不需手寫正規表示式。
5. 在 code-review-scoring 專案根目錄執行 lint，確認格式、`applies`
   是否重疊、配分加總與掃描規則皆正確：

   ```bash
   bash skills/code-review-scoring/scripts/standard.sh lint
   ```

   ```powershell
   powershell -ExecutionPolicy Bypass `
     -File skills\code-review-scoring\scripts\standard.ps1 lint
   ```

## 注意事項

- 評分途中不要修改標準。評分流程會在步驟 1 記錄標準的 sha256，
  之後標準或掃描規則被修改時流程會拒絕繼續。
- 掃描規則檔第一行記錄來源標準文件的版本與 sha256，
  標準文件修改後請重新產生或審核掃描規則檔。
- 標準文件須通過 markdownlint，僅停用 MD028 與 MD033。
