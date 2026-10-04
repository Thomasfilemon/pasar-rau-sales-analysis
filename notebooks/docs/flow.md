Sekarang baru enak. **Kita berhenti “ngoprek file” dan mulai kerja kayak data analyst beneran.**

Urutannya jangan langsung SQL, jangan langsung Power BI, apalagi bikin dashboard warna-warni dulu. Dashboard tanpa ngerti datanya itu cuma **hiasan mahal buat kebohongan**.

Kita mulai dari:

# Phase 1 — Data Understanding

Tujuan fase ini cuma satu:

> **Pahami apa isi dataset, seberapa sehat datanya, dan analisis apa yang valid dilakukan.**

File pertama yang kita kerjakan:

```text
notebooks/
└── 01_data_understanding.ipynb
```

Notebook ini bukan tempat “mencari insight bisnis” dulu. Ini tempat kita menginterogasi dataset.

---

## 1. Load raw data tanpa merusaknya

Karena CSV kita agak goblin, kita harus hati-hati saat import.

Hal pertama yang mau kita pastikan:

- delimiter yang benar
- jumlah baris valid
- jumlah kolom
- jangan sampai ID berubah jadi float
- jangan sampai decimal comma salah dibaca
- buang trailing empty garbage rows **tanpa menyentuh raw file**

Mental model-nya:

> “Sebelum gue percaya tabel ini, apakah Python membaca file ini sesuai dengan apa yang sebenarnya ada di file?”

Itu pertanyaan pertama.

---

## 2. Understand the grain

Ini salah satu konsep paling penting.

Kita harus menjawab:

> **Satu row mewakili apa?**

Dari yang sudah kita lihat, kemungkinan besar:

> satu baris = satu SKU/product line dalam sebuah sales transaction.

Tapi nanti kita **buktikan**, bukan sekadar asumsi.

Kita cek kombinasi:

```text
Invoice
Date
Outlet
SKU
Quantity
```

Kalau invoice yang sama punya banyak SKU, berarti jelas satu invoice bisa terdiri dari beberapa rows.

Ini penting karena nanti menentukan cara kita menghitung:

- sales
- orders
- customers
- products
- average order value

---

# 3. Build a data dictionary

Kita dokumentasikan setiap kolom.

Misalnya:

| Column | Meaning | Type Expected | Analyst Notes |
|---|---|---|---|
| `INVDate` | transaction date | datetime | currently text |
| `INVNumber` | invoice identifier | string | partially corrupted |
| `Outlet` | outlet identifier | string | preserve exactly |
| `Outlet Name` | customer/outlet name | text | may need normalization |
| `SKUCode` | product identifier | string | likely usable |
| `ProductName` | product description | text | check consistency |
| `GSV` | gross sales value | numeric | locale parsing needed |
| `Discount` | discount value | numeric | usually negative |
| `Tax` | tax | numeric | validate relationship |
| `Net` | final sales amount | numeric | validate formula |

This might seem boring.

It isn't.

A good analyst can explain **what their columns mean and which ones they trust**.

---

# 4. Profile the dataset

Then we ask basic structural questions.

Things like:

```python
df.shape
df.head()
df.tail()
df.info()
df.columns
```

But here's the important part:

You're not running those commands because some tutorial told you to.

Each answers something.

### `df.shape`

Question:

> How much data do we actually have?

Expected:

```text
10,678 rows
29 columns
```

### `df.head()`

Question:

> What does a typical record look like?

### `df.tail()`

Particularly important here.

Question:

> Did our parser accidentally include those hundreds of thousands of garbage lines?

### `df.info()`

Question:

> What does pandas think each column is?

If `Net` comes out as:

```text
object
```

instead of:

```text
float64
```

that's not just a technical detail.

That's evidence that the column needs cleaning.

---

# 5. Missing values

Then:

> What information is missing, and is that expected?

We'll generate something like:

```text
Column             Missing    Missing %
NOTES 1             xxxx        xx%
NOTES 2             xxxx        xx%
Weight              ...
...
```

And this is where analyst thinking matters again.

A beginner sees:

> 70% null → BAD

An analyst asks:

> **Why is it missing?**

Maybe `NOTES 1` is optional.

Maybe `Weight` only applies to certain product categories.

Maybe missingness itself carries business meaning.

So:

```text
missing ≠ automatically bad
```

---

# 6. Cardinality

Cardinality basically means:

> How many unique values does this column have?

For example:

```text
SKUCode          → ~541 unique
Outlet Name      → ~85
Salesperson      → ~7
Month            → 12
Invoice          → ???
```

This helps us identify:

### Dimensions

Things you group by:

```text
Date
Product
Outlet
Salesperson
```

### Measures

Things you calculate:

```text
Quantity
GSV
Discount
Tax
Net
```

This distinction becomes extremely important later in SQL and Power BI.

---

# 7. Identifier validation

This dataset makes this section especially valuable.

We'll investigate:

### Outlet ID

Questions:

> Does every Outlet ID map to exactly one Outlet Name?

If not:

> Is it spelling variation?

Example:

```text
TOKO MAJU
TOKO MAJU.
TOKO MAJU 1
```

Or legitimately different customers?

---

### SKU

Question:

> Does one SKU code consistently map to one product name?

Ideally:

```text
SKU123
→ PEPSODENT 190G
```

always.

If:

```text
SKU123
→ PEPSODENT 190G
→ PEPSODENT WHITE 190G
→ PEPSODENT 190 GR
```

then we need to decide whether it's merely naming variation or changing product metadata.

