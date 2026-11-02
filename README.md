# OMNI VISION AI

An offline-first, cross-platform computer-vision system, developed in verified phases. This repository currently contains the **Phase 1–2 deliverable**: the shared TypeScript contracts, model manifest and integrity manager, plus real YOLOX-Nano ONNX CPU inference plumbing and tests. It is deliberately not presented as a finished desktop/mobile/PWA product yet.

## Current implementation status

| Phase | Status | What is included |
|---|---|---|
| 1 — Shared architecture | Complete | TypeScript contracts, model manifest, model integrity/import API, workspace layout |
| 2 — Real object detection | Complete for the selected model | Licensed YOLOX-Nano ONNX weights, preprocessing, decode, class-aware NMS, ONNX Runtime CPU smoke test |
| 3 — Hand / OCR / tracking | Not started | No stub or simulated results are shipped |
| 4 — Web PWA | Not started | No PWA is claimed yet |
| 5 — Windows/Tauri | Not started | No installer is claimed yet |
| 6 — Android/Compose | Not started | No APK is claimed yet |
| 7–8 — device optimization / full platform test matrix | Not started | No device/browser testing is claimed |

The project follows the requested phase order. The next phase is hand/OCR/tracking; platform apps are intentionally not generated as empty shells or fake-feature demos.

## What works now

- **Real inference only.** `YoloxNanoDetector` runs the included ONNX model through a runtime adapter. A missing/corrupt model or a failed inference raises an explicit error; it never substitutes hard-coded detections.
- **Offline artifact included.** `models/detection/yolox_nano.onnx` is a 3,659,407-byte model under Apache-2.0. Its SHA-256 is pinned in `models/manifest.json` and checked by the test suite.
- **Cross-platform contract.** Results use one pixel-coordinate bounding-box schema, with class ID, label, confidence, model ID, image dimensions and processing time.
- **Input/output path verified.** The actual ONNX file has been executed with ONNX Runtime Node CPU in the model smoke test. That verifies graph loading, declared tensor names/shapes and finite output, not detector accuracy on a test corpus.

This default detector is **closed-set COCO-80**, not open vocabulary. It can identify only the labels in its class list; it does not promise arbitrary object/attribute recognition. In particular, it has no distinct `monitor` class (COCO has `tv`) and does not know every household object. Confidence is a model score, not certainty.

## Requirements

- Node.js 20+
- npm 10+

## Setup and checks

```bash
npm install
npm run verify:model
npm run typecheck
npm test
```

`npm test` includes an ONNX Runtime CPU smoke test. Runtime packages are development/test dependencies at this stage; later web, Tauri and Android adapters will use their platform-appropriate ONNX Runtime bindings.

## Shared detector API

The platform adapter supplies a session runner and tensor factory; the core owns preprocessing, YOLOX decoding, score filtering, class-aware NMS and mapping to original-image pixel coordinates. The model's expected input is RGB NCHW float32 in **0–255**, letterboxed top-left with pad value 114, at 416×416. Output is YOLOX's `[1, 3549, 85]` head tensor.

```ts
import { YoloxNanoDetector } from '@omni/vision-core';

const detector = new YoloxNanoDetector(runtimeAdapter);
const result = await detector.detect({ width, height, data: rgbaPixels });
// result.detections contains only detections decoded from the model output.
```

See [`docs/architecture.md`](docs/architecture.md), [`docs/model-catalog.md`](docs/model-catalog.md), [`docs/offline-setup.md`](docs/offline-setup.md), [`docs/development.md`](docs/development.md), and [`docs/test-report.md`](docs/test-report.md).

## Model and license

Model source and version are pinned in [`models/manifest.json`](models/manifest.json). YOLOX weights are Apache License 2.0; the complete upstream license and attribution are in [`models/licenses/YOLOX-Apache-2.0.txt`](models/licenses/YOLOX-Apache-2.0.txt). The repository's own code is MIT-licensed; see [`LICENSE`](LICENSE). These statements are model/code attribution, not legal advice.

## Privacy

The shared core has no network client, analytics, account system, or upload path. A future user interface must preserve local-only processing and explain permissions before camera use. The model is included in this source tree; later apps will make model installation explicit and support manual import.

## Limitations / honest test report

No Windows 11 device, Android 15 / POCO C71-class phone, Chrome/Edge PWA offline session, camera, or installer has been tested in this environment. No platform deliverable exists yet. The included tests validate contracts and algorithmic edge cases, verify the model checksum, and run the ONNX graph on CPU with a zero-valued tensor. Decoder unit tests use controlled tensors to validate math; these are test fixtures only, never displayed as inference results. A positive real-photo accuracy benchmark is not claimed.


## Web MVP (Phase 3)

The browser application is in `apps/web`. Install/update workspace dependencies from the repository root with `npm install`, then run `npm run dev:web`. See `apps/web/README.md` for browser, camera, and offline notes.
