# Food data sources: coverage, nutrient depth, licensing and cost

Research for the ticket "Food data sources: coverage, nutrient depth, licensing and cost", under the map "Intake v1 spec". All sources were checked on 2026-11-02. This is not legal advice.

## Answer

Use **Open Food Facts (OFF) for barcodes** and **barcode-plus-label capture for the misses**. Use **NEVO for generic Dutch foods only while the app is free**, behind a replaceable source boundary.

- **Paid release:** NEVO's conditions forbid charging end users for use of its data. Before charging, get RIVM's written permission or swap in Ciqual or USDA FoodData Central (FDC).
- **Commercial APIs:** do not use FatSecret, Nutritionix or Edamam in v1. Their storage terms conflict with an offline-first app that keeps editable values, and their NL coverage is unpublished. FatSecret also bars diet guidance, and its NL dataset is quote-only.
- **Corrections:** keep user corrections as a separate overlay, never as edits inside the source data. OFF's share-alike licence and NEVO's no-amendment rule both make this the safe design.

**Dutch barcode coverage has no published number from any provider.** In a small sample of one Dutch chain, OFF found 77% of food barcodes, and 63% had complete energy and macros. Expect roughly one first-time scan in three to need label capture or manual entry. Own-label products will do much worse (38% found).

## How to read this

- **[V]** I read the primary text directly.
- **[S]** The primary page was read through a summarising fetch tool, so the wording may be paraphrased. Re-read it before relying on exact wording.
- **[M]** I measured this myself on 2026-11-02.
- **[I]** My inference, not a statement by the source.

## Comparison

| Source | NL barcodes | Depth beyond macros | Cost | Limits | Offline or bulk | Caching | User corrections | Later paid release |
|---|---|---|---|---|---|---|---|---|
| **Open Food Facts** | No published rate. 114,305 NL-tagged products [M]; 77% sample hit [M] | Label values; micronutrients are sparse [M] | Free [V] | 15 product reads and 10 searches per minute per IP [V] | Nightly JSONL, CSV, Parquet and MongoDB, plus 14 days of deltas [V] | Yes, with a local cache explicitly suggested [V] | Yes, but publicly used adaptations are share-alike [V] | Yes: commercial use allowed with attribution and share-alike [V] |
| **NEVO** | None: generic foods, no barcode lookup [I] | Best: about 130 nutrients across 2,328 foods [V] | Free [V] | No API found [I] | Full dataset download [V] | Not addressed; dataset download is allowed [V] | Additions only, clearly marked; no amendments [V] | **Blocked as written: may not charge end users for use of the data** [V] |
| **USDA FDC** | 0 of 25 [M]; Branded data covers US and NZ markets [V] | Good on analytical data types; Branded is label data [V] | Free [V] | 1,000 per hour per IP; key required [V] | Full downloads [V] | Yes: CC0 [V] | Yes: CC0 [V] | Yes: CC0 [V] |
| **FatSecret** | NL region exists [S]; no NL rate; "90%+" is a global claim [V] | About 19 nutrient fields [S] | Barcode needs Premier, priced by quote per market [V] | Unlimited on Premier; 5,000 per day on Basic [V] | No bulk offer found [I] | 24 hours, except identifiers [V] | No edit right found; Premier can request database edits [V] | Possible on a paid contract; see the guidance risk below |
| **Nutritionix** | No NL statement found; ">92%" is not NL-specific [V] | Label and dietitian data; detailed nutrient fields are a premium add-on [S] | Free for 2 MAU; $499 per month for 200 MAU; $999 for 1,000 MAU; from $1,850 [V] | Tied to MAU [V] | Bulk licence by contact [V] | Food log history only, MVP plan and up [V] | Not found | Possible, at those prices |
| **Edamam** | No NL statement found; 700,000 to 790,000 UPCs overall [V] | 28 nutrients for basic foods; label data for UPC foods [V] | $14, $69 or $299 per month, or custom [V] | 50 to 300 UPC requests per minute by plan [V] | None; building a copy is prohibited [V] | Macros only, on Core and Plus; IDs and labels on Basic; full caching is an add-on [V] | Not found | Possible, but data must be returned on cancellation [V] |

