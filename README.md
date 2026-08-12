# ChEMBL Metalloproteinase Bioactivity Pipeline

SQL-based extraction and cleaning of bioactivity data for metalloproteinase
targets from a local [ChEMBL](https://chembl.gitbook.io/chembl-interface-documentation/downloads)
SQLite dump, producing a clean SMILES + activity-class dataset ready for
QSAR modeling. Same target family (MMPs) as an earlier published virtual
screening study of mine (Ramezani & Shamsara, *Mol. Divers.* 2018), though
built independently against a current ChEMBL release, not the pipeline
used for that paper.

## What it does

1. Queries the `activities`/`assays`/`compound_structures`/`target_dictionary`
   tables directly via `sqlite3`, joining on `molregno`/`tid`, filtered to
   direct single-protein-target binding assays (`confidence_score = 9`,
   `assay_type = 'B'`).
2. Converts `standard_value` (nM) to pActivity (`-log10(M)`, the same
   convention as ChEMBL's own `pchembl_value`) and classifies each compound
   active/inactive at a configurable pIC50-equivalent threshold (6.5 by
   default).
3. Drops censored measurements (`>`/`<` relations) whose reported bound
   can't actually resolve which side of the threshold the true value falls
   on.
4. Deduplicates by (`molregno`, `Target_ChEMBL_ID`): rows that disagree on
   activity class, or whose pActivity values differ by more than 3 log
   units, are dropped outright as unresolvable; genuinely consistent
   duplicates are then collapsed to one row.
5. Writes the clean `SMILES` + `activity_cat` dataset per target.

## Requirements

- A local ChEMBL SQLite dump (`chembl_<version>.db`), downloaded from
  [ChEMBL's downloads page](https://chembl.gitbook.io/chembl-interface-documentation/downloads) --
  not included here (multi-GB file).
- Python 3, `pandas`, `numpy`.

## Files

```
ChEMBL_GitHub.ipynb          the full pipeline
MMP_0.csv .. MMP_4_6.5.csv   intermediate/output CSVs from an MMP run, kept for reference
```

To run against a different target, edit the `t.pref_name LIKE '%...%'`
clause in the query cell, and the `target`/`path` variables at the top.
