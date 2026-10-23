# Code Review Scoring Rules

<!-- markdownlint-disable MD028 -->
<!-- markdownlint-disable MD033 -->

[English](../../README.md) | 繁體中文

[code-review-scoring](https://github.com/samuikaze/code-review-scoring)
使用的 PR Code Review 評分規則專案。

評分規則與評分流程分開存放：評分流程透過 `configs/settings.ini` 的
`[rules]` 區段 `root` 找到本專案，讀取 `rules.ini` 登錄的規則，
規則的更新不需要修改評分流程。

> [!IMPORTANT]
> 規則不需要建立多語言版本。評分流程只讀取 `rules.ini` 為每套規則登錄的那一個目錄，
> 不會依語言切換。本專案的 `zh-tw/`、`en/` 目錄只是示範，
> 自己的規則只要用一種語言撰寫即可，見「規則檔的語言」。

## 目錄結構

```text
code-review-scoring-rules/
  rules.ini                              規則登錄（各規則的目錄與是否啟用）
  defaults.ini                           預設的面向配分、扣分合併方式、嚴重度分級與等級
  springboot3-rule/
    zh-tw/
      rule.ini                           規則設定：標準文件、掃描規則、applies
      springboot-3-cr-scoring-rule.md    Spring Boot 3 標準文件（繁體中文）
      springboot-3-cr-scoring-rule.tsv   Spring Boot 3 的掃描規則
    en/                                  同一套規則的英文版（檔案同上）
  common-rule/
    zh-tw/
      rule.ini                           規則設定：標準文件、掃描規則、role="common"
      common-cr-scoring-rule.md          通用標準文件（繁體中文）
      common-cr-scoring-rule.tsv         通用標準的掃描規則
    en/                                  同一套規則的英文版（檔案同上）
  docs/
    zh-tw/README.md                      繁體中文 README
```

## 規則一覽

| 規則名稱 | 類型 | 適用範圍 | 繁體中文 | 英文 |
| --- | --- | --- | --- | --- |
| `springboot-3` | 框架規則 | Java／Spring Boot 3 專案 | [springboot-3-cr-scoring-rule.md](../../springboot3-rule/zh-tw/springboot-3-cr-scoring-rule.md) | [springboot-3-cr-scoring-rule.md](../../springboot3-rule/en/springboot-3-cr-scoring-rule.md) |
| `common` | 通用規則（`role="common"`） | 不屬於任何框架規則的檔案，例如 README、CI／CD 設定、Dockerfile、腳本、JSON／YAML 設定 | [common-cr-scoring-rule.md](../../common-rule/zh-tw/common-cr-scoring-rule.md) | [common-cr-scoring-rule.md](../../common-rule/en/common-cr-scoring-rule.md) |

`rules.ini` 預設登錄繁體中文版的目錄。規則的每個語言目錄都是獨立的規則目錄，以該語言撰寫，包含：

- **`rule.ini`**：規則設定，註解以所在目錄的語言撰寫。`[rule]` 指定標準文件（`standard`）、
  掃描規則檔（`scan`）與框架規則適用的檔案（`applies`），通用規則則設定
  `role="common"`；路徑相對於規則目錄。框架規則也可以覆寫 `defaults.ini` 的配分、
  扣分合併方式、嚴重度分級與等級；通用規則只能覆寫扣分合併方式與嚴重度分級（不含 `cap`），
  配分與等級一律使用 `defaults.ini`。
- **標準文件**（`.md`）：定義面向、檢查項、事先確認事項等內容。
  前置資料的 `name` 為規則名稱，須與 `rules.ini` 的區段名稱一致；
  `version` 為標準版本，會記錄在評分報告中。
- **掃描規則檔**（`.tsv`）：以正規表示式列出候選線索，只作為提示，不會直接扣分。

## 設定檔

設定檔一律為 INI 格式，值以雙引號包住，`;` 或 `#` 開頭的行是註解。

- **`rules.ini`**：登錄規則，每套規則一個區段。區段名稱為規則名稱，
  須與標準文件前置資料的 `name` 一致。兩個鍵都必填：
  - `path`：規則目錄（`rule.ini` 所在的目錄），完整路徑或相對於 `rules.ini` 所在目錄的路徑。
    本專案的規則也以它選擇語言：`zh-tw` 或 `en`（見「規則檔的語言」）。
  - `enable`：`"true"` 或 `"false"`。停用的規則不參與評分，`standard lint` 仍會檢查。
    啟用中的通用規則（`role="common"`）最多一套。

  ```ini
  [springboot-3]
  path="springboot3-rule/zh-tw"
  enable="true"
  ```

- **`defaults.ini`**：`[scoring]`、`[points]`、`[severity:代碼]` 與 `[grade:代碼]`，
  `rule.ini` 沒有設定的部分使用這裡的設定。取代以區塊群組為單位：
  `rule.ini` 有任何 `[severity:*]` 就整組取代嚴重度分級，有任何 `[grade:*]` 就整組取代等級。
- **`rule.ini`**：各區段與鍵的說明見 code-review-scoring 的規則設定模板
  （[繁體中文](https://github.com/samuikaze/code-review-scoring/blob/main/templates/zh-tw/rule-template.ini)、
  [英文](https://github.com/samuikaze/code-review-scoring/blob/main/templates/en/rule-template.ini)）。

預設的嚴重度分級中，必須修正的問題沿用 SonarQube 傳統嚴重度的 Critical、Major、Minor，
風格層面的小問題沿用 Google Code Review 指引的 Nit。預設等級為
A（95）、B（85）、C（70）、D（60）、F（0），有 Critical 扣分項時等級上限為 B。

只有在有明確理由時才覆寫規則的分級，並把理由寫在該規則的標準文件中。
嚴重度與等級的代碼不隨語言改變；報告上要顯示其他名稱時，以 `name` 設定顯示名稱。

## 規則檔的語言

規則檔不需要做 i18n。評分流程只讀取 `rules.ini` 登錄的那一個規則目錄，
不會依語言切換，所以依自己喜歡或團隊使用的語言建立一份規則即可，
不需要同時維護多種語言的版本。

本專案的 `zh-tw/` 與 `en/` 是同一套規則的兩種語言版本，
檢查項代碼、嚴重度與事先確認事項完全相同，用來示範兩種模板。
兩個目錄各有自己的 `rule.ini`，目錄中的所有內容（包含 `rule.ini` 的註解）都以該目錄的語言撰寫。
要把規則換成英文版時，把 `rules.ini` 中該規則 `path` 的 `zh-tw` 換成 `en`：

```ini
[springboot-3]
path="springboot3-rule/en"
enable="true"
```

撰寫其他語言的規則檔時請注意：

- 由腳本解析的章節名稱與面向名稱，目前只認得繁體中文或英文，
  例如「面向」或 `Dimensions`、「檢查項」或 `Check items`、
  「事先確認事項」或 `Confirmations`、「正確性」或 `Correctness`。
  其餘內容（檢查項說明、特例規則、判斷提示等）可以使用任何語言。
- 檢查項代碼、嚴重度代碼（預設為 `Critical`、`Major`、`Minor`、`Nit`）
  與全形分隔符號 `｜` 維持原樣，不翻譯。
- 掃描規則檔的 `# source:` 行記錄的是對應標準文件的 sha256，
  每種語言的標準文件都要搭配各自蓋過 source 行的掃描規則檔。

## 使用方式

1. 把本專案 clone 到本機，位置不限：

   ```bash
   git clone https://github.com/samuikaze/code-review-scoring-rules.git
   ```

2. 在 code-review-scoring 的 `configs/settings.ini` 加上 `[rules]` 區段，
   `root` 指向 clone 下來的目錄（完整路徑，或相對於 `configs/` 的路徑；
   Windows 請使用斜線）：

   ```ini
   [rules]
   root="/path/to/code-review-scoring-rules"
   ```

3. 要改用英文版時，把 `rules.ini` 中各規則 `path` 的 `zh-tw` 換成 `en`
   （見「規則檔的語言」）。
4. 在 code-review-scoring 專案根目錄執行 `standard lint` 檢查規則（見下方）。

`settings.ini` 不進版控，換電腦或換人時請各自設定 `root`。

## 新增或修改規則

規則的撰寫與檢查一律從 code-review-scoring 專案執行：
本專案只放規則與設定，模板、腳本與 `code-review-scoring-standard` Skill 都在評分專案中。

1. 為規則建立一個目錄（例如 `<規則名稱>-rule/`），放入依 code-review-scoring 的規則設定模板
   （[繁體中文](https://github.com/samuikaze/code-review-scoring/blob/main/templates/zh-tw/rule-template.ini)、
   [英文](https://github.com/samuikaze/code-review-scoring/blob/main/templates/en/rule-template.ini)）
   撰寫的 `rule.ini`，以及依標準文件模板
   （[繁體中文](https://github.com/samuikaze/code-review-scoring/blob/main/templates/zh-tw/scoring-rule-template.md)、
   [英文](https://github.com/samuikaze/code-review-scoring/blob/main/templates/en/scoring-rule-template.md)）撰寫的標準文件。
   只寫一種語言即可，見「規則檔的語言」。
2. 在 `rules.ini` 登錄這套規則，設定 `path` 與 `enable="true"`。
3. 標準文件的「面向」「檢查項」「事先確認事項」三個章節由腳本解析，格式固定；
   「特例規則」「判斷提示」只給 AI 參考，可以自由撰寫。
4. 修改標準文件後，請同步更新前置資料的 `version`。
   本專案已提供多種語言版本的規則，修改其中一種時請一併更新其他語言版本。
5. 需要掃描規則檔時，可在 code-review-scoring 中透過
   [`code-review-scoring-standard`](https://github.com/samuikaze/code-review-scoring/blob/main/skills/code-review-scoring-standard/SKILL.md)
   Skill 產生或更新，不需手寫正規表示式。
6. 在 code-review-scoring 專案根目錄執行 lint，檢查 `rules.ini`、`defaults.ini`、
   各規則的 `rule.ini`、標準文件與掃描規則：

   ```bash
   bash skills/code-review-scoring/scripts/standard.sh lint
   ```

   ```powershell
   powershell -ExecutionPolicy Bypass `
     -File skills\code-review-scoring\scripts\standard.ps1 lint
   ```

## 注意事項

- 評分途中不要修改規則。評分流程會在步驟 1 記錄標準文件、掃描規則檔、
  `rule.ini` 與 `defaults.ini` 的 sha256，之後被修改時流程會拒絕繼續。
- 掃描規則檔開頭的 `# source:` 行記錄來源標準文件的版本與 sha256，
  標準文件修改後請重新產生或審核掃描規則檔。
- 標準文件須通過 markdownlint，僅停用 MD028 與 MD033。
