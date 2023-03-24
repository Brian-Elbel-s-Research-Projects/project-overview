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
**- standardize color scheme**
  - Borne from a now closed team issue [link](https://github.com/Brian-Elbel-s-Research-Projects/project-overview/issues/61), Lloyd developed a custom color scheme for the team to use across projects and scripts, [linked here](https://nyulangone-my.sharepoint.com/personal/lloyd_heng_nyulangone_org/_layouts/15/onedrive.aspx?id=%2Fpersonal%2Flloyd%5Fheng%5Fnyulangone%5Forg%2FDocuments%2Fcolors%2Ehtml&parent=%2Fpersonal%2Flloyd%5Fheng%5Fnyulangone%5Forg%2FDocuments&ga=1). We use the L color scheme at the bottom, copied here
 ```
palette = c("#007fff", #azure
            "#00dea4", #asda green
            "#ff2052", #awesome red
            "#fbec5d", #buff mustard
            "#ffc1cc", #bubble gum
            "#ffb95a", #cape jasmine orange
            "#141414") #chinese black
```
**- felm() with robust s.e.**
  - `plm()` does not produce a robust variance-covariance matrix nor robust standard errors for estimating confidence intervals, and a basic heteroskedasticity test reveals a problem with several menu labeling analyses that likely affect other projects as well. Therefore, we switched from `plm()` estimation of the DiD to `felm()` using the same design; this required some testing and troubleshooting, but the point estimates are identical, and the SEs enlarge as expected. What follows is detail on the transition and solution to the troubleshooting.
  - If you're doing something like `calories = treat*relative + as.factor(month)` with restaurant fixed effects as a specified index, it is likely that you will get collinearity between `month` and `relative`, and you may still get a NaN result for the `treatment` variable from `felm()` even after removing the `+ as.factor(month)` term as mentioned below. A NaN result for `treatment` will continue to work for any `tidy()` procedures but break a forloop for significance testing, e.g. using `linearHypothesis` against the `presum`. 
  - Lloyd's solution to the collinearity problem was to drop the `+ factor(month)`, and also specifies the `regression_index` as a factor in the model instead. The point estimates are still identical to previously.
  - Another problem that you may run into is incorporating the weights; the felm() argument wasn't able to locate the data object's weights itself even though the data is specified in the options — to get around this, just supply the object and its weight(s) — aka something like weight = the_data$weight2.
    - This may throw a length error, though — if you shorten the data with a filter within the `data = ` argument, it will be shorter than the data supplied in the `weights = ` argument. Get around this by just filtering the whole object before feeding it into the felm(). 
  - You may also need to adjust your tidying on the mod.factor object; see below. I clean out the treatment coefficient itself, and the intercept (as it was appearing in some iterations of my testing various model specifications when I couldn't get this all to work). Make sure you specify se.type in the tidy(mod.factor) command, I think.
  - To be thorough, consider replacing not only the first index in the coefficient list but also the beta list within mod.factor and the first row&column in the vcv matrix.
  - We've also adjusted the indexing on the hypothesis testing loop, detailed in an issue [here](https://github.com/Brian-Elbel-s-Research-Projects/project-overview/issues/78)
  -  After implementing felm() be sure to include the tidy() options it gets used in the standard error computations, as so: `tidy(mod.factor,conf.level = 0.95,conf.int = TRUE, se.type = "robust")`.
  -  Examples can be found in the by-location analyses and intra-california analysis as of 3/22/23 time of writing.

**- Write objects to Excel directly**
  - Examples can be found in the by-location analyses and intra-California analysis as of 3/22/23 time of writing. 
  - Essentially, create the object/image to be written directly. 
  - Then, data objects can be used to fill in charts by specifying the object and the target location (sheet, row, column of starting) and by pre-formatting the object so it fills in the Excel sheet properly. Note that Excel formatting decisions (e.g., on data type = number, or general, or date) will be kept. 
  - Similarly images can be produced and saved to a specific sheet and location, with slightly different commands.
  - Procedure: 
    - Define an export path just by specifying the string e.g. `table_shells <- "the_file_path_to_the_destination_shells"`, load the Excel workbook with `wb = loadWorkbook("the_file_path")`, save the object with `insertImage()` or `writeData()` referring to the `wb` object, then save the workbook edits with `saveWorkbook()`

**- How did is estimated**
  -  A subset of the data is selected based on the matching identifier and the matching place. 
  - The relative2.factor variable is used as a categorical covariate, and the reference category is set to "-3".
  - A fixed-effects linear regression model is fit to the data, with e.g. `calorie` as the dependent variable, and `treat` and `relative2.factor` as independent variables. The weights are the product of the two weight variables.
  - The estimated coefficients and their standard errors are extracted from the model, and a table is created. The table includes the coefficient estimate, p-value, and confidence interval for each covariate in the model. The table is filtered to exclude the reference categories, the intercept, and the treat variable. The table is also modified to include rows with zeros for the "-3" level of the `relative2.factor` variable. This month is the beginning of wash-out. Values are measured with respect to this reference month, so monthly point estimates refer to the deviation in average calories purchased (for that month-treatment_group) from month -3. 
  - The DiD estimator is computed by calculating the difference in the mean changes in average calories purchased between the treated and control groups before and after the treatment. Separately for the first versus the second year, a treatment and a comparison object is made: the pre-treatment coefficient is the mean of the monthly coefficients from -8 to -3 and the post-treatment coefficient is the mean of the monthly coefficients after implementation (3 to 12 for first year, 13 to 24 for second year). The difference between them gives the difference between pre- and post-treatment. The equivalent is conducted for the pre- and post-control values and the difference taken between these two treated and control objects for the appropriate year gives the DiD. 
**- Robustness checks**
  - We run some combination of open time, different baseline, macronutrient, daypart/time of day, new menu item, top selling items, overall quantity of sales and per-food-item-category quantity of sales follow-up analyses. Internal consistency between these robustness checks is important as a procedure to both validate and detail the main effect analyses. Quantity is not an equivalently direct measure of calorie changes and may not reflect clearly the rest of the analsyes.

**- Group == 2 for orangey liney**
  - Be sure to incorporate a clear delineation for the difference in pre and post values in the trend object so that they can be plotted correctly; this can be accomplished by assigning the pre-post differenced values their own group (and group==0 and group==1 are already used, so this can be done with group==2).

**- trend**
  - This object is part of the chunk for creating DiD plots; the trend object is initialized as an empty object, then the DiD analysis is conducted. The `tmp1`, `tmp2`, and `tmp` objects are used to hold the (baseline and endline separately) point estimates of the monthly average calorie changes differenced from the reference month; the tmp1 and tmp2 objects are combined into a single object which contains the pre mean, post mean, and their difference; this is then significance-tested in a forloop using the `tmp` object; the p-values in `tmp` are merged onto the respective `tmp1` object's months.


**- Matching**
  - We use a 5:1 Mahalanobis match with replacement as pre-processing to trim the pool of potential matches, followed by a 3:1 without replacement CBPS match to finalize the dataset. This is conducted only for the California dataset since the smaller locations (with n=1, n=3, and one n=16) are not expected to produce effective matching. Synthetic Controls for California are scheduled to be tested, as of 3/22/23, so the Mahalanobis+CBPS approach is being phased out after Paper One.

## Miscellaneous
**- mounting g drive**
  - For security, rather than posting the location, it can be retrieved [here](https://nyulangone-my.sharepoint.com/:u:/g/personal/emil_hafeez_nyulangone_org/ETT_Bhn7evhDvgd81eYG3w4BM7Y_pCPfBvUL2a82l7VOXg?e=pvjQEb)
  - 
**- standard checks when loading in a dataset**
