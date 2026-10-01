# QD Subpixel Grid Mapper

Automated batch pipeline for subpixel localization and coordinate mapping of quantum dots (QDs) relative to lithographic reference marker grids.

Developed as part of an **engineering thesis project**. The objective is to extract high-precision emitter coordinates within marker coordinate frames to guide deterministic nanofabrication, such as aligning photonic waveguides directly over selected quantum dots.

---

## Features

- **Subpixel Accuracy:** Combines non-linear 2D Gaussian surface fitting (Levenberg-Marquardt) for QD localizations with spatial moments for alignment markers.
- **Correlative Marker Tracking:** Uses normalized cross-correlation (Template Matching) with CLAHE contrast normalization to resolve low-contrast markers in cryogenic PL images.
- **Saturation Suppression (NMS):** Spatial clustering and non-maximum suppression eliminate duplicate detections over saturated sensor pixels.
- **Automated Batch Processing:** Analyzes all `.png` files in the directory and outputs per-image CSV tables and text logs.
- **Deterministic Quality Metrics:** Computes template match confidence (`Marker Confidence`) and signal-to-noise ratio (`Mean SNR`) for every analyzed frame.

---

## Configuration

All core operational variables are exposed at the top of the script under `USER CONFIGURATION`:

| Parameter | Default | Description |
| :--- | :--- | :--- |
| `GRID_SIZE_UM` | `80.0` | Physical pitch of the square alignment grid in micrometers [$\mu\text{m}$]. |
| `THRESHOLD_SIGMA` | `3.0` | Detection sensitivity (noise sigma multiplier). Lower values detect dimmer emitters; higher values reduce false positives. |
| `MARGIN_UM` | `15.0` | Boundary margin around the grid to include surrounding emitters [$\mu\text{m}$]. |
| `MIN_DIST_NMS_PX` | `5.0` | Physical exclusion radius to merge sensor saturation plateaus [pixels]. |
| `APPROX_PIXEL_SCALE_UM_PER_PX` | `0.246` | Nominal optical scale used to constrain geometric marker diagonal matching. |

> **Note on Performance:** Processing time scales directly with the number of candidate peaks evaluated by the 2D Gaussian optimizer; raising `THRESHOLD_SIGMA` (e.g., to $\ge 3.5$) drastically speeds up execution on large batch datasets by filtering out background noise.

---

## Output Format

For each processed image, the pipeline exports a summary log (`_report.txt`) and a structured coordinate table (`_qds.csv`).

All potential emitter coordinates are **sorted descending by intensity**, prioritizing candidate selection:

| ID | Px_X | Px_Y | Coord_X_um | Coord_Y_um | Inside_Grid | Intensity | SNR |
| :- | :--- | :--- | :--------- | :--------- | :---------- | :-------- | :-- |
| 1  | 573.272 | 129.395 | 79.582 | 59.621 | YES | 255 | 18.4 |
| 2  | 320.608 | 268.872 | 16.361 | 25.860 | YES | 255 | 17.9 |

- `Inside_Grid`: Flags whether the emitter is bounded by $[0, \text{GRID\_SIZE\_UM}]$ (`YES`) or resides within the outer margin area (`NO`).

---

## Dependencies & Execution

Install required dependencies:
```bash
pip install opencv-python numpy scipy
