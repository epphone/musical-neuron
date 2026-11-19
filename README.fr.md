<div align="center">

# Le neurone musical

**The Musical Neuron**

Un outil Max/MSP de *co-comprovisation* : les phrases improvisées sont transcrites,
réduites et redonnées à l'interprète dans une boucle continue humain–machine.

[Français](README.fr.md) &nbsp;|&nbsp; [English](README.md)

</div>

---

## Le concept

Le neurone musical part d'une question qui renverse la prémisse habituelle de la
composition assistée par ordinateur (CAO) : non pas *comment l'ordinateur peut-il
aider le processus compositionnel*, mais *comment la CAO peut-elle se rapprocher
d'un processus créatif organique*.

L'interprète improvise une phrase monodique, continue, d'une seule traite —
approximativement la durée d'un souffle sur un instrument à vent — et l'ordinateur :

1. **transcrit** la phrase en temps réel (hauteurs, amplitudes, rythme) ;
2. **analyse** la ligne pour n'en retenir que les gestes signifiants : notes tenues,
   traits, et les proportions temporelles entre ces événements ;
3. **réduit** la phrase en notation spatiale : notes tenues et glissandos ;
4. redonne la version simplifiée à l'interprète pour qu'iel puisse **réimproviser
   par-dessus**, avec un degré de fidélité variable, pour un nombre d'itérations
   indéterminé.

Plutôt qu'une transcription fidèle — dont la profusion de détails refermerait la
pensée musicale sur elle-même —, l'outil *approxime* et *oublie* délibérément,
mimant le rôle de la mémoire humaine faillible dans les traditions orales, où
l'oubli et l'erreur sont les premiers générateurs de la variation. Le programme
agit comme une mémoire malléable dans un circuit symbiotique humain–machine :
il imite en déformant, tandis que l'interprète réinjecte de la rhizomaticité à
chaque itération.

La conception s'ancre dans des concepts deleuziens (en dialogue avec Bergson et
Proust) : *actuel/virtuel*, *arborescent/rhizomatique*,
*déterritorialisation/reterritorialisation* — la guêpe et l'orchidée. L'essai
conceptuel complet (en français, PDF) est inclus dans [`paper/`](paper/).

## Prérequis

- [Max/MSP](https://cycling74.com/products/max) (64 bits)
- Les librairies [Bach, Cage et Dada](https://www.bachproject.net/) — Andrea
  Agostini et Daniele Ghisi
- [fiddle~](https://vboehm.net/downloads/) — suivi de hauteur selon Miller
  Puckette, portage 64 bits pour Max par Volker Böhm
- [zsa.descriptors](http://www.e--j.com/index.php/what-is-zsa-descriptors/) —
  Mikhail Malt et Emmanuel Jourdan

Instructions d'installation et de configuration : [`docs/guide.fr.md`](docs/guide.fr.md).

## Téléchargement

Une **version application autonome pour macOS** — compilée avec le moteur de
Max, sans installation de Max requise pour l'exécuter — est publiée sur la
[page des releases](../../releases). Le patch source (`.maxpat`) est publié dans
[`patch/`](patch/).

## Fonctionnement (aperçu)

1. **Transcription** — `[fiddle~]` suit la hauteur et l'amplitude ; la phrase est
   transcrite avec `[bach.transcribe]`. Les hauteurs sont arrondies au demi-ton
   près (paramétrable au quart ou au huitième de ton), les amplitudes sont
   converties en vélocités MIDI, et les notes sont jouées en legato.
2. **Marqueurs de distribution rythmique** — `[dada.analysis.rhythmdistrib]`
   cartographie la distribution des moments d'activité et de temps suspendus ;
   une détection de transitions cale les marqueurs sur les temps d'entrée les plus
   proches (`[bach.nearest]`).
3. **Division en tranches** — `[cage.slice]` découpe la mélodie en unités
   mélodiques de base : lignes stationnaires (notes tenues) et lignes ascendantes
   ou descendantes.
4. **Petites et grandes tranches** — les tranches significativement plus *courtes*
   que la médiane des durées (souvent des appoggiatures) sont jointes à la tranche
   suivante ; les tranches significativement plus *longues* sont divisées aux
   grands sauts mélodiques (intervalles équivalents ou supérieurs à la quarte juste).
5. **Réduction mélodique** — chaque trait est comparé à des profils de parcours
   mélodique idéalisés, générés dynamiquement d'après les neumes élémentaires du
   chant grégorien tels que classés dans le TARSOM de Schaeffer : *clivis*,
   *podatus*, *torculus*, *porrectus* (`[zsa.dist]`). Les points saillants
   (première note, nadir, apogée, dernière note) recomposent la ligne, et les
   traits sont redessinés en glissandos.

Le déroulé détaillé avec tous les sous-patchs : [`docs/algorithm.fr.md`](docs/algorithm.fr.md).

## Statut

**Alpha.** L'outil est fonctionnel, mais il bénéficierait encore de retouches ;
les limites connues et les améliorations prévues sont listées dans la
[discussion de l'essai](paper/README.md) et dans
[`docs/algorithm.fr.md` § Problèmes connus & feuille de route](docs/algorithm.fr.md#problèmes-connus--feuille-de-route).

## Organisation du dépôt

```text
musical-neuron/
├── README.md            ← documentation in English
├── README.fr.md         ← ce fichier (français)
├── docs/
│   ├── guide.md         ← installation & usage (EN)
│   ├── guide.fr.md      ← installation & usage (FR)
│   ├── algorithm.md     ← the analysis pipeline in detail (EN)
│   └── algorithm.fr.md  ← le pipeline d'analyse en détail (FR)
├── patch/               ← les patchs Max/MSP (.maxpat)
├── paper/               ← l'essai conceptuel (PDF, en français)
├── LICENSE              ← CC BY-NC-SA 4.0
├── CITATION.cff
└── .gitignore
```

## Auteur

**Antoine Goudreau** ([@epphone](https://github.com/epphone))

## Remerciements

Bâti sur le travail remarquable de :

- **Andrea Agostini** et **Daniele Ghisi** — [Bach, Cage, Dada](https://www.bachproject.net/)
- **Miller Puckette** — [fiddle~](https://vboehm.net/downloads/), porté vers Max 64 bits par **Volker Böhm**
- **Mikhail Malt** et **Emmanuel Jourdan** — [zsa.descriptors](http://www.e--j.com/index.php/what-is-zsa-descriptors/)

Le cadre conceptuel est redevable à Gilles Deleuze (avec Félix Guattari, Henri
Bergson et Marcel Proust), ainsi qu'au *Traité des objets musicaux* de Pierre
Schaeffer.

## Licence

Ce dépôt — code, documentation et essai — est sous licence
[Creative Commons Attribution - Pas d'Utilisation Commerciale - Partage dans les
Mêmes Conditions 4.0 International](LICENSE)
([CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.fr)) :
partage et adaptation libres avec attribution, à des fins non commerciales
uniquement, et les adaptations doivent être diffusées sous les mêmes conditions.
