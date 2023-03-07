# Tacobell

## Constructed Variables
- `postX`

> A single discrete variable telling us how many months the restaurant has observed data in the post-implementation period. For the time period, the left bound is the month following implementation and the right bound is 24 months later, such that the maximum value `postX` can possess is `24`. Wash-out months in the post-implementation period are included in the calculation of postX.

- `openX`

>

- `weight1`

> Weight assigned by the matching process. For synthetic controls, `weight1` = 1.

- `weight2`

> $$weight2 = \frac{\text{restaurant's total transactions}}{\text{total transactions from all restaurants}} \times \text{N of restaurants}$$
> Restaurant-specific weight calculated as the proportion of transactions contributed by restaurant between relative months -3 and -12 (inclusive on both ends), centered around 1.0. **Note** that this is sample- and sub-sample specific, so `weight2` should be generated every time the sample changes. For the within-CA paper, weight2 is also treatment-specific.

- `regression_index`

> Uniquely identifies each restaurant or in within-CA's case, each treatment-control pair. `regression_index` is supplied in `plm`'s "index" option.

- `matching_identifier`

>

- `id_match`

>

- `id`

>

- `entry`

> Implementation monthno. Look [here](https://github.com/Brian-Elbel-s-Research-Projects/menu-labeling-impact-on-calories-drive-through/issues/1).

- `relative2.factor`

> (`monthno` - `entry`) as a factor variable; a value of 0 for `relative2.factor` therefore represents the implementation month.

- `policy`

> Identifies the larger geographic area that identifies a labelled/potentially labelled/reverted location. Typically at the state-level.

- `match_place`

> Similar to `policy` except control units are also assigned their matched unit's `policy`.

- `pre_mean`; `post_mean`

> `pre_mean` represents the mean of the differenced coefficients from the `plm` model between months -3 and -8.  
> <br>
> `post_mean` represents the same, except for either the first (months 3 to 12) or second year (months 13 to 24).  
> <br>
> These variables are used to calculate the first and second year DiD estimates.

- slopes and lags

> Slope and lag variables are calculated for transaction-related variables including `count`, `count_all`, `dollar`, `dollar_all`, `calorie`, and `calorie_all`.  
> <br>
> Slopes are constructed from months -3 to -8 (to check).  
> <br>
> Lags represent the value observed 3 months ago.

## Code, processes
- standardize color scheme
- felm with robust s.e.
- write objects to excel directly
- how did is estimated
- robustness checks
- group == 2 for orangey liney
- trend
- colors
- matching

## Miscellaneous
- mounting g drive
- standard checks when loading in a dataset
