Status: needs-triage

# ds-code-review 規格（討論中）

> 原始需求由使用者提出（2026-11-23），下方「決策紀錄」隨 grilling 逐輪補上。

## 目標

- 內建 `/code-review`：廣泛查找的做法好，但不驗證，誤報太多。
- `code-review:code-review`：會驗證，但檢查範圍太狹隘。
- `mattpocock-skills:code-review`：專精 spec 符合度，不找 bug，只能當輔助。

想要的是：認真檢查目標問題有沒有解決、實作架構流程邏輯有沒有問題，並且真實查驗、不亂報。

## 流程

### 1. 找出 PR 本身的目標

閱讀 PR 標題與敘述，再依提及的 ticket 延伸了解背景。延伸部分以 Capability fallback 處理：有對應 skill 就調用，沒有就盡力而為。

### 2. 審查變更內容

各角度：

- 是否達成預定目標／問題是否修好
- 是否從根本原因解決，還是在疊特例
- 是否破壞預定目標以外的原有行為
- 是否在特定輸入或時序條件下出現非預期錯誤
- 是否踩到語言本身的陷阱
- 是否過度設計，有沒有更簡單直接的方法
- 是否重複製作已有的輪子
- 是否違背 convention 規範
- 合併目標 branch 上是否已有更新的 commit，與本 PR 重複或衝突

不限制問題數量；不要漏，但不要捕風捉影。

### 3. 驗證找到的問題

先去重，再逐條徹底驗證。必要時（有對應 skill 或能自行達成條件）到真實環境取得證據。誤報淘汰；真問題記下驗證方法與證據。

### 4. 結論

統整所有有信心的問題，依該環境使用者的習慣產出報告。

## 關鍵原則

- **一定要落地**：每一步的發現與結論都寫成檔案，不直接寫進 context；session 中斷後可接續。
- **多運用 subagent**：條件允許時每一步都分派 subagent，保持乾淨視野、避免主 context 肥大。

### 補充需求（2026-11-23）：避開已有人提過的問題

審查時可能已有其他人在 review 或留言。若找到的問題已有人提過，可省略查驗，也不重複提出。（細節見決策紀錄第 3 輪）

## 決策紀錄

### 第 1 輪（2026-11-23）

- **審查對象**：GitHub PR（編號／URL，經 `gh`）與尚未開 PR 的本地 branch 對 base；其他平台走 Capability fallback。
- **目標**：有線索就推斷，寫成目標檔，**請使用者確認後才繼續**；完全沒線索就直接問。
- **事實包**：第 1 步產出只含事實、不含判斷的事實包（目標、diff、改動檔案、相關呼叫端），各角度 subagent 由此起步並可自行深挖；無 subagent 時單線依序跑各角度（ADR-0002）。
- **發現門檻**：每條發現必須有具體位置、具體失敗情境（輸入／狀態 → 錯誤結果）、所屬角度；寫不出失敗情境的只能記為「疑點」交給驗證階段。找問題的 agent 不自我過濾。
- **驗證邊界**：跑測試、在隔離 worktree 寫一次性重現、本地 build/執行、唯讀查詢可直接做；影響共用系統（部署、寫共用 DB、外部 API 寫入、PR 留言）須先問；絕不修改被審 PR 的 branch。
- **驗證結果**：CONFIRMED（附方法與證據）、REFUTED（淘汰）、UNVERIFIABLE（報告另列，附原因）。每條去重後的發現派一個全新 verifier，任務是設法推翻它，只拿該發現與事實包。
- **報告**：一律產出報告檔；對話只給摘要，語言依使用者設定；依嚴重度排序；貼到 PR 僅在明確要求且先給使用者看過文字。
- **既有問題**：另列小節。只是沒修 → 既有問題；本 PR 讓它重新暴露或更易觸發 → 列入主報告。
- **狀態位置**：該 coding agent／專案慣例優先，否則 `$(git rev-parse --git-common-dir)/ds-code-review/<審查目標>/`（細節見第 2 輪）。
- **接續**：head commit 未變 → 從最後完成的步驟接續；head 已變 → 重新審查，舊報告存檔保留。不做增量審查。
- **角度分組**（5 個 finder）：目標（達成、根本原因）／回歸（原有行為破壞、被刪除行為稽核、跨檔呼叫端追蹤）／邊界（輸入與時序、語言陷阱）／設計（過度設計、重複造輪子、convention）／base 漂移（僅在與目標 branch 新 commit 有重疊時派出）。
- **補漏輪**：finder 結束後再派一個讀過所有已知發現的 agent，專找遺漏類別。
- **模型**：不指定，沿用 agent 預設。

