# exoplanet-transit-detection
Automated TESS light curve analysis and transit detection pipeline using Box Least Squares (BLS)
# 🌌 TESS Exoplanet Transit Detection Pipeline

An end-to-end observational astronomy pipeline in Python designed to detect, model, and characterize transiting exoplanets using data from NASA's **Transiting Exoplanet Survey Satellite (TESS)**.

---

## 📌 Project Overview
This pipeline uses `lightkurve` and `astropy` to process TESS light curves, eliminate stellar noise, and detect periodic planet transits using **Box Least Squares (BLS)** periodograms.

### Key Objectives:
1. **Target Validation:** Characterize known ultra-short period exoplanet **WASP-18 b** (period, depth, radius, orbital velocity, equilibrium temperature).
2. **Atmospheric Refinement:** Apply quadratic limb-darkening corrections ($u_1, u_2$) to improve radius accuracy.
3. **Automated Target Scanning:** Execute a systematic blind search across TESS Input Catalog (TIC) targets to flag candidate transit signals automatically.

---

## 📊 Key Results

| Target | Object Type | Detected Period | Derived Radius | Key Feature |
| :--- | :--- | :--- | :--- | :--- |
| **WASP-18 b** | Confirmed Hot Jupiter | ~0.9415 days | $1.08\text{ }R_J$ | Limb-darkened model correction |
| **TIC 261136679** | Detected Candidate | ~6.2661 days | ~$0.07\text{ }R_J$ | Signal extracted via phase binning |

---

## 🛠️ Tech Stack & Dependencies
* **Python 3.x**
* **Lightkurve:** NASA light curve retrieval & BLS periodograms
* **NumPy / Matplotlib:** Data binning, array operations, and phase-folded plots
