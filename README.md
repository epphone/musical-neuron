<div align="center">

# The Musical Neuron

**le neurone musical**

A Max/MSP tool for *co-comprovisation*: improvised phrases are transcribed, reduced
and fed back to the performer in a continuous human–machine loop.

[English](README.md) &nbsp;|&nbsp; [Français](README.fr.md)

</div>

---

## Concept

The Musical Neuron starts from a question that reverses the usual premise of
computer-assisted composition (CAC): not *how can the computer help the composer*,
but *how can CAC come closer to an organic creative process*.

The performer improvises a single continuous, monodic phrase — approximately the
length of one breath on a wind instrument — and the computer:

1. **transcribes** the phrase in real time (pitches, amplitudes, timing);
2. **analyzes** it, keeping only the significant gestures: held notes, runs,
   and the temporal proportions between them;
3. **reduces** it to a spatial-notation line: held notes and glissandi;
4. returns the simplified score so the performer can **re-improvise over it**,
   with a variable degree of fidelity, for as many iterations as desired.

Rather than a faithful transcription — whose profusion of detail would close
musical thought in on itself — the tool deliberately *approximates and forgets*,
mimicking the role of flawed human memory in oral traditions, where forgetting
and error are the primary generators of variation. The program acts as a malleable
memory in a symbiotic human–machine circuit: it imitates by deforming, while the
performer re-injects "rhizomaticity" at every iteration.

The design is grounded in Deleuzian concepts (in dialogue with Bergson and
Proust): *virtual/actual*, *arborescent/rhizomatic*,
*deterritorialization/reterritorialization* — the wasp and the orchid. The full
conceptual essay (in French, PDF) is included in [`paper/`](paper/).

## Requirements

- [Max/MSP](https://cycling74.com/products/max) (64-bit)
- [Bach, Cage and Dada](https://www.bachproject.net/) — libraries by
  Andrea Agostini and Daniele Ghisi
- [fiddle~](https://vboehm.net/downloads/) — pitch tracking by Miller Puckette,
  64-bit port for Max by Volker Böhm
- [zsa.descriptors](http://www.e--j.com/index.php/what-is-zsa-descriptors/) —
  Mikhail Malt and Emmanuel Jourdan

Installation and setup instructions: [`docs/guide.md`](docs/guide.md).

## Download

A **standalone macOS application** — built with the Max runtime, no Max
installation required to run it — is published on the
[releases page](../../releases). The source patch (`.maxpat`) is published in
[`patch/`](patch/).

## How it works (overview)

1. **Transcription** — `[fiddle~]` tracks pitch and amplitude; the phrase is
   transcribed with `[bach.transcribe]`. Pitches are rounded to the nearest
   semitone (configurable to quarter or eighth tones), amplitudes are converted
   to MIDI velocities, and notes are made legato.
2. **Rhythmic-distribution markers** — `[dada.analysis.rhythmdistrib]` maps the
   distribution of activity vs. suspended time across the phrase; an edge-detection
   pass snaps markers to the nearest note onsets (`[bach.nearest]`).
3. **Slicing** — `[cage.slice]` cuts the melody into basic melodic units
   ("slices"): stationary lines (held notes) and ascending or descending lines.
4. **Small and large slices** — slices significantly *shorter* than the median
   duration (often appoggiaturas) are merged into the following slice; slices
   significantly *longer* are split at large melodic leaps (intervals of a
   perfect fourth or more).
5. **Melodic reduction** — each run is compared against dynamically generated,
   idealized contour profiles inspired by the elementary neumes of Gregorian
   chant as classified in Schaeffer's TARSOM: *clivis*, *podatus*,
   *torculus*, *porrectus* (`[zsa.dist]`). Salient points (first note, nadir,
   apex, last note) recompose the line, and runs are redrawn as glissandi.

Detailed walkthrough with all subpatchers: [`docs/algorithm.md`](docs/algorithm.md).

## Status

**Alpha.** The tool is functional but still benefits from refinement; known
limitations and planned improvements are listed in the
[essay's discussion](paper/README.md) and in
[`docs/algorithm.md` § Known issues & roadmap](docs/algorithm.md#known-issues--roadmap).

## Repository layout

```text
musical-neuron/
├── README.md            ← this file (English)
├── README.fr.md         ← documentation principale en français
├── docs/
│   ├── guide.md         ← installation & usage (EN)
│   ├── guide.fr.md      ← installation & usage (FR)
│   ├── algorithm.md     ← the analysis pipeline in detail (EN)
│   └── algorithm.fr.md  ← le pipeline d'analyse en détail (FR)
├── patch/               ← the Max/MSP patches (.maxpat)
├── paper/               ← the conceptual essay (PDF, in French)
├── LICENSE              ← CC BY-NC-SA 4.0
├── CITATION.cff
└── .gitignore
```

## Author

**Antoine Goudreau** ([@epphone](https://github.com/epphone))

## Acknowledgements

Built on the brilliant work of:

- **Andrea Agostini** and **Daniele Ghisi** — [Bach, Cage, Dada](https://www.bachproject.net/)
- **Miller Puckette** — [fiddle~](https://vboehm.net/downloads/), ported to 64-bit Max by **Volker Böhm**
- **Mikhail Malt** and **Emmanuel Jourdan** — [zsa.descriptors](http://www.e--j.com/index.php/what-is-zsa-descriptors/)

The conceptual framework is indebted to Gilles Deleuze (with Félix Guattari,
Henri Bergson and Marcel Proust), as well as to Pierre Schaeffer's *Traité des
objets musicaux*.

## License

This repository — code, documentation and essay — is licensed under
[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International](LICENSE)
([CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)):
you are free to share and adapt it, with attribution, for non-commercial
purposes only, and adaptations must be licensed under the same terms.
