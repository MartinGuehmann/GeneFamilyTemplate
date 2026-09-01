Put the bait/seed sequences for this gene family here, one FASTA file per
sequence, e.g. `AAA32805_Ga1_Arabidopsis_thaliana.fasta`. These do double
duty:

- Steps 1-4 (`00a_GetGenes.sh` in PhylogenyPipeline) use them as queries to
  search the protein databases for homologs, seeding this gene's
  `Sequences/` pool.
- `16_ExtractSequencesOfInterest.sh` uses `nw_clade` to find the smallest
  clade in each pruning tree that contains *all* bait leaves at once, and
  extracts that as "the target clade" - the whole basis for
  `SequencesOfInterest`.

That second role makes bait choice riskier than it looks: because all bait
leaves are combined into a single MRCA, every one of them has to be a
genuine, unambiguous member of the same monophyletic target group. One
sequence that's actually a distant relative - even if its database
annotation makes it look like a match - forces that MRCA outward across
whatever lies between it and the real target clade, and everything in
between gets swept into the extraction along with it. If curation later
finds such a sequence, move it out of `BaitSequences/` (see
`AnchorSequences/README.md` if the rest of its family should now be
actively excluded, and keep a copy in `AdditionalBaitSequences/` per that
directory's own README so redoing the database search stays
reproducible).

See also `RerootSequences/` (roots the tree so "the clade containing
bait" is well-defined) and `AnchorSequences/` (the opposite of bait -
known, unambiguously *distant* references that must never end up inside
the extracted clade).

Keep this file even after adding real sequences here - it documents the
role above, not just placeholder boilerplate for an otherwise-empty
directory.
