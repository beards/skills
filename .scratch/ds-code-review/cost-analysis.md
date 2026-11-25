# 成本分析：group-stellar-lib-pr-470（7 個 PR 的 Review set，兩輪）（2026-11-25）

資料：`_unified/.scratch/ds-code-review/group-stellar-lib-pr-470/{rounds/1,.}/usage.md`、verdicts、entries。
round 2 有 8 個 subagent 因週額度中斷，usage 只記續跑段，實際更高。

## Token 分布

| 階段 | round 1 | round 2 |
|---|---|---|
| goal + fact pack（含 head 中途變動的 Update 146k） | 385k（13%） | 155k |
| find（5 lens / round 2 為 6 lens） | 1,202k（41%） | ≥947k（design 未記） |
| verify（11 / 6 個 verifier） | 1,343k（46%） | 596k |
| 合計 | 約 2.93M | ≥1.70M |

每個 subagent 的固定開銷約 75k，round 1 共 18 個 subagent，光底價就約 1.35M。

## 產出

- round 1：32 entry → 推翻 9、確認 22（must-fix 2、should-fix 3、nit 16、pre-existing 1）。
- round 2 新 entry：21 → 推翻 6、確認 15（should-fix 1、其餘 nit，含 pre-existing 3）；舊項目 fixed 4、still present 5（全是 nit）。
- 兩輪合計 **must/should-fix 6 條**：F01、F02（must）、F05、F08、F12、F43。

## 各 lens 對 must/should-fix 的貢獻

| 條目 | 提出的 lens |
|---|---|
| F01 | goal, edge |
| F02 | goal, edge, regression |
| F05 | goal, edge, design |
| F08 | edge, design, regression |
| F12 | sweep（唯一） |
| F43 | sweep, regression |

- 只靠 `edge` + `sweep` 就涵蓋全部 6 條。
- `goal`、`regression`、`design` 沒有任何一條 must/should-fix 是它們獨有的；`design` 的獨有產出全是 nit（F11、F15–F18、F42…）。
- `drift`（round 2）：1 條 pre-existing nit、1 條推翻。
- 註：n=1 個 Review set，lens 重疊可能和這組 PR 的性質有關。

## 跟別人比，我們多做的事

| 多做的事 | 成本 | 這次換到什麼 |
|---|---|---|
| 每條 finding 都由獨立 verifier 驗（含 nit） | verify 46%；nit verifier 合計約 390k | 擋掉 15 條誤報；但 22 條確認中 16 條是 nit |
| 5 個 lens 各自一個 strongest finder | find 41% | 4 個 lens 高度重疊，`edge`+`sweep` 已涵蓋全部重要問題 |
| 獨立 goal、fact pack subagent | 13% | 讓 finder 共用起點；finder 越少越不划算 |
| verifier 小組（約 5 條）各自重讀 goal + fact pack | 每組底價約 75k | 分組越細越貴，跟驗幾條無關 |

## 可砍的方向（依省多少、傷多少排序）

1. **nit 不驗**：全部列為 unverified nit。省約 390k／兩輪，風險最低（nit 錯了代價小）。
2. **lens 合併**：goal、regression、edge、design 合成一個 correctness finder（strongest），保留 sweep 當第二個獨立視角。每輪省約 650k。
3. **verifier 分大組**：非 nit 的 entry 不分檔案，一兩個 verifier 處理完；strongest 只給 must-fix。round 1 的 11 個 verifier 可縮成 2–3 個，省約 600k。
4. **goal 和 fact pack 合成一個 subagent**：省一份底價約 80k。

1–4 全做，估計 round 1 從 2.9M 降到約 1M。這是估計，沒實測。

## 同場發現的 Follow-up review 問題（使用者回報）

- 升級門檻太敏感：sysmgr 照要求逐欄位 escape，delta 485 行 > 原 diff 477 的一半，觸發整組重審；使用者選不重審。
- 舊 state 沒有 `posted.md`：上一版不產生，需手動補或從 `state.md` 的 posted 紀錄／PR 上自己的 comment 推回。
- 沒有新 commit 的 member：沒說舊項目要不要重驗；這次沿用不重驗。
- 舊項目重驗不必等 finder，可以並行（兩者互不讀對方輸出）。
- 撞到額度中斷：續跑的 subagent 在 usage.md 只記續跑段。
