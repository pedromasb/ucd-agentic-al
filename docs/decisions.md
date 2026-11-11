# Methodological decisions

The choices that changed the results, what was considered, why the final option was taken, and who proposed it.
Every decision in the table was taken by me (PMB); "Proposed by" says where the idea came from.

## Contaminant removal

| # | Decision | Final choice | Alternatives considered | Why | Proposed by |
|---|---|---|---|---|---|
| 1 | Labelling protocol | Fixed random checks of each predicted class (unbiased purity), then all predicted UCDs, border and out-of-range objects; "uncertain" allowed | Label everything at random; label only predicted UCDs | Random checks measure purity without bias; active batches find the classifier's errors fastest | Agent; PMB labelled every object shown |
| 2 | Gold sample | Random, targeted and active batches labelled by eye; once the classifier was reliable, the remaining unlabelled objects were labelled as contaminants (`bulk`) | Inspect every spectrum | Most gold candidates were contaminants; inspecting all of them would add little | PMB |
| 3 | Classifier features | Post-DTL latent + AE metrics + PCA of the **full spectra** + latent distances to the Elf Owl grid and to known UCDs | PCA of the reconstruction residuals | In my tests the full-spectrum PCA separated UCDs from contaminants much better | PMB |
| 4 | Classifier threshold | P(contaminant) ≤ 0.4, chosen on the plateau of the CV recall/rejection curves | 95th percentile of the CV probabilities of UCDs | The percentile rule jumped from 0.25 to 0.6 after two new labels | Agent (both, after spotting the instability); PMB (final value) |
| 5 | Gold → main | Label main objects too and retrain on gold + main; flag out-of-range features | Apply the gold classifier to main as is | The gold-trained classifier reached ~25% purity on main outside gold | Agent (diagnosis); PMB (labelling) |
| 6 | Bulk labels | Unlabelled objects with P > 0.3 → contaminant (`bulk`); everything below inspected by eye | P > 0.5; P > 0.9 and inspect the rest | Visual UCDs reach P ≈ 0.2; a wrong bulk label becomes a false negative in the DTL | PMB |
| 7 | Relabelling | UCDs with P > 0.2 set to `uncertain` before training | Keep them as UCDs | Labels the classifier cannot fit are the least reliable | PMB |

## DTL with negative examples

| # | Decision | Final choice | Alternatives considered | Why | Proposed by |
|---|---|---|---|---|---|
| 8 | Negatives in the fine-tuning | From the pre-trained AE with the same frozen block | Positives only (original DTL); classifier only | Without negatives the AE reconstructs broad-line emitters as well as UCDs | PMB (idea); agent (margin-loss formulation and implementation) |
| 9 | Positives | Known UCDs and templates of the original DTL + visual UCDs | Original DTL positives only | The visual UCDs extend the training set to the types found in Euclid | Agent; PMB (adopted) |
| 10 | Margin | margin threshold | - | Rejected contaminants keep a safety factor above the main-sample cut | Agent |
| 11 | λ schedule | Linear ramp 0 → 1 over 50 epochs; early stopping after the ramp | Constant λ = 1 | With constant λ, validation UCD recall fell to 0.21 at epoch 25 | Agent (after the first run); PMB (adopted) |
| 12 | Negative batches | 50% visually inspected, 50% bulk contaminants | Uniform; inspected only | Bulk labels dominate and are easy; the hard, inspected ones would rarely be seen | Agent |
| 13 | Augmentation of negatives | None | Wavelength shifts ±300, ±600 Å | Train on the contaminants as observed | PMB |
