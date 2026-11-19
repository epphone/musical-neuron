# The analysis pipeline in detail

[English](algorithm.md) | [Français](algorithm.fr.md)

This document walks through the successive steps performed by the Musical
Neuron between two of the performer's phrases. It follows (and is abridged
from) the conceptual essay in [`../paper/`](../paper/README.md).

## 0. Transcription

- `[fiddle~]` (64-bit port by Volker Böhm, after Miller Puckette) tracks
  pitch and amplitude of the live signal. The transcription is armed by the
  performer with a `[toggle]` — the **space bar** in the current iteration,
  a **MIDI pedal** ideally, in performance.
- Pitches are rounded to the nearest semitone; the rounding can be
  parameterized to quarter or eighth tones.
- Amplitudes are kept for each note and converted to MIDI velocities, used
  later by the distribution analysis.
- `[fiddle~]` only outputs the pitch at note onsets, while `[bach.transcribe]`
  requires pitch, velocity *and* duration in the same MIDI-style message. Each
  note is therefore transcribed with an arbitrary duration (1000 ms), and at
  the end of the phrase the message `sel all, legato` is sent to the roll so
  that each note ends exactly when the next one begins — assuming an unbroken
  melodic line.

## 1. Marker placement from the rhythmic distribution of the notes

Subpatchers: `#-sub-divisions`, `edge-detection-rhythm`, `round2onsets`,
`add-markers`.

- `[dada.analysis.rhythmdistrib]` computes the distribution of notes over the
  duration of the melody, according to the proximity of notes to one another
  and to weights derived from their MIDI velocities. It divides the roll into
  **N slices of equal duration**, where N is the total duration of the melody
  divided by the mean duration of its notes — approximating a unit comparable
  to a measure, like an antecedent or consequent component of a phrase.
  Because the analysis is driven by note *onsets*, runs come out fused
  together and held notes survive intact: a coarse but useful blurring of
  rhythmic detail.
- The resulting list is then reduced to binary activity values: every value
  above 0.0 becomes 0, every 0 becomes 1 (`[!- 0.]`), and `[togedge]` keeps
  only the 0→1 transitions — the boundary between activity and suspension.
- The slice markers produced by `add-markers` are snapped to the nearest note
  onsets with `[bach.nearest]`, so they coincide exactly with the beginning of
  runs and held notes.

This stretching of `cage.slice`'s "slice" notion is what the essay calls the
basic melodic units: stationary lines (held notes), ascending or descending
lines.

## 2. Slicing the melody

With markers on the roll, `[cage.slice]` divides the melody into several
rolls. In the subpatcher `process-slices`, the first note of every slice
except the first is removed — a necessary correction, since `[cage.slice]`
retains the tail of the last note of the preceding slice at the start of each
new one.

## 3. Small slices

Subpatchers: `sous-mediane`, `process-below`.

Even after `round2onsets`, markers are not always precise, and the first step
frequently produces slices containing only one or two notes. These short
slices often precede runs; when they precede a held note they are considered
**appoggiaturas**, which perception fuses with the attack of the longer
event. In both cases, the small slice is merged into the *following* slice.

- `sous-mediane` collects the durations of all slices and computes their
  **median** (rather than the mean, which would be biased by outliers). A
  slice counts as *small* only if it is significantly shorter than the
  median: the threshold is the median divided by 2. The subpatcher outputs
  the addresses of the qualifying slices.
- `process-below` merges each small slice with the slice that follows it,
  using `[bach.subs]` to replace the pair with a single roll in the list of
  slices.

## 4. Large slices

Subpatcher: `process-above`.

- The median is recomputed (the slicing changed), and a slice counts as
  *large* if its duration exceeds the median × 1.5.
- Long runs of quasi-constant speed can hide inside a large slice. To bring
  large slices back to a duration comparable to the median, `process-above`
  analyzes them for **large melodic leaps** (intervals greater than or equal
  to a perfect fourth), adds markers there, and divides the slice into
  several rolls. Splitting at leaps prepares the melodic reduction and
  prevents overly crude simplifications of long runs.

## 5. Melodic reduction

Subpatchers: `>2-notes`, and the profile comparison.

- `>2-notes` separates rolls containing fewer than two notes (plain held
  notes, with or without appoggiatura) from those containing actual runs —
  only the latter need notational simplification.
- Runs generally have an overall ascending or descending profile, or a
  composite of both. To decide which, each run is compared against
  **idealized contour profiles** adapted from the melodic profile classes of
  Pierre Schaeffer's TARSOM, themselves based on the elementary neumes of
  Gregorian chant:
  - **clivis** (ascending),
  - **podatus** (descending),
  - **torculus** (ascending, then descending),
  - **porrectus** (descending, then ascending).
- `[zsa.dist]` requires equal-length lists, so the profiles are **generated
  dynamically**: arithmetic series from 0.0 to 1.0 (clivis, podatus) and from
  −1.0 to 1.0 (torculus, porrectus), rescaled to the number of notes in the
  slice, choose the winning model.
- During the comparison, salient points are retained: **first note, nadir,
  apex, last note**. These recompose the reduced line; bach's glissando
  drawing replaces the run itself, always keeping its first and last notes.
  Only *torculus* and *porrectus* additionally need the extremes.
- The performer then reads this spatial notation and re-improvises over it,
  restarting the cycle.

## Known issues & roadmap

The tool is in its **alpha** stage. Items below come from the essay's
discussion section.

- **Onset-based distribution**: `[dada.analysis.rhythmdistrib]` refers to note
  *attacks*, not to the space notes occupy. A representation taking full
  note durations into account would remove the need for
  `edge-detection-rhythm` and the corrections of `round2onsets` and
  `add-markers`.
- **Melodic-reduction precision**: the four profiles are a somewhat arbitrary
  choice. Planned additions: a stationary-line model (for lines hovering
  around a harmonic pole — trills, wavering vibrato-like lines) and straight
  lines with a rise/fall in their first or second half, which should help
  `[zsa.dist]` find *torculus* and *porrectus* more easily.
- **Data ordering**: repeated `[bach.subs]` merges/removals are costly in Max
  and do not behave reliably every run. A global optimization is needed, or
  another way to decompose/recompose the melody than `[cage.slice]`.
- **Tiny slices from `process-above`** should be merged into an adjacent
  slice; this step is not implemented yet.
- **Alternative analyses**: this iteration is based on an "energy
  distribution" of the notes; other data-analysis methods could be explored.
- **Motif detection**: repeated and transposed motifs are not detected yet;
  a planned method borrows from graphic notation (a black box containing the
  motif, extended by an ascending, descending or flat tail).
- **Live analysis modulation**: an option to change the analysis mode on the
  fly (e.g. a pointillist style) would add a dynamic numeric feedback to the
  system.
- **Melodic leaps early in the pipeline**: leaps are only used just before
  the melodic reduction, but they should also feature in the earliest
  analysis steps, as they are significant in conjunct-minded phrases.
- **Sequentiality**: fundamentally, the computer still waits for the data
  before rendering its result; the performer cannot change the analysis
  mid-run. The dynamism is technological (the score as a technology), not
  numeric.
