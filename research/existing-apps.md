# What existing apps already do against the wanted features

Checked 2 November 2026. Answers the research ticket of the same name under the "Intake v1 spec" map. This describes the market only; whether to build is a question for a later ticket.

## Short answer

- **No checked app covers the whole list.** Cronometer comes closest: every listed feature except native Daily Dozen tracking is documented, nearly all of it on the free tier. Its documented weakness for this project is packaged Dutch products.
- **Daily Dozen and calories live in separate apps.** The official Daily Dozen app does the checklist and the 21 Tweaks. No calorie tracker checked offers either natively (absence of evidence, see "Limits").
- **Dutch product coverage and full feature depth pull apart.** Mijn Eetmeter has the Dutch data but only 22 nutrients, no documented quick add and no label capture. The international apps have the features, but Cronometer and MacroFactor both document or draw complaints about thin Dutch coverage.
- **Barcode-then-label capture is rare.** Cronometer and MacroFactor both document it, and a third party reports label scanning at Lose It. MyFitnessPal and Yazio document a barcode miss leading to manual entry. MacroFactor's label scanner does not list Dutch as a supported language.
- **Exercise handling is a real split.** Cronometer, MyFitnessPal and Yazio add burned calories to the target. MacroFactor refuses to, on purpose, and adjusts from intake and weight trend.
- **Most reported friction:** barcode and other basics behind paywalls, wrong crowdsourced data, cluttered or changed interfaces, and logging effort itself.

## Evidence labels

- **[P]** First-party page read directly.
- **[E]** First-party page seen only as a search excerpt, because direct fetching was blocked (HTTP 403 on the MyFitnessPal, Yazio and Cronometer support sites). Worded by the search tool, so weaker than [P].
- **[L]** Lower trust: user reviews, forum posts by users, third-party articles.
- **[I]** My inference.
- **n/v** Not verified: nothing first-party found either way. This does not mean the feature is absent.

Source codes such as C1 are listed under "Sources".

## Feature matrix

### Cronometer, MacroFactor, MyFitnessPal

