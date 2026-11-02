# Apple Health and platform limits across iPhone, iPad, macOS and web

Checked 2026-11-02. Research only: the options below feed the "Platform and tech stack" grilling ticket, which makes the choice.

**Evidence labels:** **[V]** verified in a primary source, with a citation. **[T]** tested directly on a Mac for this note. **[I]** my inference, not stated by a source. **[R]** a report only, such as a bug-tracker comment, so test it before relying on it.

## Answer

- **Apple Health can be read on iPhone and on iPad (iPadOS 17 or later) only.** On macOS the framework links, but the app gets no data. The web has no access at all.
- **Mac and web can only get Health data through the app's own relay.** The iPhone or iPad app reads the data and passes on its own copy. That relay **should not be iCloud or CloudKit**, because App Review guideline 5.1.3(ii) says apps "may not store personal health information in iCloud".
- **Scanning works everywhere, at different quality.** iPhone and iPad have a ready-made system scanner. The Mac needs a custom capture screen and an awkward camera. The web needs a WebAssembly decoder, because the browser's own barcode API isn't usable across browsers.
- **Offline is solid on native and workable with care on the web.** Safari's storage rules are the web's main risk.
- **Every route to all four platforms needs some Swift or native HealthKit code, plus a sync layer that the web can reach.** No cross-platform framework removes either need. [I]

## What each platform can do

| | iPhone | iPad | macOS | Web |
|---|---|---|---|---|
| **Read weight and exercise from Health** | Yes | Yes, iPadOS 17+ | **No** | **No** |
| **Barcode scanning** | System scanner (`DataScannerViewController`), A12 or later | Same as iPhone | Custom: AVFoundation and Vision, webcam or iPhone as camera | Camera yes, native decoder no; use WASM |
| **Label text recognition on the device** | Yes (Vision or the system scanner) | Yes | Yes (Vision) | No browser API found; use WASM OCR or send to a server [I] |
| **Offline** | Full local store | Full local store | Full local store | Service worker and IndexedDB, with Safari's limits |

### Apple Health

- **iPhone and iPad:** "HealthKit data is also available to iPads running iPadOS 17 or later". iPhone and iPadOS 17+ each have their own store, and "HealthKit automatically syncs data between these devices". [V: 1, 2]
- **macOS:** the framework exists on "macOS 13 and later, but your app can't read or write HealthKit data", and `isHealthDataAvailable()` returns false. Apple says these devices have no copy of the store. Shared code can still compile, which helps a multiplatform app. [V: 1, 2]
  - `isHealthDataAvailable()` returned `false` on macOS 26.7.1. macOS 27 wasn't run, but Apple's current docs still say the same, and the HealthKit docs carry June 2026 updates. [T; V: 1, 3]
  - Apple's Health page names only "iPhone and iPad". [V: 4]
- **iPad app running on a Mac:** this doesn't get around the limit. iOS apps on Mac use the same frameworks as Mac Catalyst, and an Apple engineer stated in 2020 that the call returns false on Catalyst. Not tested on a Mac for this note. [V: 5, 6; I for this exact case]
- **The data itself:** `bodyMass` and `HKWorkout` are the types for weight and exercise. [V: 7, 8]

**Things the design must handle** (these apply on iPhone and iPad):

1. **A denied read looks like no data.** "From the app's point of view, no data of that type exists", so empty states can't say that permission was refused. [V: 9]
2. **Limited history is new in the 27 releases.** People can grant only a recent window of data rather than their full history. `getEarliestAuthorizedSampleDate` is available from iOS and iPadOS 27. Missing older weights may therefore not mean there are none. [V: 10, 11]
3. **The store is encrypted while the device is locked.** Queries then fail with `errorDatabaseInaccessible`, so background reads are unreliable. [V: 9, 12]
4. **Background delivery exists but is best-effort.** It needs its own entitlement, the wake-ups are rate-limited, and HealthKit stops delivering if the app fails to respond three times. It can't be tested in the Simulator. [V: 13, 14]
5. **A required-capability entry may need removing.** Xcode adds `healthkit` to the required device capabilities, which blocks installation on unsupported devices. Delete it if the app must also run where Health is absent. [V: 15]
6. **Purpose and privacy rules apply.** The use must be for health or fitness and clear in the UI. Any HealthKit app needs a privacy policy. [V: 9, 16]

### Barcode and label scanning

