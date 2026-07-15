# GeneFamilyTemplate

This is a template repository for setting up the gene-specific data and
configuration needed to run
[PhylogenyPipeline](https://github.com/MartinGuehmann/PhylogenyPipeline) on
a new gene family. It is not runnable on its own; see
[Opsins](https://github.com/MartinGuehmann/Opsins) for a repository
instantiated from this template.

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
	  Kick off gene-hit extraction (calls Scheduler-00-ExtractSequences.sh).
	  The "-NoContinue" variant stops after extraction instead of
	  automatically continuing into sequence processing.
	- 01_StartProcessing.sh / 01_StartProcessing-NoContinue.sh
	  Kick off sequence processing (Scheduler-01-PrepareSequences.sh).
	- 04_RestartProcessing.sh
	  Restart building the big combined sequence file
	  (Scheduler-04-ContinueMakeBigSequenceFile.sh).
	- 13_RestartProcessing.sh
	  Restart preparing sequences of interest for extraction
	  (Scheduler-13-ExtractSequencePreparation.sh).
	- 15_RestartProcessing.sh
	  Restart tree building for sequence-of-interest extraction
	  (Scheduler-15-ExtractSequencesOfInterestWithIQ-Tree.sh).
	- 16_TreeBuildScheduler.sh
	  Restart tree building (Scheduler-16-TreeBuildScheduler.sh).

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
`BaitSequences/` is an input you curate by hand; the rest are created by
the pipeline itself the first time it runs, so they don't exist yet in
this template:

	- BaitSequences/
	  Bait/seed sequences (fasta) used to search the protein databases
	  for hits.
	- Hits/
	  Raw per-database BLAST hit tables from the initial gene search.
	- Sequences/
	  Collected candidate sequences and the non-redundant sequence sets
	  derived from them.
	- SequencesOfInterest/
	  The sequences selected for the final alignment and tree building.
	- Alignments/
	  The final multiple sequence alignments and resulting trees.

If this gene family goes through the pruning-guide-tree step, additional
directories such as `AdditionalBaitSequences/`, `MustKeepSequences/`,
`OutgroupSequences/`, `RerootSequences/`, `SeqenceChunksForPruning/`, and
`TreesForPruningFromPASTA/` come into play — see
[PhylogenyPipeline's README](https://github.com/MartinGuehmann/PhylogenyPipeline#readme)
for what those directories are for.

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
