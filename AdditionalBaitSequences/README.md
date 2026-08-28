Extra bait sequences (fasta), one per file, included alongside
`BaitSequences/` when searching the protein databases for hits — used to
add sequences without editing `BaitSequences/` itself. Optional; the
pipeline only reads this directory if it exists.

This directory is also the reproducibility record for `BaitSequences/`:
every query sequence that has *ever* been bait and actually contributed
hits to this gene's current `Sequences/` pool needs a copy here, even
after it stops being bait. Removing a sequence from `BaitSequences/`
without keeping a copy here silently breaks that record - redoing the
search (steps 1-4) would then produce a smaller pool than the one
actually checked in, missing whatever hits only that sequence's query
found. If a former bait sequence needs to be genuinely gone from the
pool, its hits have to be hunted down and removed explicitly; simply
dropping it from both directories does not do that.

Delete this file once real sequences have been added; it only exists to
keep this otherwise-empty directory tracked in git.