| Feature | Cronometer | MacroFactor | MyFitnessPal |
|---|---|---|---|
| Quick add | Free. Calories, protein, fat or carbs as separate entries [P C1] | Energy, protein, fat, carbs, alcohol [P M1] | Calories free; macros need Premium [E F1] |
| Barcode scan | Free [P C2, C3] | Yes [P M1]. Netherlands not among the "excellent" countries [P M2] | Premium only since October 2022 [E F2; P F3]. Not on the website [E F2] |
| Editable values after scan | Amount editable. Wrong data: report it, or create a custom food [E C4] | Units and quantity before logging [P M1]. Changing nutrient values: n/v | Can re-link to a better match, which changes the barcode for everyone [E F4]. Own copy via My Foods [E F5] |
| Barcode, then label, for an unknown product | Yes, free and instant: photos of label and front make a Custom Food [P C3] | Yes: automatic switch to label scanner [P M3]. Optimised for US and Canada; experimental for English, German, French, Spanish labels. Dutch not listed (page updated January 2024) [P M4] | Manual "Create a Food", free, published to all users [E F5]. No label reading found (n/v). Meal Scan reads plates, not labels [E F6] |
| Full nutrient data, common databases | 84 nutrients free, 95 Gold [P C2]. NCCDB and USDA [E C5]; NEVO named as a source [P C6], but staff said Dutch data is mainly whole foods and meals, not packaged (2021) [P C7] | Micronutrients tracked; 1.36M verified foods plus 4M barcodes [P M5]. Open Food Facts used for barcodes [E M6]. Licence terms n/v | 20.5M+ items [P F7]. Deeper breakdowns are Premium [E F1]. New entries join the public database unverified, per a competitor's blog [L X1] |
| Saved meals showing ingredients and calories | Free. "Explode" creates one diary entry per item, each editable [P C8] | Recipes are "bundles"; "Expand Ingredients" and "Explode" [P M1; E M7, M12]. Per-ingredient calories before exploding: n/v | Meals exist [E F8]. Per-ingredient display: n/v (page blocked) |
| Calorie, fat, protein targets | Free custom macro targets; per-weekday needs Gold [P C2] | Yes, coached and adjusted weekly [P M5] | Calculated initial goals [E F9]. Custom macros and per-day goals need Premium [P F3] |
| Targets adjusted for exercise | Optional toggle adds expenditure above baseline [E C9] | **Deliberately not.** Uses intake and weight trend, not wearable or manual exercise estimates [P M8] | Added automatically; turning off needs Premium [E F1, F10] |
| Weight from Apple Health | Yes [P C10] | Yes, both directions [E M9] | Yes, both directions [E F11] |
| Exercise from Apple Health | Yes, plus general activity [P C10] | Not used for nutrition targets [P M8] | Yes, same day, one way into the app [E F11] |
| Daily Dozen or 21 Tweaks | Not native. Staff suggested custom biometrics as a workaround (2017) [P C11]. Custom biometrics now need Gold [P C2] | Not found (n/v) | Not found (n/v) |
| iPhone / iPad | Yes / yes [P C12] | Yes / yes [P M10] | Yes / yes [P F7] |
| macOS | Apple Silicon [P C12] | Apple Silicon [P M10] | No Mac entry in the Dutch listing [P F7] |
| Web | Yes [E C13] | No. "Planned" [P M5] | Yes, but no barcode or Meal Scan [E F2, F6] |
| Price (Netherlands where shown) | Free with ads. Gold €11.99 a month or €69.99 a year in the Dutch store [P C12]; $10.99 or $59.99 on its site [P C2] | No free tier, 7-day trial [P M5]. Dutch store about €12 a month, €48.99 for six months, from €73.99 a year [P M10] | Free. Premium €9.99 a month or €49.99 a year [P F3, F7]. Prices conflict, see "Open conflicts" |

### Lose It, Yazio, Mijn Eetmeter

| Feature | Lose It | Yazio | Mijn Eetmeter |
|---|---|---|---|
| Quick add | n/v (site inaccessible) | Description, calories, optional macros [E Y1] | Not documented. Search, barcode, favourites, recurring items [P E1, E2] |
| Barcode scan | Listed under Premium [P L1]. Third party cites a transition notice with access differing between accounts [L X4] | Yes [P Y2]. Free per "all calorie-counting functions are always free" [E Y3; I] | Free [P E3] |
| Editable values after scan | n/v | Editing public foods is Pro only [E Y4] | Amounts per day [P E1]. Changing nutrient values: n/v |
| Barcode, then label, for an unknown product | Third party says nutrition-label scanning is moving into Premium [L X4]. Flow after a barcode miss: n/v | Add it by hand; barcode foods become public [E Y5]. No label reading found (n/v) | Add own product by hand from the label. Not checked by the Voedingscentrum, and many own products make vitamin advice unreliable [P E1] |
| Full nutrient data, common databases | 56M+ items [P L1]. Macros, vitamins, minerals are Premium [P L1] | 4M items [P Y6]. Fatty acids, fibre, sugar, salt, vitamins, minerals are Pro [E Y3] | Up to 22 nutrients [P E2]. Levensmiddelendatabank (Voedingscentrum and RIVM) plus manufacturer data, 120,000 to 150,000 products [P E1, E3, E4] |
| Saved meals showing ingredients and calories | n/v | Saved combinations logged as one entry [E Y7]. Per-ingredient display: n/v | "Mijn gerechten"; portion size of saved meals cannot be adjusted [P E1]. Review says values per logged product are not shown [L E5] |
| Calorie, fat, protein targets | Calculated plan is Basic; macro tracking is Premium [P L1] | Calculated; macro percentages adjustable [E Y8] | Based on sex, age, activity level [P E1]. Own macro targets: n/v |
| Targets adjusted for exercise | Third party says synced exercise calories feed the daily budget [L X2] | Automatic, can be turned off [E Y9] | Activity level setting changes the advice [P E1]. Per-workout: n/v |
| Weight from Apple Health | HealthKit sync listed under Premium [P L1]. Detail n/v | Yes [E Y10] | n/v |
| Exercise from Apple Health | As above [P L1] | Yes, active energy [E Y10] | Watch links through Health to the movement meter [P E4] |
| Daily Dozen or 21 Tweaks | Not found (n/v) | Not found (n/v) | Not found. Follows the Dutch Schijf van Vijf instead [P E2] |
| iPhone / iPad | Yes / yes [P L2] | Yes / yes [P Y6] | Yes / no [P E3] |
| macOS | Apple Silicon [P L2] | Help centre says no; store listing shows Mac, see "Open conflicts" | No [P E3] |
| Web | n/v (site inaccessible) | No [E Y11] | Yes, shared account with the Voedingscentrum site [P E1, E4] |
| Price (Netherlands where shown) | Free tier. Premium €10.99 to €85.99 by plan, lifetime €59.99 to €69.99 [P L2] | Free tier. Pro 3-month €23.99 to €35.99, 12-month €11.90 to €59.90, 7-day trial [P Y6; E Y12] | Free, government funded [P E3, E4] |

