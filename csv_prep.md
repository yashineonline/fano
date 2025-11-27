### 1. File details

* **Output file name**: `collagen_band_assignments.csv`
* **Format**: Comma-separated values (CSV), UTF-8
* **No empty rows** in the middle of the file.

### 2. Columns to include

Please create a header row with the following columns **in this order**:

1. `band_label`

   * A short, machine-friendly name, e.g.

     * `amide_I_1`, `amide_III_2`, `CH2_bend`, `tyr_doublet_1`, etc.
   * Use only letters, numbers and underscores (no spaces, no accents).

2. `band_type`

   * A broad category we will use in code. Use one of:

     * `amide_I`
     * `amide_III`
     * `aromatic`
     * `CH_bend`
     * `backbone`
     * `other`
   * Choose the best match from the assignment in the thesis (e.g. amide I, amide III, ring modes, CH(_2)/CH(_3) bending, etc.).
   * If you’re uncertain, use `other` and keep the explanation in the `assignment_text` column.

3. `center_cm_1`

   * The **central wavenumber** from the table (e.g. 1665, 1450, etc.).
   * Numeric only (no units, no “cm⁻¹”). Example: `1665`

4. `lo_cm_1`

   * Lower bound of the band range in cm⁻¹.
   * If the table gives a range (e.g. 1650–1675), use the lower value (`1650`).
   * If only a single value is given, set

     * `lo_cm_1 = center_cm_1 - 5` (so 1665 → 1660).

5. `hi_cm_1`

   * Upper bound of the band range in cm⁻¹.
   * If the table gives a range (e.g. 1650–1675), use the upper value (`1675`).
   * If only a single value is given, set

     * `hi_cm_1 = center_cm_1 + 5` (so 1665 → 1670).

6. `assignment_text`

   * Copy the **verbal assignment / mode description** from the table (e.g. “amide I, C=O stretch”, “CH(_2) scissoring”, “phenyl ring breathing”, etc.).
   * It’s fine if this contains spaces. If there are commas, please either:

     * put the whole field in double quotes, **or**
     * replace commas with semicolons `;` to keep the CSV clean.

7. `reference`

   * Put: `Fields2016_phd_table3_1` for every row (this just tracks where the assignments came from).

### 3. Example rows

**Header:**

```text
band_label,band_type,center_cm_1,lo_cm_1,hi_cm_1,assignment_text,reference
```

**Example 1 (range given in table):**

```text
amide_I_1,amide_I,1665,1650,1675,"amide I, C=O stretch",Fields2016_phd_table3_1
```

**Example 2 (single value in table):**

```text
CH2_bend_1,CH_bend,1450,1445,1455,"CH2 scissoring / CH3 bending",Fields2016_phd_table3_1
```

**Example 3 (aromatic ring mode):**

```text
tyr_ring_1,aromatic,830,825,835,"tyrosine ring mode",Fields2016_phd_table3_1
```

### 4. Add additional bands

Using the table at the link below, check whether there are any **extra bands** that do not already appear in the thesis table.  
For any new bands you find, add corresponding rows to the CSV in the same format.

<https://www.nature.com/articles/s41598-019-43636-2/tables/1>

### 5. General rules

* One **row per band** listed in Table 3.1.
* Don’t leave cells blank; if something is genuinely unknown, write `unknown`.
* Use `.` as decimal separator (e.g. `1665.0` is fine).
* Please keep units consistent (always cm⁻¹, but do **not** write “cm⁻¹” in the numeric columns).
* Save and send me both:

  * the `.csv` file
  * and optionally an `.xlsx` version if you used Excel (but the CSV is what the code will read).


additional links and info for next step

https://www.mdpi.com/1422-0067/24/19/14748

components: elastin, keratin, nucleus, triolein, ceramide, melanin, water. 

approximate values

* 825–860 → `tyrosine_aromatic`
* 995–1015 → `phenylalanine_aromatic`
* 1230–1310 → `amide_III`
* 1430–1480 → `CH_bend`
* 1630–1680 → `amide_I`
* 1550–1615 → `other_aromatic`

These are vibration types; different bases just have different *weights* on them.

**Collagen** (triple-helix protein) ([MDPI][1])

* 825–860: Pro/HyPro / Tyr ring → `tyrosine_aromatic`
* 938–940: C–C stretch of collagen backbone (falls near your “phenylalanine_aromatic” window but physically it’s C–C of the helix).
* 1240–1270: amide III of collagen triple helix → `amide_III`
* 1440–1450: CH2 bending of the glycine–proline–hydroxyproline backbone → `CH_bend`
* 1660–1670: amide I (C=O stretch) → `amide_I`

