---
# 標準代號：只能使用英文小寫、數字與 -，須與規則專案 rules.ini 的區段名稱一致
name: springboot-3
# 標準版本，會記錄在評分報告中
version: v0.6
---

# Spring Boot 3 PR 程式碼品質評分規則

<!-- markdownlint-disable MD028 -->
<!-- markdownlint-disable MD033 -->

適用於 Java／Spring Boot 3 專案的 PR 程式碼品質評分。

本文件由上層規則目錄的 `rule.ini` 設定為 code-review-scoring 的評分規則，
並在規則專案的 `rules.ini` 登錄為 `springboot-3`；
自動掃描規則在同目錄的 `springboot-3-cr-scoring-rule.tsv`：

```ini
[rule]
standard="zh-tw/springboot-3-cr-scoring-rule.md"
scan="zh-tw/springboot-3-cr-scoring-rule.tsv"
applies="*.java, *.kt, *.sql, *.xml, *.yml, *.yaml, *.properties, *.gradle, *.kts"
```

評分範圍、需要證據才能扣分、上游因素等通用規則由評分流程處理，不在本文件中。

## 面向

所有標準共用以下七個面向。配分、嚴重度分級與等級使用規則專案 `defaults.ini`
的設定，可在上層規則目錄的 `rule.ini` 覆寫。檢查代碼的第一個字母代表所屬面向。

| 代碼 | 面向 | 衡量重點 |
| --- | --- | --- |
| A | 正確性 | 程式是否照預期運作、邊界與例外是否處理 |
| B | 安全性 | 是否引入安全弱點 |
| C | 測試品質 | 測試是否足夠且有意義 |
| D | 架構與設計 | 分層、職責劃分、是否符合 Spring 慣例 |
| E | 效能與資源使用 | 查詢效率、資源管理、外部呼叫 |
| F | 可讀性 | 別人是否容易看懂 |
| G | 可維護性 | 未來是否容易修改與擴充 |

## 檢查項

每個檢查項寫成一個清單項目，欄位以全形 `｜` 分隔：

- 不可調整嚴重度：`- 代碼｜內容｜預設嚴重度`
- 可調整嚴重度：`- 代碼｜內容｜預設嚴重度｜可調整為｜調整條件`

只有寫明的條件成立時才可以調整嚴重度。任何檢查項都可以降為 `Nit`。

### A. 正確性

- A1｜邊界情況沒有處理：null、空集合、`Optional` 沒有檢查就呼叫 `.get()`、
  數值溢位｜Major
- A2｜金額計算沒有使用 `BigDecimal`（例如用 `double`、`float`）｜Critical
- A3｜`@Transactional` 放在 private 方法上，或同一個類別內部呼叫自己帶有
  `@Transactional` 的方法，導致 proxy 失效｜Major
- A4｜方法會拋出 checked exception，帶有 `@Transactional` 卻沒有設定
  `rollbackFor`（checked exception 預設不會觸發 rollback）｜Major
- A5｜例外被吞掉：`catch` 區塊只印 log 或沒有任何處理，或重新拋出時沒有保留
  原本的 cause｜Major
- A6｜並行問題：Singleton Bean 中有共用的可變狀態、明顯的 Race Condition，
  或更新共用資料卻沒有樂觀鎖（`@Version`）或其他鎖機制｜Critical
- A7｜時間處理使用 `Date` / `Calendar`，或沒有明確指定時區｜Minor｜Major｜
  PR 描述或規格書明確涉及跨時區業務時
- A8｜實作和 SD 規格書或 PR 描述不一致，而且規格書的對應內容寫得清楚｜Major

### B. 安全性

- B1｜程式碼或設定檔中寫死密碼、Token、金鑰｜Critical
- B2｜SQL Injection 風險：字串串接 JPQL、Native Query，或在 `@Query` 中拼接
  參數｜Critical
- B3｜新增或修改的 API 端點缺少身分驗證或權限檢查，或存取資源時沒有檢查
  擁有者（IDOR）｜Critical
- B4｜Controller 參數沒有加 `@Valid`，或 DTO 沒有使用 `jakarta.validation`
  註解｜Major
- B5｜關閉 CSRF 防護，且沒有註解或 PR 描述說明原因｜Major
- B6｜`permitAll()` 開放的路徑範圍明顯大於需要（例如 `/**`、`/api/**`）｜Major
- B7｜log 輸出個資、Token、完整 Request Body，或錯誤回應包含
  stack trace｜Major
- B8｜CORS 設定 `allowedOrigins("*")` 並同時允許 credentials｜Major
- B9｜新增的相依套件有已知 CVE｜Major

### C. 測試品質

- C1｜新增或修改的商業邏輯沒有對應的單元測試｜Major｜Critical｜
  PR 有商業邏輯異動，但整個 PR 沒有任何測試檔案的異動時
- C2｜測試只涵蓋 Happy Path，沒有涵蓋例外與邊界情況｜Major
- C3｜測試沒有有效斷言，或只有 `assertNotNull`｜Major
- C4｜Controller 或 Repository 的測試使用 `@SpringBootTest`，而不是
  `@WebMvcTest`、`@DataJpaTest`｜Minor
- C5｜DB 測試使用 H2，但被測的查詢使用了正式環境資料庫特有的語法
  （建議改用 Testcontainers）｜Minor