---

### Salesperson

We already know salesperson codes are reused.

So we'll quantify exactly:

> Which salesman codes map to multiple names, and during which dates?

Could be reassignment.

And if the mapping changes cleanly over time, that tells us something.

For example:

```text
Salesman 21
Jan-Jun  → SAEFUL
Jul-Dec  → SUKMADEWI
```

That would strongly suggest route reassignment.

That is much more useful than simply saying:

> “IDs inconsistent lol.”

---

# 8. Validate the money

One thing we've already discovered:

```text
Net ≈ GSV + Discount + Tax
```

We will formally test it.

Create:

```python
calculated_net = gsv + discount + tax
difference = net - calculated_net
```

Then inspect:

```text
mean difference
median difference
max difference
rows outside tolerance
```

Why?

Because before using financial values for business conclusions, we should establish that they reconcile.

This is basically a **data quality control test**.

Portfolio gold.

---

# 9. Quantity relationships

We've got:

```text
Case
Dozen
Pieces
TotalQuantity(PCS)
```

So naturally:

> Is `TotalQuantity(PCS)` derived from the others?

Perhaps something resembling:

```text
Case × case_pack
+ Dozen × 12
+ Pieces
```

But we don't know product case sizes yet.

This is exactly where an analyst says:

> “I cannot validate this relationship without knowing pack-size semantics.”

Not every mystery needs to be solved by hallucinating a formula.

---

# 10. Identify returns

We'll eventually create something like:

```python
transaction_type
```

with categories:

```text
SALE
RETURN
ADJUSTMENT / UNKNOWN
```

Potential logic might involve:

```text
TotalQuantity(PCS) < 0
Net < 0
GSV < 0
```

But again, don't jump straight into:

```python
df["type"] = np.where(df["Net"] < 0, "Return", "Sale")
```

We need to see whether those indicators agree.

Maybe:

```text
quantity negative
net negative
```

usually occur together.

Maybe there are exceptions.

Those exceptions are where the interesting shit lives.

---

# 11. Duplicate investigation

Not:

```python
df.drop_duplicates(inplace=True)
```

Absolutely not yet.

First:

```python
df.duplicated().sum()
```

Then inspect them.

Questions:

> Same invoice?

> Same SKU?

> Same quantity?

> Same financial values?

> Same customer?

A perfectly identical repeated row might be an accidental duplicate.

Or maybe it's two identical product lines.

Because invoice IDs are partially corrupted, we need caution.

We document the decision rather than pretending deletion is obvious.

---

# 12. Date coverage

Then we validate:

```text
minimum date
maximum date
number of months
transactions per month
missing days
```

We already know:

```text
04 Jan 2021
→
31 Dec 2021
```

But perhaps March has 1,500 rows and September has 50.

That could indicate:

- seasonality
- incomplete data
- operational changes

At this stage we don't conclude which one.

We merely notice.

---

# Then we finish Notebook 01 with something recruiters love

A section called:

# Initial Data Quality Findings

Something along the lines of:

```text
1. Dataset contains 10,678 valid line-item records covering Jan-Dec 2021.

2. Transaction dates are complete but require explicit day-first parsing.

3. Outlet identifiers are preserved in the raw source, although
   outlet-to-name mappings require validation.

4. Invoice identifiers are partially corrupted due to scientific
   notation and should not be used blindly for transaction counts.

5. Financial columns use inconsistent number formatting and require
   locale-aware parsing.

6. Net sales reconcile closely with GSV + Discount + Tax.

7. Negative quantities and financial values indicate return/reversal
   activity.

8. Salesperson codes appear to be reassigned between employees.

9. Duplicate records exist and require investigation before removal.
```

That's already the beginning of your portfolio story.

Notice something?

**We haven't made a single sexy chart yet.**

And we're already doing actual analysis.

---

# Then comes Phase 2

Once Notebook 01 is complete:

```text
01_data_understanding.ipynb
        ↓
02_data_cleaning.ipynb
```

`02` will actually perform the transformations discovered in `01`.

Such as:

```text
parse dates
clean numeric fields
preserve IDs as strings
standardize text
flag unreliable invoices
classify returns
handle duplicates
create clean derived fields
```

Then output:

```text
data/processed/
└── pasar_rau_sales_clean.csv
```

Only **after that** do we start:

```text
03_exploratory_analysis.ipynb
```

That's where we finally ask:

> What happened to the business in 2021?

And start exploring:

- monthly sales
- top products
- product mix
- outlets
- salespeople
- returns
- discounts
- customer behavior

Then:

```text
SQL
↓
Power BI
↓
README / case study
```

---

## What I want you to do right now

Don't analyze anything manually yet.

Make sure your repo looks approximately like:

```text
pasar-rau-sales-analysis/
│
├── data/
│   ├── raw/
│   │   └── Penjualan2021-Pasarawu.csv
│   ├── interim/
│   └── processed/
│
├── notebooks/
│   └── 01_data_understanding.ipynb
│
├── sql/
├── dashboard/
├── reports/
├── images/
│
├── .gitignore
└── README.md
```

And make sure:

```text
data/raw/
```

is ignored by Git.

Then **our next actual session starts inside `01_data_understanding.ipynb`**.

I'll give you the first cells, explain why we're writing each one, we'll inspect the results together, and I'll deliberately stop at certain points and ask **“what do you notice?”** before I give you my interpretation.

That's where the actual analyst training starts.

We've finished **finding the dataset**.

Now we start **interrogating the bastard.**