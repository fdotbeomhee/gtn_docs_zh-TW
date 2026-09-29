# 實用自訂指引 (Customization Guide) {: #customization-guide }

所有專案層級的自訂配置皆存放在工作區目錄下的 `griptape-nodes-project.yml` 中。您只需宣告欲變更的項目 — 其餘未設定之處將完整繼承系統預設規範。

## 修改輸出目錄實體路徑 {: #changing-the-outputs-directory-path }

將所有節點輸出成果統一遷移至名為 `renders` 的資料夾：

```yaml
project_template_schema_version: "1.0.0"
name: "My Project"

directories:
  outputs:
    path_macro: "renders"
```

此時所有引用 `{outputs}` 的情境規則皆會自動將檔案存入 `renders` 資料夾而非預設的 `outputs`。

## 修改輸入目錄實體路徑 {: #changing-the-inputs-directory-path }

```yaml
project_template_schema_version: "1.0.0"
name: "My Project"

directories:
  inputs:
    path_macro: "source_files"
```

## 新增自訂目錄 {: #adding-a-custom-directory }

為客戶最終交付成品新增專用目錄：

```yaml
project_template_schema_version: "1.0.0"
name: "My Project"

directories:
  deliverables:
    path_macro: "client/final"
```

新增完成後，`{deliverables}` 即可作為變數在任何巨集中直接調用。

## 新增自訂情境規則 {: #adding-a-custom-situation }

新增一個將最終彩現輸出以直接覆寫原則存入 deliverables 目錄的情境規則：

```yaml
project_template_schema_version: "1.0.0"
name: "My Project"

directories:
  deliverables:
    path_macro: "client/final"

situations:
  save_deliverable:
    macro: "{deliverables}/{workflow_name?:_}{file_name_base}.{file_extension}"
    policy:
      on_collision: overwrite
      create_dirs: true
    description: "交付給客戶的最終成品"
```

## 修改現有情境規則 {: #modifying-an-existing-situation }

修改 `save_node_output`，強制在檔名前包含當前工作流程名稱：

```yaml
project_template_schema_version: "1.0.0"
name: "My Project"

situations:
  save_node_output:
    macro: "{outputs}/{workflow_name:_}{file_name_base}{_index?:03}.{file_extension}"
```

此處僅替換了 `macro` 欄位。覆寫原則 (policy)、備援規則 (fallback) 與描述文字 (description) 皆平穩繼承自系統預設值。

## 新增自訂環境變數 {: #adding-environment-variables }

定義可在巨集中隨處調用的自訂鍵值：

```yaml
project_template_schema_version: "1.0.0"
name: "My Project"

environment:
  CLIENT_CODE: "ACME"
  SEASON: "S03"
```

在目錄路徑或情境巨集中引用它們：

```yaml
directories:
  outputs:
    path_macro: "{CLIENT_CODE}/{SEASON}/renders"
```

## 依副檔名自動分流檔案 {: #routing-files-by-extension }

使用 `file_extension_directories` 將不同類型的檔案自動存入不同的子資料夾中，而無需為每種類型編寫獨立的情境規則。完整手冊請參閱[副檔名目錄分流 (File Extension Directories)](file_extension_directories.md)。

```yaml
project_template_schema_version: "1.0.0"
name: "My Project"

file_extension_directories:
  png: "images"
  jpg: "images"
  mp4: "videos"
  wav: "audio"

situations:
  save_node_output:
    macro: "{outputs}/{file_extension_directory?:/}{node_name?:_}{file_name_base}{_index?:03}.{file_extension}"
```

副檔名映射值亦支援巨集。若希望將影片導向至共用網路磁碟、而其餘檔案保持存於 `outputs` 下，可採用由分流數值主導根目錄的情境巨集：

```yaml
project_template_schema_version: "1.0.0"
name: "My Project"

file_extension_directories:
  png: "{outputs}/images"
  mp4: "{workspace_dir}/shared/videos"

situations:
  save_node_output:
    macro: "{file_extension_directory?:/}{node_name?:_}{file_name_base}{_index?:03}.{file_extension}"
```

## 引用作業系統環境變數 {: #referencing-os-environment-variables }

自啟動引擎的作業系統環境中擷取數值：

```yaml
project_template_schema_version: "1.0.0"
name: "My Project"

environment:
  SHARED_DRIVE: "$STUDIO_SHARED_STORAGE"

directories:
  outputs:
    path_macro: "{SHARED_DRIVE}/renders"
```

若作業系統環境變數 `STUDIO_SHARED_STORAGE` 設定為 `/mnt/studio`，則 `{outputs}` 將順利解析為 `/mnt/studio/renders`。

## 完整實戰配置範例 {: #putting-it-all-together }

視覺特效 (VFX) 專案的完整實體自訂專案設定檔：

```yaml
project_template_schema_version: "1.0.0"
name: "VFX Pipeline"
description: "VFX 製程專用自訂檔案架構"

environment:
  SHOW_CODE: "AURORA"

directories:
  inputs:
    path_macro: "source"
  outputs:
    path_macro: "renders"
  plates:
    path_macro: "source/plates"
  deliverables:
    path_macro: "deliverables/{SHOW_CODE}"

situations:
  save_node_output:
    macro: "{outputs}/{workflow_name?:_}{file_name_base}{_index?:03}.{file_extension}"

  save_deliverable:
    macro: "{deliverables}/{workflow_name?:_}{file_name_base}.{file_extension}"
    policy:
      on_collision: overwrite
      create_dirs: true
    fallback: save_file
    description: "最終交付成品"
```
