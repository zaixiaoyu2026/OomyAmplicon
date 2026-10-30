# Primer-region QIIME 2 artifacts

These artifacts support the final two-reference comparison using the ITS1-O / 5.8S-O-Rev region. They were generated in QIIME 2 amplicon 2025.4. Primer extraction used forward primer `CGGAAGGATCATTACCAC`, reverse primer `AGCCTAGACATCCACTGCTG`, minimum identity 0.8, minimum amplicon length 50 bp, and both orientations. Naive Bayes classifiers were fitted separately for the archived UNITE v10.0 backbone and UNITE_OomyHybrid v1.0.0. Shared representative ASVs were classified at confidence 0.7.

| File prefix | Artifact | QIIME 2 type / role |
|---|---|---|
| `unite10_primer_refs.qza` | Extracted reference reads | `FeatureData[Sequence]` |
| `unite10_primer_taxonomy.qza` | Taxonomy matching extracted UNITE reads | `FeatureData[Taxonomy]` |
| `unite10_primer_classifier.qza` | Trained UNITE classifier | `TaxonomicClassifier` |
| `hybrid_primer_refs.qza` | Extracted Hybrid reference reads | `FeatureData[Sequence]` |
| `hybrid_primer_taxonomy.qza` | Taxonomy matching extracted Hybrid reads | `FeatureData[Taxonomy]` |
| `hybrid_primer_classifier.qza` | Trained Hybrid classifier | `TaxonomicClassifier` |
| `taxonomy_unite10_primer.qza` | UNITE assignments for the shared representative sequences | `FeatureData[Taxonomy]` |
| `taxonomy_hybrid_primer.qza` | Hybrid assignments for the shared representative sequences | `FeatureData[Taxonomy]` |

Recreate these artifacts with `Code_and_Pipelines/03_classifier_comparison/primer_region_retraining/train_two_classifiers.sh`. The full-length (untrimmed) Hybrid reference artifacts are one directory above: `../ref-seqs.qza`, `../ref-taxonomy.corrected.qza`, and `../classifier.corrected.qza`. The QIIME 2 artifacts do not include raw FASTQ data.

Classifier labels are reference- and parameter-dependent predictions. Assignment counts do not measure percent identity or prove biological identity.
