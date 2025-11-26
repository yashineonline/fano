**Fano lineshape fitter**

Roughly, it takes a CSV like:

* first column: wavenumber (cm⁻¹) or implicit grid
* other columns: basis spectra for **collagen, elastin, triolein, nucleus, keratin, ceramide, melanin, water**

For each component (each column):

1. **Load the spectrum**

   ```python
   w, y = load_component(csv, comp)
   mask = (w >= WN_START) & (w <= WN_END)
   w, y = w[mask], y[mask]
   ```

   It restricts to a region of interest (ROI) in wavenumber.

2. **Detect candidate peaks**
   Using `find_peaks` (optionally after Savitzky–Golay smoothing) with two prominence thresholds (strong peaks and weaker ones) plus optional “shoulder” detection:

   ```python
   cand_locs, sg_used = pick_candidates(w, y, use_sg=use_sg)
   ```

   Then it ranks and thins them (optionally with binning across the ROI so peaks aren’t all clustered in one region).

3. **Build Fano models with different numbers of peaks**
   For each candidate model size ( k = 1, 2, ..., \text{planned_models} ):

   * It takes the first (k) candidate indices.
   * Builds initial guesses and bounds via `build_init_bounds`:

     * (I_0): amplitude
     * `inv_q` = (1/q): controls asymmetry (Fano parameter)
     * (w_0): center position
     * (\gamma): linewidth
   * Fits a sum of Fano profiles by nonlinear least squares:

     ```python
     res = least_squares(residuals_fano, x0, bounds=(lb, ub), ...)
     ```
   * The model is:

     ```python
     fano_component(w, I0, inv_q, w0, gamma)
     fano_sum = sum of these over peaks
     ```

4. **Compare two hypotheses per component**

   For each component you actually fit **two families of models**:

   * **Full Fano:** `inv_q` free (asymmetric line shapes allowed).
   * **Symmetric null:** `inv_q` ≈ 0 (effectively Lorentzian/symmetric peaks).

   That’s done here:

   ```python
   # Fano (free inv_q)
   storage_fano, aics_fano, bics_fano = scan_models(..., fix_invq=None, ...)

   # Symmetric null (inv_q = 0)
   storage_sym, aics_sym, bics_sym = scan_models(..., fix_invq=0.0, ...)
   ```

   For each family, you:

   * scan over (k),
   * compute AIC/BIC,
   * choose the best (k) model.

   Then you compare the **best Fano model** vs the **best symmetric model** via:

   ```python
   delta_aic = aics_best_sym[idxS] - aics_best[idxF]
   delta_bic = bics_best_sym[idxS] - bics_best[idxF]
   ```

   Positive ΔAIC/ΔBIC means *Fano is preferred over symmetric*.

5. **Model diagnostics + summary**

   For the chosen best Fano model per component, you:

   * Save **tables**:

     * `*_IC.tsv`: AIC/BIC vs k
     * `*_IC_null.tsv`: same for symmetric null
     * `*_BestParams.tsv`: fitted [I0, inv_q, w0, gamma]
     * `*_FitCurve.tsv`: w, data, fit, residual

   * Save **plots**:

     * `*_IC.svg`: AIC/BIC vs k
     * `*_BestFit.svg`: data + total fit + individual components
     * `*_Residuals.svg` + `*_ResidualStats.tsv`: residuals, Durbin–Watson, roughness.

   * Compute diagnostic scalars: median |q|, median gamma, roughness, DW, average prominence of matched peaks, etc., and append to a run-level:

     ```python
     out_fano_all/summary_overview.tsv
     ```

   * At the end, you auto-build a LaTeX table `summary_overview.tex` summarizing those diagnostics.

6. **Synthetic control**

   Separately, once per run (unless you disabled it in quick mode), `run_control_if_needed` creates a **synthetic symmetric spectrum** (sum of Lorentzians) and runs the same Fano vs symmetric comparison to see if your pipeline spuriously “detects Fano asymmetry” on a purely symmetric control.

---

## What QUICK PROFILE changes

When `quick=True`, you **tighten** everything to get a very fast approximate run:

```python
if quick:
    WN_START, WN_END = 900.0, 1700.0
    MINPROM1, MINDIST1 = 0.30, 8
    MINPROM2, MINDIST2 = 0.08, 4
    USE_SHOULDERS = False
    MAX_PEAKS = 4
    MAX_NFEV = 60_000
    EARLYSTOP_PATIENCE = 1
```

So quick mode:

* Uses a **narrower ROI** (900–1700 cm⁻¹).
* Requires **stronger peaks** (higher prominence).
* Limits to **at most 4 peaks**.
* Stops model-size scanning as soon as AIC worsens once (`EARLYSTOP_PATIENCE = 1`).
* Reduces max function evaluations.

It still does the **same logic**, just on fewer peaks and stricter thresholds, primarily to check speed and rough structure.

---

## What are AIC and BIC here?

You’re using standard **information criteria** to choose how many peaks you really need and whether Fano asymmetry is justified.

Given:

* observed data (y),
* model predictions (\hat{y}),
* number of free parameters (k_{\text{params}}),

you build:

```python
def aic_score(y_true, y_pred, k_params):
    n = y_true.size
    rss = np.sum((y_true - y_pred)**2)
    return 2*k_params + n*np.log(rss/max(n,1))

def bic_score(y_true, y_pred, k_params):
    n = y_true.size
    rss = np.sum((y_true - y_pred)**2)
    return k_params*np.log(max(n,1)) + n*np.log(rss/max(n,1))
```

Interpretation:

* **AIC (Akaike Information Criterion)**
  ( \mathrm{AIC} = 2k + n \log(\mathrm{RSS}) )
  Balances goodness-of-fit (RSS: residual sum of squares) against model complexity (k). Lower AIC = better trade-off.

* **BIC (Bayesian Information Criterion)**
  ( \mathrm{BIC} = k \log n + n \log(\mathrm{RSS}) )
  Similar idea but with a stronger penalty for extra parameters (especially for large n). Again, lower is better.

You use them for two things:

1. **Choosing k (number of peaks)** within the Fano family and within the symmetric family.
2. **Comparing Fano vs symmetric** at matched k via ΔAIC, ΔBIC.

So conceptually:

> “Does adding Fano asymmetry (`inv_q` free) improve the fit enough to justify the extra degrees of freedom, or is a symmetric Lorentzian-like model sufficient for this component?”

---

So yes: just from the code, it’s very clear this is a **Raman (or similar) basis-spectrum fitter using sums of Fano profiles**, with **automatic peak picking, model-order selection by AIC/BIC**, and **Fano vs symmetric model comparison** per component, plus a quick-profile mode to speed up exploratory runs.
