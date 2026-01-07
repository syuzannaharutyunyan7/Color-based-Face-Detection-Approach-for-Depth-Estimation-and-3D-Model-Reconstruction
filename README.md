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
- Pixels are extracted from facial regions and grouped based on their Green (G) values (256 groups).  
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
- Convert RGB images to **HSV (Hue, Saturation, Value/Brightness)**.  
- Desired Hue range for skin color: [3, 24].  
- Multiply Saturation and Value channels (SV composite) to better distinguish skin from background.  

---

### Automated SV Feature Generation
- ImageJ plugin reads **Saturation and Value channels**.  
- Multiplies them for each pixel → **SV composite feature**.  
- Normalized SV highlights facial regions in grayscale.

---

### Histogram-Based Analysis of SV
- Compute histogram of SV values with 256 bins.  
- Apply **conditional logarithmic transformation** and **five-point moving average** to smooth histogram.  
- Linear regression removes global trends.  
- Identify **dominant peak in bins 40–80** to determine lower and upper SV bounds for facial pixels.

---

### Face Mask Generation
- Pixels outside SV bounds → set to white (non-face).  
- Pixels within SV bounds → potential facial regions.  
- Produces **binary face masks** leaving only facial regions.  

---

### Extraction of Facial Regions
- Combine **Hue and SV filtering** to retain pixels within thresholds.  
- Non-facial regions are set to white.  
- Apply morphological filtering (**erosion, median filtering, dilation**) to remove artifacts and smooth edges.  
- Particle analysis identifies **largest connected region** (assumed to be the face).  
- Apply mask to original image → **face-only images**.

---

### Hue Remapping
- Compute **Hue histogram** of face-only images.  
- Calculate **CDF (cumulative distribution function)**.  
- Map Hue values to **reference CDF** for standardized skin-tone across dataset.  

---

### Depth Estimation and 3D Reconstruction
- Use **vertical projections** from binary layers.  
- Identify maxima and minima corresponding to key facial structures.  
- Compute ellipsoids representing **3D facial shape**.  

---

## Dataset
- **FEI Face Database**: 200 individuals × 14 images each → 2800 images.  
- Uniform white background, frontal and profile rotations.  
- Diverse appearances, hairstyles, accessories, and expressions.  

---

## Testing Results

### Face Masks
Working and non-working face masks are presented, along with the percentage of correctly detected faces and analysis of why some images are not extracted accurately.

#### The results of the *-11 facial dataset after filtering process
**Face Masks of *-11 dataset**

| Images               | Clothes remaining after face mask | Hair remaining after face mask | Facial pixel loss |
|---------------------|---------------------------------|-------------------------------|-----------------|
| Successful images    | 190                             | 181                           | 197             |
| Unsuccessful images  | 10                              | 19                            | 3               |

Some images show loss of facial pixels, others have hair colors similar to skin tone, and some still have clothing remnants. **86.5%** of images were processed successfully.

---

#### The results of the *-01 facial dataset after filtering process
**Face Masks of *-01 dataset**

| Images               | Clothes remaining after face mask | Hair remaining after face mask | Facial pixel loss |
|---------------------|---------------------------------|-------------------------------|-----------------|
| Successful images    | 191                             | 183                           | 193             |
| Unsuccessful images  | 9                               | 17                            | 7               |

Some images show loss of facial pixels, hair colors similar to skin tone, or clothing remnants. **86%** of images were processed successfully.

---

#### The results of the *-10 facial dataset after filtering process
**Face Masks of *-10 dataset**

| Images               | Clothes remaining after face mask | Hair remaining after face mask | Facial pixel loss |
|---------------------|---------------------------------|-------------------------------|-----------------|
| Successful images    | 192                             | 177                           | 195             |
| Unsuccessful images  | 8                               | 23                            | 5               |

Some images show loss of facial pixels, hair colors similar to skin tone, or clothing remnants. **84%** of images were processed successfully.

---

### Histogram Matching and Binary Layers
- Histogram matching standardizes pixel values across datasets.  
- Binary Layers plugin generates 4 binary layers based on RGB intervals for depth analysis.  
- **Vertical projection analysis** identifies maxima and minima for facial structures.  
- After histogram matching, facial features like nose, mouth, ears, and eyes become clearly visible.

---

### 3D Face Depth Reconstruction
- Depth map calculated using **binary layer projections**.  
- Parameters for ellipsoid construction:

| Parameter | Calculation / Description                     | Binary Layer |
|-----------|-----------------------------------------------|--------------|
| X1        | x coordinate of left max                      | Layer 0      |
| X2        | x coordinate of max                            | Layer 2      |
| a         | X2 − X1                                       | -            |
| D1        | x coordinate of middle max                     | Layer 0      |
| D2        | x coordinate of middle max                     | Layer 1      |
| D3        | x coordinate of min after left max             | Layer 2      |
| D4        | max x coordinate                               | Layer 2      |
| D5        | x coordinate of right max                      | Layer 2      |
| b         | max(D1,D2) − D3                               | -            |
| c         | D4 − D5                                       | -            |

- Construct ellipsoid with semiaxes `a` and `b`.  
- Generate 3D depth map for each face.

---

## Tools and Implementation
- **ImageJ**: Main image-processing platform.  
- **Custom Java Plugins**:  
  - `AllLayersBinary` → RGB-based pixel evaluation  
  - `FaceExtractorFromMasks` → Facial region extraction  
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

Testing on the FEI Face Database showed **~85% success** for clean extraction. Future work includes applying the algorithm to rotated images and automating ellipsoid generation for all faces.