### 第 2 輪（2026-11-23）

- **狀態位置判定**：專案 `AGENTS.md`／`CLAUDE.md` 明定工作檔位置**且該路徑已被 gitignore** → 用它；否則 `$(git rev-parse --git-common-dir)/ds-code-review/<審查目標>/`。會被 commit 的位置與 agent 的 session scratchpad 皆不採用。
- **呼叫方式**：`ds-code-review [PR 編號 | PR URL | branch] [--base <ref>]`。無參數時：目前 branch 有 PR → 審該 PR；否則審目前 branch 對預設 branch。
- **前置過濾**：不擋 draft；diff 為空即停止；PR 已 merge／已關閉時先提醒，由使用者決定是否照審。
- **嚴重度**：must-fix（錯誤行為、未達成 Goal、破壞既有行為）／should-fix（設計、convention、可維護性，有明確理由）／nit（報告中收合）。既有問題不分級，另列一節。
- **語言陷阱**：不放各語言清單，靠模型知識與專案規範；待 eval 顯示某語言常漏再補。
- **Eval**：第一版完成後，以 3–5 個埋有問題的 PR fixture 起步（表象修補、破壞舊行為、race condition、重複造輪子、誤報誘餌），放在 `evals/ds-code-review/`。

### 第 3 輪（2026-11-23）：已有人提過的問題

- **讀取時機**：既有留言存在 `prior-comments.md`，finder 與 verifier 都看不到；只在第 2 步合併後比對，避免被帶偏（符合 ADR-0002）。
- **範圍**：GitHub 的逐行 review 留言、整體 review、一般討論留言，人與 bot 都算；略過 CI 狀態通知與 PR 作者自己的說明。其他平台走 Capability fallback，沒有就不比對。
- **比對**：按根因，標準同合併（同一修正能解決）。
  - `prior`：未解決的留言已提過 → 不驗證、不當新問題報。
  - `extends`：我們多了實質不同的情境或明顯更高的嚴重度 → 照常驗證，註明延伸自該留言。
  - `reraised`：留言串已 resolved 或作者說已修，但 head 上仍存在 → 照常驗證並註明。
- **呈現**：報告另列「Already raised」小節（一行一條附連結）；貼回 PR 時不含 `prior` 項。

### 第 4 輪（2026-11-23）：何時確認 Goal

- 修正第 1 輪 Q2 被擴大套用的問題：只有在 Goal 只能從 PR 本身（標題、敘述、commit、diff）推得時才請使用者確認；有查到 ticket、spec 或設計文件明確寫出目標時，直接進行，不打斷。
- 動機：試跑時查 Jira ticket 花了 7 分鐘，使用者早已離開視窗，回來才發現卡在確認步驟。
- 為了讓需要確認時能盡早問：Goal 與事實包拆成兩個 subagent 平行跑，`goal.md` 一出現就判斷是否需要確認，不必等事實包完成。
- 再放寬（同日）：PR 敘述本身寫清楚目標（bug 則寫明症狀與預期行為）也算有明確來源，不問；只有必須從標題、commit、diff 拼湊推斷時才問。

### 第 5 輪（2026-11-23）：降低成本（推翻 Q14「不指定模型」）

- 模型分級為 Enhancement：skill 只寫 *strongest / standard / light* 三級，harness 能選模型時才套用。
  - Goal 與事實包：standard。finder `goal`/`regression`/`edge`/`sweep`：strongest；`design`/`drift`：standard。
  - verifier 依群組內最嚴重項：must-fix strongest、should-fix 與 suspicion standard（suspicion 需要深挖才能具體化）、nit light。
- **effort 一律不降**：使用者在 PR #94 實驗（`stellar-updatemgr/.git/ds-code-review/pr-94/experiments/comparison.md`）顯示 Sonnet 5.5 medium 跳過 ticket 查詢與呼叫端搜尋，Sonnet 5.5 xhigh 與 Opus 5.5 xhigh 完整度相當。決定完整度的是 effort，不是模型。
- verifier 依 `cause` 所在檔案分組（每組約五條上限），同組共用一個 verifier，減少重複讀程式碼；仍逐條獨立判斷。
- 事實包呼叫端附上呼叫行原文，並加上可檢查的完成條件（每個改動符號都有呼叫端清單或 `none found`；每個 ticket 參照都在 Sources 或 Unresolved）。
- 有 token 用量回報時，主流程記到 `$STATE/usage.md`，以便找出實際大宗。

### 第 6 輪（2026-11-23）：步驟交給其他 agent 執行

