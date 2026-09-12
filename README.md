# Exoplanet Transit & Telescope Anomaly Detector
---
## Project Overview
This respository contains a fully automated Python program designed to search for and study planets outside our solar system. It uses real starlight data from NASA's Transiting Exoplanet Survey Satellite (TESS).
The main goal of this project is to turn raw, messy telescope data into actual planet discoveries. The code does this by automatically downloading star information, searching for repeating shadows caused by orbiting planets, calculating how big and hot those planets are, and double-checking the camera pixels to make sure the signals are not just space illusions.

---
## AI-Assisted Development Methodology
A defining feature of this project is the use of Artificial Intelligence as a digital coding tutor and pair programmer. Rather than using AI to blindly generate code, I leveraged it to accelerate my learning and optimize my technical work flow in the following ways:
1. **Translating Physics into Code**: I used AI to help me turn complicated physics and math into clean python code. It helped me make sure all my unit conversions were accurate.
2. **Learning NASA's Tools**: NASA's data tools can be tricky to use. AI helped me understand how to combine data from different months and how to read telescope warning labels.
3. **Debugging code**: When I was building the tool to check the camera pixels, the math kept breaking. AI helped me fix the errors and calculate the exact distance between pixels.
4. **Data visualization**: AI provided guidance on advanced matplotlib formatting, making much more easier to distinguish real and fake signals.
This human-AI collaboration allowed me to focus heavily on the underlying astrophysics and logic design, resulting in a significantly more sophisticated and robust pipeline than I could have built alone in the same timeframe.
## System Architecture
1. **Automated Data Retrieval and Cleaning:** The analyze_exoplanet_target() function connects directly with the Mikulski Archive for Space Telescopes (MAST). It queries both SPOC and TESS-SPOC authors to maximize data availability.

   **Multi-Sector Stitching:** For targets observed across multiple TESS sectors, the pipeline seamlessly downloads and and stitches the arrays to increase the signal-to-noise ratio over a longer observational baseline.

   **Quality Filtering & Detrending:** To ensure data integrity, the pipeline masks anomalous flux points caused by spacecraft pointing jitters or momentum dumps. It then flattens stellar variability (using Savitzky-Golay filtering) and removes severe statistical outliers (.remove_nans().flatten().remove_outliers()).
2. **Periodicity Analysis & Signal Detection Box Least Squares (BLS):** The pipeline conducts systematic blind searches across custom frequency grids using a BLS algorithm to detect periodic transit dips.

   **Signal Detection Efficiency (SDE):** Targets with power spectrum peaks exceeding a specific threshold (e.g., pg.max_power>10) are flagged as viable transit candidates.

   **Phase Folding:** The raw light curve is folded over the optimal detected period ($P$) and centered precisely on the mid-transit epoch ($T_0$). The data is then binned (e.g., $\sim30$ points per bin) to drastically reduce noise and reveal the true transit geometry.
   
3. **Physical Characterization (Astrophysics Module):** Using the extracted transit parameters and stellar priors (mass, radius, temperature), the pipeline calculates fundamental planetary characteristics:
**Semi-Major Axis ($a$): Derived using Kepler's Third Law:**

$$a = \left( \frac{G M_* P^2}{4 \pi^2} \right)^{1/3}$$

**Orbital Velocity ($v$): Assuming a circular orbit:**

$$v = \frac{2 \pi a}{P}$$

**Planetary Radius ($R_p$): Calculated from the fractional transit depth ($\delta$):**

$$R_p = R_* \sqrt{\delta}$$

**Limb-Darkening Corrections:** For specific targets (like F-type stars), the pipeline integrates quadratic limb-darkening coefficients (e.g., $u_1 = 0.3, u_2 = 0.2$) to correct the mid-transit depth, accounting for the stellar disk's brightness profile.

**Equilibrium Temperature ($T_{eq}$):** Evaluated assuming a standard planetary Bond albedo ($A_B = 0.1$):

