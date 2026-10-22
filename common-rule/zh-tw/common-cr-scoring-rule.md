---
# 標準代號：只能使用英文小寫、數字與 -，須與 configs/standards.ini 的區段名稱一致
name: common
# 標準版本，會記錄在評分報告中
version: v0.1
---

# 通用檔案 PR 品質評分規則

<!-- markdownlint-disable MD028 -->
<!-- markdownlint-disable MD033 -->

適用於不屬於任何框架標準的檔案：

- 文件：README、Markdown 等說明文件。
- CI／CD：GitHub Actions、GitLab CI、Azure Pipelines 等流程設定。
- 容器：Dockerfile、docker-compose。
- 腳本：shell（sh、bash、zsh）、PowerShell。
- 其他設定：JSON、YAML、`.env` 範本、`.gitignore`、`.editorconfig`、`.gitattributes` 等。

檢查項依社群常用的工具與指南整理：markdownlint、GitHub Actions 安全強化指南、
Docker 官方的 Dockerfile 最佳實踐與 hadolint、ShellCheck、PSScriptAnalyzer、
OWASP Secrets Management Cheat Sheet、The Twelve-Factor App。

本文件透過 `configs/standards.ini` 外掛到 code-review-scoring 的評分流程，
作為通用標準（`role="common"`）；自動掃描規則在同目錄的 `common-cr-scoring-rule.tsv`：

```ini
[common]
path="/path/to/common-cr-scoring-rule.md"
scan="/path/to/common-cr-scoring-rule.tsv"
role="common"
```

PR 同時異動框架檔案時，通用檔案的扣分項併入所屬專案的框架報告，扣在相同的 A～G 面向；
PR 只異動通用檔案時，以本標準單獨產出一份報告。
評分範圍、需要證據才能扣分、上游因素等通用規則由評分流程處理，不在本文件中。

## 面向

所有標準共用以下七個面向。配分預設使用 `configs/settings.ini` 的 `[points]`，
可在 `configs/standards.ini` 的本標準區段覆寫（只在本標準單獨產出報告時使用）。
檢查代碼 `CM-` 之後的第一個字母代表所屬面向。

| 代碼 | 面向 | 衡量重點 |
| --- | --- | --- |
| A | 正確性 | 腳本與設定是否照預期運作、失敗時是否停止 |
| B | 安全性 | 機密、權限、供應鏈與不受信任的輸入 |
| C | 測試品質 | 測試與檢查是否仍然執行、變更是否可以驗證 |
| D | 架構與設計 | 職責劃分、重複、設定與環境分離 |
| E | 效能與資源使用 | 建置快取、映像檔大小、執行資源與逾時 |
| F | 可讀性 | 文件是否完整正確、腳本是否容易看懂 |
| G | 可維護性 | 版本固定、慣用寫法、設定說明與版控設定 |

## 檢查項

每個檢查項寫成一個清單項目，欄位以全形 `｜` 分隔：

- 不可調整嚴重度：`- 代碼｜內容｜預設嚴重度`
- 可調整嚴重度：`- 代碼｜內容｜預設嚴重度｜可調整為｜調整條件`

只有寫明的條件成立時才可以調整嚴重度。任何檢查項都可以降為 `Nit`。

### A. 正確性

- CM-A1｜shell 腳本失敗時不會停止：沒有 `set -e`（建議 `set -euo pipefail`），
  也沒有逐一檢查指令的結束碼｜Major
- CM-A2｜shell 腳本的變數展開沒有加引號，值含空白或萬用字元時會出錯，
  例如 `rm -rf $DIR/*`｜Major｜Critical｜
  展開結果用於刪除、覆寫或搬移檔案時
- CM-A3｜PowerShell 腳本沒有處理錯誤：沒有設定 `$ErrorActionPreference = 'Stop'`，
  也沒有檢查外部程式的 `$LASTEXITCODE`，或把錯誤全部設為忽略｜Major
- CM-A4｜CI／CD 或部署設定有錯：引用不存在的檔案、變數、Job 或 Stage，
  觸發條件或執行順序和預期不同｜Major｜Critical｜
  影響正式環境的部署或執行時
- CM-A5｜文件內容和本次的程式或設定變更不一致：指令、參數、路徑、
  環境變數或版本已經過時｜Minor｜Major｜
  文件是安裝、部署或維運操作的步驟說明時
