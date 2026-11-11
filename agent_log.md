# Agent log (edited extract)

How the work with the AI coding agent (Claude Code) went, in order.
For each step: what I asked, what the agent proposed or did, and what I decided. Quotes in *italics* are my
messages verbatim (typos kept); the rest is paraphrased. Steps 1–4 come from earlier sessions, which survive
only as the agent's own summary, so they are paraphrased throughout. Code, shell output and messages about the
computing environment are left out.

---

### 1. Why are contaminants reconstructed as UCDs?
- **I asked** whether some candidates were UCDs or AGN, why the autoencoder reconstructs them, and stated the
  constraint: purely spectroscopic, no colours, band ratios or photometric redshifts.
- **Agent:** proposed extra spectroscopic statistics (χ², runs test on the residuals). Contamination remained,
  and **I asked** whether the contaminants could be added to the fine-tuning as negative examples, and for a
  separate notebook to study them (dimensionality reduction, fine-tuning with negatives, other ideas).
- **Agent:** formulated the negatives as a margin (hinge) term, so contaminants are pushed above the selection
  threshold and then left alone, and proposed a classifier as an intermediate step to get labels.

### 2. Labelling the gold sample
- **I asked** for interactive labelling with buttons.
- **Agent:** built the labelling widget (spectrum, reconstruction, nearest known UCDs, back/skip, label sources),
  random and ordered batches, a review pass for `uncertain`, HDBSCAN on UMAP/PCA projections to propose
  clusters to label, and a gradient-boosting classifier with a fixed test split.
- **Agent error:** the first threshold rule (95th percentile of the CV probabilities of UCDs) jumped from
  0.25 to 0.6 when I added two labels. The agent replaced it with recall/rejection-vs-threshold curves and a
  threshold on their plateau.
- **I decided** to label the whole gold sample by eye, then bulk-labelled the rest.

### 3. From gold to the wider (main) sample
- **I asked:** *"Great, I have labelled all the gold sample. Let's create a notebook now with two main sections..."*.
- **Agent:** built the notebook. Applied to main, contaminants were still predicted as UCDs.
- **Agent diagnosis:** distribution shift. The main-only objects failed the gold cuts, so they lie outside the
  range of the training features, and trees do not extrapolate. Proposed labelling main objects, retraining on
  gold + main, and flagging out-of-range objects.

### 4. Classifier features
- **My finding:** *"Change of plans. let's change the features in the classifier. We will use the PCA from the full spectra (it discriminates way better than residual). Then, we will use this classifier on the main sample to continue the visual labelling."*
- **Agent:** switched the features.

### 5. Reading the labelling results
- **I asked** to explain the random check of the gold-trained classifier on main: 25 of 99 confidently labelled
  predicted UCDs were UCDs, with 101 more labelled uncertain.
- **Agent:** purity ≈ 25%; the gold-trained classifier does not transfer; the predicted UCDs must all be inspected; the uncertain
  fraction (half the sample) matters more than the purity.


### 6. Completing the labels 
- **I said:** *"the idea is to do active learning until I don't have any contaminants in the ucd predictions and viceversa, and then automatically label with P_CONTAM"*
- **Agent:** (1) inspect every predicted UCD rather than auto-label them; (2) stop when border
  batches find no UCDs, since active batches are biased.
- **I decided:** *"set the vlabel of these objects to uncertain: cand[(cand['vlabel']=='ucd') & (cand['p_contam']>0.2)]"*,
  then *"Label everything above 0.3 as contaminants and let me label all the unlabelled below 0.3 myself"*.

### 7. DTL with negatives
- **I asked:** *"Let's proceed with the negative autoencoder with negative examples. Anything to add before?"*
- **Agent proposed:** batches of negatives balanced between visually inspected and bulk-labelled contaminants
  (bulk labels dominate and are easy), a check of where the uncertain objects end up, and freezing a snapshot
  of the labels.
- **I decided** which of these to implement, and: *"Don't include augmentation for the contaminants"*.
- **First run:** validation UCD recall fell from 0.90 to 0.21 at epoch 25 and was still rising at the epoch limit.
  **Agent:** the negative term dominates at the start (every contaminant violates the margin); proposed a linear
  λ warm-up over 50 epochs and more epochs with early stopping after the warm-up.
- **I decided:** *"I'm proceeding to train the second model with the lambda ramp"*.
-   **Agent:** pointed out that the gap between UCDs (MSE ≲ 0.01) and contaminants (≳ 0.02) would allow a higher
  threshold, to be chosen from counts rather than fractions because of the base rate.
- **I decided:** *"Nice! I'm happy with the model."*