$$T_{eq} = T_* \sqrt{\frac{R_*}{2a}} (1 - A_B)^{1/4}$$

4. **Pixel-Level Spatial Vetting (False Positive Rejection):** A light curve dip does not guarantee a planet; it can easily be a Background Eclipsing Binary (BEB) bleeding into the target's aperture. To counter this, the pipeline analyzes raw Target Pixel Files (TPFs):
**Difference Imaging:** Isolates missing flux by subtracting in-transit pixel frames from out-of-transit frames ($\Delta F = F_{\text{out}} - F_{\text{in}}$).

**Centroid Tracking:** Calculates the flux-weighted centroid of this difference image to find the exact origin of the transit signal.

**Offset Validation:** Computes the Euclidean distance from the target star's center to the transit source. If the calculated spatial offset is >0.5 pixels, the target is automatically rejected as an off-target false positive.

---

## Key Results & Discoveries

### Case 1: WASP-18 b (Extreme Hot Jupiter)
Findings: The pipeline successfully characterized this ultra-short period exoplanet. It orbits at a staggering distance of just $0.0201 \text{ AU}$ and travels at $232.14 \text{ km/s}$, completing a full year in under 23 hours. The estimated equilibrium temperature exceeds $2300 \text{ K}$.
    Accuracy: Using raw TESS data and basic BLS modeling, the initial radius estimate was $1.01 R_J$. By implementing the pipeline's limb-darkening correction factor, the refined radius updated to $1.08 R_J$—achieving an impressive <10% error margin (9.21%) relative to the published literature benchmark of $1.19 R_J$.

### Case 2: HAT-P-7 b
Findings: The automated function successfully queried Sector 14 data and extracted a clean transit signal. The pipeline accurately derived an orbital period of $2.2045 \text{ days}$ and calculated a physical radius of $0.73 R_J$.    

### Case 3: TIC 261136679 (Successful BEB Rejection)
The Search: During a systematic loop over unvetted TIC IDs, the BLS module flagged a high-power transit signal with a $12.53\text{-day}$ period and a $3.60\text{-hour}$ duration.
The Vetting: The signal geometrically looked like a planet. However, pushing it through the TPF vetting module generated a difference image that revealed the missing flux was occurring far outside the target aperture.
Result: The pipeline calculated a Centroid Offset of 27.412 pixels. The spatial transit was definitively off-target, and the pipeline accurately and automatically reclassified the signal as a REJECTED Background Eclipsing Binary.

| Target | Object Type | Detected Period | Derived Radius | Key Feature |
| :--- | :--- | :--- | :--- | :--- |
| **WASP-18 b** | Confirmed Hot Jupiter | ~0.9415 days | 1.08 R_J | Limb-darkened model correction |
| **HAT-P-7 b** | Confirmed Planet | ~2.2045 days | 0.73 R_J | Standard BLS period search and baseline parameter calculation |
| **TIC 261136679** | Detected Candidate | ~6.2661 days | ~0.07 R_J | Signal extracted via phase binning |

---

## Technologies & Dependencies
* **Language:** Python 3.x
*  **Core libraries:**
       * **lightkurve** (NASA MAST data retrieval and TPF formatting)
       * **numpy** (Array manipulation, mathematical constants, and masking operations)
       * **astropy** (Cosmological constant definitions and unit conversions)
       * **matplotlib** (Data visualization, colormapping, and spatial plotting)
       * **shutil** (Cache management during high-volume data loops)

---
## Future Roadmap: Cloud Automation
Right now, I run this code manually on my computer. For my next major update, I plan to turn it into a fully independent software robot:
**Cloud Hosting:** I will put the code on an always-on cloud server and schedule it to automatically scan NASA's new data drops every single week.
**Smart Memory:** I will add a database so the program remembers which stars it has already checked, ensuring it only looks for brand-new planets.
**Discord Alerts:** I will program the code to instantly send a message to my phone's Discord app the moment it successfully finds and verifies a new planet shadow!