- **iPhone and iPad:** `DataScannerViewController` (iOS and iPadOS 16+, A12 Bionic or later) scans live video for text and machine-readable codes. EAN-13, EAN-8 and UPC-E are among Vision's supported symbologies. [V: 17, 18, 19]
- **macOS:** that scanner isn't available on macOS or Mac Catalyst. The building blocks are:
  - AVFoundation capture with Vision: `DetectBarcodesRequest` needs macOS 15+, and `VNDetectBarcodesRequest` is older. [V: 20, 21]
  - Text recognition with `RecognizeTextRequest` (macOS 15+). [V: 22]
  - Continuity Camera, which uses an iPhone's rear camera as a capture device. [V: 23]
  - **Dutch text recognition:** `nl-NL` is in Vision's list on macOS 26.7.1. iPhone and iPad weren't tested. [T]
  - **Ergonomics:** holding a product up to a Mac's fixed webcam will probably work poorly for small barcodes, so test it. A photo import or scanning on the iPhone may be the realistic Mac flow. [I]
- **Web:**
  - **Camera:** `getUserMedia` works across browsers, including Safari 11+, and needs HTTPS. [V: 24]
  - **`BarcodeDetector` can't be the base.** It is experimental. In Safari it sits behind a preference, Firefox lacks it, and desktop Chrome and Edge are partial; only Chrome for Android has it plainly. [V: 25] The Safari 27.0 and 27.2 feature posts don't mention it. [T: page search, 26]
  - **WASM decoders fill the gap.** For example, the MIT-licensed `barcode-detector` package wraps ZXing-C++ and supports `ean_13`. This adds a dependency and download weight. [V: 27; I for the cost]
  - **Home Screen camera access needs a device test.** The original standalone-mode bug was closed as fixed in 2020, and a separate bug was filed about repeated permission prompts. [V: 28] A search summary also pointed to later reports of a black camera stream, which I didn't trace to a source. [R]

Which approach reads labels best belongs to the "Nutrition label capture" ticket. This note only covers what each platform offers.

### Offline

- **Native:**
  - SwiftData offers local persistence on all three Apple platforms. [V: 29]
  - HealthKit data is stored on the device. [V: 9]
  - CloudKit "relies on the presence of the network", so sync waits for a connection. [V: 30]
  - SwiftData's iCloud sync has schema limits: no unique constraints, relationships must be optional, and production schemas are additive only. [V: 31]
- **Web:**
  - Service workers are available in Safari 11.1+, and `persist()` in 15.2+. [V: 32, 33]
  - **Safari's 7-day cap:** script-writable storage, including IndexedDB, is deleted after 7 days without interaction. Home Screen web apps' first-party domains are exempt. [V: 34]
  - Quotas are generous (up to 60% of disk per origin in a browser app), but data can still be evicted. WebKit grants `persist()` on heuristics such as being opened as a Home Screen web app. [V: 35]
  - **The Mac's "Add to Dock" web app starts with separate storage.** Safari copies cookies into it but no local storage. Whether the 7-day exemption also covers Dock apps isn't stated in what I read. [V: 36; open question]
  - **No background sync in Safari or Firefox**, so queued entries sync only while the app is open. [V: 37]
  - **The Netherlands is in the EU.** In 2024 Apple dropped its plan to remove Home Screen web apps there. I saw this through a search snippet of Apple's page, and the current English page no longer mentions it. Safari 26 now opens every site added to the Home Screen as a web app. Test on a real device here. [R: 38; V: 39]
- **Everywhere:** decoding a barcode needs no network, but finding its food does, unless the data is cached locally. How much to cache depends on the "Food data sources" ticket. [I]

## How Health data can reach Mac and web

| Route | Reaches | Cost | Limits |
|---|---|---|---|
| **Own backend relay:** iPhone/iPad app reads Health, sends a copy | Mac and web | Backend, auth, privacy policy; suits a shared food-log sync | Freshness depends on the app running; a locked device blocks reads [V: 9, 12] |
| **Relay on app open only** (subset of the above) | Mac and web | Smallest code; no background entitlement | Data is as old as the last time the app was opened [I] |
| **Direct device transfer** (Multipeer Connectivity or Network framework) | Mac only | No server; custom pairing and sync | Devices must be near or reachable; no web. Availability verified, the flow isn't [V: 40, 41; I] |
| **Manual export:** Health's "Export All Health Data" gives XML to import | Mac and web | XML parser and import screen; no entitlement | Manual and whole-history; no live updates [V: 42] |
| **Shortcuts automation** | Unclear | Fragile, user-maintained | I found no Apple documentation of a Health query action. **Unverified** |
| **Third-party aggregator** | Mac and web | Fee or dependency; not investigated | Still needs an iPhone app, and sharing is restricted to health or fitness services with permission [V: 9; I] |
| **iCloud or CloudKit relay** | Mac, and web via CloudKit JS | Would be cheap | **Ruled out by guideline 5.1.3(ii)** [V: 43] |

