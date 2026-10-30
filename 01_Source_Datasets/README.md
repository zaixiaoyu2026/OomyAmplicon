# NCBI-curated oomycete ITS source dataset

This 28,487-sequence exact non-redundant NCBI-derived ITS collection is supplied as a **source dataset** for Hybrid supplement screening and provenance. It is not treated as a third standalone reference database in the final UNITE versus UNITE_OomyHybrid classifier comparison.

Contents include normalized FASTA sequences, taxonomy strings, accession metadata, curation decisions, taxonomy-conflict records, and composition summaries. NCBI records were retrieved using the Oomycota taxonomic scope and ITS title terms, then filtered for sequence and TaxID/lineage completeness, 150–4,000 nt length, valid IUPAC symbols, N ≤1%, and total ambiguity ≤2%. Environmental/low-information records were excluded before exact SHA-256 sequence deduplication. See `../02_UNITE_OomyHybrid_v1.0.0/README_database_construction_for_publication.md` for how the source data were screened to select the 884 sequences added to the Hybrid reference.
