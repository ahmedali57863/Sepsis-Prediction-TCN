# Data Governance — MIMIC-IV v3.1

This project uses the MIMIC-IV v3.1 database (Johnson et al., 2024), accessed
under PhysioNet's Credentialed Health Data License 1.5.0.

## Rules we follow

1. **Individual credentialing.** Every team member with data access holds
   their own independent PhysioNet credentialed access and has signed the
   Data Use Agreement personally. Data access is never shared via a
   teammate's copy, a shared drive, or a shared account.
2. **No raw or derived patient data in version control.** `data/` is
   gitignored at every processing stage (raw, interim, processed). Only
   code, configs, and de-identified aggregate results (e.g. AUROC numbers,
   trained-model performance summaries) are committed.
3. **No third-party data sharing.** Raw/derived data is not uploaded to
   general-purpose cloud storage, sent through third-party APIs, or shared
   with online services outside PhysioNet's approved access mechanisms.
4. **Approved cloud compute path.** If local hardware is insufficient for a
   training run, compute is scaled via PhysioNet's sanctioned route: linking
   a credentialed PhysioNet account directly to Google Cloud
   (BigQuery / Cloud Storage bucket), not by manually uploading raw exports
   to a personal Drive account.

## Citation

Johnson, A., Bulgarelli, L., Pollard, T., Gow, B., Moody, B., Horng, S.,
Celi, L. A., & Mark, R. (2024). MIMIC-IV (version 3.1). PhysioNet.
RRID:SCR_007345. https://doi.org/10.13026/kpb9-mt58

Johnson, A.E.W., Bulgarelli, L., Shen, L. et al. MIMIC-IV, a freely
accessible electronic health record dataset. Sci Data 10, 1 (2023).
https://doi.org/10.1038/s41597-022-01899-x

Goldberger, A., Amaral, L., Glass, L., Hausdorff, J., Ivanov, P. C.,
Mark, R., ... & Stanley, H. E. (2000). PhysioBank, PhysioToolkit, and
PhysioNet: Components of a new research resource for complex physiologic
signals. Circulation, 101(23), e215–e220.