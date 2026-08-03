SEEDANCE 2.0 官方規則 — 通用版
================================
（任何project都可以直接使用此文件，內容為官方平台規則／prompt句式，不涉及特定project的美術風格或pipeline架構）

JOB 與 SHOT 關係
----------------
- 1 JOB = 1條 Seedance 生成影片
- ⚠️官方硬限制：單次生成 duration 必須為4-15秒（並非僅「上限15s」，4秒以下會被API拒絕）
- 1個JOB可含1~N個SHOT，SHOT時長總和須介於4-15s之間
- 排JOB時需計算：最後一個SHOT剩餘時長不足4s，應移至下一個JOB或合併，不可強行塞入
- 多於1個SHOT時，SHOT之間以 "CUT TO" 分隔
- SHOT若有對白，直接寫入該SHOT段落
- ⚠️官方提示：不應在prompt文字內為個別SHOT寫入精確秒數（例如"0-3秒"），
  timing支援不穩定，強制限時可能導致生成異常；應交由model自然分配pacing

IDENTITY-LOCK 規則
------------------
- 每個出現於SCENE內的角色／物件，identity-lock句只在SCENE開場寫一次（SCENE描述之前），
  不在個別SHOT內重複
- Identity-lock句只可包含「外觀」語言（臉、髮型、服裝、形狀、顏色、比例）
- 不可包含pose、動作、狀態、位置描述（例如 "in ready-to-launch position"、
  "unaltered by pose or angle" 之後接位置字眼）
  此類屬於SHOT動作描述，應寫在SHOT段落內
- 官方原本推薦句式：
  - 基本定義：Define [2-3個穩定靜態特徵，如服裝／髮型／外觀] in <Image/Video_N> as <Subject_N>
  - 簡單場景（未預先定義）：每次提及即以 <Subject_N>@<Image_N> 標記binding關係，
    例：Zhang San@Image 1
  - 預先定義場景：全片統一使用同一名稱（例：定義為「警察」後全程使用「警察」，不可中途改稱）
  - 官方提醒：核心特徵選取2-3個即可，須避免同一subject出現矛盾特徵描述

PREVENTATIVE CLAUSE 分層規則
----------------------------
Preventative clause分三層寫入：
- 全局性（整個SCENE皆須遵守的限制，例如不要warm light、不要色彩飽和度突變）
  → 寫在 STYLE BLOCK（以正面敘述方式）
- 單一SHOT性（僅該SHOT須遵守的限制，例如不要zoom）
  → 寫入該SHOT的鏡頭描述句內（以正面敘述方式）
  例：「TRUE SLOW PUSH-IN: camera physically drifts forward... NOT a zoom, focal length stays fixed throughout.」
- 畫面雜訊／meta元素性（deformation、flicker、subtitle、watermark等，與主體／鏡頭控制無關）
  → 寫入片尾 CONSTRAINT 句（完整句negative，詳見下方規則）

⚠️ 官方核心原則（來自官方prompt guide）
--------------------------------------------
1. Seedance = multimodal AI director，內部拆解為「spatial layer（畫面有什麼）」與
   「temporal layer（時間點變化）」兩個維度理解。良好的prompt是engineering指令，
   而非copywriting：who + where + doing what + how camera moves + 時序。
2. 官方advanced formula（元素排序）：
   precise subject → action details → scene/environment → lighting & color tone
   → camera movement → visual style → image quality → constraints
   （即主體／動作優先，style／quality／constraint置後）
3. Shot sequencing使用「Shot 1 / Shot 2 / Shot 3」timeline storyboard。
   ⚠️ 官方明確指出：不應硬性限制每個shot的精確秒數（例如0-3秒），
      timing支援不穩定，強制限時可能導致生成異常。
4. 動作描述：具體到身體部位（手／腳／頭／肩）+ 幅度／速度／力度量化。
   官方建議優先使用slow/gentle/continuous的小動作，避免sprint/big jump/violent roll等
   high-burst大動作（穩定性較差）。
