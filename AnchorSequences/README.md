Fixed reference sequences (fasta), one per file, injected into every
chunk before alignment/tree building in the tree-based pruning stage
(steps 13-16) purely to stabilize each chunk's tree topology - unlike
`BaitSequences/` (defines the target clade) or `RerootSequences/` (roots
the tree), these are known, unambiguously *distant* relatives that must
never end up inside the extracted clade. Optional; the pipeline only
reads this directory if it exists.

Why this might be needed: with only bait + reroot sequences as fixed
landmarks, a chunk's few hundred to ~900 randomly drawn sequences can
give the tree builder too little consistent context to reliably resolve
where the target clade's boundary actually is. Symptom to watch for in
`ExtractionCounts.txt`: per-chunk extraction counts that are bimodal or
otherwise inconsistent rather than smoothly variable, with inspection
showing whole unrelated sequence families swept into the "high" chunks.

If that happens, pick a handful of real, already-in-pool sequences from
clearly distant branches - not just "far from bait," but a genuinely
different family with a similar, compact single-domain architecture (to
avoid alignment/domain-mismatch problems a very different architecture,
e.g. a large extracellular domain, could cause). `16_ExtractSequencesOfInterest.sh`
excludes each one's own clade from the final extracted output the moment
any of them still lands inside it, so an imperfect choice can't
contaminate real output - it just costs some recall on the chunk where
it happens.

Delete this file once real sequences have been added; it only exists to
keep this otherwise-empty directory tracked in git.
