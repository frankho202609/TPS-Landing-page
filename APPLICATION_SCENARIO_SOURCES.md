# 應用情境設定值、模型組合與來源依據

更新日期：2026-09-24

本文件對應 `index.html` 的應用情境按鈕。使用者點選情境後，HTML 會自動套用主模型、附加模型、輸入長度、最大輸出、Batch Size 與 Concurrent Users；以下逐項說明設定值及其依據。

## 證據分級

- **官方規格**：模型開發者官網或官方 Hugging Face 模型卡明確記載，例如任務類型、最大 context、輸入／輸出模態。
- **架構依據**：官方指南或原始論文支持模型之間的搭配方式，例如 RAG、function calling、STT → LLM → TTS。
- **本站情境基準**：官方沒有替每種應用規定固定 Input、Output、Batch 或 Concurrent Users。本頁為了比較 VGA／SoC，依任務規模設定一致的測試負載。這些值必須實測驗證，不能寫成原廠保證。

## ChatGPT 還是 Llama？

- ChatGPT／OpenAI API 是雲端服務，模型權重不會載入使用者的 VGA／SoC，因此不能用本頁的本機 VRAM、TPS 與 TTFT 直接估算。
- Llama 3.3 70B Instruct 有可取得的模型權重，可自行部署，適合本頁的本機硬體比較。它不是「Llama 的 ChatGPT」，而是另一個 instruction-tuned chat LLM。
- 因此，如果目標是資料留在本機、離線推論或比較 MSI PC 能否承載模型，本頁應使用 Llama 等 open-weight 模型；如果使用 ChatGPT，應另做 API 費用、網路延遲與資料治理評估。

來源：

- OpenAI 官方 Models 文件：https://platform.openai.com/docs/models
- Meta Llama 3.3 官方模型卡：https://huggingface.co/meta-llama/Llama-3.3-70B-Instruct

### 中文限制

Llama 3.3 官方列出的正式支援語言不包含中文。它可能產生中文，但 Meta 不建議在未正式支援的語言上直接部署而沒有額外評估、微調與安全控制。因此，個人知識庫選 Llama 3.3 的理由是「現有清單內可自行部署的大型通用模型」，不是「官方認證的最佳中文模型」。正式繁體中文產品應加入具官方中文／多語支援的通用 Qwen Instruct 類模型，再以中文 RAG 測試集比較。

## Context Session 與長度定義

### Context Session

Context Session 是模型在一次請求中必須同時處理的全部 token，通常包括：

1. System prompt 與安全規則。
2. 使用者目前問題。
3. 對話歷史。
4. RAG 取回的文件片段。
5. Agent 的工具定義、工具呼叫與工具結果。
6. 圖片／音訊經模型編碼後占用的 token（依模型而定）。
7. 為本次回答保留的輸出空間。

模型的 context window 是總上限，不是建議每次塞滿的長度。OpenAI Realtime API 也說明，可用輸入空間受 context window 與最大輸出 token 共同限制：https://platform.openai.com/docs/api-reference/realtime

### HTML 的 Input / Prefill

HTML 的 Input 是本頁用於硬體估算的**典型輸入上限**。輸入越長，Prefill、KV Cache 與記憶體需求通常越高。即使模型官方支援 128K 或 256K，本頁仍可用 4K、8K、16K 或 32K 作為可比較的工作負載。

### HTML 的 Maximum Output

Maximum Output 是單次回答最多新生成的 token，不包含 prompt。Hugging Face 對 `max_new_tokens` 的官方定義：https://huggingface.co/docs/transformers.js/api/utils/generation

本站使用下列情境基準：

- 512 tokens：摘要、客服、文件回答、單輪 Agent 結果。
- 1,024 tokens：需要較完整解釋與引用的知識庫回答。
- 2,048 tokens：較長程式碼、測試與除錯說明。

模型應在回答完成後提早停止；最大輸出不代表每次一定產生同樣長度。

## HTML 預設值總表

