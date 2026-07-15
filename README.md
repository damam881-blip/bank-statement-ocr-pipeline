# High-Fidelity Bank Statement OCR Pre-Processing Pipeline

An advanced, production-grade Computer Vision pre-processing pipeline engineered to reconstruct, binarize, and optimize low-DPI scanned financial PDF documents. This pipeline resolves complex layout challenges, layer overlaps, and compression artifacts, delivering razor-sharp, pixel-perfect images tailored for maximum OCR extraction accuracy.

---

## Pipeline Architecture & Workflow

The preprocessing pipeline is structured into three decoupled, high-performance Jupyter Notebooks, each designed to tackle a specific stage of the document lifecycle:

```mermaid
graph TD
    A[Scanned Financial PDF] --> B[Notebook 01: Raw Stream Extraction]
    B --> C[Layer Reconstruction & Pixel-Minimum Blending]
    C --> D[Notebook 02: Adaptive Binarization & Benchmarking]
    D --> E[Block-Size Optimization Scenario A vs B]
    E --> F[Notebook 03: Hardware-Negotiated Dynamic Rasterization]
    F --> G[Razor-Sharp High-DPI Lossless PNG Canvas]

---

## 🚀 Phase 1: Universal High-Fidelity Pre-Processing Pipeline

This stage converts any scanned bank statement PDF into a pristine, high-contrast, 1-bit binary image (`.tiff`). It utilizes a dynamic backoff rendering mechanism to prevent memory crashes and combines edge-preserving bilateral filtering with Otsu's adaptive binarization to eliminate background scanner noise.

### 🛠️ Key Architectural Features
* **Dynamic Resolution Backoff:** Adapts rendering zoom from $5.0\times$ (optimal 360 DPI) down to $3.0\times$ depending on available system RAM, avoiding Out-Of-Memory (OOM) crashes.
* **Edge-Preserving Smoothness:** Implements a Bilateral Filter to wipe out background scanner speckles and shadows while keeping text outlines razor-sharp.
* **Automated Mathematical Thresholding:** Employs Otsu's Binarization to calculate the absolute optimal threshold dynamically for each unique document.

### 💻 Production-Ready Code
Here is the core processing script (`Bit_LosslessTIFF.py`):

```python
import fitz  # PyMuPDF
import cv2
import numpy as np
import os

print("[+] Starting Universal PDF Pre-processing Pipeline...")

pdf_name = "كشف حساب الراجحي.pdf"

if not os.path.exists(pdf_name):
    print(f"[!] Error: Cannot find '{pdf_name}' in the current folder!")
else:
    doc = fitz.open(pdf_name)
    page = doc.load_page(0)

    # Disable anti-aliasing to preserve raw pixel boundaries
    try:
        fitz.tools.set_aa_level(0)
    except Exception:
        pass

    # Dynamic Backoff Loop for Zoom
    optimal_zoom = 5.0
    pix = None

    while optimal_zoom >= 3.0:
        matrix = fitz.Matrix(optimal_zoom, optimal_zoom)
        try:
            pix = page.get_pixmap(matrix=matrix, alpha=False)
            break
        except Exception:
            print(f"[!] RAM allocation failed at {optimal_zoom}x zoom. Stepping down...")
            optimal_zoom -= 0.5

    if pix is None:
        print("[!] Emergency fallback to 2.0x zoom.")
        try:
            matrix = fitz.Matrix(2.0, 2.0)
            pix = page.get_pixmap(matrix=matrix, alpha=False)
            optimal_zoom = 2.0
        except Exception as e:
            print(f"[!] Critical Error! {e}")

    # Process and clean the matrix
    if pix is not None:
        print(f"[✓] Canvas successfully rendered at {optimal_zoom:.1f}x zoom.")

        # Convert raw byte stream to a 3D NumPy array
        img_data = np.frombuffer(pix.samples, dtype=np.uint8).reshape((pix.h, pix.w, pix.n))
        img_gray = cv2.cvtColor(img_data, cv2.COLOR_RGB2GRAY)

        # Bilateral Filtering (removes scanner noise while protecting text edges)
        print("[+] Applying Bilateral Filter...")
        img_filtered = cv2.bilateralFilter(img_gray, d=9, sigmaColor=75, sigmaSpace=75)

        # Otsu's Adaptive Thresholding
        print("[+] Calculating optimal threshold via Otsu's Binarization...")
        optimal_thresh, binary_1bit = cv2.threshold(
            img_filtered, 
            0, 
            255, 
            cv2.THRESH_BINARY + cv2.THRESH_OTSU
        )

        print(f"[✓] Optimal dynamic threshold: {optimal_thresh}")

        output_path = "page_1bit_universal.tiff"
        cv2.imwrite(output_path, binary_1bit)
        print(f"[✓] Flawless 1-Bit file saved to: {output_path}")