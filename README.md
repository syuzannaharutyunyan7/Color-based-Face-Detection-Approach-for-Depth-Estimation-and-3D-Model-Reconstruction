# Color-based Face Detection Approach for Depth Estimation and 3D Model Reconstruction

## Overview
Face detection has been a subject of various studies in computer science for many years, due to its wide range of practical applications. While deep learning-based methods can achieve high accuracy, they often require large datasets and substantial computational resources to generate results.  

Building further on insights from previous studies, the purpose of this research is to create and implement an **efficient, color-based approach for depth estimation as well as reconstruction of 3D facial models**. The methods used in this study are designed to identify face-relevant color ranges by analyzing **geometrical, statistical, and histogram-driven properties of skin-color distributions**.  

The system is implemented using **ImageJ’s image-processing tools combined with custom Java plugins and automated macros**. By relying on color-space analysis rather than computationally heavy deep learning models, the approach enables **lightweight, resource-conscious facial analysis** suitable for environments with limited computing capacity.

---

## Problem Statement
The extraction of facial regions from images is a critical foundation for 3D face reconstruction. Existing RGB-based methods for skin detection are sensitive to illumination and can mix background with individual skin tones. In addition, current methods can misclassify between non-facial and facial regions. Differences in color, brightness, and saturation across images make it harder to distinguish between skin and non-skin pixels, lowering the accuracy of depth estimation.  

This research provides an **automated approach that isolates facial regions under different conditions** by performing filtering using the **HSV color space**, while the RGB color space is used to verify the steps applied in HSV.

---

## Methodology

### Face-specific Colors in RGB
The purpose of this section is to determine a specific range for Hue that is optimal for skin region detection, based on *FLID Color Based Iterative Face Detection by Khachatryan (2010)*.  

- Collect pixels from facial regions and group them based on their Green (G) value (256 groups).  
- Compute **M = (R + B)/2** for each pixel.  
- Apply **quadratic regression curves** to identify facial-color intervals.  
- Pixels are assigned colors based on which interval they satisfy:  
  - Between curve 0 and 1 → Green  
  - Between curve 1 and 2 → Yellow  
  - Between curve 2 and 3 → Red  
  - Between curve 3 and 4 → Magenta  

**Limitation**: RGB-only detection is sensitive to illumination, background colors, and poorly separates actual skin tones from non-skin pixels.

---

### Face-specific Colors in HSV
To address RGB limitations, the approach uses the **HSV color space**:  

- The desirable **Hue range for skin color** is [3, 24].  
- Each image is separated into **RGB (Red, Green, Blue)** and **HSV (Hue, Saturation, Value/Brightness)** channels.  
- The combination of **Saturation and Value/Brightness (SV)** provides a measure that better distinguishes skin tones across images and reduces the impact of illumination variations.

---

### Automated SV Feature Generation
- The ImageJ plugin reads **Saturation and Value channels** and multiplies them for each pixel.  
- The resulting **SV composite feature** is normalized and highlights facial regions in grayscale.

---

### Histogram-Based Analysis of SV
- Compute **histogram of SV values** with 256 bins.  
- Apply **conditional logarithmic transformation** and **five-point moving average** to smooth the histogram.  
- Apply **linear regression to remove global trends**.  
- Identify the **dominant peak in bins 40–80** to determine lower and upper SV bounds for skin pixels.

---

### Face Mask Generation
- Pixels outside SV bounds → set to white (non-face).  
- Pixels within SV bounds → potential facial regions.  
- Produces **binary masks leaving only facial regions**.  

---

### Extraction of Facial Regions
- Combine **Hue and SV filtering** to retain pixels within thresholds.  
- Non-facial regions are set to white.  
- Morphological filtering (**erosion, median filtering, dilation**) removes artifacts and smooths edges.  
- Particle analysis identifies the **largest connected region** (assumed to be the face).  
- Apply the mask to the original image → **face-only images**.

---

### Hue Remapping
- Compute **Hue histogram** of face-only images.  
- Calculate **CDF (cumulative distribution function)**.  
- Map Hue values to an **ideal reference CDF** to standardize skin-tone colors across images.  

---

### Depth Estimation and 3D Reconstruction
- Use **vertical projection patterns** from binary layers.  
- Identify **maxima and minima corresponding to key facial structures**.  
- Compute ellipsoids representing the **3D facial shape**.

---

## Dataset
- **FEI Face Database**: 200 individuals × 14 images each → 2800 images.  
- Uniform white background, frontal and profile rotations.  
- Diverse appearances, hairstyles, accessories, and expressions.  

---

## Results
- 86.5% of images successfully processed.  
- RGB-only methods failed under varying illumination and background conditions.  
- Hue + SV filtering produced **clean and precise facial masks**.  
- Hue remapping improved **color consistency across the dataset**.  

---

## Tools and Implementation
- **ImageJ**: Main image-processing platform.  
- **Custom Java Plugins**:  
  - `AllLayersBinary` → RGB-based pixel evaluation  
  - `FaceExtractorFromMasks` → Final facial region extraction  
  - `HueRemapPlugin` → Hue normalization  
  - `ProjectionsVer` → Vertical projection curves for depth estimation  
- **Macros**: Automate extraction of RGB and HSV channels, SV computation, and morphological filtering.  

---

## Usage
1. Place dataset images in a folder (e.g., `FEI Dataset`).  
2. Run macros to extract **RGB and HSV channels**.  
3. Generate **SV composite images** using the plugin.  
4. Apply **SV histogram analysis** to identify face masks.  
5. Combine **Hue and SV filtering** to extract facial regions.  
6. Apply **morphological filtering and largest region selection**.  
7. Run **Hue remapping** for consistent color across images.  
8. Use **vertical projection analysis** for depth estimation and 3D reconstruction.

---

## Conclusion
By relying on **color-space analysis, histogram normalization, and morphological filtering**, this approach efficiently extracts facial regions and produces clean datasets suitable for **depth estimation and 3D reconstruction**, without requiring heavy deep learning models.