| 應用情境 | 主模型 | 附加模型 | Input | Output | Batch | Concurrent | 設定性質 |
|---|---|---|---:|---:|---:|---:|---|
| 個人知識庫 | Llama 3.3 70B Instruct | BGE Small ZH + BGE Reranker v2 M3 | 32,768 | 1,024 | 沿用全域值 | 沿用全域值 | 模型用途有官方／論文依據；長度為本站基準 |
| 程式開發 | Qwen2.5-Coder 32B Instruct | 無 | 8,192 | 2,048 | 沿用全域值 | 沿用全域值 | 模型用途有官方依據；長度為本站基準 |
| Agentic AI | Gemma 4 E4B | 外部工具，不是附加模型 | 16,384 | 512 | 1 | 4 | Function calling 有官方依據；負載值為本站基準 |
| 客服摘要 | Ministral 3 8B | 無 | 4,096 | 512 | 沿用全域值 | 沿用全域值 | 模型能力有官方依據；長度為本站基準 |
| 圖片生成 | Ministral 3 8B | Qwen2.5-VL 7B + SDXL | 4,096 | 512 | 沿用全域值 | 沿用全域值 | 組合角色有官方依據；文字長度為本站基準 |
| 文件問答 | Ministral 3 8B | GLM-OCR + BGE Small ZH + BGE Reranker | 8,192 | 512 | 沿用全域值 | 沿用全域值 | Document QA／OCR／RAG 有官方依據；長度為本站基準 |
| 視覺辨識 | Gemma 4 E4B | Qwen2.5-VL 7B + YOLO11 | 4,096 | 512 | 沿用全域值 | 沿用全域值 | Detection／VLM／Agent 角色有官方依據；長度為本站基準 |
| 語音客服 | Ministral 3 8B | Whisper Large v3 + Kokoro-82M | 4,096 | 512 | 沿用全域值 | 沿用全域值 | STT／LLM／TTS 角色有官方依據；長度為本站基準 |

> 「沿用全域值」表示目前情境卡沒有覆寫該欄位。若要讓情境比較可重現，後續建議在 HTML 為每張卡明確指定 Batch 與 Concurrent Users，而不是依賴使用者先前留下的值。

## 逐一說明 HTML 為什麼套用這些值

### 1. 個人知識庫

HTML：`llama3_3_70b` + `bge_small_zh` + `bge_reranker`，Input 32,768，Output 1,024。

- **模型組合**：這是 RAG 流程。Embedding 先檢索候選段落；Reranker 以 query＋passage 計算相關性並重排；LLM 根據取回內容回答。
- **為何使用 BGE Small ZH**：BAAI 官方標示為 Chinese embedding model，適合中文文件檢索。
- **為何加入 Reranker**：BAAI 官方說明推薦先以 embedding 取回 top-k，再用 cross-encoder reranker 選出更相關的結果。
- **為何 Input 32K**：可容納 system prompt、對話歷史與多段檢索內容，同時低於 Llama 3.3 的官方 128K 上限，避免把最大能力當成一般負載。
- **為何 Output 1K**：比摘要情境多，保留完整回答、條列與引述空間；屬本站應用基準。

來源：

- RAG 原始論文：https://arxiv.org/abs/2005.11401
- Llama 3.3：https://huggingface.co/meta-llama/Llama-3.3-70B-Instruct
- BGE Small ZH：https://huggingface.co/BAAI/bge-small-zh-v1.5
- BGE Reranker：https://huggingface.co/BAAI/bge-reranker-v2-m3

### 2. 程式開發

HTML：`qwen25_32b`，Input 8,192，Output 2,048，無強制附加模型。

- **模型組合**：Qwen 官方將 Qwen2.5-Coder 定位為程式碼生成、程式推理與修復。基本程式開發只需 Coder LLM。
- **為何沒有附加模型**：Repository index、code RAG、編譯器、測試執行器或 IDE Agent 都是可選工具，不是完成基本程式推論的必要模型。
- **為何 Input 8K**：用來容納需求、相關函式、錯誤訊息與少量上下文；Qwen 32B 官方最大 context 為 128K，但 8K 是較常見且可比較的單次任務基準。
- **為何 Output 2K**：程式碼、測試與修改說明通常比摘要長；屬本站基準。

