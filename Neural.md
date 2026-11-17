# Neural — Possession Classification

## Objective and output contract

This submission produces one possession label for every frame of the Barcelona–Athletic Club
tactical-video timeline. The CSV uses the required `frame,label` schema, with 199,319 ordered
frames (`0` through `199318`) and labels from `H`, `A`, `D`, and `X`.

The prepared file is `Neural.csv`. The chosen method is the organizer's Smooth-50 reference
baseline. This is a reproducible reference choice; it is not a claim that it is the best
full-match method.

## Permitted inputs and data handling

The method uses only the supplied tactical ATD tracking metadata and raw tactical ATD tracking:

- `tactical/BAR-ATH_tactical_atd.xml`
- `tactical/BAR-ATH_tactical_atd.txt`

It does not use broadcast video or tracking, Smart Tagging, homography, private ground truth, or
manual frame correction. The match data remains outside version control. No raw tracking exports,
videos, private labels, or derived artifacts capable of reconstructing restricted data are included
in this document.

## Method

For each tactical frame with a tracked ball, the baseline finds the tracked player nearest to the
ball and assigns that player's team. Team membership comes from the ATD metadata: the first listed
team is home (`H`) and the other team is away (`A`).

ATD coordinates are normalized, so the implementation scales horizontal and vertical coordinates
by the official pitch dimensions before comparing distances. ATD frame IDs are one-based; the
submission timeline is zero-based, so the implementation writes a raw ATD frame to `frame_id - 1`.

The raw H/A observations are then smoothed with a centred majority vote over `±50` frames. Within
each window, the side with more H/A observations is emitted; ties go to `H`. If a window contains
no H/A observation, the output is `D`.

## Missing observations

When the raw tracking record has no ball observation, the baseline leaves that raw frame as
`NOBODY`. The centred smoothing stage emits `D` only where its window contains no H/A observations.
This is the baseline's operational rule, not a claim that every missing ball observation means the
ball is out of play. Missing tracking can occur for several reasons, and the method does not model
dead-ball events separately.

## Reproduction

Tested environment: Windows x86-64 and CPython 3.13.17 with the pinned dependencies. Set
`MTAH_DATA_DIR` to the external data root containing the `tactical/` folder.

Example PowerShell commands from the repository root (use a new output directory and do not copy
match data into the repository):

```powershell
$env:MTAH_DATA_DIR = 'path\to\authorised-data'
python -m pip install --require-hashes -r requirements.lock
python starter\baseline.py --window 50 --out generated
```

The generated output should be compared with the recorded hash. The organizer's separately
available validator checks the header, row count, frame ordering, and allowed label vocabulary.

## Development disclosure

We also manually annotated segments outside the public ground-truth window and used those labels
in research comparisons. The selected submission is the unchanged organizer Smooth-50 baseline,
which does not use these labels to generate predictions; comparisons informed its fallback
selection. The participant supplied organizer responses permitting such annotations and
validation selection, but the original messages and sender metadata were not independently
verified. Published rules and the supplied clarification should be reconciled with organizers.

Our separate supervised research used manually annotated training and validation segments.
The 2,000-frame validation subset was previously used and its class counts were seen; it was
not a fresh blind or independent confirmation. No valid final official spell-F1 comparison
established that a supervised candidate beats Smooth-50. We retained the baseline as a
conservative fallback, not as an improvement claim. Historical public-ground-truth influence
was not comprehensively ruled out for every experiment.

## Recorded validation evidence

Recorded verification of `Neural.csv` found:

- 199,319 data rows, with frames `0` through `199318` in order;
- header `frame,label`;
- labels `H`, `A`, and `D` only, with no blank labels;
- official validator pass; and
- byte-for-byte equality with `evaluation/results/baseline_smooth50.csv`.

The recorded SHA-256 is:

```text
D444CAA1AEBB6FB58386033A5967C83302B8DD984EA91C86381BF208332702FF
```

## Limitations

The nearest-player rule can be unreliable in duels, loose-ball situations, deflections, occlusion,
or tracking error. The method has no explicit model of possession transitions or dead-ball events.

The organizer's public evaluation slice contains only 17 possession spells. It is useful for
checking alignment and file validity, but it cannot establish full-match superiority. Private
full-match ground truth was not used and is unavailable for local evaluation.

## Data protection and reproducibility

The public repository should contain source code, dependencies, and instructions sufficient to
regenerate the submission from the authorised bundle. It must not contain the gated match data,
private ground truth, videos, raw tracking exports, or restricted derived data.
