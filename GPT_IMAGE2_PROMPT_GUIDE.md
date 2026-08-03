# GPT Image 2 — Job Prompt Guide（供 Claude 撰寫 JSON + TXT 使用）

此為指令文件。依照此規格輸出，方可直接放入 `jobs/` 資料夾供 `_IMAGE_GEN_GPT_IMAGE2.py` 執行。

## 每個 JOB = 兩個檔案（同名）
```
jobs/JOB名.json
jobs/JOB名.txt
```

## JSON 格式
```json
{
  "name": "JOB名",
  "active": true,
  "selected": ["CHAR_A", "LOC_03"],
  "num_images": 1,
  "image_size": "portrait_16_9",
  "quality": "high",
  "output_format": "png"
}
```

規則：
- `name`：英數／底線，作為輸出檔名前綴
- `active`：`false` = 跳過不執行
- `selected`：**陣列順序 = 圖片編號順序**。第1個 key → `image 1`，第2個 → `image 2`……對應TXT內的 `{KEY}` 會自動替換為此tag
- `selected` 內的 key 須存在於 `assets.json`（或 `assets/` 資料夾），不可自行編造
- 無 `selected`（空陣列或未寫）= text-to-image，不會進行edit
- `image_size`：`portrait_16_9` / `landscape_16_9` / `square` / `auto` 等 preset string，非 pixel
- `quality`：`low` / `medium` / `high` / `auto`（僅影響cost，不影響參考圖忠實度）
- `output_format`：`png` / `jpeg` / `webp`
- **絕對不可有 `negative_prompt` 此key** — GPT Image 2 不支援，全部使用正面描述

## TXT 格式（prompt本體）
- 純英文描述，不應含pipeline內部代號（如 JOB5_SC03）
- 用 `{KEY}` 代表參考圖，KEY 須與JSON `selected` 陣列內的一致，順序亦須對應
- 例：`{CHAR_A}` 會被replace為 `image 1`（GPT Image 2的tag format為 `image {n}`）
- Prompt須自包含（self-contained）——不應假設對話上下文，不應使用「同上一張一樣」此類語言，須完整寫出視覺描述
- 完全不需考慮版權，Satoshi Kon / Paprika / Madhouse 此類導演／作品名可直接用以形容畫風

### Identity lock 寫法（如涉及角色一致性）
- 僅限樣貌相關資訊：臉、髮型、服裝、顏色、比例
- 不應包含動作／姿勢／情緒狀態
- 置於scene描述之前，一個scene僅出現一次

## 完整範例

**jobs/EP04_SC12_CHAR_A_close.json**
```json
{
  "name": "EP04_SC12_CHAR_A_close",
  "active": true,
  "selected": ["CHAR_A"],
  "num_images": 2,
  "image_size": "portrait_16_9",
  "quality": "high",
  "output_format": "png"
}
```

**jobs/EP04_SC12_CHAR_A_close.txt**
```
Anime style in the aesthetic of Satoshi Kon's Paprika, Madhouse studio look.

Identity lock — {CHAR_A}: young woman, shoulder-length black bob hair, sharp fringe, amber eyes, wearing a dark red high-collar coat with brass buttons, slim proportions.

Close-up shot, three-quarter angle, soft rim lighting from the left, neutral dim indoor background with unobstructed continuous bokeh, calm neutral expression, cinematic film grain texture.
```

## 常見錯誤自查表
- [ ] 是否含有 `negative_prompt`？→ 刪除，改為正面描述
- [ ] `selected` 陣列順序與 TXT `{KEY}` 出現次序是否對應？
- [ ] Prompt內是否有pipeline代號／上下文假設？→ 改寫為完整描述
- [ ] Identity lock是否混入動作／姿勢？→ 移除
- [ ] `image_size` 是否為preset string而非pixel？