- 動機：Claude Code 帳號額度用完時，想把 verify 之類的步驟丟給 Codex。
- 分層：skill 只提供「步驟可外派」的接口；「哪一步交給誰」寫在使用者或專案的 agent 指令（`AGENTS.md`／`CLAUDE.md`），不寫進 skill，也不放到 workflow（Orchestrator skill）——派給誰是環境決定的（帳號、已裝 CLI、額度），不是流程邏輯，而且單獨跑 skill 時也要能用。
- 規則：指令中為某步驟（`goal-and-facts`／`find`／`verify`／`report`，沿用 `state.md` 名稱）指定執行指令時，該步驟的 subagent 改經該指令派出，輸入、輸出檔與一行狀態回報不變；主流程確認輸出檔存在才打勾。
- `usage.md`：tier 欄記指令名稱，無 token 回報記 `-`。
- 附帶好處：finder 與 verifier 用不同家模型，盲點較不重疊。
- 未驗證：Codex `workspace-write` 沙箱能否寫入 `.git/` 下的 `$STATE`；沙箱等級由使用者在自己的指令中決定。

### 第 7 輪（2026-11-24）：nit 上限與分組（依 PR #470 試跑）

- 試跑數據：26 entry、6 個 verifier、共約 1.29M tokens；每個 subagent 固定開銷約 70–80k，與驗幾條關係不大（haiku 驗 2 條 88k，sonnet 驗 5 條 85–99k）。confirmed 16 條中 13 條是 nit。
- verifier 分組：所有 nit（最多 10 條）不分檔案，合成一組交給一個 light verifier；其他 entry 維持依 cause 檔案分組、每組約 5 條。理由：每個 verifier 的主要開銷是讀 goal 與 fact pack，而非讀程式碼。
- nit 驗證上限 10 條：第 2 步合併與比對 prior comment 後，主流程依對 Goal 與後續維護的重要性排序 nit，第十條以後的 entry 標 `verify: skipped (nit cap)`，第 3 步不驗。
- 未驗證的 nit 仍寫進最終報告，另列收合的「Unverified nits」小節並註明未驗證、可能有誤，附 entry id 供使用者自行深挖；Stats 記「nits not verified」。
- 報告範本修正：Evidence 與 Fix 原本相鄰無空行，Markdown 併成一段導致修法被看漏；改為各自成段並加粗。

### 第 8 輪（2026-11-24）：一次審多個相關 PR

- **指定方式**：參數可給多個 PR URL 或 `owner/repo#n`，可混用。本地 clone 先在目前目錄與同層目錄找 remote 相符者；找不到就自動 `gh repo clone` 到 `$STATE` 下，不問。
- **狀態目錄**（單 PR 也適用）：以下指令的目前目錄為準。在 repo 內 → 沿用原規則（文件指定且已 gitignore 的工作目錄，否則 git common dir）；不在 repo 內 → `<目前目錄>/.ds-code-review/<target-id>/`。多 PR 的 target id 為 `group-<第一個 target id>`。
- **Goal**：一份 `goal.md`，上半共同目標，下半每個 PR 一小段寫其角色；確認規則照舊（任一部分是推斷才問）。
- **finder**：每個 lens 一個 finder 看全部 PR，不依 PR 拆。
- **歸屬與報告**：問題歸 `cause` 所在 PR，surface 可在別的 PR；一份 `report.md`，共同 header 後每 PR 一節、各自 verdict；prior comment 各 PR 分開比對；貼回 PR 只貼該 PR 的節。
- **已 merge／關閉的 PR**：自動當背景，diff 與 Goal 進 fact pack，不歸屬問題、不出報告節。
- **中途 head 變動**（單 PR 也適用）：每步開始前檢查各 PR head；有變動則保留已完成 finder，fact pack 附加「Update」節記新 commit，後續步驟（sweep、verify）一律在最新 head 跑；`state.md` 記各步驟跑在哪個 head。
- **實作**：多 PR 細節放 `references/multi-pr.md`（只有 Review set 才讀）；狀態目錄、相依 PR 偵測、中途 head 變動寫在 `SKILL.md`；`GLOSSARY.md` 新增 Review set、Background PR。
- **相依 PR 偵測**：只給一個 PR 時，Step 0 讀完 PR 敘述後、查 ticket 前，從 PR 敘述與 commit message 找明確標示相依（雙向：depends on、requires、part of、consumers 等）的 PR 連結。仍開著的 → 問使用者是否一起審；已 merge／關閉的 → 不問，直接當背景。無人可回答時只審指定 PR，報告 header 列出「偵測到但未一起審的相依 PR」。不從 ticket 挖。

### 第 9 輪（2026-11-25）：verdict 證據等級（借鏡 pstack blast-radius）

