The clean mental model is:

> **Observation = What happened?**  
> **Interpretation = Why might it matter / what could explain it?**  
> **Business implication = So what should the business care about or do next?**

That’s the whole skeleton.

The trick is to **not let those three bleed into each other**, because that’s where people start turning one chart into a fanfic.

## 1. Observation = only what the data directly shows

Think:

> “If someone challenged me, can I point to a table/chart and prove this sentence?”

Good observation:

> Net sales were highest in October and lowest in February.

Better:

> October recorded the highest net sales of the year, while February recorded the lowest.

Bad:

> October performed best because customers were more active.

Why bad? Because **“because customers were more active”** is already an explanation. The chart may not prove that.

A reusable pattern:

```text
[Metric] was [higher/lower/largest/smallest] for [segment/time/product],
compared with [comparison point].
```

Or:

```text
[Segment A] contributed X% of [metric], while [Segment B] contributed Y%.
```

Or:

```text
[Metric] increased/decreased by X% between [period A] and [period B].
```

The strongest observations usually contain **comparison**, not just a number.

Weak:

> Product A generated Rp500 million.

Better:

> Product A generated Rp500 million in net sales, the highest among all products and 35% more than Product B.

Now the number has context.

---

# 2. Interpretation = what the pattern could mean

Now ask:

> **Why is this interesting?**

and:

> **What plausible explanation fits what I see?**

But be careful with certainty.

Use language like:

- may indicate
- suggests
- appears consistent with
- could be related to
- warrants further investigation

Instead of:

- proves
- caused
- definitely happened because
- therefore X caused Y

For example:

### Observation

> December had relatively high gross sales but also the highest return value.

### Interpretation

> This suggests that strong December selling activity was partially offset by unusually high returns.

That's defensible because you're connecting two measured things.

This would be weaker:

> Customers bought too many products for Christmas and returned them afterward.

Maybe. Maybe not. Cute story, zero evidence. Straight to analyst jail.

A reusable pattern:

```text
This suggests that [business phenomenon] may be associated with [observed pattern].
```

Or:

```text
The difference may reflect [factor A], [factor B], or [factor C];
additional analysis would be needed to identify the main driver.
```

That second one is incredibly useful when you're uncertain.

---

# 3. Business implication = why anyone should give a damn

This is the **“so what?”** part.

Ask:

> If I'm a manager reading this, what decision does this affect?

Possible implications usually fall into a few buckets:

- investigate something
- monitor something
- prioritize something
- reduce something
- protect something
- replicate something successful
- allocate resources differently

For example:

### Observation

> Five outlets contribute 48% of total net sales.

### Interpretation

> Sales are highly concentrated among a small group of outlets.

### Business implication

> Losing or reducing activity from one of these outlets could materially affect revenue, so management may want to monitor key-account retention and diversify sales across additional outlets.

Now that's an insight.

The chart alone was merely:

> “Top 5 outlets big.”

The implication turns it into something a business could care about.

---

# The framework I want you to memorize

Whenever you see an interesting result, mentally run:

```text
1. WHAT?
What exactly happened?

2. COMPARED TO WHAT?
Is it high, low, growing, declining, unusual?

3. WHY MIGHT THAT MATTER?
What reasonable explanation or business meaning could exist?

4. SO WHAT?
What decision, risk, opportunity, or further investigation follows?
```

You could honestly put this beside your monitor.

## Example from your monthly section

Suppose you calculate:

```text
Jan:  Rp120M
Feb:   Rp95M
Mar:  Rp130M
...
Oct:  Rp180M
```

Don't immediately write:

> October sales were Rp180M.

Instead:

### Observation

> October recorded the highest net sales of 2021 at Rp180 million, approximately 38% above the monthly average.

### Interpretation

> The unusually strong October performance suggests either higher transaction volume, a stronger product mix, increased outlet activity, or a combination of these factors.

### Business implication

> October should be investigated further to determine which products, outlets, or salespeople drove the increase. If the drivers are repeatable rather than seasonal, they could inform future sales planning.

Notice something sneaky?

Your implication also created the **next analysis question**.

That's very common.

Good analysis is basically:

```text
Answer
↓
new question
↓
answer
↓
new question
```

until eventually somebody makes a decision or runs out of budget.

---

# Example: product analysis

Imagine:

```text
Product A:
Net Sales = #1
Quantity = #14
```

### Observation

> Product A generated the highest net sales despite ranking only 14th by units sold.

### Interpretation

> Its revenue performance appears to be driven more by unit value than by sales volume.

### Business implication

> Product A may be strategically important for revenue generation, so management should monitor its availability and avoid evaluating product importance based only on unit volume.

That’s a damn good portfolio insight because you're showing:

> volume ≠ value.

---

# Example: outlet analysis

Suppose your top 10 outlets generate 70% of sales.

### Observation

> The top 10 outlets account for approximately 70% of total net sales.

### Interpretation

> Revenue is concentrated among a relatively small group of customers.

### Business implication

> This concentration creates both an opportunity and a risk: maintaining relationships with these outlets is commercially important, while expanding sales among smaller outlets could reduce dependence on a limited customer group.

Look at the layers:

