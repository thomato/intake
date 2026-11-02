# Nutrition label capture: accuracy, Apple on-device vs AI vision, catching misreads

Research note for Intake (iPhone, iPad, macOS, web; first market NL). All sources checked 2026-11-02.
Claim labels: **Verified** = I read the source. **Verified, local run** = I ran it myself on Apple's SDK. **Inference** = my reasoning, not stated by a source.

## 1. Answer

- **Nobody has published how accurately either approach reads Dutch or EU nutrition labels.** I found no per-field figures for Dutch or EU labels, from Apple's OCR or from any current AI model. Apple publishes no accuracy numbers for its text recognition.
- **The closest evidence is weak but points one way: don't save a read unattended.** Older vision LLMs got 83-90% of element+value pairs correct on filtered, legible English labels after post-processing, and 74-80% on Arabic ([Assiri et al., J. Imaging 2025](https://pmc.ncbi.nlm.nih.gov/articles/PMC12387780/)). (Verified) That is roughly one wrong or missing line in every 6-10.
- **v1 recommendation (Inference):**
  - On iPhone and iPad: barcode first, then Apple Vision OCR, a deterministic EU-label parser, validators derived from the Regulation, and mandatory confirm-before-save.
  - Keep one vision LLM behind a server proxy as an optional fallback for bad reads and for web. It becomes the paid feature later.
  - Run the 100-label evaluation in section 9 before deciding which path is primary.