- verdict 加 `evidence:` 欄，記決定性證據達到的最高等級：`reproduced`（在執行中的程式或真實環境重現）> `ran`（跑了針對真實 code 的測試或腳本）> `traced`（逐行追路徑並引用每一步）> `pointed`（引用決定性的那一行、規則、既有工具或 commit）。
- CONFIRMED 與 REFUTED 都要填；連 `pointed` 都達不到的就是 UNVERIFIABLE（對應 blast-radius 的第 1 級「只是說」）。
- 有更高等級可達卻沒做，要在 Method 說明原因（blast-radius 的「說明停在哪一級」）。
- 用名稱不用數字：名稱本身就描述做了什麼，報告讀者不用查表。
- 報告：Evidence 標籤後加等級 `**Evidence** (ran): …`；Stats 列 confirmed 各等級數量。
- 未做：依等級擋報告（例如 must-fix 至少要 `ran`）。先觀察實跑的等級分布再決定。

### 第 10 輪（2026-11-25）：Follow-up review（修正後再審）

- 參考：Matt `code-review` 只有「review since <fixed point>」、不記上次 finding；pstack autopilot-full 依改動大小分級重驗、shipping 用 `git patch-id` 讓結論綁 patch。
- **觸發**：`$STATE` 有完成的 report 且 head 變了 → 自動 follow-up；`--full` 才歸檔重來。上次沒跑完 → 照「中途 head 變動」接續。順帶解掉 resume 與中途 head 變動規則的矛盾。
- **檔案**：上一輪移到 `$STATE/rounds/<n>/`；新 entry id 接續；舊問題重驗的 verdict 加 `round`、`status`。
- **delta**：merge-base 沒變用 `git diff <old head> <new head>`；有 rebase 用 `git range-diff`，只因 rebase 變的 commit 不算，base 的新 commit 交給 `drift`。
- **升級完整重審**：delta 變動行數超過原 diff 一半、range-diff 對不起來、Goal 變了；Review set 任一 member 升級就整組升級。
- **Goal 與 fact pack**：沿用上一輪 `goal.md`（重抓 PR 敘述與 ticket 比對）；fact pack 沿用並加 `## Follow-up <n>` 節，一個 standard subagent。
- **find**：delta 上跑 `regression`、`edge`（重點：修正本身有沒有弄壞別處），條件成立加 `drift`，最後 `sweep`；prior comment 只比對新出現的；nit 上限只算新 entry。
- **重驗**：上次 CONFIRMED（非 pre-existing）全部重驗；UNVERIFIABLE 只在 delta 碰到 cause 檔時重驗；其他沿用。依 cause 檔分組、每組約 10 條、standard；nit 一組 light。verifier 拿到 entry、上一輪 verdict、該 comment thread 的作者回覆。
- **status**：`fixed`、`still present`、`partial`、`accepted`（作者不修且理由成立）、`disputed`（作者不修但理由不成立，並列雙方）。
- **報告**：header 標 round 與對比 head；「Previous items」每條一行，仍有問題的展開；其他部分只放新 entry；verdict 只算仍存在 + 新問題。
- **貼回 PR**：舊問題回原 thread（`fixed`/`accepted` 一行確認），新問題開新 comment；所有貼文記在 `posted.md`，供下一輪找 thread。貼出仍需使用者逐則同意。
- **實作**：`references/follow-up.md`；`SKILL.md` 只改 Step 0（`--full`、resume 分支）與貼文記錄；`GLOSSARY.md` 新增 Follow-up review。
- 未驗證：尚未實跑；`range-diff` 配對失敗的判斷（「大多數」）是否夠明確。

### 待處理觀察（2026-11-24，stellar-enforce PR #1072 試跑）

- 12 個 subagent、約 1.38M tokens，PR 只有 +85/−30。大宗是 finder 深入讀跨 repo（lib、deploy）程式碼，且不少 finding 其實是 lib 的問題，在 consumer PR 又找一次。→ 第 8 輪：一次審多個相關 PR。
- review 中途 head 被推新 commit：skill 只在重新叫起時處理 head 變動。這次手動做法：已完成的四個 finder 保留，fact pack 補上新 commit，sweep 與所有 verifier 改在新 head 跑。結果：新 commit 已修的 F01 被正確推翻，sweep 抓到新 commit 帶進的 2 個問題。
- verifier 調整嚴重度：2 個 must-fix 降為 should-fix，1 個 nit（CI 沒跑）升為 should-fix。佐證 nit 上限用的是驗證前的嚴重度，被跳過的 nit 可能其實是 should-fix。