5. 單一SHOT只寫1種camera movement，不應同時要求push+pull+pan+move（會造成抖動）。
6. 情緒須外化為具體動作（不應寫"very sad"，應寫"lowering head, shoulders trembling,
   eyes reddening"），官方備有情緒→動作對照表可供參考。

⚠️ 官方符號規範（特殊符號標記資訊類型）
--------------------------------------
官方建議以符號區分不同資訊類型，協助模型準確理解：
- 音樂：（）  例：（fast-paced rock music playing in background）
- 音效：<>  例：< dog barking in the distance >
- 對白：{}  例：{Hello, world}  （非中英文須標示語言，如 says in Japanese {こんにちは}）
- 字幕：【】 例：【Chapter One: Departure】
註：（）<>音效符號依SHOT-level寫法（詳見下方「AUDIO 文字描述」章節）；
   {}對白與【】字幕亦依SHOT處理。

⚠️ Character ID drift（換臉／漂移）官方對策
------------------------------------------
症狀：生成角色中途「換臉」、變得與名人相似而遭審查封鎖。
根因：face reference強度不足（混合圖過雜、face佔比過小）。
對策：
- 除全身圖外，另準備乾淨headshot（僅頭部、最好無表情、去除背景／肩頸干擾）
- prompt須說明：<Subject 1> 以 image1(headshot) 作為facial reference、image2(full-body) 作為styling reference
- 越需要精準reference的asset，在prompt／selected中應越前置（官方："place important assets first"）
- ⚠️不應使用multi-view／三視圖作為角色reference——模型容易誤判為多個不同subject，反而加劇drift
  （turnaround sheet應裁剪為單張frontal portrait）

⚠️ Asset配置策略（官方建議）
----------------------------
官方建議每JOB使用4-5個asset即可，不應用盡上限（過多asset會使模型難以判斷優先次序，容易造成style衝突／subject模糊）。
四種功能角色：角色錨定(character image) + 場景定調(scene image) +
運鏡參考(camera movement video) + 節奏氛圍(audio)。
建議組合：1-2張角色圖 + 1張場景圖 + 1條運鏡video + 1條audio。

⚠️ 其他任務類型官方句式（除Multimodal Reference外，其餘僅供參考）
----------------------------------------------------------------------
- Video Editing（局部／全局修改原片，未提及的部分預設不變）：
  加元素：Clearly describe [Element_Features] + [Timing] + [Location]
  改元素：Strictly edit <Video_N>, and modify [Original_Characteristic] in it to [New_Characteristic]
  刪元素：明確說明欲刪除之內容；欲保留之內容須特別強調方能保留完好
- Video Extension（沿時間軸延續原片，audio-video風格／主體／敘事須連貫）：
  延伸：Extend <Video_N> forward/backward to generate...
  多段接駁：<Video_1> + [Transition_Description] + followed by <Video_2> + ...
  ⚠️官方提醒：此類task須以"<Video_N>"直接指代，不應使用"reference <Video_N>"
  （會被誤判為reference task）
- Combined（reference一個asset + edit另一個asset）：
  Reference [Reference_Dimension] of <Image/Video_N>, strictly edit <Video_X>, [Specific_Edits]

⚠️ 官方FAQ重點（實務問題與對策）
-----------------------------------------------------
1. 對白語言須統一：dialogue內不應中英文夾雜（專有名詞除外）
2. 角色數量：reference真人數量建議不超過4人，超過則穩定性明顯下降
   （容易出現人數錯亂、重複角色）；多人場景建議分組生成image、再以image生成video
3. 片尾雜音：有narration的片段尾段容易出現abrupt click／斷尾雜音，
   後製可用Premiere/CapCut volume envelope做fade-out處理
4. Video extension接駁跳幀：前後兩段連接處容易出現畫面跳動／回退，
   後製對策：前段尾剪去6格、後段頭剪去1格，逐個接駁點重複處理
5. 反覆extension畫質衰退：多次continuation會累積畫質降級（尤其臉部易出現色塊），
   對策：將原片轉為「純白3D模型」中繼版本再continuation
   （prompt參考："Convert the video into a white 3D model, no color, no texture, no shadows"）
