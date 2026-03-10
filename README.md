Hi, nice to meet you! This section shows how to use the AI-assisted segmentation tool to train brightfield cell “thickness” quantification as a practical standard for cell quality assessment.

Workflow: Start with an example input/output, then iteratively tune the segmentation parameters (based on Otsu thresholding) and manually mark/correct results. With each correction, the auto output becomes more accurate and efficient.

<p align="center">
  <img src="https://github.com/user-attachments/assets/83c04034-f2d2-4681-8e7a-bbd75637a9d6" height="320">
  <img src="https://github.com/user-attachments/assets/d4c05345-bb26-444d-baaa-3bb0e6859342" height="320">
</p>

Software interface (Streamlit):

<img width="1916" height="948" alt="image" src="https://github.com/user-attachments/assets/c704d699-3f01-4e02-bd7a-ea023d23e3b7" /> <img width="1537" height="848" alt="image" src="https://github.com/user-attachments/assets/038f8d44-7608-4f05-903e-e83bbe51c26d" />


## 🔍 Core Algorithm (Mathematical Overview)

BCQA processes a bright-field microscopy image and outputs:

1. Binary cell mask (segmentation)
2. Per-cell morphology + intensity metrics
3. Greyscale "thickness heatmap" overlay

---

### 1️⃣ Global Image Features (for optional AI parameter suggestion)

From grayscale image I:

- Mean intensity  
  μ = (1/N) * Σ I(p)

- Standard deviation  
  σ = sqrt( (1/N) * Σ (I(p) − μ)^2 )

These features optionally feed:

- MLPClassifier → selects threshold type (Otsu or Adaptive)
- MLPRegressor → predicts morphology kernel sizes

---

### 2️⃣ Segmentation

Convert to grayscale:

I = Gray(RGB)

**Otsu threshold (binary inverse):**

BW = 1 if I < T_otsu else 0

**Adaptive threshold:**

BW(x,y) = 1 if I(x,y) < (local_mean − C)

Morphological cleanup:

- Opening → remove small noise
- Closing → fill holes and connect fragments

Kernel sizes: open_k, close_k

---

### 3️⃣ Per-Cell Measurements

Keep objects with:

Area ≥ A_min

For each cell:

- Area (A)
- Perimeter (P)
- Circularity = 4πA / P²
- Aspect Ratio = width / height
- Mean Intensity (thickness proxy):

I_mean = (1 / |M|) * Σ I(p) for p in cell mask

---

### 4️⃣ Thickness Heatmap Mapping

Let:

I_min = minimum cell mean intensity  
I_max = maximum cell mean intensity  

Grey value mapping (linear interpolation):

Low intensity → light grey (200)  
High intensity → dark grey (50)

g = linear_map(I_mean → [200, 50])

Cells are filled with (g, g, g)  
Contours are drawn in green.

---

### 📄 CSV Output Columns

- Area
- Perimeter
- Circularity
- Aspect_Ratio
- Mean_Intensity
- Grey_Value
- X, Y, Width, Height
<img width="1521" height="590" alt="image" src="https://github.com/user-attachments/assets/38a858dc-f542-49cb-8b2e-2f8349dd206a" />
<img width="289" height="542" alt="image" src="https://github.com/user-attachments/assets/e0848142-51e8-43ba-a220-ca85844f524e" />

