# Cumulative BMP Signaling and Gene Activation

A small simulation and machine-learning project on how cells in the early *Drosophila* embryo might decide when to switch on BMP target genes.

> **All data in this project are simulated.** The model checks the internal logic of published claims and does not test the underlying biology.

## Background

During dorsal patterning of the *Drosophila* embryo, BMP signaling starts out broad and sharpens toward the dorsal midline within about 30-40 minutes. Cells must interpret this changing signal quickly.

This project is based on the paper:

> Brantley, S.E., Janssen, J., Chao, A., Vergassola, M., Blythe, S.A., and Di Talia, S. (2026). Rapid transcriptional response to a dynamic morphogen by time integration. *Current Biology* 36, 2853-2865. https://doi.org/10.1016/j.cub.2026.04.064

The paper makes two main claims, which this project explores:

1. Nuclei time the activation of BMP target genes according to **time-integrated (cumulative) BMP signaling** rather than the instantaneous level.
2. This rule breaks down in lateral regions of *sog* mutants, which suggests that additional spatial regulation restricts gene expression there.

This is an independent educational project and is not affiliated with the authors.

## Question

Can a simple classifier predict whether a nucleus has switched on a gene better from the **total signal received so far** or from the **signal at that moment**? And does a rule learned in normal embryos still work when something else restricts expression far from the midline?

## Approach

1. **Setup:** one side of the dorsal region, with 13 positions from the midline out to 60 µm (5 µm steps), followed from minute 20 to minute 60 after anaphase of nuclear cycle 13 (1-minute steps).
2. **Simulated BMP signal:** two bell-shaped profiles centered on the midline.
   - *Normal:* the signal rises over time and the gradient narrows (peak about 0.04, arbitrary units).
   - *sog-like:* the signal rises weakly and stays broad (peak about 0.012).
3. **Lateral restriction term:** a position-dependent factor that reduces responsiveness beyond about 45 µm. It stands in for whatever restricts expression laterally (for example low Zen or repressors such as Brinker). It is not a model of any specific factor.
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

1. Open the notebook in [Google Colab](https://colab.research.google.com/) (File → Upload notebook, or open from GitHub).
2. Run the cells from top to bottom.

**Requirements:** Python 3, `numpy`, `scikit-learn`. All three are available by default in Colab.

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

- **Simulated data only.** Signal shapes, peak heights, firing strengths and the 45 µm boundary are my own choices, so the absolute numbers have no biological meaning. Only the patterns matter.
- **The main result is partly built in.** Genes are simulated to respond to accumulated signal, so total signal was expected to win. The project shows the logic is consistent, not that embryos work this way.
- **Signal shapes are only qualitatively similar to the paper's.** For example, lateral signals in the normal case decline late in the simulation, whereas the paper's data plateau, and the *sog*-like gradient is broader than the measured one.
- **Different firing strengths** are used for the two cases (60 and 150), so absolute values should not be compared across cases.
- **Restriction term is generic.** It does not model Zen or Brinker specifically, and other boundary positions were not tested.
- **Simple model.** Logistic regression is a basic classifier, chosen so the comparison is about the signals and not the model.
- **No noise or small-sample effects** are modeled, although some live-imaging experiments in the paper use very few embryos.

## Possible extensions

- Test other boundary positions for the restriction term.
- Add measurement noise and small sample sizes.
- Add position as a feature and check whether the model recovers the restriction.
- Compare with a nonlinear model such as a random forest.
- Replace simulated signals with real nuclear Medea measurements. The authors report that their segmented nuclei data and analysis code are available at https://github.com/susanna3/GFPMedDynamics.