6. 特效不符預期（例如倒數數字亂跳）：文字描述特效效果不穩定，
   可改用reference video定義該特效（"the way X appears should reference video 1"）
7. 中文發音錯誤（如有中文對白）：多音字／生僻字容易讀錯，
   可用同音字替換（例："螭龙山"改為"吃龙山"）以協助發音準確——此僅為workaround，不保證100%
8. Reference audio聲線不準確：欲準確還原某把聲音，prompt須加入聲音特質形容詞
   （例："Use the low, thick, warm, finely grainy middle-aged male voice of @Audio 1 to say"），
   台詞語氣風格越貼近reference audio，還原度越高

⚠️ 排序原則
-----------
Seedance 2.0 開頭20-30字權重最高，模型以此鎖定主體，
因此主體（IDENTITY LOCK）應優先，STYLE BLOCK此類全局形容詞應置後，
以避免長篇style文字稀釋開頭對主體的鎖定。

⚠️ Pipeline共用（所有project同一套py/json結構）
----------------------
{KEY} Tag系統規則
- Prompt文字內以 {KEY} tag（如 {CHAR_A}, {PROP_2}）標記角色／物件出現位置
- {KEY} 須對應 assets.json 內的key
- JSON只有一個 selected array，image/video/audio不分開三個array，
  由script讀取URL副檔名自動判斷media type：
  - .mp4 / .mov / .webm → 判定為video，依序resolve為 Video 1/2/3
  - .wav / .mp3 / .m4a / .flac → 判定為audio，依序resolve為 Audio 1/2/3
  - 其他（.jpg/.png等） → 判定為image，依序resolve為 Image 1/2...
  三種type各自獨立由0開始計數，不會共用同一編號
- 每次須同時輸出：(a) prompt文字（含{KEY} tag） (b) 對應的selected array
  （順序須與tag出現次序相符——同一array內image/video/audio key可混合寫入，script會自動分類）
- 內部代號({KEY})僅出現於此中介tag系統，不會直接寫死為"image1"/"video1"/"audio1"等字眼
  （make_jobs()負責將{KEY}轉為"Image N"/"Video N"/"Audio N"文字，依URL副檔名判斷，而非依assets.json的category）
- 額外獨立reference：JSON可加`"ref_videos": ["URL", ...]`，用於放置不在assets.json內的獨立video URL
  （此類不會經{KEY} tag resolve，直接於build_content()加入content array，不會出現於prompt文字內）
- ⚠️ script（Seedance_gen_BYTEPLUS.py）副檔名判斷包含.webm/.m4a/.flac，
  此類格式在官方文件中未見明文支援，若使用須留意是否會分類錯誤或遭API拒絕

⚠️ {KEY}資產tag（{CHAR_A}等）與官方{}對白符號不會衝突：
   make_jobs()在送prompt至API之前，已將{CHAR_A}此類{KEY}轉換為"Image N"文字，
   送達model時prompt內已不含{CHAR_A}此類字串——
   剩餘的{}僅為手寫的對白內容，兩者運作階段完全分開，不會互相污染。

IDENTITY-LOCK句式（本pipeline實際用法）
使用「{KEY}: preserve exact...」句式：
  {KEY}: preserve exact [face/hairstyle/outfit 或 shape/color/proportions] from reference.
（官方原本推薦句式見上方「IDENTITY-LOCK 規則」，供debug／對照參考）

JOB JSON 結構說明
每個JOB對應一個 .json（settings）+ 一個 .txt（prompt文字），JSON欄位如下：

