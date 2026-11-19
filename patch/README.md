# The Max/MSP patch

[English](README.md) | [Français](#le-patch-maxmsp)

> **Status:** the `.maxpat` file(s) will be published here shortly.

## What goes here

The main patch of the Musical Neuron, plus any required abstractions and
supporting files. See the [installation guide](../docs/guide.md) for the
environment setup (Max/MSP 64-bit, Bach/Cage/Dada, fiddle~, zsa.descriptors).

## Subpatchers referenced in the documentation

| Name | Step |
|---|---|
| `#-sub-divisions` | rhythm-distribution analysis |
| `edge-detection-rhythm` | activity/suspension edge detection |
| `round2onsets` | snap markers to note onsets |
| `add-markers` | marker placement on the roll |
| `process-slices` | first-note removal after slicing |
| `sous-mediane` | median computation & small-slice addresses |
| `process-below` | merging of small slices |
| `process-above` | splitting of large slices at melodic leaps |
| `>2-notes` | separation of held notes from runs |

<!-- Le patch Max/MSP : le fichier `.maxpat` principal du neurone musical,
     avec ses abstractions et fichiers de support. Voir le
     [guide d'installation](../docs/guide.fr.md) pour la configuration de
     l'environnement (Max/MSP 64 bits, Bach/Cage/Dada, fiddle~,
     zsa.descriptors). -->

## Hardware

- A microphone or pickup on the (wind) instrument.
- Recommended for performance: a **MIDI pedal** to arm/stop the live
  transcription (the space bar is used in the current iteration).
