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

This is good work. More importantly, you’ve reached the point where we can stop asking **“what’s wrong with the data?”** and start deciding **“what are we going to do about it?”**

And before we move on, there are two analyst-level corrections I want you to make.

First, this:

> “There are 21 rows of duplicates.”

If you got that from:

```python
df.duplicated().sum()
```

and every duplicate group occurs exactly twice, the more precise interpretation is:

> **There are 21 duplicated records beyond their original occurrence, forming 21 duplicate pairs and involving 42 rows in total.**

That distinction matters. `duplicated().sum()` counts the *extra copy*, not every row participating in duplicate groups.

Second, those Net differences are effectively negligible. You found:

```text
min    -0.81
max     0.81
median  0.00
```

Assuming these financial fields are in rupiah, we're talking about differences of **less than Rp1**. That's not a meaningful financial discrepancy on transactions worth thousands/millions of rupiah.

So I wouldn't write:

> “1,166 Net mismatches.”

That sounds way more dramatic than reality.

I'd write:

> **1,166 rows exceed a strict Rp0.10 tolerance, but all reconciliation differences remain below Rp1, indicating minor rounding differences rather than material financial inconsistencies.**

That is much better analytical communication.

And your tax result is even stronger:

> **No rows differed from the expected 10% tax calculation by more than Rp0.10.**

That gives us pretty damn strong evidence for:

\[
Tax = 10\% \times (GSV + Discount)
\]

at least for this dataset.

---

# Your Notebook 01 is basically finished

I'd tighten your final summary slightly before closing it.

For the identifier section, change this:

> Invoice and outlet identifiers are preserved in the raw source and can be used for traceability...

because we already know `INVNumber` is partially damaged.

Use something closer to:

> **Outlet identifiers are preserved in the raw source but require mapping validation. Invoice identifiers are only partially preserved, with many values affected by scientific notation and therefore unsuitable for unrestricted invoice-level analysis.**

And I'd change:

> Transaction dates are generally complete...

to:

> **All valid transaction rows contain parseable transaction dates covering 4 January through 31 December 2021.**

Specific beats vague.

Your financial conclusion can become:

> **Net reconciles to GSV + Discount + Tax within less than Rp1 across the dataset, indicating only immaterial rounding differences. Tax reconciles to approximately 10% of GSV after discount within the selected tolerance.**

That's portfolio-quality wording.

---

# Now: Phase 2 — Data Cleaning

Create:

```text
notebooks/
└── 02_data_cleaning.ipynb
```

This notebook has a very different purpose.

Notebook 01 asked:

> **What problems exist?**

Notebook 02 asks:

> **What transformations should we apply so downstream analysis is consistent, reproducible, and defensible?**

Your workflow should be:

```text
RAW
 ↓
Parse
 ↓
Standardize
 ↓
Validate
 ↓
Flag questionable records
 ↓
Create analytical fields
 ↓
Save processed dataset
```

The important word there is **flag**.

We're not going around deleting every row that looks funny like some trigger-happy intern with `dropna()`.

---

# Step 1 — Document your cleaning rules first

Start the notebook with Markdown:

```markdown
# Pasar Rau Sales Analysis — Data Cleaning

## Objective

Transform the raw sales data into a consistent analytical dataset while
preserving the original business information and documenting all assumptions.

## Cleaning Principles

1. Preserve the raw source unchanged.
2. Convert fields into appropriate analytical data types.
3. Do not fabricate missing or corrupted identifiers.
4. Preserve returns and reversals as valid business activity.
5. Flag questionable records rather than silently deleting them.
6. Remove records only when there is sufficient evidence that they are invalid.
7. Validate important transformations before exporting the processed dataset.
```

That section is not fluff.

It tells a recruiter how you think.

---

# Step 2 — Reuse the raw ingestion logic

Load the file exactly the same deterministic way as Notebook 01.

Then immediately verify:

```python
assert df.shape[0] == 10678
assert df.shape[1] == 29
```

This is your first little quality-control gate.

If suddenly:

```text
10,678 → 8,423
```

your pipeline should scream instead of quietly producing nonsense.

---

# Step 3 — Rename ugly columns

Now we're finally allowed to modify names.

I'd use snake_case:

```python
column_mapping = {
    "Week": "week",
    "Month": "month",
    "Quart": "quarter",
    "DistID": "distributor_id",
    "DistName": "distributor_name",
    "Salesman": "salesman_code",
    "SalesmaneNama": "salesman_name",
    "INVNumber": "invoice_number",
    "INVDate": "invoice_date",
    "Outlet": "outlet_id",
    "Outlet Name": "outlet_name",
    "OutletAddress": "outlet_address",
    "SKUCode": "sku_code",
    "ProductName": "product_name",
    "Case": "case_qty",
    "Dozen": "dozen_qty",
    "Pieces": "piece_qty",
    "Weight": "weight",
    "TotalQuantity(PCS)": "total_quantity_pcs",
    "PriceCase": "price_case",
    "GSV": "gsv",
    "Discount": "discount",
    "Tax": "tax",
    "Net": "net_sales",
    "PERIOD": "period",
    "NOTES 1": "notes_1",
    "NOTES 2": "notes_2",
    "1000000": "gsv_millions",
    "PJP": "pjp",
}

df = df.rename(columns=column_mapping)
```

Now this:

```python
df["SalesmaneNama"]
```

stops assaulting our eyeballs.

---

# Step 4 — Convert dates

```python
df["invoice_date"] = pd.to_datetime(
    df["invoice_date"],
    dayfirst=True,
    errors="coerce"
)
```

Then validate:

```python
df["invoice_date"].isna().sum()
```

Expected:

```text
0
```

And:

```python
df["invoice_date"].min(), df["invoice_date"].max()
```

Should still give your known 2021 coverage.

This pattern matters:

> **transform → validate**

Every time.

Not:

> transform → assume → continue → discover catastrophe three notebooks later.

---

# Step 5 — Convert numeric fields

Now reuse your Indonesian-number parser.

Potential numeric columns include:

```python
numeric_columns = [
    "case_qty",
    "dozen_qty",
    "piece_qty",
    "weight",
    "total_quantity_pcs",
    "price_case",
    "gsv",
    "discount",
    "tax",
    "net_sales",
    "gsv_millions"
]
```

Then:

```python
for col in numeric_columns:
    df[col] = df[col].map(parse_indonesian_number)
```

Then inspect:

```python
df[numeric_columns].dtypes
```

and:

```python
df[numeric_columns].isna().sum()
```

Again:

**don't accept conversion silently.**

---

# Step 6 — Preserve identifiers as strings

These should **never** become floats:

```python
identifier_columns = [
    "distributor_id",
    "salesman_code",
    "invoice_number",
    "outlet_id",
    "sku_code",
]

for col in identifier_columns:
    df[col] = df[col].astype("string").str.strip()
```

Especially:

```text
outlet_id
invoice_number
sku_code
```

An ID is a label.

Not mathematics.

---

# Step 7 — Create an invoice reliability flag

This dataset gives us a beautiful example of why flags are useful.

Don't “fix”:

```text
2,1003E+11
```

We don't know the missing digits.

Instead:

```python
df["invoice_id_reliable"] = ~df["invoice_number"].str.contains(
    r"[Ee]\+",
    regex=True,
    na=False
)
```

Then:

```python
df["invoice_id_reliable"].value_counts()
```

Now downstream analysis can deliberately choose:

```python
df[df["invoice_id_reliable"]]
```

for metrics requiring trustworthy invoice numbers.

That's much better than either:

> use corrupted IDs anyway

or:

> delete half the dataset.

---

# Step 8 — Flag duplicates, but do NOT delete them yet

This is my recommendation given what you found.

Create:

```python
df["is_exact_duplicate"] = df.duplicated(
    subset=[col for col in df.columns if col != "is_exact_duplicate"],
    keep=False
)
```

But actually, since adding the flag changes the DataFrame, cleaner is to calculate before adding it:

```python
duplicate_mask = df.duplicated(keep=False)

df["is_exact_duplicate"] = duplicate_mask
```

Now:

```python
df["is_exact_duplicate"].value_counts()
```

should show roughly:

```text
False    10636
True        42
```

assuming 21 pairs.

### Why I don't want to delete them yet

Because you found:

- duplicates heavily concentrated among returns,
- many occurred on the same unusual date,
- multiple outlets are involved,
- each duplicated group occurs exactly twice.

That pattern is suspicious, but it is **not proof of accidental duplication**.

Could be batch processing.

Could be double-posted returns.