{
    "name": "JOB編號_SC場次_SHOT範圍",      // 檔名／job識別，通常對應SCENE+SHOT區間
    "active": false,                        // true=此JOB會被執行；false=跳過
    "selected": ["CHAR_A", "PROP_A1", ...], // 此JOB使用的asset key（image/video/audio可混合），順序=tag被resolve為Image N/Video N/Audio N的順序
    "ref_videos": [],                       // 額外獨立video reference URL（不在assets.json內）
    "mode": "REFERENCE",                    // REFERENCE=以selected reference image生成；FIRSTFRAME=僅first_frame；LASTFRAME=僅last_frame；FIRSTLAST=first+last frame同時使用
    "first_frame": "",                      // mode=FIRSTFRAME/FIRSTLAST時必填，填asset key（非URL）——script會以ASSETS[key]查找URL，⚠️不可放入selected array
    "last_frame": "",                       // mode=LASTFRAME/FIRSTLAST時必填，填asset key（非URL），邏輯同上，⚠️不可放入selected array
    "ratio": "9:16",                        // 畫面比例，固定依STYLE BLOCK
    "duration": 12,                         // 秒數，= 此JOB內所有SHOT時長總和，官方硬限制4-15秒
    "resolution": "480p",                   // draft用低解析度，定稿再提高
    "seed": 2026,                           // 固定seed；retry邏輯會以random覆蓋
    "audio": "",                            // ⚠️已棄用 — 原本lookup audio_descs.json，已改用SHOT-level寫法，此欄位保留但script不再讀取
    "content_filter": false                 // 內容審查開關（false=關閉審查，billing +10%）
}
// ⚠️ generate_audio並非job JSON欄位——create_task()寫死generate_audio=True送往API，job.json加此key亦無效，不會被讀取

- selected array數量／順序須與prompt .txt內{KEY}出現次序相符，錯位會導致tag對錯圖
- ⚠️ first_frame/last_frame獨立於selected array之外，填ASSET KEY（非URL，與selected array內的key格式相同），script會以ASSETS[key]自動查找URL，不經{KEY} tag／prompt文字resolve
- ⚠️ LASTFRAME mode（僅last_frame、無first_frame）script有實作，但官方文件僅列出「first_frame單獨」與「first+last同時」兩種組合，未見「last_frame單獨」此種——使用此mode前建議先實測確認API是否接受
- reference asset送API時的role值：`selected`內的image/video/audio分別使用`reference_image`/`reference_video`/`reference_audio`（script自訂，未必為官方標準field名，但為現行實際行為）
- duration須等於JOB拆分邏輯計算出的累計秒數，不可隨意填寫
- active控制單一JOB是否被pipeline執行，方便批量開關而無須刪除JSON
- JOB JSON無`negative_prompt`欄位（不存在此API參數），constraint須寫在.txt prompt文字最後

通用PROMPT骨架
--------------
[IDENTITY LOCK BLOCK — 此SCENE使用的所有角色／物件，各一句]
[STYLE BLOCK]
[VIDEO REF BLOCK — 如selected有video reference，官方句式：
 Reference <Action/Camera_movement/Style> in <Video_N> to generate...（可省略）]

SCENE_XX
[場景描述 — 地點、固定視覺元素、氛圍，跨SHOT共用]

SHOT 1 — [Xs]
[角色動作(具體動詞、現在式)/鏡頭(單一主要camera movement)/構圖/對白(如有)/單一SHOT性preventative clause]
[(音樂描述，如有)]
[<音效描述，如有>]

CUT TO

SHOT 2 — [Xs]
...

[...累計最多15s...]

[CONSTRAINT 句 — 完整句negative寫法]

⚠️ AUDIO 文字描述（SHOT-level做法）
-----------------------
官方建議audio資訊依individual SHOT撰寫（每個SHOT尾段以符號標記：音樂用()、音效用<>），
而非整個JOB結尾以一句概括。

寫法：
- 每個SHOT自身尾段加(音樂)/<音效>，不應以整個JOB一句總括的寫法處理
- (音樂)/<音效>只描述**該SHOT內已存在之事物**所發出的聲音——
  即該SHOT角色動作描述句已提及的動作／物件，方可在audio處配置對應聲音
  （例：SHOT描述角色正在行樓梯，方可寫<footsteps on stairs>；
  不可SHOT未提及之事物，僅於audio處首次出現）
- 此為防止視覺bleed的核心防線：audio不可引入SHOT視覺描述未提及的新物件／新動作，
  純粹為已寫好的畫面配聲，不做「敘事補完」
- 環境音（風、人群murmur此類無具體物件的background texture）風險較低，可照常撰寫
- 對白繼續使用{}，依SHOT內角色台詞句撰寫，不須獨立成行
- 若SHOT-level寫法仍出現unrelated視覺內容，退後策略：僅保留純環境音texture
  （風／人群／室內迴響此類無具體物件的），有具體物件聲響的一律移除，交由後製處理