### Other relevant apps

- **Dr. Greger's Daily Dozen (NutritionFacts.org).** Free with no purchases. iPhone, iPad and Apple Silicon Mac; no web found. Daily Dozen checklist, 21 Tweaks, weight through HealthKit, charts, reminders, data export, 19 languages. Version 4.0.1 dated March 2025 [P D1]. Its description mentions no calories, macros or barcode scanning, so I read it as a checklist only [I].
- **Fitatu.** Lightly checked, marketing page only. Claims Albert Heijn, Jumbo, Lidl and Aldi products, barcode scanning, 39+ nutrients, two-way Apple Health and an Apple Watch app. Photo estimation is Premium+, and computer access is a Premium feature. Prices not checked [P Z1].

## Where users report the most friction

Lower trust throughout except the peer-reviewed studies in item 1. Ordering is my judgement from what I sampled, not a measured ranking [I]. I read the review excerpts visible on Dutch App Store pages, not a full sample. Reddit was not checked.

1. **Logging effort.** The one high-trust finding: usability was the most frequent barrier across studies in a systematic review, with apps called confusing, complex and time-consuming, plus database difficulties such as too many options and missing entries [P R1]. A 2026 scoping review reports 15 to 20 minutes of logging a day, calls manual entry a major deterrent, and notes regional foods missing from databases [P R2]. In a mixed-methods study, high time investment was the main reason former users quit (14 of 38) [P R3].
2. **Basics behind paywalls.** MyFitnessPal's barcode paywall is still the loudest complaint [L X3, F12]. A third-party analysis of low-star reviews attributes 29% to paywall and upsell. It publishes no sample size and comes from a review-tool vendor, so treat the number as indicative only [L X3].
3. **Wrong data.** The same analysis gives 18% to database errors [L X3]. Examples: Lose It portions and calorie values [L L3]; Yazio user-entered values "kloppen totaal niet" [L Y13]; Cronometer values differing from the Dutch packaging, in a 2022 forum thread [L C14].
4. **Thin Dutch and European coverage in international apps.** MacroFactor reviewers report hardly any European products, 4 to 5 manual entries per shopping trip, and one regretting a yearly plan [L M11]. A Dutch Cronometer user had to create custom foods for most Albert Heijn own-brand items (2022) [L C14]. Reviewers also ask Cronometer for a Dutch interface [L C15].
5. **Cluttered or changed interfaces.** Yazio: pop-ups, "dozens of screens", childish gamification [L Y13]. MyFitnessPal: new interface and macro goals not on one page [L F12]. MacroFactor: extra steps when logging [L M11].
6. **Exercise and watch expectations.** A MacroFactor reviewer objects to paying about €80 with no Apple Watch activity sync [L M11]. This is the documented design choice above [P M8].
7. **Lock-in, connection and reliability.** Cronometer reviewers cite no export, an always-on internet requirement and logouts [L C15]. Eetmeter reviewers cite account errors and a scanner needing several tries, and ask for fitness app links and meal times [L E5].