來源：

- Qwen2.5-Coder 官方說明：https://qwenlm.github.io/blog/qwen2.5-coder-family/
- Qwen2.5-Coder 32B 模型卡：https://huggingface.co/Qwen/Qwen2.5-Coder-32B-Instruct

### 3. Agentic AI

HTML：`gemma4_e4b`，Input 16,384，Output 512，Batch 1，Concurrent Users 4。

- **模型組合**：Agent 的必要搭配是應用程式提供的工具，不一定是另一個 AI 模型。主模型輸出 function call，程式執行工具，再把結果送回主模型。
- **為何 Gemma 4**：Google 官方明列 built-in function calling 與 agentic workflows，並提供完整工具呼叫流程。
- **為何 Input 16K**：多輪任務要保留工具 schema、先前步驟與工具結果；屬本站互動型 Agent 基準。
- **為何 Output 512**：每一輪通常是工具呼叫或精簡結果；整個任務可能有多輪，因此 512 是單輪上限。
- **為何 Batch 1**：互動式 Agent 優先降低單一請求延遲。
- **為何 Concurrent 4**：模擬小型團隊同時使用；不是 Google 官方值，需按實際人數壓測。

來源：

- Gemma 4 官方概覽：https://ai.google.dev/gemma/docs/core
- Gemma 4 Function Calling：https://ai.google.dev/gemma/docs/capabilities/text/function-calling-gemma4
- Gemma 4 Prompt／Agentic Context：https://ai.google.dev/gemma/docs/core/prompt-formatting-gemma4

### 4. 客服摘要

HTML：`ministral3_8b`，Input 4,096，Output 512。

- **模型組合**：摘要是 text-to-text 任務，單一 LLM 即可。Ministral 3 官方支援 structured outputs 與 batching，並定位於 edge deployment。
- **為何 Input 4K**：可容納一般客服對話與摘要格式；長逐字稿應分段或提高 input。
- **為何 Output 512**：足以輸出摘要、問題分類、處理結果與後續行動；屬本站基準。

來源：

- Ministral 3 8B 官方文件：https://docs.mistral.ai/models/ministral-3-8b-25-12

### 5. 圖片生成

HTML：`ministral3_8b` + `qwen25_vl_7b` + `sdxl`，Input 4,096，Output 512。

- **模型組合**：SDXL 負責 text-to-image；LLM 整理 prompt；VLM 理解參考圖片或檢查生成結果。
- **必要與選配**：只做文字生圖時，SDXL 是必要核心，VLM 不是必要；目前組合代表「能對話、能看圖、能生成」的完整助理。
- **為何 4K／512**：只代表 LLM 的文字 prompt 階段。圖片輸出不是 512 tokens，影像模型記憶體應另算。

來源：

- SDXL：https://huggingface.co/stabilityai/stable-diffusion-xl-base-1.0
- Qwen2.5-VL：https://huggingface.co/Qwen/Qwen2.5-VL-7B-Instruct
- Ministral 3 8B：https://docs.mistral.ai/models/ministral-3-8b-25-12

### 6. 文件問答

HTML：`ministral3_8b` + `glm_ocr` + `bge_small_zh` + `bge_reranker`，Input 8,192，Output 512。

- **模型組合**：掃描文件先以 OCR 擷取文字、表格與公式，再以 Embedding 檢索、Reranker 重排、LLM 回答。
- **何時不需要 OCR**：若 PDF 已有可讀取的乾淨文字層，可直接進入 RAG。
- **為何 Input 8K**：比一般摘要多，用於問題、多段文件內容與少量對話歷史。
- **為何 Output 512**：適合直接回答與簡短依據；長報告可改為 1,024 或 2,048。