Apple documents no server API for a person's Health data, and Health's iCloud copy is end-to-end encrypted when two-factor authentication is on. [V: 44; I that no route exists through it]

### The iCloud rule

- **App Review guideline 5.1.3(ii),** page dated 16 October 2026: apps "may not store personal health information in iCloud". [V: 43]
- **Developer licence, section 3.3.3(D):** unless Apple permits otherwise in writing, no use of Apple software or services "(including iCloud, the iCloud Storage APIs, CloudKit APIs...)" to "create, receive, maintain or transmit any sensitive, individually-identifiable health information". The clause also refers to the US HIPAA definitions. [V: 45]
- **Weight and exercise copied from Health** clearly fall under the guideline, so keep them out of CloudKit. [I]
- **Open question: do food logs count?** Apple doesn't define the term in what I read, and calorie or nutrition logs could arguably be included. This decides whether SwiftData with CloudKit is safe for the food log at all. Ask Apple, or avoid the question by not using iCloud. [I]
- **Personal use:** App Review only sees what is submitted, so a personal build wouldn't meet the guideline, but the licence still applies. Since the map keeps a paid release possible, building on CloudKit now could force a sync rewrite later. [I]

## Technical approaches

All costs are my estimates for one developer. [I] The facts in the middle columns are verified where cited.

| Approach | Coverage | Health and scanning | Main costs |
|---|---|---|---|
| **A. SwiftUI app + separate web app** | SwiftUI: iOS, iPadOS, macOS, not web [V: 46] | Best native access to all of it | Two UIs, two languages (Swift, TypeScript), logic such as targets duplicated or moved to the server. Backend needed for web |
| **B. Web app everywhere + small native Health companion** | One web UI on all four; companion on iPhone/iPad | Companion reads Health and relays. Scanning via WASM, with web limits | Lowest UI work. Daily use lives with Safari's camera and storage limits, and needs two apps on iPhone |
| **C. Capacitor (web UI in a native shell)** | iOS, Android, web first-party; macOS not listed [V: 47] | Own or community HealthKit plugin; native scanner via plugin | One UI. Mac relies on web, or the iOS app on an Apple silicon Mac. Plugin upkeep; some Swift anyway |
| **D. Flutter** | iOS, macOS, web all supported [V: 48] | `health` package covers iOS and Android only [V: 49]; Mac and web need other code paths | One codebase, new language (Dart). Non-native look on Apple. Web offline is DIY, as Flutter no longer generates a caching service worker [V: 50] |
| **E. React Native** | Core is iOS and Android. macOS comes from Microsoft, web from the community [V: 51]. Expo documents Android, iOS and web [V: 52] | Community HealthKit libraries (not assessed) | Three separately maintained platforms that must stay compatible. The highest upgrade risk here |
| **F. Kotlin Multiplatform** | iOS and desktop JVM stable. Compose web is Beta; Kotlin/JS is stable [V: 53] | HealthKit through iOS interop | Gradle and JVM toolchain. Mac UI runs on the JVM unless shared logic sits under SwiftUI. Better as shared logic than as shared UI |

**Ways to reduce the cost:**

- **Cheap Mac version:** an iPad app can run unmodified on an Apple silicon Mac, and Mac Catalyst allows customising it. Don't assume a front or rear camera is present there. This fits A or C. [V: 5]
- **Sharing Swift with the web:** Swift has WebAssembly support from 6.2. The getting-started page says nothing about SwiftUI, and Apple doesn't list web for it, so this could share logic only. Unproven for this app. [V: 54, 46; I]
- **CloudKit JS for the web:** it reaches the same containers from a web page, but doesn't solve the health-data rule. [V: 55, 43]

**Questions that decide between these** (for the "Platform and tech stack" ticket):