## Open conflicts and limits

- **MyFitnessPal price.** Its premium page and the Dutch store show €9.99 a month [P F3, F7]. Support text says $19.99 a month, with Premium+ at $24.99 [E F13]. A Dutch review mentions €21.99 a month [L F12]. Prices seem to vary by region or date, and Premium+ did not appear on the page served to me [I].
- **Yazio on Mac.** The help centre says there is no Mac version [E Y11], while the Dutch store lists Mac [P Y6]. Probably the iPad app running on Apple Silicon [I].
- **Eetmeter own products.** The manual documents adding them [P E1], while a review says this is impossible [L E5]. I trust the manual.
- **Eetmeter product count.** 120,000 in the store listing, 150,000 on its site [P E3, E4].
- **Stale listing.** The store page read for MacroFactor showed a last update of September 2025, which may be an old cached copy.
- **Lose It is the weakest column.** Its website and help centre could not be read at all, so it rests on App Store listings.
- **"Not found" for Daily Dozen** is a failure to find, not proof of absence.
- **Pages read through a summariser.** Direct page reads were summarised by a fetch tool, so re-check any exact wording before relying on it.
- **No app was used hands-on.** How well a feature works is taken from documentation and reviews, not tested. Dutch barcode hit rates were not measured for any app.
- **Database licences not examined.** Nothing here establishes what any data source permits for a later paid release.

## Sources

All accessed 2 November 2026.

