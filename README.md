# Tree Segmentation from Drone RGB — Resolution Comparison (2 cm vs 10 cm vs 30 cm)
<img width="1984" height="1115" alt="image" src="https://github.com/user-attachments/assets/c9707833-68ff-4c23-8890-aacb7083f538" />

This repo shows how changing image resolution affects tree-crown detection in **ArcGIS Pro** using a pretrained **TreeSegmentation (`.dlpk`)** model.  
We compare **2 cm**, **10 cm**, and **30 cm** inputs and show that **30 cm** improves detection of **large crowns** without manual retraining.

---

## Introduction

- **Goal:** detect individual crowns (polygons) from a high-res drone RGB orthomosaic.  
- **Observation:** pretrained models are typically trained on coarser imagery (≈10–30 cm GSD).  
- **Outcome:** at **2 cm**, crowns are over-segmented and large ones are missed; resampling to **10 cm** stabilizes small/medium trees; **30 cm** captures **large** trees best.

**Software / Model**
- ArcGIS Pro (Image Analyst license)  
- Deep Learning Libraries for ArcGIS Pro (PyTorch + fastai v1)  
- Pretrained model: `TreeSegmentation.dlpk` (ModelType `ObjectDetection`, RGB)

---

## Methods

**Pipeline**
1. Start from 2 cm RGB.  
2. **Resample** to **10 cm** and **30 cm** (bilinear).  
3. Run **Detect Objects Using Deep Learning** on each resolution with tuned parameters.  
4. Compare counts and polygon areas (esp. “large crown” area share).  
5. Visually compare by overlaying each detection layer.

**Click sequence**
1. *Data Management → Raster → Resample* (to 10 cm and 30 cm).  
2. *Image Analyst → Detect Objects Using Deep Learning* (run for each resolution).  
3. (Optional) *Add Geometry Attributes* → compute areas for comparison.  

---

## Resample

Use **Resample** with **Bilinear**:

- `RGB_10cm.tif` → **Output cell size:** `0.10` (meters)  
- `RGB_30cm.tif` → **Output cell size:** `0.30` (meters)

---

## Parameters

### Common (all runs)
- **Model:** `TreeSegmentation.dlpk`  
- **Padding:** `64`  
- **Tile Size (Environments → Raster Analysis):** `1024`  
- **Batch Size:** `2–4` (GPU) / `1` (CPU)  
- **Exclude Padding Detections:** `True`  
- **Prompt Type:** `box`

### Per-resolution settings

#### 2 cm (baseline, tends to over-segment)
- **Input:** original 2 cm RGB  
- **Confidence:** `0.55`  
- **NMS Overlap:** `0.35`  
- **Test Time Augmentation:** `False`  
> Purpose: reduce fragments; serves as a baseline showing why 2 cm is problematic.

#### 10 cm (small–medium crowns; stable)
- **Input:** `RGB_10cm.tif`  
- **Confidence:** `0.50`  
- **NMS Overlap:** `0.35`  
- **Test Time Augmentation:** `False`  
> Expected: single polygons for most small/medium trees.

#### 30 cm (large crowns)
- **Input:** `RGB_30cm.tif`  
- **Confidence:** `0.20–0.25`  
- **NMS Overlap:** `0.60`  
- **Test Time Augmentation:** `True`  
> Expected: better recall on big trees; small ones may be fewer (that’s OK for this comparison).

---

## Example results

Add images like:

```md
### Input RGB (2 cm)
<img width="1035" height="675" alt="image" src="https://github.com/user-attachments/assets/846627fa-2afa-4d44-b9c3-56aa98db334e" />


### Detections @ 2 cm
![2cm](images/det_2cm.png)

### Detections @ 10 cm
<img width="1031" height="676" alt="image" src="https://github.com/user-attachments/assets/a503b326-ec39-4642-8f49-39ee0cd2c08e" />

### Detections @ 30 cm (better large-tree recall)
<img width="1028" height="675" alt="image" src="https://github.com/user-attachments/assets/b20d1d29-59e5-467f-9eb3-8f7e8e1257b6" />