1. How much will web and Mac really be used, versus iPhone? Fast logging may matter most on the phone.
2. Must Mac and web show Health data, or only the numbers derived from it, such as adjusted targets? The relay could then be much smaller. See "How targets are calculated and how exercise adjusts them".
3. Is a backend acceptable? Only pure-device setups avoid it, and those drop the web.

## Distribution for personal use

TestFlight builds last up to 90 days, so a personal install needs a fresh upload at least that often. Apple describes App Review for external testers; its overview also says a group's first build goes to review, so check how this applies to internal-only testing when setting up. [V: 56, 57]

## Not verified

- No iPhone, iPad, macOS 27 or browser device tests were run. The only direct tests were the two Mac calls marked [T].
- iPad behaviour rests on Apple's docs. I assume it needs Health data syncing through iCloud to see iPhone data. Test on a real iPad. [I]
- Whether `AVCaptureMetadataOutput`, listed for macOS 13+, actually decodes EAN-13 on a Mac wasn't checked. The Vision route doesn't depend on it. [58]
- HealthKit background rate limits for `bodyMass` and workouts aren't stated on the page I read. [13]
- Community plugins weren't assessed for quality or upkeep.

## Method

Apple's documentation pages render in the browser, so plain fetches returned only titles. I read Apple's JSON documentation source directly instead. I also read MDN's browser-compat data, the App Review guidelines and the licence from their raw source.

An automated page summary quoted a "3.3.7(D)" iCloud clause naming HealthKit. That sentence **isn't in the licence text**: section 3.3.7(D) only points to the iCloud terms. I discarded it and quoted section 3.3.3(D) from the source. Don't reuse the "3.3.7(D)" wording if it turns up elsewhere.

Other framework facts came through page summaries of the official docs, so re-check exact wording before quoting them: 27, 28, 34–36, 39, 47–53, 54, 56 and 57.

## Sources

All checked 2026-11-02.