**Cronometer**
- C1 [Transition from MyFitnessPal (2025)](https://cronometer.com/blog/my-fitness-pal-to-cronometer/)
- C2 [Cronometer Gold](https://cronometer.com/gold/)
- C3 [How to use the barcode scanner](https://cronometer.com/blog/how-to-use-the-barcode-scanner/)
- C4 [Mobile: report an issue](https://support.cronometer.com/hc/en-us/articles/360021237511-Mobile-Report-an-Issue)
- C5 [Data sources](https://support.cronometer.com/hc/en-us/articles/360018239472-Data-Sources)
- C6 [Accurate databases](https://cronometer.com/features/accurate-databases.html)
- C7 [Forum: European food databases, staff reply 2021](https://forums.cronometer.com/discussion/3968/is-it-planned-to-support-european-food-databases-as-well)
- C8 [Custom meals](https://cronometer.com/blog/custom-meals/)
- C9 [Energy target](https://support.cronometer.com/hc/en-us/articles/31975503009044-Energy-Target)
- C10 [Apple Health integration](https://cronometer.com/blog/apple-health/)
- C11 [Forum: Daily Dozen, staff reply 2017](https://forums.cronometer.com/discussion/98/dr-greger-039-s-daily-dozen-checklist-counter)
- C12 [Dutch App Store listing](https://apps.apple.com/nl/app/cronometer-calorie-counter/id1145935738)
- C13 [Create custom meal (web)](https://support.cronometer.com/hc/en-us/articles/17687459173908-Create-Custom-Meal)
- C14 [Forum: Dutch brands, user posts 2022](https://forums.cronometer.com/discussion/5017/request-for-certain-brands-to-be-added)
- C15 [Dutch App Store reviews](https://apps.apple.com/nl/app/cronometer-calorie-counter/id1145935738?see-all=reviews&platform=iphone)

**MacroFactor**
- M1 [How to log food](https://help.macrofactorapp.com/en/articles/215-how-to-log-food-in-macrofactor)
- M2 [Regional database coverage](https://help.macrofactorapp.com/en/articles/25-how-robust-is-the-database-coverage-in-my-region)
- M3 [Label scanner](https://help.macrofactorapp.com/en/articles/213-label-scanner)
- M4 [Label scanner announcement](https://macrofactor.com/label-scanner-announcement/)
- M5 [Product page](https://macrofactor.com/macrofactor/)
- M6 [European barcode expansion (2023)](https://macrofactor.com/mm-april-2023/)
- M7 [Save a meal for later use](https://help.macrofactorapp.com/en/articles/239-save-a-meal-for-later-use)
- M8 [Drawbacks of wearables for targets (updated 2026)](https://macrofactor.com/wearables/)
- M9 [Apple Health connection](https://help.macrofactorapp.com/en/articles/65-connect-health-connect-or-apple-health)
- M10 [Dutch App Store listing](https://apps.apple.com/nl/app/macrofactor-macro-tracker/id1553503471)
- M11 [Dutch App Store reviews](https://apps.apple.com/nl/app/macrofactor-macro-tracker/id1553503471?see-all=reviews&platform=iphone)
- M12 [Explode recipes](https://help.macrofactorapp.com/en/articles/3-explode-recipes)

**MyFitnessPal**
- F1 [Premium features](https://support.myfitnesspal.com/hc/en-us/articles/360032625951-MyFitnessPal-Premium-features)
- F2 [Barcode scanner](https://support.myfitnesspal.com/hc/en-us/articles/360032624771-How-to-use-the-Barcode-Scanner)
- F3 [Premium page](https://www.myfitnesspal.com/premium)
- F4 [Scanner returned the wrong item](https://support.myfitnesspal.com/hc/en-us/articles/360032270912-The-barcode-scanner-returned-the-wrong-item)
- F5 [Food not in the database](https://support.myfitnesspal.com/hc/en-us/articles/360032271992-How-to-log-a-food-that-s-not-in-the-database)
- F6 [Meal Scan](https://support.myfitnesspal.com/hc/en-us/articles/360045761612-Meal-Scan-FAQ)
- F7 [Dutch App Store listing](https://apps.apple.com/nl/app/myfitnesspal-calorie-counter/id341232718)
- F8 [Create meals and recipes](https://blog.myfitnesspal.com/create-meals-recipes-myfitnesspal/)
- F9 [Initial goals](https://support.myfitnesspal.com/hc/en-us/articles/360032625391-How-does-MyFitnessPal-calculate-my-initial-goals)
- F10 [Why goals change with exercise](https://support.myfitnesspal.com/hc/en-us/articles/360032623851-Why-your-daily-nutrient-values-and-calorie-goal-change-you-log-exercise)
- F11 [Apple Health sync](https://support.myfitnesspal.com/hc/en-us/articles/360032271092-Apple-Health-connection-and-syncing)
- F12 [Dutch App Store reviews](https://apps.apple.com/nl/app/myfitnesspal-calorie-counter/id341232718?see-all=reviews&platform=iphone)
- F13 [Free, Premium and Premium+](https://support.myfitnesspal.com/hc/en-us/articles/34889191368077-The-difference-between-Free-Premium-and-Premium)

**Lose It**
- L1 [UK App Store listing](https://apps.apple.com/gb/app/lose-it-calorie-counter/id297368629)
- L2 [Dutch App Store listing](https://apps.apple.com/nl/app/lose-it-calorie-counter/id297368629)
- L3 [US App Store listing, reviews](https://apps.apple.com/us/app/lose-it-calorie-counter/id297368629)

**Yazio**
- Y1 [Log calories without ingredients](https://help.yazio.com/hc/en-us/articles/360000530018-Can-I-log-my-meal-s-calories-without-adding-the-individual-ingredients)
- Y2 [iOS app page](https://www.yazio.com/en/ios-app)
- Y3 [How Pro supports goals](https://help.yazio.com/hc/en-us/articles/208847005-How-does-Yazio-Pro-support-my-goals) and [Why pay for Pro](https://help.yazio.com/hc/en-us/articles/360000592809-Why-do-I-have-to-pay-for-YAZIO-PRO)
- Y4 [Edit public foods](https://help.yazio.com/hc/en-us/articles/4576053290001-How-can-I-edit-public-foods)
- Y5 [Scanned product not recognised](https://help.yazio.com/hc/en-us/articles/208555549-The-scanned-product-will-not-be-recognized-why) and [Own foods](https://help.yazio.com/hc/en-us/articles/360000629817-How-can-I-create-and-manage-my-own-foods)
- Y6 [Dutch App Store listing](https://apps.apple.com/nl/app/yazio-calorie-counter-diet/id946099227)
- Y7 [What are meals](https://help.yazio.com/hc/en-us/articles/360000620798-What-are-meals)
- Y8 [Adjust goals](https://help.yazio.com/hc/en-us/articles/29832253731857-How-can-I-adjust-my-goals) and [Macronutrient ratio](https://help.yazio.com/hc/en-us/articles/360000932785-Do-I-have-to-stick-to-the-suggested-macronutrient-ratio)
- Y9 [Why goals change](https://help.yazio.com/hc/en-us/articles/360002474498-Why-do-my-calorie-goal-and-nutrient-goals-change)
- Y10 [Yazio and Apple Health](https://help.yazio.com/hc/en-us/articles/360018833998-Yazio-and-Apple-Health)
- Y11 [Technical requirements](https://help.yazio.com/hc/en-us/articles/360000752725-Technical-Requirements-for-Yazio)
- Y12 [7-day free trial](https://help.yazio.com/hc/en-us/articles/4436727821073-What-is-our-7-Day-Free-Trial-Subscription)
- Y13 [Dutch App Store reviews](https://apps.apple.com/nl/app/yazio-calorie-counter-diet/id946099227?see-all=reviews&platform=iphone)

**Mijn Eetmeter (Voedingscentrum)**
- E1 [Help](https://mijn.voedingscentrum.nl/nl/dashboard/eetmeter/help/)
- E2 [Handleiding](https://www.voedingscentrum.nl/nl/thema/apps-en-tools-voedingscentrum/mijn-eetmeter-app-online/handleiding-mijn-eetmeter.aspx)
- E3 [Dutch App Store listing](https://apps.apple.com/nl/app/mijn-eetmeter/id663382012)
- E4 [Product page](https://www.voedingscentrum.nl/nl/thema/apps-en-tools-voedingscentrum/mijn-eetmeter-app-online.aspx) and [App Store description](https://apps.apple.com/by/app/mijn-eetmeter/id663382012)
- E5 [Dutch App Store reviews](https://apps.apple.com/nl/app/mijn-eetmeter/id663382012?see-all=reviews&platform=iphone)

**Others**
- D1 [Dr. Greger's Daily Dozen, Dutch App Store](https://apps.apple.com/nl/app/dr-gregers-daily-dozen/id1060700802)
- Z1 [Fitatu Netherlands](https://www.fitatu.com/nl)
- R1 [König et al. 2021, JMIR mHealth and uHealth: barriers to and facilitators for using nutrition apps](https://pmc.ncbi.nlm.nih.gov/articles/PMC8409150/)
- R2 [Dugas et al. 2026, JMIR mHealth and uHealth: calorie-counting apps scoping review](https://pmc.ncbi.nlm.nih.gov/articles/PMC13085986/)
- R3 [Incorporating consumers' needs in nutrition apps: mixed methods study](https://pmc.ncbi.nlm.nih.gov/articles/PMC10337335/)
- X1 [Nutrola: barcode scanner comparison (competitor blog)](https://nutrola.app/en/blog/nutrola-vs-myfitnesspal-vs-lose-it-best-barcode-scanner)
- X2 [Calorie Trackers: Lose It review](https://calorie-trackers.com/reviews/lose-it/)
- X3 [Unstar: calorie apps ranked by 1-star reviews](https://unstar.app/blog/calorie-tracking-apps-ranked-1-star-reviews-2026)
- X4 [Amy Food Journal: Lose It review](https://amyfoodjournal.com/blog/lose-it-app-review)
