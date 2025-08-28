# esr-test
This can inform gpt decision making
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


```
# Fit a Voigt line, auto-baseline, and integrate absorption over 3×FWHM:
python -m esrproc fit-integrate \
  --input data/example_esr.csv \
  --units Oe \
  --signal-kind derivative \
  --model voigt \
  --baseline asls --asls-lambda 1e6 --asls-p 0.001 \
  --smooth-savgol 11 3 \
  --out results/example_run.json \
  --plot results/example_run.png

  ```
import pandas as pd
from esrproc import Pipeline, BaselineAsLS, PeakModel

df = pd.read_csv("data/example_esr.csv")  # columns: field, signal

pipe = Pipeline(
    signal_kind="derivative",             # or "absorption"
    baseline=BaselineAsLS(lmbda=1e6, p=1e-3),
    model=PeakModel(kind="voigt"),        # "gauss", "lorentz", "voigt"
    smooth_savgol=(11, 3)                 # optional
)

result = pipe.run(df.field.values, df.signal.values, units="Oe")

print(result.center, result.fwhm, result.area_3fwhm)
print(result.window_bounds)  # (Bmin, Bmax)
# result.as_dict() -> for JSON serialization
```
{
  "file": "data/example_esr.csv",
  "units": "Oe",
  "signal_kind": "derivative",
  "baseline": {"kind": "asls", "lambda": 1000000.0, "p": 0.001},
  "model": "voigt",
  "fit": {
    "center": 3342.1,
    "fwhm": 9.37,
    "amplitude": 1.82,
    "params": {"gauss_sigma": 3.8, "lorentz_gamma": 2.7},
    "param_covariance_diag": {"center": 0.03, "fwhm": 0.05}
  },
  "window": {"min": 3332.0, "max": 3352.2, "width": 28.11},
  "area_3fwhm": 124.6,
  "area_uncertainty_est": 3.2,
  "notes": "absorption reconstructed from derivative prior to integration",
  "versions": {"numpy": "…", "scipy": "…", "lmfit": "…"}
}

