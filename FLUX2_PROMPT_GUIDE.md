# FLUX.2 Job Prompt 產生指南（供Claude參閱）

現在的任務為：與用戶討論清楚其所需畫面之後，產出**一對檔案**——`{name}.json` 與 `{name}.txt`——此兩個檔案會直接放入一個Python batch script的 `jobs/` 資料夾內執行，不再經人手修改。因此格式須嚴格依循以下規格。

---

## 一、輸出格式規格（須嚴格遵循）

### 1. `{name}.txt` —— 純prompt文字

- 內容即為最終傳給FLUX.2的prompt，可用自然語言或JSON scene格式（見下方第二節）。
- 若該job會用到參考圖，於prompt內以 `{KEY}` 此花括號寫法指涉某張參考圖，**不應自行寫入「image 1」「image 2」此類字眼**——script會自動將 `{KEY}` 換成模型適用的tag格式。
  - 例如：`保持{CHAR_Y}的樣貌不變，換上{OUTFIT_A}套裝，背景換為{SCENE_15}的場景`
- `{KEY}` 須與 `.json` 內 `selected` 陣列的KEY**一字不差**（大小寫、底線皆須match）。

### 2. `{name}.json` —— job設定

```json
{
  "name": "ep04_scene12_closeup",
  "active": true,
  "selected": ["CHAR_Y", "SCENE_15"],
  "num_images": 1,
  "image_size": "landscape_16_9",
  "output_format": "png"
}
```

可用欄位：

| 欄位 | 必要？ | 說明 |
|---|---|---|
| `name` | ✅ | job識別名，用作output檔名prefix；須與`.json`/`.txt`檔名（不含副檔名）一致 |
| `active` | 建議填寫 | `true`執行此job，`false`跳過。未填寫預設為`true` |
| `selected` | 有參考圖時才需要 | 陣列，依序列出此job使用的assets KEY（對應`assets/`資料夾內的檔名，或`assets.json`定義的別名）。純文字生圖（無參考圖）則留空陣列 `[]` |
| `num_images` | 可選 | 一次生成數量，預設1 |
| `image_size` | 可選 | `square_hd` / `square` / `portrait_4_3` / `portrait_16_9` / `landscape_4_3` / `landscape_16_9`，或`auto` |
| `output_format` | 可選 | `png`（無損）或`jpeg`（較小檔案） |
| `seed` | 可選 | 欲重現同一結果時填寫 |
| `mask_image_url` | 局部編輯時才需要 | 有mask時才加 |
| `safety_tolerance` / `enable_safety_checker` / `sync_mode` | 可選 | 一般無須理會，有特殊需求時才加 |
| `guidance_scale` / `num_inference_steps` / `negative_prompt` / `loras` | 僅限Flex/Dev variant | 使用`pro` variant時此類欄位會被忽略（pro為zero-config），僅Flex或Dev才傳入此類欄位 |

⚠️ 一個job最多可使用多少張參考圖（`selected`的長度），視乎使用的FLUX.2 variant，Pro官方確認上限為9張，其他variant上限未必相同，不清楚時應向用戶確認。

---

## 二、Prompt寫法原則（自然語言 / JSON scene 兩種寫法）

### 自然語言寫法 —— 四大支柱，順序影響權重

FLUX.2讀取prompt具有hierarchy，越前面的字權重越高，因此應永遠依此次序撰寫：

1. **Subject** —— 主體為何（人／物／產品）
2. **Action** —— 主體的動作、姿勢、狀態
3. **Style** —— 欲呈現的美術／攝影風格
4. **Context** —— 環境、燈光、氣氛、構圖

例：不應寫「用暖色調柔光做一張皮革手袋的圖」，應寫「奢華皮革手袋，放在大理石面上，柔和方向性燈光，暖琥珀色調」——主體優先，形容詞置於其後。

**顏色**：欲準確控制品牌色，直接以HEX code寫入prompt，例如「主色#FF6B35，輔色#004E89」。

**鏡頭語言**（善用此類詞彙FLUX.2表現會顯著提升）：
- 角度：eye level（自然）／low angle（有氣勢）／bird's-eye（建築感）／over-the-shoulder（親密感）
- 鏡頭：14-24mm（廣角戲劇性）／35-50mm（自然）／70-85mm（人像）／100mm+（長焦）
- 光圈：f/1.4-f/2.8（淺景深）／f/4-f/5.6（中等）／f/8-f/16（全清）

**燈光**盡量寫具體方向與性質，例如「柔和漫射攝影棚燈光，從上方打落」，不應僅寫「打好看的燈」。

**畫面內須有文字**（例如海報、產品標籤）：以引號寫明實際文字內容，並指定位置與字體風格，例如「雜誌封面，標題『FUTURE DESIGN』粗體無襯線字」。

⚠️ 不應將互相矛盾的形容詞併於一處（例如「大晴天」+「陰暗戲劇性陰影」），會使模型混淆。

### 有參考圖 / 局部編輯（使用`edit`模型）時

僅須說明**欲修改的部分**，無須重新形容整張圖。例如不應寫「整張圖換成穿紅色裙站在公園有樹」，應寫「將裙的顏色換為紅色」或「加入暖色夕陽燈光」——模型會自行保留其他未提及的部分。

### JSON scene寫法 —— 複雜多主體場景才需使用

單一主體、快速測試、隨意創作 → 使用自然語言即可。
多主體、須符合品牌規格、複雜相機參數 → 使用JSON scene結構，例如：

```json
{
  "scene": "Modern minimalist kitchen",
  "subjects": [
    {"type": "ceramic coffee mug", "description": "matte black finish, gold interior, steam rising", "position": "foreground"}
  ],
  "style": "lifestyle photography",
  "color_palette": ["black", "gold", "white"],
  "lighting": "soft morning sunlight from left",
  "mood": "calm and sophisticated",
  "camera": {"angle": "slightly low", "lens": "50mm", "f-number": "f/2.8"}
}
```

此種JSON scene寫入**prompt本身**（即`.txt`檔的內容可為此段JSON字串），並非job設定`.json`——兩者不應混淆。

---

## 三、Claude的工作流程

1. 詢問用戶所需畫面（主體、動作、風格、環境），若用戶尚未講清楚，應主動詢問：是否需要參考圖？欲以文字或JSON scene寫法？欲跑`pro`或`flex`/`dev` variant？
2. 若用戶已提供assets KEY清單（例如CHAR_Y、SCENE_15），則使用該等KEY撰寫`{KEY}`；若未提及，則假設為純文字生圖（`selected: []`）。
3. 一次性輸出兩個code block：
   - 第一個為 ```json``` block，內容為 `{name}.json`
   - 第二個為 純文字 block（或獨立code block），內容為 `{name}.txt` 的prompt
4. `name`須用英文／數字／底線，不應有空格或中文，方便直接作為檔名。
5. 若用戶欲一次性製作多個變奏（例如同一主體、數個不同構圖），則每個變奏各自輸出一組`.json`+`.txt`，`name`加編號區分（例如 `ep04_scene12_v1`、`ep04_scene12_v2`）。
