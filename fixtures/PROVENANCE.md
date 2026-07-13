# Fixture provenance

Every fixture window is a real example drawn from the parent project's own
published docs (not invented), with full source citations in each manifest
entry's `provenance` block. The windows are exactly the ones already
human-reviewed there; coordinates were never re-picked. As of 2026-07-10 the
manifest is in a **partially public** state, honestly split per entry:

- **`gate0_s1_known_text`** (positive) is `"status": "FETCHED_VERIFIED"`:
  its source raster was traced to a public URL in the official Vesuvius
  Challenge AWS open-data mirror, byte-identity was verified against the
  working file the parent project's Gate-0 check actually ran on (exact
  size + full-file sha256 equality; see the entry's
  `provenance.identity_verification`), and `fetch_fixtures.py` has been
  run live against it: it downloads the 72 MB TIF, slices the manifest's
  exact window, saves `fixtures/cache/gate0_s1_known_text.npy`, and
  stamps real sha256 hashes of both the source file and the produced
  fixture into the manifest. Anyone can reproduce this with one command:

  ```bash
  python fetch_fixtures.py --manifest fixtures/manifest.json
  ```

- **the three `blank` entries** are `"status": "NOT_PUBLICLY_FETCHABLE"`:
  their window *coordinates* translate 1:1 to public segment frames (the
  mosaic pixel grids are exactly the public segments' full-resolution PPM
  grids -- verified against the public PPM headers, recorded per entry in
  `region.coordinate_frame`), but their pixel *content* is a
  locally-computed canonical-model ink render that was never published
  anywhere. The full re-derivation recipe from fully public inputs (PPM +
  intensity volume zarr + model checkpoint + the exact render/stitch
  scripts and patch grid) is recorded in each entry's `provenance`, but
  re-running it needs a GPU and is not guaranteed byte-identical across
  hardware, so these entries are honestly not fetchable by download. The
  exact human-reviewed arrays are pinned by sha256 in each entry's
  `content_pin`; on a machine that has the parent project checkout,
  `fetch_fixtures.py --local-source-root <parent>` slices them from the
  local mosaics, verifies them against the pins (refusing to write if
  the source changed since review), and saves them to `fixtures/cache/`
  -- without upgrading their status, because a local slice is not a
  public fetch.

`fetch_fixtures.py` never fabricates a checksum: every sha256 it stamps
is computed from bytes it actually downloaded or sliced in that run.

## Per-fixture citations

- **`gate0_s1_known_text`** (positive) -- the exact crop used as a
  known-text reference window in the parent project's own Gate-0 canonical-
  inference check (`data/letters/gate0_infer_mac.py`): rows 14280-16060,
  cols 5840-8120 of the official ink-detection TIF for Scroll 1 /
  PHercParis4 segment `20260603005223-5753_-1` (published 2026-07-03 in
  the Vesuvius Challenge AWS open-data mirror; the parent project's local
  working alias for the same bytes was `ink_m1_fullres.tif`). That
  script's own re-inference of this window correlated at r=0.945
  (full-res) / 0.901 (4x downsampled) against this map at this same crop
  (`data/letters/gate0_mac.log`), both above its own r>=0.9 success bar.
  Scale: 2.4 um/px, derived from the segment's public surface-volume zarr
  `.zattrs` (declared scale [2.4, 2.4, 2.4] um, canvas 15600x80880 == the
  TIF's dims exactly) and cross-checked against the mirror's root
  `metadata.json` (`pixel_size_um: 2.4` for volume 20260411134726).
- **`s3p1_y700_x2100`, `s3p2_y0_x8148`, `s3p1_y0_x2100`** (blank) -- three
  Scroll 3 windows from the exhaustive Scroll 2/3 sweep described in the
  parent project's `release/publication/NEGATIVE_RESULTS.md` section (f).
  All three were originally flagged "interesting" by an automated literacy
  classifier, then downgraded to "noise" by a follow-up human review with
  3x3 spatial context (`data/letters/s1_atlas/triage/
  s2s3_context_review_final.json`, the per-window data behind that
  section's "commit 135221d" review round) -- exactly the "looked
  interesting at first glance, refuted on closer look" pattern this tool
  exists to automate. Window pixel coordinates and sizes come from that
  same triage file plus the `S3WIN` window-size constants in
  `data/letters/s1_atlas/train_proxy_s2s3_v1.py`; bounds were checked
  against the actual mosaic array shapes (`s3p1_mosaic.npy` (2739, 25309),
  `s3p2_mosaic.npy` (2491, 25706)), which in turn match the public PPM
  headers of Scroll 3 segments `20240716140050` and `20240618142020`
  exactly -- so the windows are addressable in the public segments' own
  coordinate frames. Scale: 7.91 um/px from the segments' public
  `meta.json` -> volume `20231117143551` -> `voxelsize: 7.91` (an earlier
  draft of this manifest said 2.4 um/px; that described the rescan
  intensity volume the render *sampled*, not the render grid pitch, and
  was corrected on 2026-07-10).

Every number above is cited to a specific file in the parent project; none
are invented. See each manifest entry's own `provenance` block for the
complete citation.