- CM-A6｜腳本依賴特定環境卻沒有檢查或說明：寫死絕對路徑、假設目前目錄、
  使用 bash 才有的語法卻以 `sh` 執行｜Minor

### B. 安全性

- CM-B1｜檔案中寫死密碼、Token、金鑰或含密碼的連線字串，
  或把含機密的檔案（例如 `.env`、私鑰）加入版控｜Critical
- CM-B2｜CI 把不受信任的輸入直接插入指令，例如 GitHub Actions 的 `run`
  直接使用 `${{ github.event.issue.title }}`、`${{ github.head_ref }}`｜Critical
- CM-B3｜CI 引用第三方 Action、Orb 或範本沒有固定到完整的 commit SHA｜Minor｜Major｜
  引用分支名稱（例如 `@main`），或該步驟能取得機密或寫入權限時
- CM-B4｜CI 權限過大：沒有設定最小的 `permissions`、使用 `write-all`，
  在 `pull_request_target` 中簽出並執行 PR 的程式碼，
  或把機密傳給不需要的步驟｜Major｜Critical｜
  不受信任的程式碼能取得機密或寫入權限時
- CM-B5｜容器以 root 執行：Dockerfile 最後沒有以 `USER` 切換到非 root 使用者，
  或 compose 設定 `privileged: true`｜Major
- CM-B6｜機密進入映像檔或紀錄：以 `ENV`、`ARG` 傳入機密、`COPY` 含機密的檔案，
  或在腳本、CI 中印出機密｜Critical
- CM-B7｜下載後直接執行沒有驗證的程式：`curl … | sh`、`iex (iwr …)`，
  或下載的檔案沒有驗證 checksum 或簽章｜Major
- CM-B8｜腳本以 `eval`、`Invoke-Expression` 執行組合出來的字串｜Major｜Critical｜
  字串含有外部或使用者輸入時
- CM-B9｜新增或更新的基底映像檔、工具或套件有已知弱點｜Major

### C. 測試品質

- CM-C1｜CI 移除、略過或放寬測試與檢查步驟：刪除測試 Job、
  加上 `continue-on-error: true` 或 `|| true`，而且沒有說明原因｜Major
- CM-C2｜新增或修改的腳本沒有驗證方式：沒有測試、沒有 dry-run 選項，
  PR 描述也沒有說明如何驗證｜Minor

### D. 架構與設計

- CM-D1｜CI 設定大量重複：多個 Job 或 Workflow 複製相同步驟，
  沒有使用可重用的 workflow、composite action、範本或 YAML anchor｜Minor
- CM-D2｜Dockerfile 沒有分離建置與執行環境：建置工具、原始碼或快取留在最終映像檔
  （建議使用 multi-stage build）｜Minor
- CM-D3｜和環境相關的設定寫死在映像檔、腳本或設定檔中，
  沒有改由環境變數或外部設定提供｜Major
- CM-D4｜腳本職責混雜：單一腳本混合建置、部署、資料處理等不相關的工作，
  或重複既有腳本的功能｜Minor

### E. 效能與資源使用

- CM-E1｜Dockerfile 的指令順序讓快取失效：先複製全部原始碼再安裝相依套件，
  或把經常變動的步驟放在前面｜Minor
- CM-E2｜映像檔包含不必要的內容：安裝套件後沒有在同一層清除快取
  （例如 `apt-get` 沒有 `--no-install-recommends`、沒有刪除 `/var/lib/apt/lists`），
  或缺少 `.dockerignore`｜Minor
- CM-E3｜CI 沒有快取相依套件、重複下載或建置相同內容，
  或觸發條件過寬（每次推送都執行全部流程）｜Minor
- CM-E4｜CI Job 或容器沒有設定逾時或資源限制（例如 `timeout-minutes`、
  記憶體與 CPU 限制），失控時會長時間佔用資源｜Minor

### F. 可讀性

- CM-F1｜README 缺少必要內容：用途、安裝或執行方式、設定方式，
  或新增功能、腳本、設定後沒有更新使用說明｜Minor
- CM-F2｜Markdown 結構錯誤：標題層級跳級、程式碼區塊沒有標示語言、
  連結或圖片路徑失效｜Nit｜Minor｜
  連結失效讓讀者找不到必要的文件時
- CM-F3｜腳本或設定中不直觀的指令、旗標或特殊處理沒有註解說明原因｜Minor
- CM-F4｜PowerShell 腳本使用別名（例如 `ls`、`%`、`?`）或位置參數，
  沒有寫出完整的 cmdlet 與參數名稱｜Nit

