Outgroup sequences (fasta), one per file. These end up in three different
places, not just "the pruning-guide tree":

- `08_ExtractSequencesOfInterest.sh` adds them alongside `BaitSequences/`
  (and `AdditionalBaitSequences/`) as extra landmarks when extracting
  labels of interest from the single guide tree built by
  `07_MakeTreeForPruning.sh`.
- `13_SplitNonRedundantSequences.sh` injects them into every chunk built
  for the per-chunk pruning trees (alongside bait/reroot/anchor
  sequences), same as those.
- `16_ExtractSequencesOfInterest.sh` appends them directly into the final
  `SequencesOfInterest.fasta` - the delivered sequence set, not just a
  tree-building aid. That's what actually roots the final alignment/tree
  built in `Alignments/` (steps 9-10): without an outgroup in the
  delivered set itself, that tree has nothing to root against.

That third role is what sets these apart from `RerootSequences/` and
`AnchorSequences/`, which only ever affect the transient per-chunk
pruning trees and never appear in the final output.

Delete this file once real sequences have been added; it only exists to
keep this otherwise-empty directory tracked in git.
