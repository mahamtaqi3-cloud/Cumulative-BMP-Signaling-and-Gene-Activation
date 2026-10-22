# Cumulative BMP Signaling and Gene Activation

A small simulation and machine-learning project on how cells in an early embryo might decide when to switch on target genes in response to a rapidly changing signal.

> **All data in this project are simulated.** The model explores the logic of an idea and does not test the underlying biology.

## Background

In the early *Drosophila* embryo, BMP signaling starts out broad and sharpens toward the dorsal midline within a short time window. Cells have to respond to this changing signal quickly.

One idea is that a cell does not react to the signal level at a single moment, but to the **total signal it has received so far**. A second idea is that this rule may not be enough on its own, and that position-dependent factors could also restrict where genes switch on.

## Question

Can a simple classifier predict whether a nucleus has switched on a gene better from the **total signal received so far** or from the **signal at that moment**? And does a rule learned in a normal gradient still work when something else restricts expression far from the midline?

## Approach

1. **Setup:** one side of the dorsal region, with 13 positions from the midline out to 60 µm (5 µm steps), followed over 40 minutes (1-minute steps).
2. **Simulated signal:** two bell-shaped profiles centered on the midline.
   - *Normal:* the signal rises over time and the gradient narrows (peak about 0.04, arbitrary units).
   - *sog-like:* the signal rises weakly and stays broad (peak about 0.012).
3. **Lateral restriction term:** a position-dependent factor that reduces responsiveness beyond about 45 µm. It is a generic stand-in for any factor that restricts expression laterally, not a model of a specific molecule.
4. **Simulated nuclei:** about 40 nuclei per position. Each has a random sensitivity and switches on once its accumulated signal passes it, so the chance of being on depends on the total signal received so far. Each nucleus is labeled on or off at every minute (about 21,000 records).
5. **Train and test:** logistic regression is trained on 70% of the records and tested on 30%, once with instantaneous signal and once with total (cumulative) signal.
6. **Transfer test:** a model trained on the normal case is tested on the *sog*-like case with the restriction term.

## Results

Results from one run (random seed 0). Exact values change slightly with different seeds.

| Test | Accuracy | AUC |
|---|---|---|
| Normal, instantaneous signal | ~0.81 | ~0.89 |
| Normal, total signal | ~0.87 | ~0.96 |
| Train on normal, test on *sog*-like with restriction | ~0.80 | ~0.91 |

- Total signal predicts activation better than instantaneous signal.
- A rule learned in the normal case transfers only partly when a position-dependent restriction is present. The drop is modest.

## How to run

1. Open the notebook in [Google Colab](https://colab.research.google.com/) (File → Upload notebook).
2. Run the cells from top to bottom.

**Requirements:** Python 3, `numpy`, `scikit-learn`. All are available by default in Colab.

## Notebook structure

| Cell | Section |
|---|---|
| 1 | Setup: positions and time |
| 2 | BMP signal: normal vs *sog*-like |
| 3 | Lateral restriction term |
| 4 | Simulated nuclei |
| 5 | Train and test |
| 6 | Transfer test |

## Limitations

- **Simulated data only.** Signal shapes, peak heights, firing strengths and the 45 µm boundary are my own choices, so absolute numbers have no biological meaning. Only the patterns matter.
- **The main result is partly built in.** Genes are simulated to respond to accumulated signal, so total signal was expected to win. The project shows the logic is consistent, not that embryos work this way.
- **Simplified signal shapes.** For example, lateral signals in the normal case decline late in the simulation, and the *sog*-like gradient is broader than a real one would be.
- **Different firing strengths** are used for the two cases (60 and 150), so absolute values should not be compared across cases.
- **Generic restriction term.** Other boundary positions were not tested.
- **Simple model.** Logistic regression is a basic classifier, chosen so the comparison is about the signals and not the model.
- **No noise or small-sample effects** are modeled.

## Possible extensions

- Test other boundary positions for the restriction term.
- Add measurement noise and small sample sizes.
- Add position as a feature and check whether the model recovers the restriction.
- Compare with a nonlinear model such as a random forest.
- Replace simulated signals with real experimental measurements.