### G. 可維護性

- CM-G1｜基底映像檔、工具或套件版本沒有固定：使用 `latest` 標籤、沒有標籤，
  或安裝時沒有指定版本｜Major
- CM-G2｜Dockerfile 寫法不符慣例：以 `ADD` 複製一般檔案或下載遠端檔案
  （應使用 `COPY` 或 `curl` 並驗證），或 `CMD`、`ENTRYPOINT` 使用 shell 形式，
  讓容器收不到停止訊號｜Minor
- CM-G3｜新增的設定項或環境變數沒有預設值，也沒有在範本或文件中說明
  （例如 `.env.example`、README）｜Minor
- CM-G4｜新增產生檔、建置產物或本機設定卻沒有加入 `.gitignore`，
  或 `.gitignore` 排除了應該進版控的檔案｜Minor
- CM-G5｜檔案的換行、編碼或縮排不符專案設定（`.editorconfig`、`.gitattributes`），
  例如 shell 腳本使用 CRLF｜Minor｜Major｜
  導致腳本無法在目標平台執行時

## 事先確認事項

評分流程會在開始時，把這裡列出的事項與缺少的文件一起詢問。
diff 或文件已經能確認的事項不會再問。
格式：`- 相關代碼｜要確認的事`，多個代碼以半形逗號分隔。

- CM-A4,CM-B4｜修改的 CI／CD 或部署流程是否會部署到正式環境、能否取得正式環境的機密
  （只有 diff 修改 CI／CD 或部署設定時才問）
- CM-B9｜是否有映像檔或相依套件的弱點掃描結果
  （只有 diff 修改 Dockerfile、基底映像檔或相依套件時才問）

## 特例規則

- CM-B9：只有提供弱點掃描結果時才評；沒有提供時記為不適用。
- CM-A1、CM-A3：只用於會被直接執行的腳本。被其他腳本以 `source`、`.` 或
  dot-source 載入的函式庫檔案，依載入它的腳本是否已處理錯誤判斷。
- CM-B1：測試資料或範例中明顯的假值（例如 `changeme`、`<token>`、`example`），
  以及引用環境變數或機密管理服務的寫法（例如 `${DB_PASSWORD}`、`${{ secrets.X }}`）不扣分。
- CM-B5：基底映像檔預設就以非 root 使用者執行（例如 distroless 的 `nonroot`）時不扣分；
  只在建置階段使用 root、最終階段已切換的 multi-stage build 也不扣分。
- CM-G1：開發用、明確標示不用於正式環境的 compose 或範例檔，可降為 `Nit`。
- 提供的靜態分析報告（ShellCheck、hadolint、markdownlint、actionlint 等）已列出的問題，
  不重複扣分；沒有提供報告時，純格式問題（縮排、空白）不扣分，CM-G5 除外。

## 判斷提示

- 依檔案類型找適用的檢查項：
  - 文件：CM-A5、CM-F1、CM-F2。
  - CI／CD：CM-A4、CM-B2～CM-B4、CM-B6、CM-C1、CM-D1、CM-E3、CM-E4。
  - 容器：CM-B5～CM-B7、CM-B9、CM-D2、CM-E1、CM-E2、CM-G1、CM-G2。
  - 腳本：CM-A1～CM-A3、CM-A6、CM-B7、CM-B8、CM-C2、CM-D4、CM-F3、CM-F4。
  - 其他設定：CM-B1、CM-D3、CM-G3～CM-G5。
- CM-B2：GitHub Actions 中，不受信任的輸入應先放進 `env:`，在指令中以環境變數引用；
  `if:` 條件中使用不算。
- CM-B3：GitHub 官方的 `actions/*` 也建議固定到 commit SHA；固定 SHA 時，
  以註解標示對應的版本號是常見寫法。
- CM-A5、CM-A6、CM-A4、CM-C2、CM-D1、CM-D2、CM-D3、CM-D4、CM-E3、CM-E4、
  CM-F1、CM-F3、CM-G3、CM-G4、CM-G5 需要閱讀整個檔案或脈絡判斷，
  不會出現在自動掃描的候選清單中，要自行檢查。
- 自動掃描的候選只是線索：例如 `FROM builder` 可能是引用前面的建置階段，
  不是沒有標籤的映像檔；每一個都要對照檔案確認。
