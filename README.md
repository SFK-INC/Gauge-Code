## SFK Inc. Gauge Code Specification

**Version 1.0 – Effective Jan 2021**

### 1. Purpose

The SFK Gauge Code provides a compact, fixed-length (six-digit) representation of linear dimensions expressed in feet, inches, and fractional inches (to the nearest ⅛ inch). It is designed for loss-less round-trip conversion and effortless machine parsing within SFK’s survey, auditing, and automation pipelines.

---

### 2. Data Structure

| Field    | Width    | Range                          | Encoding                | Notes                                   |
| -------- | -------- | ------------------------------ | ----------------------- | --------------------------------------- |
| Feet     | 2 digits | 00 – 99 ft                     | Zero-padded decimal     | Supports objects up to 99 ft inclusive. |
| Inches   | 2 digits | 00 – 11 in                     | Zero-padded decimal     | Always base-12 (no “12”).               |
| Fraction | 2 digits | 00, 12, 14, 18, 34, 38, 58, 78 | Custom lookup (see § 3) | Resolution: ⅛ inch.                     |

The concatenation **Feet Inches Fraction → F I F** yields a six-digit ASCII string—for example, **11 ft 2 ¾ in → `110234`**.

---

### 3. Fraction Lookup Table

| Fractional Inch | Gauge Code | Binary ⅛-inch equivalent |
| --------------- | ---------- | ------------------------ |
| 0 (exact inch)  | 00         | 0⁄8                      |
| ⅛ in            | 18         | 1⁄8                      |
| ¼ in  (= 2⁄8)   | 14         | 2⁄8                      |
| ⅜ in            | 38         | 3⁄8                      |
| ½ in  (= 4⁄8)   | 12         | 4⁄8                      |
| 5⁄8 in          | 58         | 5⁄8                      |
| ¾ in  (= 6⁄8)   | 34         | 6⁄8                      |
| 7⁄8 in          | 78         | 7⁄8                      |

**Rounding convention** – Measurements must be rounded to the nearest eighth-inch before encoding. Tie-break at ¹⁄₁₆ in (0.0625 in) goes **up**.

---

### 4. Encoding Procedure

1. **Normalize input**

   * Accept any standard imperial format (`ft in`, `' "`, or mixed).
   * Convert fractions to denominator 8.

2. **Range check**

   * 0 ≤ feet ≤ 99; 0 ≤ inches ≤ 11; fraction ∈ {0, 1⁄8, …, 7⁄8}.

3. **Pad and map**

   * Feet: `{:02d}`.
   * Inches: `{:02d}`.
   * Fraction: table lookup (§ 3).

4. **Concatenate** → six-digit string.

> **Example**: 7 ft 5 ½ in
>
> * Feet = 07 → “07”
> * Inches = 05 → “05”
> * Fraction = ½ in → code 12
> * Output **070512**

---

### 5. Decoding Procedure

Reverse the process: split the six-digit string into two-digit segments, translate the fraction code back via § 3, then render in any desired imperial notation.

---

### 6. Validation Rules

| Check    | Condition                     |
| -------- | ----------------------------- |
| Length   | Exactly 6 ASCII digits (0–9). |
| Feet     | 00–99 inclusive.              |
| Inches   | 00–11 inclusive.              |
| Fraction | Must match one entry in § 3.  |

Invalid records should raise a *GaugeCodeError* (or equivalent in host language).

---

### 7. Implementation Hints

* **Regular expression** (validation): `^(?:[0-9]{2})(?:0[0-9]|1[01])(?:00|12|14|18|34|38|58|78)$`
* **Storage** – Treat as CHAR(6) or VARCHAR(6); avoid integer fields to preserve leading zeros.
* **Sorting** – Lexicographic order equals numeric order because the string is zero-padded.

---

### 8. Worked Examples

| Imperial Input     | Gauge Code |
| ------------------ | ---------- |
| 9 ft 4 ⅛ in        | 090418     |
| 11 ft 2 ¾ in       | 110234     |
| 8 ft 11 in (exact) | 081100     |
| 8 ft 5 ⅝ in        | 080558     |
| 3 ft 11 ⅜ in       | 031138     |
| 7 ft 5 ½ in        | 070512     |

---

### 9. Limitations & Future Extensions

* **Resolution**: Fixed at ⅛ in. For finer tolerances, the field layout would require revision.
* **Negative lengths**: Not supported; prepend metadata outside the code if directionality is required.
* **> 99 ft**: Unsupported in current six-digit schema; adopt an expanded field if necessary.

---

**© 2025 SFK Inc.**  All rights reserved.
