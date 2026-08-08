# Scalable Hash Indexes

## Trigger

Gitmatica currently stores deterministic raw `.lvcidx` files. Raw indexes pack
well with Git delta compression and should remain the default unless measurement
shows that index size or rewrite cost is a real bottleneck.

## Planned Evaluation

- Benchmark representative large projects and long histories before changing
  the format.
- Measure packed repository size, loose working-tree size, read latency, write
  latency, peak memory, and Git delta quality.
- Compare the current flat index with deterministic chunked or paged indexes.
- Preserve stable ordering and reproducible bytes.
- Define an explicit version and migration path before introducing a new index
  encoding.
- Avoid coupling index layout to editable manifest settings.

No format change is planned without evidence that the current raw index is a
measurable constraint.
