# Installation & usage guide

[English](guide.md) | [Français](guide.fr.md)

## Requirements

| Component | Role | Link |
|---|---|---|
| Max/MSP (64-bit) | visual programming environment | <https://cycling74.com/products/max> |
| Bach, Cage, Dada | music notation & analysis libraries | <https://www.bachproject.net/> |
| fiddle~ | real-time pitch tracking | <https://vboehm.net/downloads/> |
| zsa.descriptors | distance/similarity measures | <http://www.e--j.com/index.php/what-is-zsa-descriptors/> |

## 1. Install Max/MSP

Install the 64-bit version of Max/MSP from Cycling '74. The patch has been
developed with Bach/Cage/Dada, which target recent versions of Max.

## 2. Install the Bach, Cage and Dada libraries

1. Download the latest unified installer from <https://www.bachproject.net/>
   (the bach+ cage+ dada+ family is distributed together).
2. Follow the official installation instructions for your platform.
3. Restart Max and check that the objects appear (e.g. type `bach.roll` in a
   new object box).

## 3. Install fiddle~ and zsa.descriptors

1. **fiddle~**: download the 64-bit external by Volker Böhm from
   <https://vboehm.net/downloads/> and place it in the Max search path (e.g.
   the `externals` folder of the Max installation, or a folder listed in
   *Options ▸ File Preferences*).
2. **zsa.descriptors**: download the library from
   <http://www.e--j.com/index.php/what-is-zsa-descriptors/> and install it the
   same way.

Restart Max so the externals are detected, then verify that `[fiddle~]` and
`[zsa.dist]` instantiate correctly.

## 4. Open the patch

Open the `.maxpat` file from the [`patch/`](../patch/) folder. Make sure the
Bach/Cage/Dada package folders are in Max's search path.

## 5. Audio & MIDI setup

- Connect your instrument (microphone or pickup) to the audio input of the
  patch. A reasonably clean monophonic signal gives the best transcription.
- The transcription is triggered manually. In the current iteration, use the
  **space bar**; in performance, a **MIDI pedal** is the recommended trigger
  (ideally wired to start/stop the analysis).

## How to perform with it

The intended context is an improvised piece for a solo wind instrument, but the
tool can also be used outside of performance, as a composition aid.

1. **Breathe in, start the analysis** (space bar / pedal): transcription is
   now live.
2. **Improvise one phrase in a single breath**, following the contour of a line
   that spans the whole duration of the breath.
3. **Breathe**: while you breathe in, the analysis runs. It is fast enough to
   finish during the breath, so you can immediately resume.
4. **Read the reduced line**: a simplified version of your phrase appears in
   spatial notation — held notes and glissandi.
5. **Re-improvise over it**, with whatever degree of fidelity you want. Do not
   try to reproduce it exactly: the blurring is a feature, not a bug.
6. **Repeat** for as many iterations as you wish. The line keeps evolving.

### Improvisation conventions (presupposed by the tool)

The algorithm expects phrases that respect a few musical constraints. They do
not restrict the performer's freedom, they just define the genre of material
the analysis is designed for:

- **continuous** phrases: no rests or pauses between notes;
- an **alternation of long held notes and short runs** (scale or mode runs,
  with possible ornamentation), favoring **stepwise motion** (seconds,
  thirds). Larger melodic leaps are perfectly allowed;
- **rhythmically free**: no meter, no tempo.

### Parameters

- **Pitch rounding**: semitone by default; can be set to quarter or eighth
  tones.
- **Note duration in transcription**: since `[fiddle~]` only reports pitch at
  note onsets, each note is transcribed with an arbitrary duration (1000 ms)
  and afterwards made legato (`sel all, legato` on the roll).
- **Slice count**: the phrase is subdivided according to its total duration
  divided by the mean note duration — approximating one slice per
  measure-like unit.

### Performance tips

- **Trust the blur.** The value of the tool lies in what it forgets. If you
  try to play back the reduced line too faithfully, the human–machine loop
  risks settling into musical stagnation (an *hysteresis loop*): vary your
  fidelity from one iteration to the next.
- The analysis conforms to what the program considers melodic units — expect
  appoggiaturas to be absorbed into the note or run that follows them.
