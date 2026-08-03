# SEED AUDIO — JSON + TXT 生成指南（供 Claude 使用）

此為指令文件，供另一個 Claude 對話用以輸出「一對」檔案：
`{job_name}.json` + `{job_name}.txt`，放入 `_EP04/04_AUDIO/jobs/`（或對應專案 jobs 資料夾），
須與 `SEEDAUDIO_MAIN_gen_FAL.py` 直接契合，無須人手修改即可執行。

模型：`bytedance/seed-audio-1.0`（經 fal.ai）

---

## 1. 輸出兩個檔案

**`{job_name}.txt`** — 純 prompt 內容，內含 `{KEY}` placeholder（見第3節）
**`{job_name}.json`** — job 設定，格式如下：

```json
{
  "active": true,
  "mode": "audio",
  "selected": ["CHAR_A_VOICE", "CHAR_B_VOICE"],
  "output_format": "mp3",
  "sample_rate": 44100,
  "speed": 1.0,
  "volume": 1.0,
  "voice": null,
  "pitch_shift": null
}
```

### 欄位規則
| 欄位 | 必填 | 說明 |
|---|---|---|
| `active` | 是 | `false` 會被 script skip |
| `mode` | 是 | `"audio"` / `"image"` / `"none"`（見第2節） |
| `selected` | 是 | asset key 陣列，順序 = `{KEY}` 編號意圖，順序錯誤 = @Audio 對應錯位 |
| `output_format` | 是 | 沿用專案 `defaults.json` 慣用值，無特別要求可依舊 job 抄用 |
| `sample_rate` / `speed` / `volume` | 是 | 同上，依 `defaults.json` 慣例，不應隨意改動數值 |
| `voice` | 否 | 無reference voice時使用，內建 voice id，不清楚時填 `null` |
| `pitch_shift` | 否 | 無特別需要填 `null`，不應無端加入 |

`selected` 內的 key 須存在於根目錄 `assets.json`（跳過 `_note` key），不可自行編造 key 名。

---

## 2. mode 對應邏輯（此為最重要部分，須嚴格遵循）

`build_prompt()` 會處理 `.txt` 內的 `{KEY}`：

- **`mode = "audio"`**：`{KEY}` 會依 `selected` 順序換成 `@Audio1`、`@Audio2`、`@Audio3`。
  → 此為「使用已存 reference 聲音」（TA2A / TTS with saved voice）。
  → **prompt 內不應自行寫入 `@Audio1`，須寫 `{KEY}`**，交由 script 自動轉換。
  → 最多 3 個 reference（`selected` 最多3個 key）。

- **`mode = "image"`**：`{KEY}` 會被移除（image reference 並非以 @tag 引用，而是另以 `image_url` 傳入 API）。
  → `selected` 僅取第一個 key 作為 image_url，其餘忽略；prompt 純文字無需提及image。

- **`mode = "none"`**：`{KEY}` 不會被替換，`selected` 應為空陣列 `[]`。
  → 純 T2A（text-to-audio）或全新聲音的 TTS，全部聲音靠文字形容，不使用任何 reference。

---

## 3. `{KEY}` 命名規則

- `{KEY}` 須與 `assets.json` 內的 asset key 完全一致（大小寫敏感）。
- `{KEY}` 在 txt 內出現的次序，須與 JSON `selected` 陣列次序一致 —
  第一個出現的 `{KEY}` 對應 `selected[0]`，依此類推。
- 例：`selected: ["DEX_VOICE_REF", "PRIYA_VOICE_REF"]`
  → txt 內第一次出現的 reference 聲音須寫 `{DEX_VOICE_REF}`，
     第二個寫 `{PRIYA_VOICE_REF}`（不可對調，亦不可遺漏）。
- 一個 `{KEY}` 可在 txt 內重複出現多次（同一聲音講數句對白），
  重複出現時全部會換成同一個 `@AudioN`。

---

## 4. Prompt 寫法規則（內容遵循 Seed Audio 官方 prompting 邏輯）

### 4.1 語言
- 每個 prompt 僅使用一種語言（English 或 Chinese），開場清楚表明語言／口音，
  例如 `American accent`、`Cantonese` 等。混雜語言效果較差。

### 4.2 T2A / TTS 完整場景結構（sound-design-heavy，`mode: none` 或 `image`）
6 段式結構（不一定全部6段皆須具備，但順序不應打亂）：
```
1. [Genre + environment + mood]  — 風格／場景／情緒
2. [Continuous sound bed]        — 貫穿全場的底層環境音
3. SpeakerName (voice attrs + emotion + pace) delivery verb: "Dialogue."
4. [Concrete sound effect / transition] — 具體一次性音效
5. SpeakerName (...) delivery verb: "Dialogue."
6. [Silence / closing sound / music cue / fade] — 收尾
```
- Speaker 描述須包含：性別／年齡、口音、聲線質感、情緒、語速。
- 一個場景音效不宜貪多，一條主 sound bed + 數個精準 cue 已足夠。

### 4.3 TA2A（`mode: "audio"`，使用 reference 聲音）
- 用 `{KEY}` 代替 `@AudioN`，人物首次出場須形容聲線特徵（可簡短），
  之後對白可直接寫 `SpeakerName ({KEY}), emotion, verb: "..."`。
- 例：
  ```
  Dex ({DEX_VOICE_REF}), relaxed and amused, says: "Alright, what did you do this time?"
  Priya ({PRIYA_VOICE_REF}), defensive and fast, replies: "It means the story has paperwork."
  ```

### 4.4 建立 reference voice（`mode: "none"`, 用以生成一段乾淨單人聲音作為未來 reference）
- 單一講者，30秒內，70–85字，無音樂、無第二把聲、環境音極少。
- 內容結構：`[極輕微環境音].` + `SpeakerName (聲線描述) says, 語氣: "整段對白。"`

### 4.5 Sound bed / SFX 描述用語
- 用具體聲音名詞（roar of flames, metal groan, rain lashing windows），
  不應使用抽象詞（"scary sound"）。
- Continuous sound 以 `[...]` 開頭方括號獨立成行，一次性 SFX 亦以 `[...]` 表示。
- 對白台詞以 `"..."` 包住，delivery verb 置於台詞前（says / shouts / whispers / blurts）。

### 4.6 不應執行事項
- 不應使用負面／排除語言（"no background music", "avoid silence"）— Seed Audio 無 negative prompt 支援，
  一律使用正面描述（若需要靜默，應寫 `[A brief silence.]`）。
- 不應在 txt 內自行寫入 `@Audio1` 等字樣，一律使用 `{KEY}`，由 python 處理轉換。
- 不應在 prompt 內出現內部 pipeline 代碼（JOB編號、SC編號等），須完全自足、無上下文依賴。

---

## 5. 限制（Seed Audio 1.0 API 硬性限制）

- 單一 prompt 最多 **2,048 字元**。
- 單次生成音訊最長 **2 分鐘**；長內容須分段生成再後製拼接。
- 最多 **3 個 reference audio**，每個 **≤30 秒**。
- 僅支援 **英文 / 中文**。

---

## 6. 輸出格式要求

回覆時請分別以 code block 各自輸出：

1. `{job_name}.txt` 內容（純 prompt，含 `{KEY}`）
2. `{job_name}.json` 內容（合法 JSON，依第1節欄位）

檔名依專案慣用命名（例如 `EP04_SC03_AUDIO01`），兩個檔案 basename 須完全一致。
無需解釋原理，直接輸出檔案內容。
