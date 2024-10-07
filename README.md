# Enke and Shubatt (2023)
## Repository Structure

- **Jupyter Notebooks**:
  - `CP Confidence.ipynb`: Analysis related to confidence, using subject-level data.
  - `CP Confidence No Subject.ipynb`: Confidence analysis without subject-specific data.
  - `CP Risk.ipynb`: Risk analysis using subject-level data.
  - `CP Risk No Subject.ipynb`: Risk analysis without subject-specific data.
  - `Demeaned Confidence.ipynb`: Confidence analysis with demeaned data for statistical adjustments.
  - `Demeaned Risk.ipynb`: Risk analysis with demeaned data.
  - `Peterson.ipynb`: Analysis or replication related to a study by Peterson (details inside the notebook).
  - `Shubatt Replication.ipynb`: Replication of a study by Shubatt (details inside the notebook).

- **Stata `.do` File**:
  - `calc_index.do`: A Stata script to calculate complexity indices from [Quantifying Lottery Choice Complexity (Enke & Shubatt 2023)](https://benjamin-enke.com/pdf/Quant_complexity.pdf):
- **OPC**: Objective Problem Complexity
- **SPC**: Subjective Problem Complexity
- **OAC**: Objective Aggregation Complexity
- **SAC**: Subjective Aggregation Complexity
- **OLC**: Objective Lottery Complexity
- **SLC**: Subjective Lottery Complexity

Depending on the Input, the tool calculates these indices and saves them in the `output` folder, including the necessary features to obtain these indices. 


## Input 
The data I used are provided by [Peterson et al., 2021](https://www.science.org/doi/10.1126/science.abe2629), which could be found at [choice13k](https://github.com/jcpeterson/choices13k). 
- Max number of lotteries: 2
- Max number of states per lottery: 9
- Payout value is ignored if its probability is 0
- Payouts should be distinct in each lottery
- Problem ID is optional and should be indicated by the column name `problem`.
- CSV format 
- Not existing probabilities and states can be indicated by `,"",`, `,"NA",` or `,,` 
- Additional columns can be in the input data and will not be manipulated by code, as long as it does not match the pattern `x_`, `p_`, `_a_`, `_a`, `_b_`, `_b` and `cor_`

### Choice Complexity
If two lotteries are supplied, the column names should be as displayed in the table below. The columns indicating payouts should take the form `x_{l}_{i}`, where $l \in \{a,b\}$ indicates which lottery the payout belongs to, and $i$ indexes the lottery states. Similarly, the column names indicating probabilities should take the form `p_{l}_{i}`. The state index $i$ should take values between 1 and $k_{\{l\}}$, where $k_{\{l\}}$ is the maximum number of states in any of the $l$ lotteries. As written, the code can process a maximum of $k_{\{l\}} = 9$ distinct states.  If a payout is repeated in a lottery, they will be treated as separate states. To calculate the complexity-indices in [(Enke & Shubatt 2023)](https://benjamin-enke.com/pdf/Quant_complexity), we always first collapsed problems to treat same payouts as same state. If both lotteries take on a maximum of two states, then you would include the columns `x_a_1`, `x_a_2`, `p_a_1`, `p_a_2`, and similarly for $b$.


### Lottery Complexity
If only one lottery is supplied, the lottery column names can be either as displayed above (using only the `_a_` columns and no `_b_` columns). Alternatively, the `_a_` segment may be omitted, in which case payoff columns will take the form `x_{i}` and probability columns will take the form `p_{i}`. Again, $i$ will range between 0 and $k$, the largest number of distinct states in any lottery; the code can handle a maximum value of $k = 9$. For an example of the correct input format, see `sample_just_OLC_SLC_calculation_1.csv` or `sample_just_OLC_SLC_calculation_2.csv` in the `sample_data` folder.

## Output
The results will be saved in `output` with `index_calculated_stata.csv` depending on which script you run, including features which are necessary for the calculations. (Additionally `.dta`, are saved, depending on the executed script).

### Choice Complexity
If two lotteries are supplied as above, all 6 indices are automatically calculated. The results are ordered in the CSV as follows: Problem [Optional] | Supplied probabilities and payouts | OPC | SPC | OAC | SAC | OLC_a | SLC_a |  OLC_a | SLC_a | then in the same order of the indices the necessary features for their calculations| and in the end any other columns which were also in the input data but not used by the code |. The endings `_a` or `_b` at `OLC` and `SLC` indicate to which lottery the lottery complexity index is referring. `compound` is optional. 

The features for each index are the following. Please see `Section 4` for the development of the indices and appendix *Potential Complexity Features* for details about feature definition in [Quantifying Lottery Choice Complexity](https://benjamin-enke.com/pdf/Quant_complexity.pdf). <br>
Consider a choice between two lotteries indexed by $j$ and denoted by letters $A$ and $B$. Each lottery is characterized by payout probabilities $(p_1^j,...p^j_{k_j})$ and payoff $(x_1^j,...x^j_{k_j})$ where $k_j$ denotes the number of distinct payout states of lottery $j$.

#### Features of `OPC` and `SPC`
- <u>Log excess dissimilarity (`ln_excess_dissimilarity`):</u><br>
When $F_A(x)$ and $F_B(x)$ are the CDFs of Lottery A and B with EV(.) indicating the expected value of a lottery then *Log excess dissimilarity* is defined as 

$$log\Big( 1+\int_\mathbb{R} |F_A(x) - F_B(x)|dx - |EV(A) - EV(B)|\Big) $$

- <u>No dominance (`no_dominance`):</u><br>
$$\exists x_1 , x_2: F_A(x_1) < F_B(x_1) \land F_A(x_2) >F_B(x_2)  $$
- <u>Average log payout magnitude (`ave_ln_payout_magn`):</u><br>
$$\frac{1}{2} \Big [ log \Big(1 +  1/k_A \sum_{s=1}^{k_A} |x_s^A| \Big) + log \Big (1 + 1/k_B \sum_{s=1}^{k_B} |x_s^B|) \Big) \Big ]$$
- <u>Average log number of states (`ave_ln_num_states_a`):</u><br>
$$\frac{log(1 + k_A) + log(1 + k_B)}{2}$$
- <u>Frac. lotteries involving loss (`frac_involves_losses`)</u><br>
- <u>If one lottery in the choice is compound (according to definition [above](#Compound))</u><br>
- <u>Absolute expected value difference (`abs_ev_diff`):</u><br>
$$|EV(A) - EV(B)|$$
- <u>Absolute expected value difference squared (`abs_ev_diff_sq`):</u><br>
$$|EV(A) - EV(B)|^2$$

#### Features of `OAC` and `SAC`
As above but without `abs_ev_diff` and `abs_ev_diff_sq` features.

### Lottery Complexity

If  just one lottery is supplied the lottery complexity is calculated (OLC/SLC). In principle, the ordering of the output is the same as for the [Choice Complexity](#Choice-Complexity) output. However, payouts and probabilities are now named **x_1, x_2, ... p_1, p_2**. Additionally, the features and indices don't have the appendix `_a` as just one lottery is in the dataset.


#### Features of `OLC_a/b` and `SLC_a/b`
The following defines features for both lotteries indicated with $j\in \{A,B \}$.<br>

- <u>Log Variance (`ln_variance_a/b`):</u><br>
$$log\Big ( 1+  \sum_{s=1}^{k_j} p_s^j(x_i^j)^2 -( \sum_{s=1}^{k_j} p_s^jx_s^j)^2 \Big)$$

- <u>Log payout magnitude (`ln_payout_magn_a/b`):</u><BR>
$$log\Big( 1 + 1/k_j\sum_{s=1}^{k_j}|x_s^j|  \Big)$$

		

- <u>Log number of states (`ln_num_states_a/b`):</u><br>

$$ log \Big ( 1 + k_j \Big )$$

- <u>1 if involves loss (`involves_loss_a/b`)</u><br>
- <u>1 if involves compound probability(`compound`)</u><br> 


### Running the Python Script
 See `Shubatt.ipynb` for the replication code.