**Data fact:**

> 70%.

**Meaning:**

> concentration.

**Business consequence:**

> retention risk + diversification opportunity.

That's what recruiters want to see.

---

# Example: salesperson analysis

Suppose:

```text
Salesperson A = highest total sales
Salesperson B = lower total sales
Salesperson B = highest sales per outlet
```

### Observation

> Salesperson A generated the highest total net sales, while Salesperson B generated the highest net sales per outlet.

### Interpretation

> The ranking changes depending on whether performance is measured by absolute sales or outlet productivity.

### Business implication

> Salesperson performance should not be evaluated solely on total revenue; portfolio size and productivity per outlet should also be considered.

This one is especially good.

You're basically saying:

> Please don't fire Bob just because Alice has twice as many accounts, you absolute spreadsheet goblin.

---

# Example: returns

Suppose one SKU has very high return value.

### Observation

> SKU X had the largest return value among all products and represented 18% of total returned sales value.

### Interpretation

> Returns appear disproportionately concentrated in this product rather than evenly distributed across the portfolio.

### Business implication

> SKU X should be investigated for possible product-quality, ordering, fulfillment, or customer-fit issues before assuming the returns are normal.

Notice that we **did not say**:

> Product quality is bad.

We don't know that.

We give plausible hypotheses:

```text
quality
ordering
fulfillment
customer fit
```

Then recommend investigation.

Very defensible.

---

# Example: discounts

Suppose heavily discounted products also have high quantities.

Don't write:

### ❌ Bad

> High discounts caused customers to purchase more.

Your data probably cannot prove causality.

Instead:

### Observation

> Transactions with higher discount rates also tended to have higher quantities.

### Interpretation

> Higher discounts are associated with larger transaction volumes, although the current analysis does not establish whether discounts caused the increase.

### Business implication

> Management may want to evaluate whether discounted transactions generate sufficient incremental volume to justify the reduced gross sales value.

That's analyst language.

You see the pattern?

**Observe → explain carefully → connect to a decision.**

---

# A very useful test: “Could I swap these sentences?”

Usually no.

If your “Observation” contains words like:

> because  
> likely due to  
> caused by  
> indicates customers wanted

you're probably interpreting.

If your “Interpretation” says:

> Management should...

that's probably an implication.

If your “Business implication” merely repeats:

> October had the highest sales.

you haven't answered **so what?**

---

# I also like using this five-level ladder

Think of an analytical result as:

```text
DATA
↓
OBSERVATION
↓
INTERPRETATION
↓
IMPLICATION
↓
ACTION
```

Example:

### Data

```text
Outlet A: 17%
Outlet B: 13%
Outlet C: 11%
```

### Observation

> The top three outlets contribute 41% of sales.

### Interpretation

> Revenue is concentrated in a small number of customers.

### Implication

> The business may be exposed to customer-concentration risk.

### Action

> Monitor retention of these accounts and identify opportunities to grow mid-tier outlets.

You don't always need to recommend an action. Sometimes your action should simply be:

> Investigate further.

That's totally valid.

---

# Another powerful question: “What changed?”

Most useful insights come from one of six patterns.

When you're staring at a table/chart, look for:

| Pattern | Question |
|---|---|
| **Ranking** | Who is highest/lowest? |
| **Trend** | Is something increasing/decreasing? |
| **Difference** | Why is A different from B? |
| **Concentration** | Does a small group dominate? |
| **Outlier** | What behaves unusually? |
| **Relationship** | Do two variables move together? |

If you're stuck, scan for those six.

For example:

### Monthly sales

Look for **trend/outlier**.

### Products

Look for **ranking/concentration**.

### Outlets

Look for **ranking/concentration**.

### Salespeople

Look for **difference**.

### Discounts

Look for **relationship**.

### Returns

Look for **outliers/concentration/trend**.

You now have a reusable lens for basically any dataset.

---

# Your writing should also get progressively shorter

Notebook exploration might contain:

> October recorded net sales of RpX, which was Y% higher than September and Z% above the monthly average.

Dashboard version:

> **October was the strongest sales month, X% above average.**

Executive-summary version:

> **Sales peaked in October.**

Same insight, different compression.

That's another skill we'll practice later.

---

## One final rule

Don't force every chart to produce a profound business recommendation.

Sometimes the correct answer is:

> **No meaningful pattern was observed.**

or:

> **The available data is insufficient to determine the cause.**

That is infinitely better than manufacturing some MBA horoscope like:

> “Synergies indicate an opportunity to leverage consumer engagement.”

I would personally throw the laptop out the window.

For this project, use this exact mini-template whenever you find something worth discussing:

```markdown id="ajvhjg"
### Finding: [short descriptive title]

**Observation:**  
What does the data directly show? Include comparison or magnitude where useful.

**Interpretation:**  
What might explain the pattern, and how confident are we?

**Business implication:**  
Why does this matter? What risk, opportunity, decision, or next investigation follows?
```

And before you accept your own finding, ask yourself:

> **Can I prove the observation? Am I overstating the interpretation? Does the implication actually follow?**

If all three answers are good, you've got an actual analytical insight rather than a caption under a chart.