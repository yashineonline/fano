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


