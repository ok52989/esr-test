# ESR Baseline & Area (3×FWHM)

A small, test-friendly ESR processing tool that:

1) baseline-corrects an ESR trace,  
2) fits the main peak to estimate **center** and **FWHM**, and  
3) integrates the **absorption** signal within a **3×FWHM** window centered at the peak.

Designed for quick experiments, reproducible checks, and drop-in use for teaching or lab notebooks.

---

## Why this exists

In ESR, raw signals often include a sloping or wavy background. Quantitative comparison (e.g., relative spin counts) requires (a) a stable baseline, (b) a consistent window tied to the physics of the line, and (c) integrating the **absorption** (not the derivative). This project wires those together with minimal ceremony.

---

## Method (at a glance)

1. **Load data** (CSV/TSV) with at least two columns:
   - `field` (magnetic field; Oe or T)
   - `signal` (ESR amplitude; derivative or absorption)

2. **(Optional)** Denoise with Savitzky–Golay (for display / robust baselining).

3. **Baseline correction** (default: Asymmetric Least Squares, AsLS):
   - Removes slow background while preserving the line.
   - Tunable asymmetry `p` and smoothness `λ`.

4. **Peak fitting** on the **baseline-corrected** signal:
   - Models: Gaussian, Lorentzian, or Voigt (default: Voigt).
   - Returns **center** \(B₀\) and **FWHM**.
   - If your input is a **derivative** ESR signal, fitting is performed on the derivative model; the absorption is recovered by numerical integration before area computation.

5. **Integration window**:
   - Width = **3 × FWHM**.
   - Bounds = \([B₀ - 1.5\,\text{FWHM},\; B₀ + 1.5\,\text{FWHM}]\).

6. **Area computation**:
   - Convert to absorption if needed (cumulative integral of derivative).
   - Numerically integrate absorption over the 3×FWHM window (trapezoidal rule).
   - Report area, uncertainty (via fit covariance & simple propagation), and all parameters.

---

## Installation

```bash
# Python 3.10+
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -U pip

# Install project dependencies
pip install -r requirements.txt