- C6｜測試之間互相依賴、依賴執行順序，或使用 `Thread.sleep`｜Minor
- C7｜測試方法名稱看不出測試的情境與預期結果｜Minor

### D. 架構與設計

- D1｜Controller 直接呼叫 Repository，或商業邏輯寫在 Controller、
  Entity Listener 中｜Major
- D2｜JPA Entity 直接作為 API 的 Request 或 Response｜Major｜Critical｜
  Entity 含有敏感欄位（密碼雜湊、內部 ID、個資）時
- D3｜單一類別或方法承擔多個不相關的職責（God Service），或明顯違反 SOLID
  原則｜Major
- D4｜使用 field injection（欄位上直接加 `@Autowired`），而不是建構子
  注入｜Minor
- D5｜錯誤處理沒有透過 `@RestControllerAdvice` 與 `ProblemDetail`
  （RFC 9457），或回應格式和既有慣例不一致｜Minor
- D6｜使用 `javax.*` 命名空間（Spring Boot 3 應為 `jakarta.*`；
  `javax.crypto` 等 JDK 套件不在此限）｜Minor

### E. 效能與資源使用

- E1｜JPA N+1 查詢：在迴圈中觸發 lazy loading 或逐筆查詢
  （建議使用 `@EntityGraph` 或 fetch join）｜Major
- E2｜查詢沒有分頁，一次載入整張表｜Minor｜Major｜
  PR 描述、規格書或資料表名稱顯示是交易、訂單、日誌等會持續成長的資料，
  或資料量大時
- E3｜在 `@Transactional` 範圍內呼叫外部 API，導致交易時間過長｜Major
- E4｜呼叫外部系統沒有設定 timeout、retry 或 Circuit Breaker｜Major
- E5｜Stream、檔案、連線等資源沒有使用 try-with-resources 或其他方式
  確保關閉｜Major
- E6｜關聯設定使用 `FetchType.EAGER`，且看不出必要性｜Minor
- E7｜新增的查詢條件欄位，在同一個 PR 的 migration 中沒有建立索引，
  也沒有說明｜Minor

### F. 可讀性

- F1｜方法超過 50 行，或巢狀超過 3 層｜Minor
- F2｜命名無法表達用途，或和同一個 diff 中其他程式碼的命名慣例
  不一致｜Minor
- F3｜出現沒有命名的魔術數字或有特殊意義的字串常值｜Minor
- F4｜複雜的邏輯沒有註解說明為什麼這樣寫｜Minor
- F5｜留下被註解掉的程式碼，或使用 `System.out.println`｜Minor

### G. 可維護性

- G1｜同一段邏輯在 diff 中重複出現兩次以上，可以抽成共用卻沒有抽｜Major
- G2｜應該放在設定檔的值寫死在程式碼中，或新增分散的 `@Value` 而不是
  `@ConfigurationProperties`｜Minor
- G3｜log 等級使用不當（例如正常流程使用 `ERROR`），或 log 使用字串串接
  而不是 SLF4J 參數化寫法｜Minor
- G4｜新增的公開 API 沒有 OpenAPI 註解｜Minor
- G5｜TODO 沒有附 issue 編號｜Minor
- G6｜修改已存在的 DB migration 檔案（Flyway / Liquibase），或有破壞性
  schema 變更但沒有說明｜Major

## 事先確認事項

評分流程會在開始時，把這裡列出的事項與缺少的文件一起詢問。
diff 或文件已經能確認的事項不會再問。

- A7｜功能是否涉及跨時區業務（只有 diff 涉及時間處理時才問）
- E2｜新增查詢的資料表是否會持續成長，以及大約的資料筆數
  （只有 diff 涉及查詢時才問）
- G4｜專案是否使用 springdoc
- B9｜是否有相依套件的弱點掃描結果

## 特例規則

- A8：沒有提供 SD 規格書時，只比對 PR 描述。
- B9：只有提供弱點掃描結果時才評；沒有提供時記為不適用。
- C1：只有 PR 改了商業邏輯、但完全沒有異動任何測試檔案時才升為
  Critical；有測試但沒有涵蓋到該邏輯，維持 Major。
- G4：只有在 diff 或既有程式碼中可以看到專案使用 springdoc 時才評；
  否則記為不適用。
- 第三方 workaround 豁免：為了繞過第三方核心系統問題而寫的程式碼，
  如果有註解說明原因，**並且**附上廠商問題單號，F 與 G 面向的檢查項
  不扣分。缺少其中任何一項時照常扣分。

## 判斷提示

- E2：資料量小的設定表維持 Minor；訂單表這類資料量大或持續成長的資料表
  改為 Major。資料量從 diff 看不出來時，依「事先確認事項」詢問。
- 靜態分析報告通常來自 Checkstyle、SpotBugs、SonarQube；報告中已列出的
  問題不重複扣分。
- 新增或修改的 API 端點，逐一檢查 B3（身分驗證、權限、資源擁有者）
  與 B4。
- A6、A8、B3、C2、D3、E3、F2、F4、G1 需要閱讀程式碼脈絡判斷，
  不會出現在自動掃描的候選清單中，要自行檢查。
- B9、E7、F1 也沒有自動掃描規則，同樣要自行檢查。
- 自動掃描的候選只是線索，誤判很常見：例如 `).get()` 不一定是
  `Optional`，測試資料中的假密碼不算 B1。每一個都要對照程式碼確認。
