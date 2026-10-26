# Investigating Accuracy in Data Presentation: the Fandango Star Ratings

Verifying Walt Hickey's 2015 FiveThirtyEight finding that Fandango inflated its displayed star ratings, then testing whether the behaviour changed afterwards. 146 films from 2015, 214 from 2016 and 2017.

Dataquest guided project, Data Scientist in Python path.

Two questions, two very different levels of confidence. The second one is where this notebook departs from the conventional answer.

## Question one: was Fandango inflating ratings in 2015? Yes, decisively.

The 2015 dataset contains both numbers: what Fandango displayed and what its own underlying rating was. The discrepancy is therefore directly measurable, with no cross-sample assumptions.

| | Count | Share |
|---|---|---|
| **Rounded up** | **130** | 89.0% |
| **Rounded down** | **0** | **0.0%** |
| Exact match | 16 | 11.0% |

Mean discrepancy **+0.2438 stars**, maximum +0.5, minimum 0.0. Three independent tests:

**1. The discrepancies are arithmetically impossible under nearest-rounding.** Rounding to the nearest half-star cannot move a value more than 0.25. **72 of 146 films (49.3%)** exceed that.

**2. The round-up rule reproduces the data; nearest-rounding does not.**

| Rule | Films reproduced |
|---|---|
| Round **up** to the next half-star | **133 of 146 (91.1%)** |
| Round to the **nearest** half-star | 74 of 146 (50.7%) |

**3. The asymmetry is not chance.** Of 117 films whose actual rating was not already on a half-star boundary, **117 rounded up and 0 rounded down**. Under a fair rule, by exact binomial:

**P = 6.0185e-36**

A film rated 4.0 by users was displayed as 4.5, on a five-point scale shown next to a buy-tickets button. That is not a rounding convention, it is a different number.

## Question two: did Fandango fix it? The data cannot say.

The conventional answer compares the 2015 and 2016 Fandango distributions, finds a drop of about 0.2 stars, and concludes the system changed. The raw comparison does show it:

| Year | n | Mean displayed rating |
|---|---|---|
| 2015 | 129 | 4.0853 |
| 2016 | 191 | 3.8874 |

A fall of 0.1978 stars, bootstrap CI [-0.314, -0.080], excluding zero. Taken alone it looks conclusive.

**It is not, because the two samples were built by different people using different rules.** Hickey's sample was 2015 films with 30+ Fandango reviews and an RT rating; the 2016-17 sample was films with "a significant number of votes", and that file records no Fandango vote count at all, so its rule cannot even be verified.

The other rating sites changed nothing between 2015 and 2016, which makes them a control group: whatever happens to their scores across the two samples measures the sampling difference alone.

| Site | Mean 2015 | Mean 2016 | Change |
|---|---|---|---|
| **Fandango** | 4.0853 | 3.8874 | **-0.1978** |
| Metacritic | 2.8403 | 2.6825 | -0.1578 |
| IMDB | 3.3419 | 3.2296 | -0.1123 |
| **RT audience** | 3.1415 | 2.9063 | **-0.2352** |
| | | **Control average** | **-0.1684** |

**Every site fell, and Rotten Tomatoes audience scores fell further than Fandango did.** The 2016 sample simply contains lower-rated films.

The difference-in-differences estimate, Fandango's change minus the control average:

| | Value |
|---|---|
| Difference-in-differences | **-0.0296 stars** |
| 95% bootstrap CI | **[-0.1895, 0.1317]** |
| Excludes zero | **no** |
| P(Fandango fell more than controls) | 0.6508 |

**About 85% of Fandango's apparent drop is explained by which films were sampled.** The Fandango-specific residual is 0.03 stars, with an interval spanning zero and a 0.65 probability that the sign is even negative, barely better than a coin flip.

Fandango may well have fixed it, and it publicly said it would. But this data does not establish it, and the 0.2-star headline is mostly an artefact of sample composition.

## A detail that cuts the same way

| Year | n | Share rated 4.5 or 5 | Minimum rating |
|---|---|---|---|
| 2015 | 129 | 45.0% | 3.0 |
| 2016 | 191 | **25.1%** | 2.5 |
| 2017 | 23 | **43.5%** | 3.0 |

If the rounding had been fixed in 2016, 2017 should resemble 2016. Instead **2017 reverts almost exactly to the 2015 pattern.** With 23 films that is weak evidence and is reported as a hint, not a result, but it points away from a permanent change.

Every displayed value in all three years sits on a half-star boundary, so display granularity never changed. What moved was where the mass sat, which is what a different set of films produces.

## The finding that survives regardless

Separately from rounding, Fandango sits **far above every other site in both years**: 4.09 against Metacritic's 2.84 in 2015, 3.89 against 2.68 in 2016, on the same 0 to 5 scale. That ~1.2-star gap barely moved.

This is a different phenomenon and should not be conflated with the rounding. A ticket-selling site whose reviewers already chose to buy a ticket will skew positive for ordinary selection reasons. Worth knowing when reading a Fandango score, and never what the 2015 article was about.

## Method

1. Measure the displayed-minus-actual discrepancy directly within the 2015 data.
2. Test it three ways: the arithmetic impossibility of discrepancies above 0.25, which candidate rule reproduces the displayed values, and an exact binomial test on the up/down asymmetry (hand-implemented, with sanity checks against known values).
3. Document the sampling-rule mismatch between the two datasets before comparing them.
4. Compare the raw Fandango distributions by year.
5. **Difference-in-differences** against Metacritic, IMDB and RT audience as controls, with bootstrapped confidence intervals on both the per-site changes and the DiD itself.
6. Check whether the displayed values still look like round-up output.

## Limitations

- The 2016-17 data has no underlying rating, only the displayed value, so the decisive test from question one cannot be repeated for 2016. The distribution comparison is a weak substitute.
- The DiD assumes the sampling difference affects all sites equally. Much weaker than assuming no sampling difference, which is the point, but not nothing: if the 2016 sample over-represents films where audience and critic opinion diverge, the control is imperfect.
- Neither sample is random, so neither supports inference to all films in its year. The bootstrap quantifies sampling variability; it cannot repair selection bias.
- 2017 has only 23 films, too few to carry the argument it illustrates.
- Only four rating sources were available. More controls would tighten the DiD, and the current [-0.19, 0.13] interval is wide enough to miss a real effect of plausible size.
- The causal reading rests on Fandango being the only thing that changed, which the data cannot verify.

## Stack

Python, pandas, NumPy, matplotlib. The binomial test and the bootstrap are both hand-implemented; no SciPy or scikit-learn.

## Files

- `notebook.ipynb`: the full analysis with executed outputs and figures.