## Measured Dutch coverage

The API counts come from OFF's search endpoint. Of 114,305 NL-tagged products, 66,444 (58%) have the `nutrition-facts-completed` state, out of 4,890,815 products worldwide [M]. The two NL counts came from different OFF hostnames using the same country filter. These counts say nothing about the share of the Dutch shelf that is covered.

To avoid picking barcodes from OFF itself, I took them from a Dutch supermarket:

1. I shuffled Hoogvliet's public catalogue sitemap with a fixed seed.
2. I read `gtin13` from 100 product pages, with paced requests. Its robots rules do not disallow these pages.
3. I looked each barcode up in OFF, within the published rate limit.
4. I set aside 8 household and pet products by hand, leaving 92 food and drink items.

| Group | n | Found in OFF | Found with energy, protein, fat and carbs |
|---|---:|---:|---:|
| All food and drink | 92 | 71 (77%; 95% CI 68–85) | 58 (63%; 53–72) |
| National and other brands | 71 | 63 (89%; 79–94) | 51 (72%; 60–81) |
| Chain own-label (G'woon, Hoogvliet) | 21 | 8 (38%; 21–59) | 7 (33%; 17–55) |
| Fresh and counter items (overlaps the rows above) | 17 | 8 (47%; 26–69) | 8 (47%; 26–69) |

- **Depth, of the 71 found:** fibre on 37; any micronutrient on 11.
- **Freshness, of the 66 found in my first food filter:** median last edit about 4 months ago; 3 were untouched for over 3 years.
- **FDC:** 0 of the first 25 food barcodes, using the public `DEMO_KEY` from USDA's docs. A positive control with a US barcode did match.

**Limits of this test:**

- It covers one mid-sized regional chain on one day, so the results may not carry over.
- Albert Heijn and Jumbo were not tested. Both refused my robots.txt request, and neither was sampled. Their own-label ranges matter a lot, so this is the largest gap.
- The sample is by catalogue item, not weighted by what people buy. Real scans lean towards popular brands, so the real hit rate could be higher (**[I]**).
- It checks presence and completeness, not whether values match the pack.
- FatSecret, Nutritionix and Edamam could not be tested without signing up.

A better test is cheap: scan 50 items from your own kitchen against OFF.

## Source notes

### Open Food Facts

- **Licences [V]:** the database is ODbL, its contents are under the Database Contents License, and images are CC BY-SA. OFF says these allow reuse "for all purposes, including commercial use", under attribution and share-alike. Packaging images may carry separate third-party rights. ([terms](https://world.openfoodfacts.org/terms-of-use))
- **Share-alike [V]:** if you publicly use an adapted database, or a work produced from one, you must offer it under ODbL. You must also offer the whole database or a file of the changes, free over the internet. Incompatible content may not be added to it. ([ODbL 4.4 and 4.6](https://opendatacommons.org/licenses/odbl/1-0/))
- **What stays outside share-alike [V]:** an unmodified copy in a collection of independent databases is a Collective Database, not a derivative. "Publicly" means use by persons other than you. ([ODbL 1.0 and 4.5](https://opendatacommons.org/licenses/odbl/1-0/))
- **OFF's reuse checklist [V]:** clickable attribution in the UI, bulk exports for caching, and "please send" user contributions back. ([data page](https://world.openfoodfacts.org/data))
- **API rules [V]:** production use is welcome if "1 API call = 1 real scan". Search must not power search-as-you-type. A custom User-Agent with a contact address is expected. ([API docs](https://openfoodfacts.github.io/openfoodfacts-server/api/))
- **Quality [V]:** values are entered by contributors, may contain errors, and carry no guarantee. They "must not be used for medical purposes". ([terms](https://world.openfoodfacts.org/terms-of-use))

What this means for the app (**[I]**):

- Per-IP limits suit direct calls from each device, because one person's scanning stays far below 15 per minute. A shared server would pool all users under one IP limit, so it should use exports instead.
- Barcode search needs a local or server index built from the exports, not the search API.
- Private per-user corrections look like non-public use. Corrections shown to other users would be a public derivative database, so they would need to be published or contributed back. Get this confirmed by reuse@openfoodfacts.org before a paid release.
- By simple proportion, an NL subset is about 2% of the export, so it should be small enough to bundle or sync. I did not measure this.

### NEVO (RIVM)

All from the [conditions for version 2025/9.0](https://www.rivm.nl/sites/default/files/2025-11/Conditions-of-use-NEVO-online-2025-dataset_0.pdf) [V]:

- "The user is **not** allowed to charge (end)users for the use of NEVO online version 2025/9.0 data."
- Additions are allowed if clearly marked as additions. "The user is **not** entitled to make amendment" to the dataset.
- Software output must carry the reference "Based on data from NEVO online version 2025/9.0, RIVM, Bilthoven", with "and other data sources" added where relevant.
- Users are asked to move to each new version when it is released.

Other facts [V]:

- It is the official Dutch food composition database, and the source of each value is known. ([RIVM](https://www.rivm.nl/en/dutch-food-composition-database))
- Version 2025/9.0 covers about 130 nutrients across 2,328 foods per 100 g. ([manual](https://www.rivm.nl/en/dutch-food-composition-database/use-of-nevo-online/manual))
- The whole dataset is a free download once you agree to the conditions. ([download](https://www.rivm.nl/en/dutch-food-composition-database/use-of-nevo-online/request-dataset))

I did not download the dataset, because requesting it enters the agreement. Field-level claims therefore come from RIVM's documents.

**Open points [I]:**

- The conditions do not say whether a free tier inside a paid app is allowed, or whether NEVO values already saved in a paying user's history are affected.
- They do not explicitly cover redistributing the dataset inside an app. Ask nevo@rivm.nl.
- Treat a user's correction as a new user food "based on" the NEVO item, not as an edit to it.

### Paid-safe generic food fallbacks

- **USDA FDC:** "in the public domain", published under CC0. No permission is needed, though citation is requested. The limit is 1,000 requests per hour per IP, or 30 for `DEMO_KEY`. Keys must not be public, so an app should bundle the downloads rather than ship a key. ([API guide](https://fdc.nal.usda.gov/api-guide/)) [V]
  - Foundation Foods and Branded were last released 10/2026; SR Legacy is frozen at 04/2018. ([downloads](https://fdc.nal.usda.gov/download-datasets/)) [V]
  - Branded data is label data supplied by industry; NZ products have been added. ([FAQ](https://fdc.nal.usda.gov/faq/)) [V]
  - **Weakness [I]:** foods and names are American, not Dutch.
- **Ciqual (ANSES, France):** version 2025 covers 3,484 foods and is described as open data. ([ANSES](https://www.anses.fr/en/content/anses-ciqual-food-composition-table)) [S] The 2020 release is listed under Licence Ouverte. ([data.gouv.fr](https://www.data.gouv.fr/datasets/table-de-composition-nutritionnelle-des-aliments-ciqual-2020/informations)) [S] **The licence on the 2025 release is unconfirmed**; the Ciqual site returned a 503. It is a closer European fit than FDC (**[I]**), but it has fewer nutrients than NEVO and is not Dutch.

### FatSecret

- **Editions [V]:** Basic is free at 5,000 calls per day, US only, with no barcode. Premier Free is US only, for companies under US$1 million in revenue and funding, with non-US data available "at a discount" on request. Premier has 62+ countries, priced by market. Price is not published. ([editions](https://platform.fatsecret.com/api-editions))
- **Storage [V]:** content must be removed or re-requested within 24 hours unless explicitly storable indefinitely. ([terms 1.5](https://platform.fatsecret.com/terms)) Only identifiers such as `food_id` and `serving_id` are listed as storable. ([storable data](https://platform.fatsecret.com/docs/guides/storable-data)) [S] The editions page ticks "Caching" for Premier without saying what that allows, so this would need settling in a contract.
- **Guidance risk:** the terms bar using the API "to provide diet, nutrition or health advice, guidance or diagnosis". ([terms 1.7](https://platform.fatsecret.com/terms)) [V] **[I]** Calculated targets and Daily Dozen guidance could plausibly fall under this.
- **Other [V]:** attribution is required on free tiers. The licence can be terminated "at any time for any reason". Barcode is ["Premier Exclusive"](https://platform.fatsecret.com/docs/v1/food.find_id_for_barcode) [S]. NL is a [listed region](https://platform.fatsecret.com/docs/guides/localization) [S].

This is the one commercial option worth a quote if OFF's coverage proves too thin (**[I]**).

### Nutritionix (Syndigo)

- **Prices and plans [V]:** see the table. Plans are billed annually, and attribution is required below the top plan. Free and Starter have no caching. There are no non-commercial free trials. ([API page](https://www.nutritionix.com/api))
- **Caching [S]:** the only pre-approved case is a user's historical food log. Data cached for one user may not be shown to another. ([FAQ](https://docx.syndigo.com/developers/docs/faqs))
- **Scale [V]:** the same page claims both over 1M barcoded grocery foods and "over 600K UPCs", so the size is unclear.
- **Not found:** the API-specific terms. Syndigo's general [terms of use](https://www.syndigo.com/terms-of-use/) do not cover the API [S], and published rate limits were not found either.

### Edamam

All from the [Food Database API page](https://developer.edamam.com/food-database-api) [V] unless noted:

- Requests must be human-driven, and data may be used only for presentation to the requesting user unless storage is explicitly permitted.
- Saved data must stay in the end user's password-protected account, and may not form a copy of Edamam's data or a food search.
- An active subscription is required for as long as the data is used.
- On cancelling you "have to return all nutritional information" and can no longer use it. ([FAQ](https://developer.edamam.com/api/faq))
- Attribution is required. ([attribution](https://developer.edamam.com/attribution))
- **Not read:** the API Terms of Use page returned only an empty shell to my fetch.

### Authoritative Dutch sources that are not open to apps

- **LEDA (Voedingscentrum and RIVM):** label data for over 150,000 products sold in the Netherlands. The page describes use by those two bodies and gives no external access route. ([LEDA](https://www.voedingscentrum.nl/professionals/productaanbod-en-levensmiddelendatabank/what-is-the-leda-branded-food-database-.aspx)) [S]
- **GS1 Data Source:** the supplier-facing data pool that feeds these databases. I found no developer access route in a short search, and this was not investigated further.

## Recommended combination

| Need | Free personal v1 | Paid release |
|---|---|---|
| Barcode lookup | OFF API per real scan, optionally an NL subset from exports for offline | Same, plus attribution and a share-alike review |
| Barcode miss or incomplete data | Barcode-plus-label capture, editable, stored as the user's own food | Same; AI reading is already planned as a paid feature |
| Generic foods | NEVO | RIVM permission, or Ciqual or FDC |
| Corrections | Per-user overlay; optional contribution to OFF | Same; anything shared between users must be open or contributed |

Design consequences (**[I]**):

- Tag every stored value with its source, version and licence.
- Keep OFF, NEVO and user data in separate stores, and never merge them into one table. ODbL 4.4(d) bars adding incompatible content to a derivative, and NEVO bars amendments.
- Snapshot nutrition values onto each diary entry, so that a source swap doesn't change history.
- Show attribution for OFF and the required NEVO reference.

## Not verified

- Current prices and terms can change without notice; all are as read on 2026-11-02.
- FatSecret Premier's price, and whether its NL dataset holds Dutch supermarket barcodes.
- Barcode hit rates for Albert Heijn, Jumbo and other chains.
- Accuracy of OFF values against actual packs.
- `www.rivm.nl` serves an incomplete TLS chain. I completed it with the intermediate certificate named in its own certificate, rather than turning verification off. A search index independently returned the same NEVO cost and reference wording.