1. [isHealthDataAvailable()](https://developer.apple.com/documentation/healthkit/hkhealthstore/ishealthdataavailable())
2. [About the HealthKit framework](https://developer.apple.com/documentation/healthkit/about-the-healthkit-framework)
3. [HealthKit updates](https://developer.apple.com/documentation/updates/healthkit)
4. [Apple: Health](https://www.apple.com/health/)
5. [Running your iOS apps in macOS](https://developer.apple.com/documentation/apple-silicon/running-your-ios-apps-in-macos)
6. [Apple Developer Forums: HealthKit for iPadOS?](https://developer.apple.com/forums/thread/650299)
7. [bodyMass](https://developer.apple.com/documentation/healthkit/hkquantitytypeidentifier/bodymass)
8. [HKWorkout](https://developer.apple.com/documentation/healthkit/hkworkout)
9. [Protecting user privacy](https://developer.apple.com/documentation/healthkit/protecting-user-privacy)
10. [Authorizing access to health data](https://developer.apple.com/documentation/healthkit/authorizing-access-to-health-data)
11. [getEarliestAuthorizedSampleDate(for:completion:)](https://developer.apple.com/documentation/healthkit/hkhealthstore/getearliestauthorizedsampledate(for:completion:))
12. [errorDatabaseInaccessible](https://developer.apple.com/documentation/healthkit/hkerror/code/errordatabaseinaccessible)
13. [enableBackgroundDelivery(for:frequency:withCompletion:)](https://developer.apple.com/documentation/healthkit/hkhealthstore/enablebackgrounddelivery(for:frequency:withcompletion:))
14. [HKObserverQuery](https://developer.apple.com/documentation/healthkit/hkobserverquery)
15. [Setting up HealthKit](https://developer.apple.com/documentation/healthkit/setting-up-healthkit)
16. [Apple Developer Program License Agreement, 3.3.3(H)](https://developer.apple.com/support/terms/apple-developer-program-license-agreement/)
17. [DataScannerViewController](https://developer.apple.com/documentation/visionkit/datascannerviewcontroller)
18. [DataScannerViewController.isSupported](https://developer.apple.com/documentation/visionkit/datascannerviewcontroller/issupported)
19. [VNBarcodeSymbology](https://developer.apple.com/documentation/vision/vnbarcodesymbology)
20. [DetectBarcodesRequest](https://developer.apple.com/documentation/vision/detectbarcodesrequest)
21. [VNDetectBarcodesRequest](https://developer.apple.com/documentation/vision/vndetectbarcodesrequest)
22. [RecognizeTextRequest](https://developer.apple.com/documentation/vision/recognizetextrequest)
23. [Supporting Continuity Camera in your macOS app](https://developer.apple.com/documentation/avfoundation/supporting-continuity-camera-in-your-macos-app)
24. [MDN browser-compat data: MediaDevices](https://github.com/mdn/browser-compat-data/blob/main/api/MediaDevices.json)
25. [MDN browser-compat data: BarcodeDetector](https://github.com/mdn/browser-compat-data/blob/main/api/BarcodeDetector.json); [WebKit bug: Shape Detection API on iOS](https://bugs.webkit.org/show_bug.cgi?id=281848)
26. [WebKit features for Safari 27.0](https://webkit.org/blog/18325/webkit-features-for-safari-27-0/); [Safari 27.2](https://webkit.org/blog/18515/webkit-features-for-safari-27-2/)
27. [barcode-detector](https://github.com/Sec-ant/barcode-detector)
28. [WebKit bug: getUserMedia in standalone mode](https://bugs.webkit.org/show_bug.cgi?id=185448); [follow-up bug](https://bugs.webkit.org/show_bug.cgi?id=215884)
29. [SwiftData](https://developer.apple.com/documentation/swiftdata)
30. [CloudKit](https://developer.apple.com/documentation/cloudkit)
31. [Syncing model data across a person's devices](https://developer.apple.com/documentation/swiftdata/syncing-model-data-across-a-persons-devices)
32. [MDN browser-compat data: ServiceWorker](https://github.com/mdn/browser-compat-data/blob/main/api/ServiceWorker.json)
33. [MDN browser-compat data: StorageManager](https://github.com/mdn/browser-compat-data/blob/main/api/StorageManager.json)
34. [WebKit: Tracking Prevention](https://webkit.org/tracking-prevention/)
35. [WebKit: Updates to Storage Policy](https://webkit.org/blog/14403/updates-to-storage-policy/)
36. [WebKit features in Safari 17.0](https://webkit.org/blog/14445/webkit-features-in-safari-17-0/)
37. [MDN browser-compat data: SyncManager](https://github.com/mdn/browser-compat-data/blob/main/api/SyncManager.json)
38. [Apple: Update on apps distributed in the EU](https://developer.apple.com/support/dma-and-apps-in-the-eu/)
39. [WebKit features in Safari 26.0](https://webkit.org/blog/17333/webkit-features-in-safari-26-0/)
40. [Multipeer Connectivity](https://developer.apple.com/documentation/multipeerconnectivity)
41. [Network](https://developer.apple.com/documentation/network)
42. [Share your data in Health on iPhone](https://support.apple.com/guide/iphone/share-your-health-data-iph5ede58c3d/ios); [on iPad](https://support.apple.com/guide/ipad/share-your-health-data-ipade9e6b160/ipados)
43. [App Review Guidelines, 5.1.3](https://developer.apple.com/app-store/review/guidelines/)
44. [Apple Platform Security: Protecting access to user's health data](https://support.apple.com/guide/security/protecting-access-to-users-health-data-sec88be9900f/web)
45. [Apple Developer Program License Agreement, 3.3.3(D)](https://developer.apple.com/support/terms/apple-developer-program-license-agreement/)
46. [SwiftUI](https://developer.apple.com/documentation/swiftui)
47. [Capacitor docs](https://capacitorjs.com/docs/)
48. [Flutter: Supported deployment platforms](https://docs.flutter.dev/reference/supported-platforms)
49. [health package on pub.dev](https://pub.dev/packages/health)
50. [Flutter web FAQ](https://docs.flutter.dev/platform-integration/web/faq)
51. [React Native: Out-of-tree platforms](https://reactnative.dev/docs/out-of-tree-platforms)
52. [Expo docs](https://docs.expo.dev/)
53. [Kotlin Multiplatform: Supported platforms](https://kotlinlang.org/docs/multiplatform/supported-platforms.html)
54. [Swift: Getting started with WebAssembly](https://www.swift.org/documentation/articles/wasm-getting-started.html)
55. [CloudKit JS](https://developer.apple.com/documentation/cloudkitjs)
56. [TestFlight](https://developer.apple.com/testflight/)
57. [App Store Connect: TestFlight overview](https://developer.apple.com/help/app-store-connect/test-a-beta-version/testflight-overview/)
58. [AVCaptureMetadataOutput](https://developer.apple.com/documentation/avfoundation/avcapturemetadataoutput)
