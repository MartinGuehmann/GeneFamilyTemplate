Sequences (fasta) used to reroot the per-chunk pruning trees, one per
file. `13_SplitNonRedundantSequences.sh` injects them into every chunk;
`16_ExtractSequencesOfInterest.sh` reroots each chunk's tree on them
before `nw_clade` looks for the smallest clade containing all
`BaitSequences/` leaves - without a consistent root, "the clade
containing bait" isn't well-defined.

These are transient, unlike `OutgroupSequences/`: they only ever affect
the per-chunk pruning trees themselves and are never added to the final
delivered `SequencesOfInterest.fasta`. If a gene's pruning trees are
unrooted trifurcations (as IQ-Tree's `.contree`/`.treefile` output is)
and more than one reroot leaf is given, they can all fall into different
branches of that trifurcation at once, in which case `nw_reroot` fails
for that chunk - `16_ExtractSequencesOfInterest.sh` retries each leaf
individually before giving up and extracting unrerooted.

Keep this file even after adding real sequences here - it documents the
role above, not just placeholder boilerplate for an otherwise-empty
directory.