Could be legitimate duplicated lines.

Without stronger evidence, canonical cleaned data should retain them.

Later we can compare business metrics:

```text
Including exact duplicates
vs
Excluding exact duplicates
```

and see whether they materially affect conclusions.

That's defensible analysis.

---

# Step 9 — Classify transaction direction

Now we can create one of our first business-friendly fields.

Start simple:

```python
def classify_transaction(row):
    if row["total_quantity_pcs"] < 0 and row["net_sales"] < 0:
        return "return"
    elif row["total_quantity_pcs"] > 0 and row["net_sales"] > 0:
        return "sale"
    else:
        return "adjustment_or_mixed"

df["transaction_type"] = df.apply(classify_transaction, axis=1)
```

Then:

```python
df["transaction_type"].value_counts()
```

That third category is intentional.

If quantity and money disagree, I **don't** want us forcing the record into `SALE` or `RETURN`.

Those records deserve inspection.

---

# Step 10 — Create useful financial fields

These are legitimate derived variables:

```python
df["sales_after_discount"] = (
    df["gsv"] + df["discount"]
)
```

And:

```python
df["discount_rate"] = np.where(
    df["gsv"].abs() > 0,
    df["discount"].abs() / df["gsv"].abs(),
    np.nan
)
```

Notice the zero protection.

That's because you already discovered zero-GSV transactions.

This:

```python
discount / gsv
```

without protection is how one summons `inf` from the ninth circle of pandas hell.

---

# Step 11 — Don't normalize quantity units yet

You correctly listed this as unresolved:

```text
Case
Dozen
Pieces
TotalQuantity(PCS)
PriceCase
```

Leave them alone.

We already have:

```text
total_quantity_pcs
```

which appears intended as the normalized piece quantity.

Until we understand package-size semantics, we don't fabricate conversion logic.

That's another important lesson:

> **Not cleaning something can be the correct analytical decision.**

---

# Step 12 — Run validation checks

Before export:

```python
assert len(df) == 10678
```

Then:

```python
print("Rows:", len(df))
print("Date nulls:", df["invoice_date"].isna().sum())
print("SKU nulls:", df["sku_code"].isna().sum())
print("Net nulls:", df["net_sales"].isna().sum())
print("Exact duplicate rows:", df["is_exact_duplicate"].sum())
```

Financial check again:

```python
net_diff = (
    df["net_sales"]
    - (df["gsv"] + df["discount"] + df["tax"])
)

net_diff.abs().max()
```

Expected:

```text
< 1
```

That's basically a unit test for your cleaning pipeline.

---

# Step 13 — Export the cleaned dataset

Then finally:

```python
OUTPUT_PATH = Path("../data/processed/pasar_rau_sales_clean.csv")

df.to_csv(
    OUTPUT_PATH,
    index=False,
    encoding="utf-8"
)

OUTPUT_PATH
```

Because `data/processed/` is Git-ignored, the actual customer/business data still doesn't enter GitHub.

---

## There's one architectural decision I want us to make now

For this project, I recommend we **do not create one magically “perfect” dataset**.

Instead, our cleaned dataset retains business reality and gives questionable records flags:

```text
invoice_id_reliable
is_exact_duplicate
transaction_type
```

That lets future analyses explicitly say things like:

> “Invoice-level KPIs were calculated only from records with reliable invoice identifiers.”

or:

> “Return analysis includes all records, while sensitivity analysis was performed excluding exact duplicate pairs.”

That's much more professional than secretly deleting inconvenient rows until the dashboard looks pretty.

---

## Then comes the fun notebook

Once `02_data_cleaning.ipynb` works from top to bottom and generates:

```text
pasar_rau_sales_clean.csv
```

we move into:

```text
03_exploratory_analysis.ipynb
```

And that's where we stop talking primarily about **data quality** and start asking actual business questions:

> How much did we sell in 2021?

> Which months performed best?

> Which products drive revenue?

> How concentrated are sales?

> Which outlets matter most?

> What's happening with returns?

> How heavily are we discounting?

> Which salesperson performs well—and what does “performs well” actually mean?

At that point you're going to start seeing why all this supposedly boring profiling mattered. Without it, we'd have built revenue charts on dirty numbers, counted fake invoices, potentially deleted legitimate returns, and called ourselves “data-driven.”

Basically LinkedIn would approve. Reality would not.