- **Cost will not decide this.** A cheap vision LLM costs about $0.0004-0.0013 per scan under the assumptions in section 4. Trust, offline use and privacy will decide it. (Inference from Verified prices)
- **Validators catch big errors, not all errors.** Energy arithmetic catches decimal slips in fat and protein. It cannot see a carbs/protein swap (both are 17 kJ/g), and most sugars, saturates and salt misreads pass it. Confirm-before-save stays necessary. (Inference from [Annex XIV](http://publications.europa.eu/resource/celex/02011R1169-20180101))

## 2. Comparison

| | Apple Vision OCR + own parser | Foundation Models, on-device | Cheap vision LLM (Haiku 5.5, gpt-6-luna, Gemini Flash-Lite) | Mistral OCR 4.1 |
|---|---|---|---|---|
| **Accuracy on NL/EU labels** | No data. Apple publishes none. | No data | No data for current models. Older models: 83-90% (English), 74-80% (Arabic), on a non-EU dataset | No data |
| **Speed** | 175-330 ms per image on an M1 Max with a synthetic label (Verified, local run). iPhone not measured | Text structuring only: 11-14 s on an M1 Max (Verified, local run). Image input not tested | No per-image figure published. Plausibly a few seconds (Inference, section 4) | Not published |
| **Cost per scan** | $0 | $0 | $0.0004-0.0012, or $0.0004-0.0013 on EU endpoints | $0.004, or $0.005 annotated |
| **Offline** | Yes | Yes, on Apple Intelligence devices only | No | No |
| **Privacy** | Image stays on device | On device. Private Cloud Compute is the optional server path | Image goes to a vendor, typically with up to 30 days of abuse logging | EU-hosted by default, abuse logging (period unverified) |
| **Platforms** | iOS, iPadOS, macOS. No web | iOS, iPadOS, macOS 26+; image input needs 27+ | All, including web, via your server | All, via your server |
| **Structured output** | None. You write the parser; tables come from `RecognizeDocumentsRequest` | Yes (`@Generable`) | Yes (JSON schema) | Yes (annotations) |
| **Confidence** | Line-level, 0-1 | None found | No per-field score found in the docs I read | Word and block level |

## 3. Apple on-device

**APIs and availability** (Verified)

| API | Availability | Notes |
|---|---|---|
| [`VNRecognizeTextRequest`](https://developer.apple.com/documentation/vision/vnrecognizetextrequest) | iOS/iPadOS 13+, macOS 10.15+ | Not marked deprecated |
| [`RecognizeTextRequest`](https://developer.apple.com/documentation/vision/recognizetextrequest) | iOS/iPadOS 18+, macOS 15+ | Swift API |
| [`RecognizeDocumentsRequest`](https://developer.apple.com/documentation/vision/recognizedocumentsrequest) | iOS/iPadOS/macOS 26+ | Apple's docs name "nutritional labels" as a use. Exposes [tables with rows, columns and cells](https://developer.apple.com/documentation/vision/documentobservation/container/table) |
| [`DataScannerViewController`](https://developer.apple.com/documentation/visionkit/datascannerviewcontroller) | iOS/iPadOS 16+, visionOS 1+. **macOS is not listed** | Live camera, barcodes and text, `capturePhoto()`. [Needs A12 or later](https://developer.apple.com/documentation/visionkit/datascannerviewcontroller/issupported) |
| [Foundation Models](https://developer.apple.com/documentation/foundationmodels) | iOS/iPadOS/macOS 26+ | Needs an Apple Intelligence device and region |
| Foundation Models [image attachments](https://developer.apple.com/documentation/foundationmodels/attachment) | iOS/iPadOS/macOS 27+ | Text and image prompts |

- **Web:** none of these list a web platform, so a browser client cannot use them. (Inference from the platform lists)
- **On-device:** Apple states "all vision APIs run entirely on-device." ([WWDC25-272](https://developer.apple.com/videos/play/wwdc2025/272/)) (Verified)

**Dutch** (Verified, local run: `supportedRecognitionLanguages` on macOS 26.7.1)
- `nl-NL` is supported at `.accurate` and by `RecognizeTextRequest` and `RecognizeDocumentsRequest`.
- **`.fast` level does not list Dutch.** It returns only en, fr, it, de, es and pt. Use `.accurate`.
- Apple's docs do not print the list; it is a runtime query. Re-run it on the iOS deployment target. I could not query `DataScannerViewController.supportedTextRecognitionLanguages` on a Mac.
- Apple Intelligence [supports Dutch](https://support.apple.com/en-us/121115), and `SystemLanguageModel` reported `nl_NL` as supported locally. (Verified; Verified, local run)

**Accuracy numbers:** none in the Vision docs I read or in the WWDC25-272 and WWDC26-237 sessions. (Verified for those pages; I cannot rule out a figure elsewhere.)

**Foundation Models** (Verified unless marked)
- Guided generation uses [constrained sampling](https://developer.apple.com/documentation/foundationmodels/generating-swift-data-structures-with-guided-generation) to produce typed output. That guarantees the format, not that the values are right. (Second sentence is Inference.)
- The [image guide](https://developer.apple.com/documentation/foundationmodels/analyzing-images-with-multimodal-prompting) recommends isolating a region of interest and offers an `OCRTool`. Apple describes that tool as ["good for helping models read really fine or dense text"](https://developer.apple.com/videos/play/wwdc2026/237/) and says it covers over 30 languages. That suggests the model alone is not the tool for dense label text. (Last sentence is Inference.)
- [Private Cloud Compute](https://developer.apple.com/documentation/foundationmodels/adding-server-side-intelligence-with-private-cloud-compute) (iOS 27+) gives a 32K-token context, requires a managed entitlement and a network, and applies a per-person daily quota. It has [no cloud API cost under 2 million first-time downloads, and Apple says "no prompts are ever stored"](https://developer.apple.com/videos/play/wwdc2026/241/).
- On-device structuring is not needed for v1 (Inference). In my test it took 11-14 s, while a deterministic parser should take milliseconds.

**Local smoke test, not accuracy evidence** (Verified, local run; M1 Max, macOS 26.7.1). I drew one synthetic Dutch table in code with invented values, in a clean font. I rendered it flat, with simulated cylinder warps of 60° and 80°, and with a simulated glare band.

| Finding | Result |
|---|---|
| OCR text | All lines correct in all 5 variants, decimal commas preserved ("18,5 g") |
| OCR line confidence | 1.00 on every line, every variant |
| `RecognizeDocumentsRequest` table, flat | **0 tables detected**; the text came back column by column |
| Table, 60° warp and glare | Correct rows |
| Table, 80° warp | **Rows silently shifted:** "Energie" paired with the fat values, "Vezels" with protein, "Eiwitten" with salt, and fat merged with saturates |
| `DocumentObservation.confidence` | 0.0 in every variant |
| Foundation Models structuring (OCR text in, 9 fields out) | 6 of 6 runs correct, 11.4-14.4 s each |

What this suggests (Inference):
- Real curvature and glare are harder than these simulations, so the clean OCR results say little about real packs.
- The useful finding is structural. Table detection can be absent or shift rows while the characters are perfect, and no confidence score shows it.
- So the parser should build rows itself from line bounding boxes and treat Vision's table as a hint. The validators must catch row shifts.

## 4. AI vision APIs: models, tokenising, cost

Model IDs and standard-tier prices in the cost table are Verified on the [Anthropic](https://platform.claude.com/docs/en/about-claude/pricing), [OpenAI](https://developers.openai.com/api/docs/pricing), [Gemini](https://aistudio.google.com/docs/pricing), [Vertex](https://cloud.google.com/vertex-ai/generative-ai/pricing) and [Mistral](https://docs.mistral.ai/models/ocr-4-1) pages.

**How images are tokenised** (Verified)
- **Claude:** [`⌈w/28⌉ × ⌈h/28⌉`](https://platform.claude.com/docs/en/build-with-claude/vision). Claude 4.7+ images are capped at a 2,576 px long edge and 4,784 tokens.
- **OpenAI:** [`⌈w/32⌉ × ⌈h/32⌉` patches × a model multiplier](https://developers.openai.com/api/docs/guides/images-vision). The multiplier is 1.2 for gpt-6-astra and gpt-5.6-luna. **gpt-6-luna and gpt-6.1-terra are not in the table, so I assumed 1.2.** (Inference)
- **Gemini:** [fixed by `media_resolution`](https://aistudio.google.com/docs/media-resolution). Gemini 3 and 4 use 1,120 tokens at default/`high` (recommended for images) and 2,240 at `ultra_high`.
- **Mistral OCR:** [priced per page](https://docs.mistral.ai/models/ocr-4-1).

**Assumptions for cost per scan (Inference):** a label photo cropped to 1176×1568 px, a 400-token prompt with schema, 300 output tokens, no caching, no batch.
- Claude image tokens: 42 × 56 = 2,352.
- OpenAI image tokens: 37 × 49 = 1,813 patches × 1.2 = 2,176.
- Gemini image tokens: 1,120.

| Model (API ID) | In $/M | Out $/M | Input cost | Output cost | **Per scan** | 300 scans |
|---|---|---|---|---|---:|---:|
| `gpt-6-luna` | 0.10 | 0.50 | 2,576 × 0.10 = $0.000258 | 300 × 0.50 = $0.000150 | **$0.00041** | $0.12 |
| `claude-haiku-5-5` (prompts ≤100k) | 0.10 | 0.50 | 2,752 × 0.10 = $0.000275 | $0.000150 | **$0.00043** | $0.13 |
| `gemini-3.1-flash-lite` | 0.25 | 1.50 | 1,520 × 0.25 = $0.000380 | $0.000450 | **$0.00083** | $0.25 |
| `gemini-3.5-flash-lite` | 0.30 | 2.50 | 1,520 × 0.30 = $0.000456 | $0.000750 | **$0.00121** | $0.36 |
| `gemini-3.8-flash` to 2026-12-31 | 0.75 | 3.75 | $0.001140 | $0.001125 | **$0.00227** | $0.68 |
| `gemini-3.8-flash` from 2027-01-01 | 1.50 | 7.50 | $0.002280 | $0.002250 | **$0.00453** | $1.36 |
| `gpt-6.1-terra` | 1.00 | 5.00 | $0.002576 | $0.001500 | **$0.00408** | $1.22 |
| `claude-sonnet-5-5` | 2.00 | 10.00 | $0.005504 | $0.003000 | **$0.00850** | $2.55 |
| `claude-opus-5-5` | 4.00 | 20.00 | $0.011008 | $0.006000 | **$0.01701** | $5.10 |
| Mistral OCR 4.1 (`mistral-ocr-4-1+2`) | - | - | $4 / 1,000 pages | - | **$0.004** | $1.20 |
| Same, annotated | - | - | $5 / 1,000 pages | - | **$0.005** | $1.50 |

**EU-routed variants (arithmetic is Inference):**
- `gemini-3.5-flash-lite` on non-global Vertex at [$0.33 / $2.75](https://cloud.google.com/vertex-ai/generative-ai/pricing): 1,520 × 0.33 + 300 × 2.75 = **$0.00133**.
- `gpt-6-luna` on an EU endpoint: +[10% regional uplift](https://developers.openai.com/api/docs/pricing) for models released on or after 2026-03-05 = **$0.00045**. The release date is assumed from its May 2026 knowledge cutoff.
- Haiku 5.5 regional on Bedrock: +[10%](https://platform.claude.com/docs/en/build-with-claude/claude-in-amazon-bedrock) = $0.00047, if it is offered in an EU profile (not confirmed; see section 5).

**Sensitivity:** reasoning tokens are billed as output on [Gemini](https://aistudio.google.com/docs/pricing). I assume the same elsewhere. (Inference) At 1,500 output tokens, Haiku 5.5 costs $0.00103 and Gemini 3.5 Flash-Lite $0.00421. Turn reasoning off or set it low. Haiku 5.5 is [adaptive with default effort `medium`](https://platform.claude.com/docs/en/about-claude/models/overview); gpt-6-luna [goes down to effort "none"](https://developers.openai.com/api/docs/models/gpt-6-luna). (Verified)

**Structured output** (Verified)
- [Anthropic](https://platform.claude.com/docs/en/build-with-claude/structured-outputs): `output_config.format`, guaranteed schema shape. `minimum`/`maximum` are not supported, so ranges must be checked in code.
- [OpenAI](https://developers.openai.com/api/docs/guides/structured-outputs): JSON schema. [gpt-6-luna lists structured outputs and image input](https://developers.openai.com/api/docs/models/gpt-6-luna).
- [Gemini](https://aistudio.google.com/docs/structured-output): JSON schema. Google warns: "always validate values in your application," and to handle "schema-compliant but semantically incorrect outputs."
- [Mistral](https://docs.mistral.ai/capabilities/document_ai/basic_ocr/): annotations plus page, block and word confidence.

So a schema guarantees a valid-looking `"protein": 84`, not a correct one. (Inference)

**Latency** (Verified that I found no vendor per-image figure)
- Anthropic gives [relative labels only](https://platform.claude.com/docs/en/about-claude/models/overview) (Haiku "Fastest"). The [Gemini models page](https://aistudio.google.com/docs/models) publishes no metrics.
- Third-party [Artificial Analysis](https://artificialanalysis.ai/models/claude-haiku-5-5) reports output speed of 235 t/s for Haiku 5.5, [423 t/s for Gemini 3.5 Flash-Lite](https://artificialanalysis.ai/models/gemini-3-5-flash-lite) and [117 t/s for GPT-6 Luna](https://artificialanalysis.ai/models/gpt-6-luna).
- Its time-to-first-token figures (370 s for Haiku "Max", 122 s for Luna "Max", 6.22 s for Flash-Lite) include thinking time on maximum-effort configurations. They do not represent a low-effort image call, so **do not use them.**
- At those speeds, 300 output tokens take about 0.7-2.6 s to generate, plus upload and first-token time. That is why I say "a few seconds," which is an estimate to replace with measurements. (Inference)

## 5. Privacy

**Is a label photo personal data?** Mostly not, in itself, but the surrounding request usually is. (Inference, from [GDPR](http://publications.europa.eu/resource/celex/32016R0679) Art. 4(1) and Recital 26)
- A clean crop of a printed table doesn't relate to an identifiable person.
- It can become personal data through EXIF location, a hand or receipt in frame, or a request tied to a user account.
- **Mitigation:** crop to the label on device, strip metadata, re-encode, and send through a proxy with no user ID.

**Is the log health data?** Possibly; treat it with care. (Inference)
- [Art. 4(15)](http://publications.europa.eu/resource/celex/32016R0679) covers data that "reveal information about his or her health status." The CJEU treated [online pharmacy-only order data as health data](https://curia.europa.eu/jcms/upload/docs/application/pdf/2024-10/cp240159en.pdf) in C-21/23 (read the press release, not the full judgment).
- I found no EDPB or Dutch DPA guidance specifically on food logs. A calorie log with targets could plausibly reveal health status, which argues for sending label images only, never the log.
- Purely personal use falls outside the GDPR under [Recital 18](http://publications.europa.eu/resource/celex/32016R0679) (Verified). A paid release ends that. Get legal review before then.
- **App Store:** [guideline 5.1.2(i)](https://developer.apple.com/app-store/review/guidelines/) requires clear disclosure and explicit permission before sharing personal data "with third-party AI." (Verified)

| Vendor | Training on API data | Retention | EU option (Verified) |
|---|---|---|---|
| **Anthropic** | [No by default](https://privacy.claude.com/en/articles/7996868-is-my-data-used-for-model-training) | [Deleted within 30 days](https://privacy.claude.com/en/articles/7996866-how-long-do-you-store-my-organization-s-data); flagged violations up to 2 years, scores up to 7. ZDR by agreement | **None on the first-party API:** [`inference_geo` is only `us` or `global`, workspace geo only `us`](https://platform.claude.com/docs/en/manage-claude/data-residency). [Bedrock lists EU profiles](https://platform.claude.com/docs/en/build-with-claude/claude-in-amazon-bedrock) but not by model, so Haiku 5.5 is unconfirmed, and [Google's list](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/learn/data-residency) shows no Haiku at `eu` |
| **OpenAI** | [No unless opted in](https://developers.openai.com/api/docs/guides/your-data) | Abuse logs up to 30 days. Image inputs are CSAM-scanned and may be retained for review even under ZDR | `eu.api.openai.com` (EEA + Switzerland) with processing. **Barrier:** the table marks Europe with "Image support in these regions requires approval for enhanced Zero Data Retention or enhanced Modified Abuse Monitoring," and the same features are sales-led |
| **Google** | [Paid: no. In EEA/CH/UK, paid-tier data terms apply even to unpaid quota](https://aistudio.google.com/docs/terms) | [Abuse logging for "a limited period"](https://aistudio.google.com/docs/zdr); stronger ZDR on the enterprise platform | [`eu` multi-region endpoint](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/learn/data-residency) lists Gemini 3.5 Flash-Lite and others. The doc warns `global` gives no residency guarantee. I found no Gemini Flash-Lite in `europe-west4` |
| **Mistral** | [Studio pay-as-you-go: opt-out available; free plan trains by default](https://help.mistral.ai/en/articles/347617-do-you-use-my-user-data-to-train-your-artificial-intelligence-models). **Confirm the opt-out.** | **Period unverified:** a search snippet says 30 rolling days for abuse monitoring, but the [DPA](https://legal.mistral.ai/terms/data-processing-addendum/) I read gives no timeframe. [ZDR on request, covers `/v1/ocr`](https://docs.mistral.ai/admin/monitor-comply/zero-data-retention) | [EU by default](https://help.mistral.ai/en/articles/347629-where-do-you-store-my-data-or-my-organization-s-data), with possible temporary transfers to subprocessors |
| **Apple** | n/a | On-device: image never leaves. PCC: "no prompts are ever stored" | On device |

## 6. Accuracy evidence

**State of the evidence: no study reports field-level nutrition extraction on Dutch labels. I also found none on EU labels end to end, and none testing Apple Vision on nutrition labels.** These are search negatives, so absence of evidence only. Everything below is extrapolation if applied to Dutch labels.

| Source | Data | Tested | Result (Verified) | Why it doesn't transfer |
|---|---|---|---|---|
| [Assiri et al. 2025](https://pmc.ncbi.nlm.nih.gov/articles/PMC12387780/) | 294 English-Arabic Gulf labels from databases and online listings; blurred images removed | GPT-4V, GPT-4o, Gemini (old) | **English element+value, after post-processing:** GPT-4V 90.1%, GPT-4o 87.3%, Gemini 83.2% correct (Jaccard 0.836, 0.795, 0.745). **Arabic:** 80.0%, 79.6%, 74.0%. Before processing, English was 88.0, 82.5, 73.8 and Arabic only 19-28% | Filtered, not fresh phone shots. No curved subset. Old models. No OCR baseline run. Post-processing used dictionaries tuned to the data |
| [OFF nutrition-extractor](https://huggingface.co/openfoodfacts/nutrition-extractor) on [dataset](https://huggingface.co/datasets/openfoodfacts/nutrient-detection-layout) | ~3,080 crowd photos, multilingual, includes Dutch | LayoutLMv3 on Google Cloud Vision OCR | Precision 0.9545, recall 0.9647, F1 0.9596 (self-reported, `verified: false`) | The nearest to EU labels, but token classification, not end-to-end correct values. No per-language split. Model is CC-BY-NC-SA (non-commercial) |
| [Nagayi et al. 2025](https://arxiv.org/abs/2510.03570) | South Africa, 113 ground-truth images, handheld; [glare, curves](https://arxiv.org/html/2510.03570v1) named as conditions | Tesseract, EasyOCR, PaddleOCR, TrOCR | CER 0.912-1.075, WER 6.26-7.41 (can exceed 1 from insertions); 0.58-6.24 s per image | Whole-text metrics over mixed regions, with [no nutrient-level figures](https://arxiv.org/html/2510.03570v1). No Apple or LLM. Shows raw OCR text is noisy on packs |
| [Nagayi et al. 2026](https://arxiv.org/pdf/2609.15427) | South Africa, 442 products, 3,929 images | PaddleOCR workflow vs Qwen2.5-VL 7B, sodium only | Conservative workflow sent 47 of 60 audited products to REVIEW. Of the 13 it decided, 12 agreed with manual review. Workflows agreed on 69.5% of final outcomes | Single nutrient. Evidence for review routing, not raw accuracy |
| [HalalBench 2026](https://arxiv.org/html/2604.22754v1) | 50 real + 993 synthetic, 14 languages; Dutch grouped with Finnish and Portuguese in **3 images, no Dutch F1** | ML Kit, docTR, EasyOCR, RapidOCR | Best F1 0.193 (ingredient exact match). Analysed failures: small font 30.6%, multilingual merging 22.2%, curved surfaces 19.4%. Below 7 pt, recognition under 10%. "Not corrected by any tested engine" | Ingredients, not nutrition tables. Nonetheless the best named evidence on curves and multilingual merging |
| [Grubert & Gao](https://stacks.stanford.edu/file/druid:bf950qp8995/Grubert_Gao.pdf) (student report) | 97 US labels, iPhone 5/5S | Tesseract + rectification | 63.8% of line items right, vs 34.9% OCR alone. Crowd barcode data had 14% error on a subset of 80 | Old, US format. Lists curvature as unsolved |
| [OmniFood-Bench](https://arxiv.org/pdf/2607.08423) | Mixed food images | Vision LLMs incl. gpt-5.1, gemini-3-flash | Packaged-food recognition was lowest (e.g. 42.86%); authors say OCR is "likely" a factor | Not label reading |
| [Berengueres 2026](https://arxiv.org/pdf/2608.12359) | 132 forced choices, front-of-pack, 4 markets | AI agents | 88.9% global products; 59.5% local Swedish (not above chance) | Inference from the front of pack, not reading. Warns about relying on model memory for local products |

Excluded: **MM-Food-100K** ([arXiv 2508.10429](https://arxiv.org/abs/2508.10429)) is image-based food nutrition prediction. Its abstract doesn't mention labels.

**Curved surfaces and glare:** sources name them as causes, but **I found no controlled measurement of error rate against curvature or glare on nutrition tables.** HalalBench's 19.4% is a share of analysed failures, not a degradation rate.

**Dutch data available to test with (Verified, local run):** in the [OFF dataset](https://huggingface.co/datasets/openfoodfacts/nutrient-detection-layout) I counted rows with 3 or more Dutch nutrient words: **10 of 199 test and 75 of 2,884 train.** This is my keyword heuristic. It includes multilingual labels and misses others.

## 7. EU label rules that underpin validation

From [Regulation (EU) 1169/2011, consolidated 01.01.2018](http://publications.europa.eu/resource/celex/02011R1169-20180101). (Verified) This is the latest consolidation the Publications Office returned. EUR-Lex blocked direct fetching, so I could not confirm no later one exists. The consolidated text has no legal effect by its own disclaimer.

| Rule | Text | Consequence for the parser |
|---|---|---|
| Mandatory, Art. 9(1)(l), 30(1) | Energy, fat, saturates, carbohydrate, sugars, protein, salt | Expect 7 values. Missing ones trigger a review |
| Optional, Art. 30(2) | Mono-unsaturates, polyunsaturates, polyols, starch, fibre, some vitamins and minerals | Handle their absence, especially in the energy check |
| Basis, Art. 32(2) | Per 100 g or per 100 ml is required | Primary column, with exceptions below |
| Portion, Art. 33 | May be added if the portion is quantified and the number of portions stated | Second column. Header gives the factor |
| %RI, Art. 32(4)-(5) | Optional; "8 400 kJ/2 000 kcal" statement | Third column trap |
| Prepared food, Art. 31(3) | Values may relate to the food after preparation | Column trap ("bereid") |
| Exceptions, Art. 33(2)-(3) | Values repeated under Art. 30(3)(b), shown in the principal field of vision, may be per portion alone; so may non-prepacked foods under Art. 30(5) | Don't capture from the front of pack (Inference) |
| Order and units, Annex XV | Energy kJ/kcal, fat, of which (saturates, mono, poly), carbohydrate, of which (sugars, polyols, starch), fibre, protein, salt | Language-independent order "where appropriate" (Art. 34(1)), usable as an extra row check |
| Format, Art. 34(2) | Tabular "if space permits," **otherwise linear** | Parser must handle running text |
| Negligible, Art. 34(5) | May be replaced by "contains negligible amounts of…" | A missing value isn't always a misread |
| Salt, Annex I(11) | "Salt" = sodium × 2,5 | If an import says sodium, convert |
| Carbs, Annex I(7) | Includes polyols. Fibre is defined separately (I(12)) | Don't add polyols twice; fibre is outside carbs (Inference from definitions) |
| Language, Art. 15 | A language easily understood where marketed; several languages allowed | Multilingual tables are legal and expected |
| Legibility, Art. 13(2)-(3) | x-height ≥1.2 mm, or ≥0.9 mm on packs under 80 cm² | Crop close; avoid vendor downscaling |
| Exempt, Annex V | Single-ingredient unprocessed foods, water, herbs, salt, packs under 25 cm², others | "No table" is a valid state |

**Terms from the Annex XV language versions** (Verified: [NL, DE, FR via the same resource](http://publications.europa.eu/resource/celex/02011R1169-20180101))

| EN | NL | DE | FR |
|---|---|---|---|
| energy | energie | Energie | énergie |
| fat | vetten | Fett | matières grasses |
| saturates | verzadigde vetzuren | gesättigte Fettsäuren | acides gras saturés |
| mono-unsaturates | enkelvoudig onverzadigde vetzuren | einfach ungesättigte Fettsäuren | acides gras mono-insaturés |
| polyunsaturates | meervoudig onverzadigde vetzuren | mehrfach ungesättigte Fettsäuren | acides gras polyinsaturés |
| carbohydrate | koolhydraten | Kohlenhydrate | glucides |
| sugars | suikers | Zucker | sucres |
| polyols | polyolen | mehrwertige Alkohole | polyols |
| starch | zetmeel | Stärke | amidon |
| fibre | vezels | Ballaststoffe | fibres alimentaires |
| protein | eiwitten | Eiweiß | protéines |
| salt | zout | Salz | sel |
| of which | waarvan | davon | dont |

**Rounding and tolerances** ([EC guidance, Dec 2012](https://food.ec.europa.eu/system/files/2016-10/labelling_nutrition-vitamins_minerals-guidance_tolerances_1212_en.pdf); [summary table](https://food.ec.europa.eu/system/files/2016-10/labelling_nutrition-vitamins_minerals-guidance_tolerances_summary_table_012013_en.pdf)). (Verified) The guidance states it has **no formal legal status**.

| Nutrient | Rounding | Control tolerance |
|---|---|---|
| Energy | Nearest 1 kJ/kcal | None listed in Table 1 |
| Fat, carbs, sugars, protein, fibre | ≥10 g: 1 g. Under 10 g and over 0.5 g: 0.1 g. ≤0.5 g: "0 g" or "<0.5 g" | Under 10 g: ±2 g (fat ±1.5 g). 10-40 g: ±20%. Over 40 g: ±8 g |
| Saturates, mono, poly | ≥10 g: 1 g. Under 10 g and over 0.1 g: 0.1 g. ≤0.1 g: "0 g" or "<0.1 g" | Under 4 g: ±0.8 g. ≥4 g: ±20% |
| Salt | ≥1 g: 0.1 g. Under 1 g and over 0.0125 g: 0.01 g. Below: "0 g" or "<0.01 g" | Under 1.25 g: ±0.375 g. ≥1.25 g: ±20% |

**Implications (Inference):**
- Declared values are averages ([Art. 31(4)](http://publications.europa.eu/resource/celex/02011R1169-20180101)), and the tolerances compare a lab result against the label. So they do **not** say how far a label's own numbers may disagree with each other.
- Our checks compare printed numbers, so rounding, not the ±20%, sets the strict bound. Real labels will still break it, so failures are warnings, not rejections.
- Foods also legitimately vary from the label, so a perfect read isn't a perfect measurement. Don't imply more precision than the label has.

## 8. Catching misreads

**Evidence that the checks make sense**
- [FoodRepo](https://pmc.ncbi.nlm.nih.gov/articles/PMC6040205/) used the same family for catching *transcription errors*: p+f+c ≤ 100, subclass ≤ parent, and energy ≈ 4p + 9f + 4c. It reports no detection rates. (Verified)
- Grubert & Gao used [numeric rectification](https://stacks.stanford.edu/file/druid:bf950qp8995/Grubert_Gao.pdf) to catch "42%" read as "32%." (Verified)
- **Labelled lead, blog, unverified:** an [audit of Open Food Facts France](https://dev.to/leanapp/auditing-857655-open-food-facts-labels-with-pandas-how-many-nutrition-facts-contradict-themselves-4cen) flagged 5.5% of complete products as incoherent (over 10% *and* over 30 kcal/100 g). Treat that as a rough hint at false-positive load only.
- **No source measures how many real label misreads these checks catch.** All thresholds below are proposals to tune in the evaluation.

**Checks** (facts Verified as cited; thresholds and severities are Inference)

| # | Check | Rule | Action |
|---|---|---|---|
| 1 | **Energy from macros** | kJ ≈ 37·fat + 17·(carbs − polyols) + 10·polyols + 17·protein + 8·fibre + 29·alcohol + 13·organic acid + 25·salatrims; erythritol 0. kcal uses 9, 4, 2.4, 4, 2, 7, 3, 6, 0 ([Annex XIV](http://publications.europa.eu/resource/celex/02011R1169-20180101)). Compute the allowed gap by propagating each printed value's rounding. Worst case with fat, carbs, protein and fibre all ≥10 g: 37×0.5 + 17×0.5 + 17×0.5 + 8×0.5 + 0.5 = **40 kJ** (or 10 kcal) | Within the rounding gap or 5%: pass. 5-15%: warn. Over 15%: highlight fields |
| 2 | **Unlisted components** | Fibre may be omitted, and alcohol, organic acids and salatrims rarely appear, so computed energy can run low. Allow for that, and mark drinks with alcohol as outside check 1 | Soften check 1 when these are likely |
| 3 | **kJ/kcal ratio** | Annex XIV ratios span 4.00 (fibre) to 4.33 (organic acid); fat 4.11, carbs and protein 4.25. Expect kJ/kcal in **about 4.0-4.33**, not exactly 4.184, widened for rounding at low energy (compare (kJ±0.5)/(kcal±0.5)) | Outside: highlight energy |
| 4 | **Sub-nutrient ≤ parent** | Saturates + mono + poly ≤ fat, and sugars + polyols + starch ≤ carbs ([Annex XV](http://publications.europa.eu/resource/celex/02011R1169-20180101)), +0.5 g for rounding | Highlight both |
| 5 | **Mass sum** | Per 100 g: fat + carbs + fibre + protein + salt ≤ about 102 g. Don't add sub-nutrients. **Skip or relax for per 100 ml,** where density can exceed 1 | Highlight the largest |
| 6 | **Column resolution** | Read the headers. Accept "per/pro/pour 100 g/ml" as primary. Reject "%", "RI", "RI*", "referentie-inname". Flag "bereid", "prepared", "zubereitet", "préparé" | Unknown: ask, never guess |
| 7 | **Portion consistency** | Each portion value ÷ its per-100 value should match grams ÷ 100, within rounding. Portion may exceed 100 g, so "larger column = per 100" is unsafe | Mismatch: swap or highlight |
| 8 | **Annex XV order** | Rows should appear in order per language block | Out of order: row shift suspected |
| 9 | **Number parsing** | Treat comma as decimal. Dot followed by exactly 3 digits on energy is thousands ("1.906 kJ"). Otherwise a dot is decimal, for imports. Parse "<0,5" as censored, stored as such. The Regulation's own text uses "2,4 kcal/g" but sets no separator rule | Ambiguous: highlight |
| 10 | **Unit check** | g for macros, kJ and kcal for energy, and mg/µg only for vitamins and minerals; "sodium" in mg → ÷1000 × 2.5 as salt | Highlight |
| 11 | **Decimal-loss pattern** | Value is ≥10 and has decimals missing while energy disagrees; test the candidate with a comma inserted | Offer the fix, don't apply it |
| 12 | **Multilingual merge** | Match terms per language block. A single value should map to one nutrient | Merge suspected: highlight |
| 13 | **Barcode cross-check** | Compare with database per-100 g values after normalising rounding. **Neither source is truth:** crowd data had [14% error in one old sample](https://stacks.stanford.edu/file/druid:bf950qp8995/Grubert_Gao.pdf), and recipes change | Disagree: show both, user picks |
| 14 | **Confirm before save** | Show the crop beside editable values. Highlight any flagged field. Require one tap to accept. Store provenance per value (OCR, LLM, barcode, user) | Always |

**What these checks can and can't catch.** Worked example (synthetic): fat 18,5, carbs 62, protein 8,4, fibre 3,1. Then 37×18.5 + 17×62 + 17×8.4 + 8×3.1 = 684.5 + 1054 + 142.8 + 24.8 = 1906.1 kJ against a declared 1906; kcal = 454.3 against 454; ratio 4.198. (Arithmetic; Inference on detectability)

| Misread | Effect | Caught? |
|---|---|---|
| Fat 18,5 → 185 | +6,160 kJ, mass over 100 | Yes |
| Protein 8,4 → 84 | +1,285 kJ | Yes |
| Fat 7,2 row shifted into fat | -418 kJ, 22% | Yes |
| kcal 454 → 404 | Ratio 4.72 | Yes |
| Sugars 31 → 81 | Exceeds carbs 62 | Yes |
| **Protein 8,4 → 3,4** | -85 kJ, 4.5% | **Only at the strict rounding gap. Passes the 5% band.** |
| **Carbs ↔ protein swap** | 0 kJ, same factor | **No.** Needs check 8 or 13 |
| **Sugars 31 → 51** | Under carbs | **No** |
| **Saturates 7,2 → 1,2** | Under fat | **No** |
| **Salt 0,58 → 5,8** | Barely moves mass | **No** |

Intake's targets are calories, protein and fat, the fields these checks protect best, apart from the swap and small protein errors. Carbs and other nutrients get less protection. (Inference)

**Common confusions**
- Verified: "16O" for 160, "io" for 10, "42%" for "32%" ([Grubert & Gao](https://stacks.stanford.edu/file/druid:bf950qp8995/Grubert_Gao.pdf)); merged cells and misaligned columns ([Assiri et al.](https://pmc.ncbi.nlm.nih.gov/articles/PMC12387780/)); text merged across adjacent languages ([HalalBench](https://arxiv.org/html/2604.22754v1)).
- Inference, to be confirmed by the evaluation: lost comma (1,5 → 15), "5 g" → "59", S/5, B/8, l/1, `kJ` split from its number. Consider turning `usesLanguageCorrection` off for numeric cells and testing it, since its effect on digits isn't documented in what I read.

**Confidence**
- Apple exposes [0-1 confidence on recognised text](https://developer.apple.com/documentation/vision/vnrecognizedtext/confidence) (Verified). I saw no calibration statement, and in my smoke test it was 1.00 even where the table structure was wrong. (Verified, local run) Use it as a weak input for highlighting and never as a pass.
- Schema-constrained LLM output has no per-field confidence in the docs I read. Asking the model to self-report gives an uncalibrated number. I didn't check logprob features. (Inference)
- Mistral OCR and the [OFF pipeline](https://openfoodfacts.github.io/robotoff/references/predictions/nutrient-extraction/) do expose confidence scores. (Verified)

## 9. Recommendation for the spec

All Inference, built on the Verified facts above.

**Pipeline**
1. Scan barcode. On a database hit, show values for editing, per the existing flow.
2. On a miss, or on request, capture a still photo of the label with framing guidance. Crop on device, and check blur and size before processing.
3. Run OCR with `RecognizeDocumentsRequest` for text and table hints, with languages nl, en, de, fr (my smoke test used only nl and en). If you use the plain text requests, set `.accurate`, because `.fast` lacks Dutch. Build rows from line geometry.
4. Parse with the Annex XV term table, handling tabular and linear formats.
5. Run validators and show confirm-before-save with the crop and highlights.
6. Offer "Try AI read" when required fields are missing or validators fail. Validate its output the same way.

**Platforms**

| Platform | v1 |
|---|---|
| iPhone, iPad | Full flow. Target iOS/iPadOS 26+ for `RecognizeDocumentsRequest`. `DataScannerViewController` is fine for barcode and live guidance |
| macOS | No `DataScannerViewController`. Run Vision on an imported or dragged photo |
| Web | No Apple APIs. v1: manual entry and edit only. Label capture via the server LLM can come with the paid tier, which suits it |
| Sharing logic | The parser and validators should be plain, platform-free code with shared test fixtures, so that web and Apple clients agree |

**Cloud fallback**
- Keep the API key on a server, never in the app. Send only the cropped, metadata-free image.
- Put vendors behind one interface, since prices and models will move: Gemini 3.8 Flash [doubles on 2027-01-01](https://aistudio.google.com/docs/pricing), and several models are being retired.
- **Tentative default:** `gemini-3.5-flash-lite` on the Google `eu` endpoint, about $0.0013 per scan. It was the only low-cost LLM path I found with EU processing and no approval step, but I have no accuracy evidence favouring it.
- **Evaluate alongside it:** `claude-haiku-5-5` and `gpt-6-luna` (cheaper, but global routing or an image approval barrier), and Mistral OCR 4.1 fed through our parser (EU default, gives confidence).
- Disclosure: this note was written by a Claude model. Treat the Anthropic row as you would the others, and let the evaluation decide.

**Paid tier**
- Free: barcode, on-device OCR, validators. Paid: AI read.
- At about $0.001 per scan, even 300 scans a month is under $0.40, so a per-scan meter is not needed. Set a fair-use cap to bound abuse.
- Add explicit consent for third-party AI ([5.1.2(i)](https://developer.apple.com/app-store/review/guidelines/)), a DPA and a privacy notice before release.
- Check PCC with `OCRTool` as a possible no-cost alternative on Apple devices, subject to entitlement and daily limits.

**Evaluation plan to fill the Dutch-label gap**

*Step 0, free pre-screen.* Run the parser on the [~85 Dutch-keyword OFF rows](https://huggingface.co/datasets/openfoodfacts/nutrient-detection-layout) (CC-BY-SA per the card; check the image terms). This debugs the parser only, as its ground truth is value tokens, not a full Intake record.

*Step 1, sample.* **100 Dutch-market products**, photographed handheld in normal kitchen and shop light, one photo per product, retakes not allowed. Keep images out of the public repo.

| Stratum | n |
|---|---:|
| Flat cartons | 30 |
| Curved (bottles, cans, tubs) | 30 |
| Glossy or flexible film | 20 |
| Dense multilingual or linear format | 20 |

*Step 2, ground truth.* Key values from the physical pack, then re-key them blind on another day and resolve differences. Allow 3-4 hours in total (estimate).

*Step 3, arms.*
- A: Apple OCR + parser.
- B: Gemini 3.5 Flash-Lite.
- C: Haiku 5.5.
- D: gpt-6-luna.
- E: Mistral OCR + parser.
- F (optional): on-device Foundation Models with image and `OCRTool` on iOS 27.

Cloud cost across B-E is under $1 for 100 images, at the section 4 assumptions. Use a fixed prompt and record model IDs.

*Step 4, metrics, reported overall and per stratum.*

| Metric | Definition |
|---|---|
| **Silent-error rate (primary)** | Labels where a calorie, protein or fat value is wrong and *no* validator flags it |
| Macro-perfect labels | All of energy, fat, protein and carbs exact after normalisation |
| Field exact match | Per field, including the other nutrients |
| Column and row errors | Wrong basis column, shifted row |
| Flag rate and false flags | How often correct reads are bothered |
| Absolute error | kcal per 100 g, and grams for protein and fat |
| Latency, cost | p50 and p95 on an actual iPhone and network, tokens used |
| Validator catch rate | Share of wrong reads flagged, per check. This fills the evidence gap in section 8 |

*Step 5, precision to expect.* (Inference; arithmetic shown)
- At label level with n=100, an observed 95% has a Wilson 95% interval of **88.8-97.8%**.
- Zero silent errors in 100 labels still only bounds the true rate at about **3.0%** (1 − 0.05^(1/100)).
- Don't treat ~900 fields as independent. The naive interval of 93.4-96.2% is too narrow, so bootstrap by label. Compare arms with paired tests (McNemar) on the same images.
- n=100 can separate a 95% pipeline from an 80% one, not 95% from 92%.

*Step 6, decision rule (proposal).*
- Keep Apple as primary if its silent-error rate is ≤3 of 100 labels and macro-perfect is ≥85%.
- Make the LLM primary, where online, if it beats Apple by at least 10 points on macro-perfect (paired test significant) with no worse silent-error rate.
- Whichever wins, keep confirm-before-save unless silent errors reach 0 in 100 and a further sample agrees.

## 10. New questions surfaced

1. **Minimum OS:** is iOS/macOS 26 acceptable as the floor? iOS 27 would add on-device image input.
2. **Web label capture:** is web a v1 feature at all, or edit-only until the paid tier?
3. **Does personal v1 need an EU route?** It matters little for personal use but changes the vendor shortlist for release. Should the paid release require EU processing?
4. **iCloud and nutrition logs:** [guideline 5.1.3(ii)](https://developer.apple.com/app-store/review/guidelines/) says apps "may not store personal health information in iCloud." Does a food log with targets count? This could affect the sync choice and needs an answer before a paid release.
5. **Is a food log health data under GDPR Art. 4(15)?** This needs legal input before release, given C-21/23's broad reading.
6. **Photo retention:** keep label crops for audit and re-reading, or discard after save? They help correction, but add storage and privacy surface.
7. **Conflicts between barcode data and label:** which value wins by default, and should user corrections be contributed back to the database?
8. **Is a read wrong if the label is internally inconsistent?** Decide whether Intake records the printed number (FoodRepo's approach) or lets the user override with a note.
9. **PCC entitlement:** can a solo developer get it, and what are the actual daily limits? Apple's docs don't state the numbers I read.
10. **Re-checking prices:** Gemini 3.8 Flash and 3.6 Flash pricing changes on 2027-01-01. Re-check before committing.

## Sources (all checked 2026-11-02)

**Apple** (docs read through Apple's documentation JSON, since the pages need JavaScript)
- [VNRecognizeTextRequest](https://developer.apple.com/documentation/vision/vnrecognizetextrequest), [RecognizeTextRequest](https://developer.apple.com/documentation/vision/recognizetextrequest), [RecognizeDocumentsRequest](https://developer.apple.com/documentation/vision/recognizedocumentsrequest), [DocumentObservation](https://developer.apple.com/documentation/vision/documentobservation), [Table](https://developer.apple.com/documentation/vision/documentobservation/container/table), [confidence](https://developer.apple.com/documentation/vision/vnrecognizedtext/confidence)
- [DataScannerViewController](https://developer.apple.com/documentation/visionkit/datascannerviewcontroller), [isSupported](https://developer.apple.com/documentation/visionkit/datascannerviewcontroller/issupported)
- [Foundation Models](https://developer.apple.com/documentation/foundationmodels), [SystemLanguageModel](https://developer.apple.com/documentation/foundationmodels/systemlanguagemodel), [multimodal guide](https://developer.apple.com/documentation/foundationmodels/analyzing-images-with-multimodal-prompting), [Attachment](https://developer.apple.com/documentation/foundationmodels/attachment), [guided generation](https://developer.apple.com/documentation/foundationmodels/generating-swift-data-structures-with-guided-generation), [languages](https://developer.apple.com/documentation/foundationmodels/supporting-languages-and-locales-with-foundation-models), [PCC guide](https://developer.apple.com/documentation/foundationmodels/adding-server-side-intelligence-with-private-cloud-compute), [PrivateCloudComputeLanguageModel](https://developer.apple.com/documentation/foundationmodels/privatecloudcomputelanguagemodel)
- [WWDC25-272](https://developer.apple.com/videos/play/wwdc2025/272/), [WWDC26-237](https://developer.apple.com/videos/play/wwdc2026/237/), [WWDC26-241](https://developer.apple.com/videos/play/wwdc2026/241/) (transcripts)
- [Apple Intelligence requirements](https://support.apple.com/en-us/121115), [App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/)

**Vendors**
- Anthropic: [pricing](https://platform.claude.com/docs/en/about-claude/pricing), [models](https://platform.claude.com/docs/en/about-claude/models/overview), [vision](https://platform.claude.com/docs/en/build-with-claude/vision), [structured outputs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs), [data residency](https://platform.claude.com/docs/en/manage-claude/data-residency), [Bedrock](https://platform.claude.com/docs/en/build-with-claude/claude-in-amazon-bedrock), [retention](https://privacy.claude.com/en/articles/7996866-how-long-do-you-store-my-organization-s-data), [training](https://privacy.claude.com/en/articles/7996868-is-my-data-used-for-model-training)
- OpenAI: [pricing](https://developers.openai.com/api/docs/pricing), [vision](https://developers.openai.com/api/docs/guides/images-vision), [structured outputs](https://developers.openai.com/api/docs/guides/structured-outputs), [gpt-6-luna](https://developers.openai.com/api/docs/models/gpt-6-luna), [data controls](https://developers.openai.com/api/docs/guides/your-data)
- Google: [pricing](https://aistudio.google.com/docs/pricing), [models](https://aistudio.google.com/docs/models), [image understanding](https://aistudio.google.com/docs/image-understanding), [media resolution](https://aistudio.google.com/docs/media-resolution), [structured output](https://aistudio.google.com/docs/structured-output), [terms](https://aistudio.google.com/docs/terms), [ZDR](https://aistudio.google.com/docs/zdr), [data residency](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/learn/data-residency), [Vertex pricing](https://cloud.google.com/vertex-ai/generative-ai/pricing)
- Mistral: [OCR 4.1](https://docs.mistral.ai/models/ocr-4-1), [OCR docs](https://docs.mistral.ai/capabilities/document_ai/basic_ocr/), [models](https://docs.mistral.ai/getting-started/models/), [training](https://help.mistral.ai/en/articles/347617-do-you-use-my-user-data-to-train-your-artificial-intelligence-models), [hosting](https://help.mistral.ai/en/articles/347629-where-do-you-store-my-data-or-my-organization-s-data), [ZDR](https://docs.mistral.ai/admin/monitor-comply/zero-data-retention)
- Third-party benchmark: Artificial Analysis ([Haiku 5.5](https://artificialanalysis.ai/models/claude-haiku-5-5), [Flash-Lite](https://artificialanalysis.ai/models/gemini-3-5-flash-lite), [Luna](https://artificialanalysis.ai/models/gpt-6-luna))

**Law and guidance**
- [Regulation 1169/2011, consolidated](http://publications.europa.eu/resource/celex/02011R1169-20180101), EN, NL, DE and FR via the EU Publications Office
- [GDPR](http://publications.europa.eu/resource/celex/32016R0679), [CJEU press release, C-21/23](https://curia.europa.eu/jcms/upload/docs/application/pdf/2024-10/cp240159en.pdf)
- [EC tolerance guidance](https://food.ec.europa.eu/system/files/2016-10/labelling_nutrition-vitamins_minerals-guidance_tolerances_1212_en.pdf), [summary table](https://food.ec.europa.eu/system/files/2016-10/labelling_nutrition-vitamins_minerals-guidance_tolerances_summary_table_012013_en.pdf)

**Papers and datasets:** linked in section 6, plus [FoodRepo](https://pmc.ncbi.nlm.nih.gov/articles/PMC6040205/) and [Robotoff docs](https://openfoodfacts.github.io/robotoff/references/predictions/nutrient-extraction/).

**Could not fetch**
- EUR-Lex directly: empty response on every URL tried. Same text obtained from the EU Publications Office.
- [openai.com/api/pricing](https://openai.com/api/pricing/): 403. Used OpenAI's developer pricing page.
- [mdpi.com article](https://www.mdpi.com/2313-433X/11/8/271): 403. Used the PMC copy.
- [mistral.ai/pricing](https://mistral.ai/pricing): no figures. `docs.mistral.ai/deployment/ai-studio/pricing/`: 404. Used the OCR 4.1 model page.
- Apple's HTML docs: title only. `foundationmodels/ocrtool`: 404, so `OCRTool` details come from the guide and WWDC only.
- **Not used, could not trace to a source:** two search-snippet claims (95% attribute accuracy; 95.5% detection and 76.7% value extraction via ML Kit).
