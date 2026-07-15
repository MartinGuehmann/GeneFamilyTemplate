# GeneFamilyTemplate

This is a template repository for setting up the gene-specific data and
configuration needed to run
[PhylogenyPipeline](https://github.com/MartinGuehmann/PhylogenyPipeline) on
a new gene family. It is not runnable on its own; it was generalized from
[Opsins](https://github.com/MartinGuehmann/Opsins), which remains useful
as an example.

## Using this template

1. Clone or use GitHub's "Use this template" button to create a new
   repository from this one, and rename it to the gene family it will
   hold (the repository/directory name is used as the gene name, see
   Layout below).
2. Add bait/seed sequences to `BaitSequences/` and remove the placeholder
   `BaitSequences/README.md`.
3. Fill in `Clades.csv` and `InterestingTaxa.csv` with reference sequences
   and taxa for this gene family, and rename the ingroup row in
   `InterestingTaxa.csv` accordingly.
4. Fill in `AdditionalTaxonIdentifiers.csv` if needed.
5. Optionally fill in `_SpecialAminoAcids.txt` and drop its leading
   underscore if there is a specific amino acid position of interest to
   annotate.
6. If this gene family won't go through the pruning-guide-tree step,
   delete `AdditionalBaitSequences/`, `MustKeepSequences/`,
   `OutgroupSequences/`, and `RerootSequences/` — otherwise fill them in
   as needed and remove their placeholder README.md files (see
   Subdirectories below).

## Layout

Repositories created from this template must be cloned as a sibling of
`PhylogenyPipeline`, i.e. both checked out next to each other under the
same parent directory:

	some-parent-directory/
		PhylogenyPipeline/
		YourGeneFamily/

The scripts here call `../PhylogenyPipeline/...` directly, and pass the
pipeline `-g "../YourGeneFamily"` so that the pipeline writes its per-gene
output (`Hits/`, `Alignments/`, `SequencesOfInterest/`, etc.) back into
this repository instead of into `PhylogenyPipeline`.

## Running

	- 00_StartExtraction.sh / 00_StartExtraction-NoContinue.sh
	  Kick off gene-hit extraction (step 0; calls
	  Scheduler-00-ExtractSequences.sh). The "-NoContinue" variant stops
	  after step 0 instead of automatically continuing into sequence
	  processing (step 1).
	- 01_StartProcessing.sh / 01_StartProcessing-NoContinue.sh
	  Kick off sequence processing (steps 1 to 4;
	  Scheduler-01-PrepareSequences.sh). The "-NoContinue" variant stops
	  after step 4 instead of automatically continuing into
	  sequence-of-interest preparation (step 13).
	- 04_RestartProcessing.sh
	  Restart building the big combined sequence file (step 4;
	  Scheduler-04-ContinueMakeBigSequenceFile.sh), then automatically
	  continues into sequence-of-interest preparation (step 13;
	  13_RestartProcessing.sh's
	  Scheduler-13-ExtractSequencePreparation.sh) — there is no
	  "-NoContinue" variant of this script to stop that.
	- 13_RestartProcessing.sh
	  Restart preparing sequences of interest for extraction
	  (Scheduler-13-ExtractSequencePreparation.sh). useFullDataset is set
	  here, so this runs step 17 instead of step 13, skipping the
	  pruning-guide-tree extraction and using the full non-redundant
	  dataset directly, then automatically continues into tree building
	  (16_TreeBuildScheduler.sh's Scheduler-16-TreeBuildScheduler.sh) —
	  again with no "-NoContinue" variant.
	- 15_RestartProcessing.sh
	  Restart tree building for sequence-of-interest extraction (step
	  15; Scheduler-15-ExtractSequencesOfInterestWithIQ-Tree.sh), then
	  automatically continues into tree building
	  (16_TreeBuildScheduler.sh's Scheduler-16-TreeBuildScheduler.sh) —
	  again with no "-NoContinue" variant.
	- 16_TreeBuildScheduler.sh
	  Restart tree building (Scheduler-16-TreeBuildScheduler.sh):
	  repeatedly runs the alignment/rogue-removal step (step 9) across
	  aligners to build the final trees. The final step; nothing to
	  continue into.

Each of these picks the gene name up from its own directory name, so they
only work correctly when run from within this checkout.

## Standalone checks

Beyond the numbered scripts above, it's common to add small standalone
scripts for gene-family-specific sanity checks that aren't part of the
main pipeline run, e.g. comparing hit counts with and without a particular
bait sequence, or checking an alignment for a conserved residue at a
known position. See `CheckForLysine.sh`/`CheckPlacopsins.sh` in Opsins for
examples. None are included in this template; add them as needed.

## Configuration files

	- Clades.csv
	  Reference sequence, clade label, and two colors per line, used to
	  color and label clades in the output tree figures. The last entry
	  is used as the outgroup for rooting the tree. Additional files
	  named `*Clades.csv` can be added for separate clade-focused
	  reruns — 12a_ConvertTreesToFiguresForAllClades.sh picks up any
	  file matching that pattern.
	- AdditionalTaxonIdentifiers.csv
	  Lookup table mapping misspelled or non-standard higher-taxon names
	  to their correct name in the NCBI taxon database.
	- InterestingTaxa.csv
	  Taxa to highlight with specific colors in the output trees.

`_SpecialAminoAcids.txt` configures 12_ConvertTreesToFigures.py to
annotate a specific amino acid position of interest, numbered against a
given reference sequence, in the output trees and sequence logos. The
leading underscore takes it out of the way of the `SpecialAminoAcids.txt`
filename the script actually looks for, so as shipped in this template it
has no effect; drop the underscore once it is filled in.

## Subdirectories

These follow PhylogenyPipeline's `$DIR/$gene/...` convention for per-gene
data (see
[its README](https://github.com/MartinGuehmann/PhylogenyPipeline#readme)).
Some are inputs you curate by hand, the rest are created by the pipeline
itself the first time it runs, so they don't exist yet in this template.

Inputs:

	- BaitSequences/
	  Bait/seed sequences (fasta) used to search the protein databases
	  for hits.
	- AdditionalBaitSequences/
	  Extra bait sequences included alongside BaitSequences/. Optional;
	  the pipeline only reads this directory if it exists.
	- MustKeepSequences/
	  Reference sequences that must survive non-redundancy filtering
	  regardless of similarity to other sequences.
	- OutgroupSequences/
	  Outgroup sequences added to the pruning-guide tree and used for
	  rooting it.
	- RerootSequences/
	  Sequences used to reroot trees.

Generated by a pipeline run:

	- Hits/
	  Raw per-database BLAST hit tables from the initial gene search.
	- Sequences/
	  Collected candidate sequences and the non-redundant sequence sets
	  derived from them.
	- SequenceChunksForPruning/
	  Sequence chunks split off for building the pruning-guide tree.
	- TreesForPruningFromPASTA/
	  PASTA trees built over those chunks, used to decide which
	  sequences to prune before the final alignment.
	- SequencesOfInterest/
	  The sequences selected for the final alignment and tree building.
	- Alignments/
	  The final multiple sequence alignments and resulting trees.

`AdditionalBaitSequences/`, `MustKeepSequences/`, `OutgroupSequences/`,
`RerootSequences/`, `SequenceChunksForPruning/`, and
`TreesForPruningFromPASTA/` are only relevant if this gene family goes
through the pruning-guide-tree step; delete the input directories among
them if not using it (the generated ones simply won't be created). See
[PhylogenyPipeline's README](https://github.com/MartinGuehmann/PhylogenyPipeline#readme)
for more on that step.

## Data

Once a pipeline run has populated `Hits/`, `Alignments/`, `Sequences/`,
`SequencesOfInterest/`, and `BaitSequences/`, those directories tend to be
tracked in git rather than gitignored, so a full clone of a repository
instantiated from this template can get large. To get just the scripts
and configuration without the generated data, use a partial, sparse
clone:

	git clone --filter=blob:none --no-checkout <repository-url>
	cd YourGeneFamily
	git sparse-checkout init --cone
	git sparse-checkout set
	git checkout master

Cone mode checks out files directly at the repository root by default —
the scripts and top-level configuration files above — but requires any
subdirectory to be listed explicitly with `git sparse-checkout set` to be
included, so add any standalone-check subdirectories to that command if
needed.
