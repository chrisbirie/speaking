# Designing Data Pipelines That Don't Hate You Six Months Later

**Stir Trek 2026** · AMC Easton, Columbus, OH · May 1, 2026

[▶ Watch the recording](https://www.youtube.com/watch?v=Dwg9FyWeezU&list=PLaHMqLt8nxCx3TgokegypUH5_K0_kVpXA&index=15) · [Slides (PDF)](./slides.pdf) · [Transcript](./transcript.md)

Most pipelines work fine the day you ship them. The pain shows up six months later: the schema changed, a backfill double-counted half a table, and nobody can explain why last Tuesday's numbers are wrong.

This talk is about the design choices that decide which of those futures you get — schema evolution, idempotency, observability, and testability. "Design for change" isn't a slogan. It's a set of concrete decisions you make early, before the pipeline has any data worth breaking.

This was my first conference talk. It's part of the [Stir Trek 2026 playlist](https://www.youtube.com/playlist?list=PLaHMqLt8nxCx3TgokegypUH5_K0_kVpXA).