⚠️ CONSTRAINT 句規則（官方認可）
-----------------------
位置：整個prompt最後（官方Example置於全片描述最後一句）
✅ 官方確認：Seedance支援完整句negative寫法，不必勉強改為全部正面敘述。
   官方原文示範用語：
   - 防字幕："keep it subtitle-free" / "avoid generating any text or subtitles"
   - 防logo："do not generate a logo"
   - 防浮水印："do not generate a watermark"
   - 防變形／閃爍："face remains stable without deformation; movements natural and smooth, no stutter or flicker"
   - 防style drift："2D Japanese anime style"（以正面style word鎖定，官方建議）
   - 防twin／分身（多角色場景官方固定句）：
     "Throughout the video, characters with completely identical appearance, clothing, and accessories are prohibited.
      Do not generate duplicate avatars or a twin effect. Keep only a single corresponding character in the same frame."

寫法：使用完整句子（"do not generate X" / "X remains stable, no Y"），而非"No X, No Y"逗號堆疊。
   （逗號堆疊僅為坊間慣例；官方示範全部為完整句，句式清晰模型理解更穩定）

⚠️ 但此類constraint皆無法100%杜絕（官方明確指出subtitle/twin/watermark均僅為降低機率），
   如出現殘留，可配合：landscape生成後再crop（字幕機率較低）、reference圖先去水印／文字、換seed重生。

分層原則：
- 角色外觀限制 → IDENTITY LOCK BLOCK（正面preserve exact）
- 單一SHOT鏡頭限制 → 該SHOT描述句
- 全局畫面／style限制 → STYLE BLOCK或片尾constraint句

JOB拆分邏輯範例
--------------
SHOT1  4s  -> JOB1
SHOT2  6s  -> JOB1（累計10s）
SHOT3  7s  -> 超出15s，移至JOB2
⚠️ 若最後剩餘的SHOT單獨取出少於4秒（例如僅剩2s），
   不可自行開設一個JOB（低於官方4秒下限會被API拒絕）——
   須與前一個JOB合併，或調整該SHOT時長至≥4s

REFERENCE 數量上限（官方準確數字）
------------------
- image：1-9張（R2V multimodal reference），每張<30MB
- video：最多3條，單條時長2-15秒，全部reference video總長度合計不可超過15秒
  格式限定：.mp4 / .mov 為官方文件列明支援（.webm未見官方文件確認）
- audio：最多3條，單條時長2-15秒，全部reference audio總長度合計不可超過15秒
  格式限定：.wav / .mp3 為官方文件列明支援（.m4a/.flac未見官方文件確認，使用前建議實測）
- 三種type獨立計數，不會共用同一上限

⚠️ MODEL LIMITATION / CONSTRAINT（官方文件對照）
-------------------------------------
- ⚠️官方明文：Seedance 2.0系列**不支援直接上傳含真人臉孔的reference image/video**
  （如reference圖來源混入真人相／真人動作capture片，須留意此限制；
  官方另有「Create with ease」文件說明變通方法，需要時可再查閱）
- JOB JSON無獨立`negative_prompt`欄位／API參數，不應在JSON內加入（此為真實限制）
- 但prompt文字內可使用negative句式——官方明確示範使用
  "do not generate X" / "keep it X-free" / "no stutter or flicker" 此類完整句
  （「prompt完全不可使用negative」此說法為錯誤，Seedance與GPT Image 2不同）
- 寫法使用完整句子，而非"No X, No Y"逗號堆疊（官方示範全部為完整句）
- ⚠️ constraint不保證100%生效（官方明確指出subtitle/twin/watermark均僅為降低機率），
  殘留時可配合landscape再crop、reference先去文字、換seed重生
- ⚠️ duration官方硬限制4-15秒（不僅為「上限」，亦有下限）
- ⚠️ 單一JOB reference角色人數官方建議不超過4人，
  超過4人穩定性下降、容易出現人數錯亂／重複角色
