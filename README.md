# A/B Testing: Do Discount and BOGO Offers Increase Purchase Conversion?

A 3-arm A/B test analysis in Python on a public marketing promotion dataset.

## Business question

Does sending a customer a **Discount** or a **Buy One Get One (BOGO)** offer increase the chance they buy, compared with sending **no offer**? And if both work, which one works better?

**Primary metric:** purchase conversion rate (customers who bought ÷ customers in the group).

## Data

- Source: Kaggle, *Marketing Promotion Campaign Uplift Modelling* — [https://www.kaggle.com/datasets/davinwijaya/customer-retention]
- 64,000 rows (raw), 9 columns: `recency`, `history`, `used_discount`, `used_bogo`, `zip_code`, `is_referral`, `channel`, `offer`, `conversion`.
- `offer` has three groups of roughly equal size: Discount, Buy One Get One, No Offer. `conversion` is 0/1.
- **The dataset is fictional (built for practice), not a real company experiment.** Results demonstrate the method, not real customer behaviour.

## Cleaning

- No missing values.
- 6,603 rows were exact copies of another row across all 9 columns. The dataset has no customer ID, so these could not be confirmed as true duplicates. I treated identical rows as duplicates and removed them: **64,000 raw → 57,397 after cleaning**. This is an assumption (see Limitations).
- **Sensitivity check:** the conclusions were the same with and without duplicate removal.
- Group sizes after cleaning: Discount 19,166, BOGO 19,159, No Offer 19,072.

## Method

1. **Randomization check.** Compared the three groups on customer information assumed to exist before the offer was sent (`recency`, `history`, `is_referral`, `used_discount`, `used_bogo`, `channel`, `zip_code`). Continuous columns were compared by mean and median, categorical columns by percentage share per group. See the balance table below.
2. **Chi-square test** on the offer × conversion table to test whether conversion differs anywhere across the three groups.
3. **Three pairwise two-proportion z-tests** to find which groups differ, in which direction, and by how much:
   - Discount vs No Offer
   - BOGO vs No Offer
   - Discount vs BOGO
4. **Holm correction** across the three z-tests (family-wise α = 0.05), because running several tests raises the chance of a false positive.
5. **95% confidence intervals** for each difference in conversion rate.

Tools: Python, pandas, SciPy, statsmodels, Jupyter.

## Randomization check (balance across groups)

| Column | Statistic | Buy One Get One | Discount | No Offer |
| --- | --- | --- | --- | --- |
| `recency` | mean (median) | 5.65 (5.0) | 5.69 (5.0) | 5.65 (5.0) |
| `history` | mean (median) | 267.22 (185.06) | 266.58 (182.74) | 265.48 (183.38) |
| `is_referral` | share = 1 | 50.4% | 50.2% | 50.1% |
| `used_discount` | share = 1 | 55.4% | 55.7% | 55.8% |
| `used_bogo` | share = 1 | 55.6% | 55.7% | 55.5% |


All checked columns are closely matched across the three groups, which is what random assignment should look like.

## Results

### Conversion rate by group

| Group | Customers | Purchased | Conversion rate |
| --- | --- | --- | --- |
| No Offer | 19,072 | 2,189 | 11.5% |
| Buy One Get One | 19,159 | 3,069 | 16.0% |
| Discount | 19,166 | 3,703 | 19.3% |

### Overall test

Chi-square = 449.9, df = 2, p < 0.001. Conversion rates are not the same across the three groups. This test does not say which group differs or in which direction, so pairwise tests follow.

### Pairwise comparisons

| Comparison | Difference (percentage points) | 95% CI | z | Holm-adjusted p |
| --- | --- | --- | --- | --- |
| Discount vs No Offer | +7.8 | 7.1 to 8.6 | 21.24 | < 0.001 |
| BOGO vs No Offer | +4.5 | 3.9 to 5.2 | 12.89 | < 0.001 |
| Discount vs BOGO | +3.3 | 2.5 to 4.1 | 8.47 | < 0.001 |

All three comparisons remain statistically significant after Holm correction, and every confidence interval is entirely above zero.

In relative terms: Discount converts about 68% higher than No Offer, BOGO about 40% higher than No Offer, and Discount about 21% higher than BOGO.

## Conclusion

In this dataset, both offers were associated with higher conversion than no offer, and Discount converted better than BOGO.

- Discount lifted conversion by about 7.8 points over No Offer.
- BOGO lifted conversion by about 4.5 points over No Offer.
- Discount outperformed BOGO by about 3.3 points.

Because the groups were balanced on pre-offer characteristics, the differences are consistent with an effect of the offers themselves.

**This does not mean Discount is the better business choice.** A discount costs margin, and this dataset has no information on cost or order value. A cheaper BOGO could still be more profitable.

## Limitations

- **Fictional data.** The dataset was created for practice, so the numbers do not describe real customers.
- **Duplicate rows.** There is no customer ID, so removing 6,603 identical rows is an assumption, not a confirmed fix. Results were the same with and without removal.
- **Conversion only.** There is no order value, revenue, or cost column, so profitability and average order value could not be analysed.
- **Randomization is inferred.** Balanced pre-offer characteristics support random assignment, but the dataset does not document how offers were assigned. It is also assumed that `used_discount` and `used_bogo` describe behaviour before the offer.
- **Short-term outcome.** Conversion is a single yes/no result. Repeat purchases and long-term retention are not measured.

## Repository contents

- `ab_testing.ipynb`: full analysis notebook
- `testing_data.csv`: dataset used in the analysis
- `requirements.txt`: Python dependencies
- `README.md`: this file
