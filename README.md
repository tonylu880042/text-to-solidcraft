# text-to-solidcraft

**text-to-solidcraft** 是一套以 SolidCraft 為建模引擎的 AI 參數化 CAD skill。你可以用自然語言描述零件、組裝方式與尺寸，讓 AI 協助規劃草圖和特徵樹，透過 SolidCraft MCP 建立、檢查及修改可編輯的 3D 模型。

模型的參數、草圖與特徵歷史保存在 SolidCraft 原生設計檔中。需要交付時，可依需求匯出 STEP、STL 等格式。

## 能做什麼

- 透過分階段提問，將設計需求整理成尺寸、命名參數、約束與特徵順序。
- 建立草圖、擠出、切除、旋轉、圓角、倒角及陣列等參數化特徵。
- 檢視零件、裝配、特徵歷史與目前選取的幾何。
- 修改既有參數與特徵，並重新量測受影響的尺寸。
- 檢查尺寸、間隙與製造相關問題，並標示無法確認的項目。
- 保存可編輯的 SolidCraft 設計檔，匯出支援的製造或交換格式。

```text
你的需求 → CAD 設計與參數規劃 → SolidCraft MCP
        → 可編輯模型與特徵歷史 → 量測與檢查 → STEP／STL 等檔案
```

## 開始使用

### 1. 準備 SolidCraft

安裝 [SolidCraft](https://solidcraft.io/)，並確認 `solidcraft-cli` 可由終端機執行：

```bash
solidcraft-cli --version
```

macOS 若未將 CLI 加入 PATH，可改用 `/Applications/SolidCraft.app/Contents/MacOS/solidcraft-cli`。實際路徑依安裝位置而定。

### 2. 取得專案並連接 MCP

```bash
git clone --branch refactor/cad-solidcraft-backend https://github.com/tonylu880042/text-to-solidcraft.git
cd text-to-solidcraft
codex mcp add solidcraft -- solidcraft-cli mcp
codex mcp list
```

如果 CLI 不在 PATH，在 macOS 可使用：

```bash
codex mcp add solidcraft -- /Applications/SolidCraft.app/Contents/MacOS/solidcraft-cli mcp
```

設定後重新開啟 Codex 或開始新的工作階段，確認 `solidcraft` MCP 工具可用。在 SolidCraft 開啟桌面控制通道時，agent 可以連到視窗中的設計；使用獨立無視窗工作階段時，啟動參數可改為 `mcp --headless`。其他 MCP 客戶端請依其設定方式，將 `solidcraft-cli mcp` 加入伺服器清單。

目前此儲存庫沿用的插件設定仍會啟動 **cadgen**，不會自動註冊 SolidCraft MCP。請另外完成上述連線設定。伺服器的可讀寫路徑也取決於 SolidCraft 的檔案存取設定；需要指定工作資料夾時，請參考 [SolidCraft MCP 文件](https://github.com/tangency-labs/solidcraft/blob/main/docs/mcp.md)。

### 3. 描述你要的模型

在專案工作目錄中，可以直接告訴 agent：

> 請依照 `skills/cad/SKILL.md`，建立一個 100 × 70 × 30 mm 的電子外殼，壁厚 2 mm、上蓋可拆。先列出設計假設與特徵規劃，再建立可編輯模型，檢查尺寸並匯出 STEP 和 STL。

不需要一開始就知道所有尺寸。描述用途、已知尺寸與製造方式後，agent 會分批詢問影響功能與配合的關鍵問題，協助整理設計規格。簡單且資訊完整的修改不需要重新填寫需求表。

#### 如何準備設計需求

可先提供下列已知資訊；不確定的項目可以直接回答「未知」：

- **用途與使用方式**：模型要做什麼、支撐或連接哪些物件。
- **尺寸與基準**：長、寬、高、孔徑、孔距及單位；角度相對於哪個方向，尺寸是內徑還是外徑。
- **配合與活動空間**：接觸位置、插入深度、移動範圍、避空區域及其他零件的尺寸。
- **使用與製造條件**：重量、受力方向、材料、製造方式、可用空間及需要的輸出格式。

建模前，agent 會簡要整理已確認的需求、可調參數、設計假設、待量測項目與預計檢查方式。例如設計飛機模型支架時，會詢問噴嘴固定方式、機腹支撐位置、起落架避空、角度基準及模型重量。使用者要求先做概念版時，可以採用明列的暫定尺寸；實際配合、承重及製造可行性仍需量測與驗證。

修改範例：

> 將外殼寬度從 100 mm 改成 115 mm，壁厚改成 2.5 mm，並把開孔向右移動 4 mm。請修改現有參數和特徵，再重新檢查並匯出。

若你的 agent 已將此資料夾安裝為可自動發現的 skill，也可以使用 `$cad`；否則直接指定上面的 `skills/cad/SKILL.md` 路徑即可。

## 輸出檔案

| 格式 | 用途 |
| --- | --- |
| `.sc` | SolidCraft 原生設計；保留可編輯的參數、草圖與特徵歷史 |
| `.step` | 實體幾何交換，適合交給其他 CAD 軟體或加工廠 |
| `.stl` | 三角網格，常用於 3D 列印；交付時應註明單位 |
| `.3mf`、`.obj`、`.urdf` | 依工作需求及目前 SolidCraft MCP 版本支援的功能匯出 |

STEP、STL 等匯出檔是當下模型的快照；設計修改後需要重新匯出。原生 `.sc` 是持續編輯時的主要檔案。

## 專案結構

```text
skills/cad/
├── SKILL.md                 # CAD 設計與建模流程
├── references/              # 參數、約束、製造、組裝與驗證知識
└── backends/solidcraft.md   # SolidCraft MCP 操作與限制

docs/solidcraft-cad-refactor.md          # 遷移稽核與設計決策
docs/solidcraft-upstream-validation.md  # 驗證結果與涵蓋範圍
```

CAD skill 的工程推理與 SolidCraft 執行細節分開維護。本專案仍包含從上游繼承的其他 skills、插件設定與工具；它們尚未全部遷移至 SolidCraft。

## 測試與現況

本機曾使用 **SolidCraft 0.27.0** 的獨立 headless MCP 工作階段，完成 19 個對應案例，以及參考上游的幾何量測與 STEP 檢查。結果與限制詳見[驗證報告](docs/solidcraft-upstream-validation.md)。這些結果涵蓋已測案例，並不代表所有模型、桌面 UI 或製造條件都已驗證。

SolidCraft MCP 目前沒有經確認可涵蓋所有拓撲、自交與布林穩健性的單一通用驗證工具。模型量測、裝配間隙與製造檢查應依實際用途和材料判讀；無法完成的檢查須明確標示。

此儲存庫保留的舊版插件／其他 skills 仍可能使用 cadgen。其 runtime 的 Python 版本需求以 `packages/cadgen/pyproject.toml` 為準；如需關閉該舊 runtime 的遙測，可執行：

```bash
uvx --no-config --managed-python --python 3.13 --from cadgen==0.8.3 cadgen telemetry off
```

## 授權與致謝

本專案根據 [MIT License](LICENSE) 發布，並保留原有版權與授權聲明。

CAD skill 改編自 [earthtojake/text-to-cad](https://github.com/earthtojake/text-to-cad) 的工作流程與內容；感謝原作者及貢獻者。SolidCraft 是獨立的 CAD 專案，請參考其[官方網站](https://solidcraft.io/)與[原始碼儲存庫](https://github.com/tangency-labs/solidcraft)。