來源：

- GLM-OCR 官方模型卡：https://huggingface.co/zai-org/GLM-OCR
- Transformers GLM-OCR 文件：https://huggingface.co/docs/transformers/main/model_doc/glm_ocr
- BGE Small ZH：https://huggingface.co/BAAI/bge-small-zh-v1.5
- BGE Reranker：https://huggingface.co/BAAI/bge-reranker-v2-m3
- Ministral Document Q&A：https://docs.mistral.ai/models/ministral-3-8b-25-12

### 7. 視覺辨識

HTML：`gemma4_e4b` + `qwen25_vl_7b` + `yolo11`，Input 4,096，Output 512。

- **模型組合**：YOLO11 做 detection；VLM 做高階圖片語意理解；Gemma Agent 決定何時呼叫工具並整理結果。
- **必要與選配**：若只需物件框選、分類或計數，YOLO 即可；需要場景解釋與自然語言問答時才加入 VLM／LLM。
- **為何 4K／512**：是 Agent 文字階段基準；圖片尺寸與視覺 token 仍須依實際框架計算。

來源：

- YOLO11：https://docs.ultralytics.com/models/yolo11
- Qwen2.5-VL：https://huggingface.co/Qwen/Qwen2.5-VL-7B-Instruct
- Gemma Function Calling：https://ai.google.dev/gemma/docs/capabilities/text/function-calling-gemma4

### 8. 語音客服

HTML：`ministral3_8b` + `whisper_v3` + `kokoro_82m`，Input 4,096，Output 512。

- **模型組合**：完整資料流是 STT → LLM → TTS。Whisper 把語音轉文字；LLM 產生回答；Kokoro 把文字轉回語音。
- **為何三者都需要**：Whisper 官方任務是 automatic speech recognition；Kokoro 官方任務是 text-to-speech，兩者角色不同，不能互相替代。
- **為何 Input 4K**：容納近期通話內容、客服規則與必要知識。
- **為何 Output 512**：作為安全上限；實際語音客服應鼓勵更短回答，以降低使用者等待時間。
- **語言注意**：正式上線前須確認 Whisper 辨識率、Kokoro 的目標語言與聲音品質，不能只用模型標籤判定可用。

來源：

- Whisper Large v3：https://huggingface.co/openai/whisper-large-v3
- Kokoro-82M：https://huggingface.co/hexgrad/Kokoro-82M
- Ministral 3 8B：https://docs.mistral.ai/models/ministral-3-8b-25-12

## 其他 HTML 數值的界線

| HTML 設定 | 是否官方建議 | 說明 |
|---|---|---|
| 模型參數量、官方最大 context、模態 | 是官方規格 | 直接引用開發者文件或官方模型卡。 |
| 4K／8K／16K／32K Input | 否 | 本站為各情境建立的比較基準。 |
| 512／1K／2K Output | 否 | 依任務產物長度與延遲設定。 |
| Q4_K_M 等量化 | 不一定 | 必須確認 checkpoint 與推論框架支援。 |
| 附加模型 VRAM 預留 | 否 | 受權重格式、精度、框架、輸入尺寸與暫存區影響。 |
| Batch Size／Concurrent Users | 否 | 工作負載假設，需用實際服務流量壓測。 |

## 維護規則

1. 每張應用情境卡都必須在本文件找到對應的主模型、附加模型、Input、Output、Batch、Concurrent 與來源。
2. 優先引用模型開發者官網或官方 Hugging Face 帳號；第三方文章只能作補充。
3. 不把最大 context 寫成推薦 context，也不把本站 VRAM 或負載估算寫成官方數據。
4. 若情境語言是繁體中文，必須確認主模型與 Embedding 的官方語言支援。
5. 圖片、音訊、向量與偵測結果不是一般文字 output tokens；TPS／TTFT 只對應 LLM 文字推論階段。
6. 正式發布前，以相同測試集實測回答品質、TTFT、TPS、峰值 VRAM 與多使用者負載。
