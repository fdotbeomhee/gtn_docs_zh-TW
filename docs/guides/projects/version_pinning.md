# 鎖定引擎與程式庫版本 (Pinning engine and library versions) {: #pinning-engine-and-library-versions }

本指南專為向團隊成員分發專案、並需要確保該專案一律在經過驗證的特定引擎版本與特定程式庫版本集合下執行的管理人員所設計。版本鎖定使專案成為單一事實來源 (Single Source of Truth)：當使用者啟用該專案時，若當前引擎版本不相容將拒絕執行，並自動將每筆鎖定的程式庫佈建至專案所宣告的相應版本。

兩項鎖定設定皆存放於**專案同級組態 (Project-adjacent config)** — 即放置於專案 `griptape-nodes-project.yml` 旁邊的 `griptape_nodes_config.json` 檔案中。專案 YAML 本身不承載版本資料，而是由同級的 JSON 組態負責。此檔案如何疊加於使用者組態之上的層級關係請參閱[工作區組態檔案](workspace.md#config-files)。

```
/MyProject/
  griptape-nodes-project.yml      <- 專案本體（此處不設定版本鎖定）
  griptape_nodes_config.json      <- 引擎與程式庫版本鎖定設定存放於此
```

請將這兩個檔案一同打包分發。因為專案同級組態層級高於使用者的本機全域組態、但低於個別工作區組態，因此您定義的鎖定設定會自動套用至每位啟用該專案的使用者，且不會被永久寫死在其個人電腦的全域環境中。

## 鎖定引擎版本 {: #pin-the-engine-version }

將 `requires_engine` 設定為符合 [PEP 440 版本規範 (PEP 440 version specifier)](https://peps.python.org/pep-0440/#version-specifiers) 的字串。當專案啟用時，運行中的引擎版本必須滿足該規範，否則將阻斷專案啟用。

```json
{
  "app_events": {
    "on_app_initialization_complete": {
      "requires_engine": ">=0.80,<1.0"
    }
  }
}
```

- 規範條件會與當前正在執行的引擎實體版本進行比對。
- 版本不相符將**阻斷啟用**：專案拒絕載入，並向使用者清楚提示所需的版本與目前正在執行的版本。
- 省略此鍵值（或設為 `null`）將完全跳過引擎版本檢查。

建議使用封閉區間（如 `>=0.80,<1.0`）而非開放的下限範圍，以防止專案在未來行為可能產生重大變更的下一個主版號引擎中被靜默啟用。

## 鎖定程式庫版本 {: #pin-library-versions }

`libraries_to_download` 列出引擎代表本專案所自動佈建的擴充程式庫清單。每筆條目可為純 Git URL 字串（傳統行為 — 直接自原始碼 Clone，不強制約束版本），亦可為附加了版本鎖定物件的結構：

```json
{
  "app_events": {
    "on_app_initialization_complete": {
      "libraries_to_download": [
        {
          "name": "Griptape Nodes Library",
          "version": "==0.79.0",
          "git_url": "griptape-ai/griptape-nodes-library-standard@v0.79.0"
        }
      ]
    }
  }
}
```

| 欄位名稱 | 必填 | 說明 |
| --------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `git_url` | 是 | `url@ref` 格式的 Git 來源：完整 URL 或 `user/repo` 簡稱，並帶有選填的 `@branch\|tag\|commit` 後綴。未宣告 `@ref` 則預設拉取該倉庫的預設主分支。 |
| `version` | 否 | 已安裝程式庫必須滿足的 PEP 440 版本規範（例如 `==0.79.0`、`>=1.2,<2`）。省略則僅依據來源鎖定。 |
| `name` | 否 | 該程式庫資訊清單中的 `name`。設定後，系統會依名稱比對本機已安裝複本，以精確判定是否需要重新下載。 |

建議將 `git_url` 的 ref 標籤**與** `version` 版本規範**同時**鎖定於同一個發布版本（例如 `@v0.79.0` 與 `==0.79.0`），確保 Clone 的原始碼與強制的版本號彼此完全同步，絕不產生漂移。

### 僅有被下載的程式庫才會被覆寫 {: #only-downloaded-libraries-are-overwritten }

列於 `libraries_to_download` 中的程式庫，是引擎為了滿足版本鎖定而允許**覆寫 (Overwrite)** 的唯一對象。僅被註冊的程式庫（在 `libraries_to_register` 中以實體路徑列出）會按原樣直接載入，**絕不會**因專案啟用而被覆寫。若您希望專案具備強制約束某個程式庫版本的能力，該程式庫必須列於下載清單中，而不僅僅是註冊清單。

您無需手動將該程式庫額外追加至 `libraries_to_register`：成功下載後，引擎會自動將解析後的資訊清單路徑追加至註冊清單中，因此下載至載入的完整鏈路完全自動化運作。

### 下載套件的實體存放落點 {: #where-the-downloads-land }

`libraries_to_download` 宣告了**下載什麼**以及鎖定哪個版本；而 [`libraries_dir`](projects.md#libraries-directory)（專案 YAML 中的設定欄位）則決定了這些程式庫**存放與解析至何處**。兩者有機結合：每個鎖定下載的套件皆會被佈建至專案解析後的程式庫目錄中。當專案未宣告 `libraries_dir`（且未自父專案繼承）時，下載檔案將落於工作區相對路徑下的 `libraries` 資料夾中，維持傳統相容行為。跨專案樹共用的 `libraries_dir` 意味著由父專案下載過一次的鎖定程式庫，其下所有子專案皆可直接重複使用，無需重複下載。

## 專案啟用時的執行邏輯 {: #what-happens-on-activation }

當使用者啟用鎖定了版本的專案時，引擎會將 `libraries_to_download` 中的每筆條目與本機已安裝的實際狀況進行嚴格比對，並規劃以下三種執行方案之一：

| 執行方案 | 觸發時機 | 具體效果 |
| ------------- | ------------------------------------------------ | --------------------------------------------------------------------------- |
| **SKIP (略過)** | 本機已安裝版本已完全滿足鎖定規範 | 保持原樣，不做任何修改。 |
| **INSTALL (安裝)** | 該程式庫尚未安裝於本機中 | 自動 Clone 鎖定的原始碼。具備非破壞性。 |
| **OVERWRITE (覆寫)** | 本機已安裝該程式庫，但版本不符合鎖定規範 | 刪除本機舊資料夾並重新 Clone 鎖定版本。**具備破壞性。** |

在執行任何具備破壞性的 OVERWRITE 操作前，編輯器會彈出唯讀的**執行計畫預覽視窗**，並暫停等待使用者親自授權。若使用者拒絕，系統會乾淨俐落地取消操作：保持先前的專案活躍，且不改動任何程式庫檔案。若正在運行的引擎版本不符合鎖定需求 (`requires_engine`)，預覽視窗亦會明確提示錯誤並鎖定核准按鈕。

## 完整實戰範例 {: #worked-example }

要求引擎版本滿足 `>=0.80,<1.0` 且將標準程式庫鎖定為 `0.79.0` 的專案配置範例：

`/MyProject/griptape-nodes-project.yml`

```yaml
project_template_schema_version: "1.0.0"
name: "my-pinned-project"
description: "運行於引擎 0.80-0.x 環境，並將標準程式庫鎖定於 0.79.0 版本。"
```

`/MyProject/griptape_nodes_config.json`

```json
{
  "app_events": {
    "on_app_initialization_complete": {
      "requires_engine": ">=0.80,<1.0",
      "libraries_to_download": [
        {
          "name": "Griptape Nodes Library",
          "version": "==0.79.0",
          "git_url": "griptape-ai/griptape-nodes-library-standard@v0.79.0"
        }
      ]
    }
  }
}
```

專案啟用時的行為表現：

1. **引擎檢查** — 若當前運行的引擎版本不在 `>=0.80,<1.0` 區間內，系統直接阻斷啟用並彈出版本不符提示。
1. **首次啟用（全新電腦環境）** — 本機不存在標準程式庫，因此執行計畫為 INSTALL：自動 Clone `v0.79.0` 並完成登錄。
1. **再次啟用（`0.79.0` 已就緒）** — 鎖定條件已滿足，執行計畫為 SKIP。
1. **本機已存在衝突版本**（例如先前其他專案安裝了 `0.78.0`）— 由於 `0.78.0` 不符合 `==0.79.0` 條件，執行計畫判定為破壞性 OVERWRITE：介面彈出預覽視窗提示覆寫，待使用者確認授權後，刪除本機舊資料夾並重新 Clone `v0.79.0`。

## 注意事項與常見陷阱 {: #notes-and-gotchas }

- **傳統純字串格式依然相容**。現有的 `"libraries_to_download": ["user/repo"]` 清單會持續直接從原始碼 Clone 而不進行強制版本約束。唯有使用物件結構時才會強制驗證 `version`。
- **使用者個人覆寫具備最高優先權**。使用者的工作區組態層級高於專案同級組態（參閱[工作區組態解析順序](workspace.md#config-resolution-order)）。個別使用者可在本機覆寫您的鎖定設定；這些鎖定是隨專案分發的標準預設規範，而非強制硬性鎖定。
- **CLI 命令列替代方案**。在無外設伺服器環境中進行自動化維運的管理員，可透過 `griptape-nodes libraries download <git_url>` 下載程式庫，並透過 `griptape-nodes libraries sync` 進行同步更新。詳見[節點程式庫 → CLI 替代方案](../libraries.md#cli-alternatives)與 [CLI 命令列參考手冊](../../reference/command_line_interface.md)。上述宣告式組態則是將相同版本約束隨專案攜帶分發的最佳實踐。