**Elastin** (more disordered protein, some lipid-like contribution) ([MDPI][1])

* 850–860: Tyr / Pro ring modes (same `tyrosine_aromatic`)
* ~1003: Phe ring → `phenylalanine_aromatic`
* 1240–1300: amide III, but more mixed/random coil → `amide_III`
* 1440–1455: CH2/CH3 bending from both protein + associated lipids → `CH_bend`
* 1655–1665: amide I (α-helix / random coil mix) → `amide_I`

**Keratin** (hair/stratum corneum protein, rich in cystine) ([MDPI][1])

* ~830–860: Tyr doublet → `tyrosine_aromatic`
* ~1003: very strong Phe ring → `phenylalanine_aromatic`
* 1245–1270: amide III (α-helix of keratin) → `amide_III`
* 1448–1455: CH2 bending (keratin side chains + lipids) → `CH_bend`
* 1655: amide I (α-helix) → `amide_I`
* 510–540 (outside our current ROI) would be S–S stretch of cystine (disulfide)

**Triolein** (TAG lipid) ([MDPI][1])

* ~1060–1128: C–C stretching in fatty acyl chains (will map to “other” in your function, but physically lipid backbone)
* 1265–1305: =CH in-plane deformation / CH2 twisting of unsaturated chains → falls into your `amide_III` window but is really lipid CH modes.
* 1440–1450: CH2 scissoring in long chains → `CH_bend`
* 1655–1660: C=C stretch of olefinic bonds (cis-double bonds) → in your `amide_I` range but physically lipid C=C.
* ~1740 (if present in the data): C=O ester stretch (outside current `BAND_RANGES`).

**Ceramide** (sphingolipid) ([MDPI][1])

* 1060–1130: C–C stretching of long chains (again “other”).
* 1295–1310: CH2 twisting/wagging of saturated chains → will fall in `amide_III` window but is lipid CH.
* 1430–1465: CH2 bending/scissoring of saturated chains → `CH_bend`.
* 1650–1665: mainly C=C (if unsaturated) + some amide I from the headgroup → `amide_I`.

**Nucleus** (DNA + histones)
In 800–1800 cm⁻¹ ROI, expect:

* ~785–790: ring breathing of cytosine/uracil + backbone modes (will map to “other”).
* ~1092–1100: PO₂⁻ symmetric stretch of DNA/RNA backbone.
* ~1335–1340: base ring stretching (A,G) / CH deformation.
* ~1575–1585: ring stretching of bases (A,G). → will fall partly into `other_aromatic`.
* 1655: amide I of histones (protein) + some base C=O.

When nucleus peaks land in `other_aromatic` or `amide_I` in the code, physically they are **nucleic-acid base** modes plus histone amide I, not aromatic amino acids.

**Melanin** ([ResearchGate][2])

* ~1350: “D”-like band, disordered indole/quinone ring stretching and C–N. → falls into `other_aromatic` band.
* ~1580–1590: “G”-like band, in-plane C=C stretching of aromatic/graphitic domains → also `other_aromatic`.
* 1440–1450: CH modes of associated proteins / lipids → `CH_bend`.

**Water** (residual liquid / bound water)
In 800–1800 cm⁻¹ the *true* H–O–H bending is at ~1640 cm⁻¹; in tissues it overlaps amide I. So any broad background around 1630–1660 that `amide_I` bin picks up in the “water” basis is mostly H–O–H bending plus some protein leakage. The sharp OH stretch bands are way above your ROI.

---

* **Code-wise**: `assign_band_type(w0)` already applies this *same* set of windows for **all** components. 
* **Physically**: we interpret the label differently depending on the basis:

  * In proteins: `amide_III`, `amide_I`, `CH_bend`, `tyrosine_aromatic`, `phenylalanine_aromatic` are literal.
  * In lipids: `amide_III` window mostly = CH₂ twisting; `amide_I` window mostly = C=C stretch.
  * In nucleus: `other_aromatic` ≈ nucleic base modes; `amide_I` ≈ histone + some base C=O.
  * In melanin: `other_aromatic` ≈ indole/graphitic C=C modes.

[1]: https://www.mdpi.com/1422-0067/24/19/14748 "Multispectral Raman Differentiation of Malignant Skin Neoplasms In Vitro: Search for Specific Biomarkers and Optimal Wavelengths"
[2]: https://www.researchgate.net/figure/a-Polarized-and-depolarized-Raman-spectra-of-synthetic-melanin-measured-using-the_fig7_8158501?utm_source=chatgpt.com "(a) Polarized and depolarized Raman spectra of synthetic ..."






