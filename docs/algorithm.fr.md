# Le pipeline d'analyse en détail

[Français](algorithm.fr.md) | [English](algorithm.md)

Ce document décrit les étapes successives effectuées par le neurone musical
entre deux phrases de l'interprète. Il suit (et abrège) l'essai conceptuel
dans [`../paper/`](../paper/README.md).

## 0. Transcription

- `[fiddle~]` (portage 64 bits par Volker Böhm, d'après Miller Puckette) suit
  la hauteur et l'amplitude du signal. La transcription est armée par
  l'interprète avec un `[toggle]` — la **barre d'espace** dans l'itération
  actuelle, idéalement une **pédale MIDI** en performance.
- Les hauteurs sont arrondies au demi-ton près ; l'arrondi peut être
  paramétré pour les quarts ou les huitièmes de ton.
- Les amplitudes respectives de chaque note sont retenues et converties en
  vélocités MIDI, utilisées plus tard par l'analyse de distribution.
- `[fiddle~]` n'envoie la hauteur qu'à l'attaque de la note, alors que
  `[bach.transcribe]` exige hauteur, vélocité *et* durée dans le même message,
  selon les conventions du protocole MIDI. Chaque note est donc transcrite
  avec une durée arbitraire (1000 ms), et à la fin de la phrase le message
  `sel all, legato` est envoyé au roll afin que chaque note se termine à
  l'arrivée de la suivante — en supposant une ligne mélodique sans brisure.

## 1. Placement des marqueurs selon la distribution rythmique

Sous-patchs : `#-sub-divisions`, `edge-detection-rhythm`, `round2onsets`,
`add-markers`.

- `[dada.analysis.rhythmdistrib]` calcule la distribution des notes sur la
  durée de la mélodie selon la proximité des notes les unes aux autres et
  selon les poids attribués aux notes en fonction de leurs vélocités MIDI. Il
  découpe le roll en **N tranches de même durée**, N étant la durée totale de
  la mélodie divisée par la moyenne des durées de ses notes — une unité
  comparable à une mesure, analogue à un antécédent ou à un conséquent dans
  une phrase. Puisque l'analyse se base sur les *attaques* des notes, les
  traits se retrouvent jumelés et les notes tenues demeurent intactes : un
  floutage grossier mais utile des détails rythmiques.
- La liste produite est ensuite réduite à des valeurs binaires d'activité :
  toute valeur supérieure à 0.0 devient 0, tous les 0 deviennent 1
  (`[!- 0.]`), et `[togedge]` ne garde que les transitions 0→1 — la frontière
  entre activité et suspension.
- Les marqueurs placés par `add-markers` sont calés sur les temps d'entrée des
  notes les plus proches avec `[bach.nearest]`, de sorte qu'ils coïncident
  précisément avec les débuts des traits et des notes tenues.

Cet étirement de la notion de tranche (*slice*) de `[cage.slice]` correspond
à ce que l'essai nomme les unités mélodiques de base : lignes stationnaires
(notes tenues), lignes ascendantes ou descendantes.

## 2. Division de la mélodie

Avec les marqueurs en place, `[cage.slice]` divise la mélodie en plusieurs
rolls. Dans le sous-patch `process-slices`, la première note de chaque tranche
sauf la première est enlevée — une correction nécessaire, puisque
`[cage.slice]` retient la toute fin de la dernière note de la tranche
précédente au début de chaque nouvelle tranche.

## 3. Traitement des petites tranches

Sous-patchs : `sous-mediane`, `process-below`.

Même après le passage de `round2onsets`, la correction des positions reste
trop imprécise pour correspondre parfaitement aux débuts des unités
mélodiques : la première étape produit fréquemment des tranches d'une ou deux
notes. Ces petites tranches précèdent souvent des traits de notes ; quand
elles précèdent une note tenue, elles sont considérées comme des
**appoggiatures**, que la perception fusionne au transitoire de l'événement
plus long. Dans les deux cas, la petite tranche est jointe à la tranche
*suivante*.

- `sous-mediane` recueille les durées respectives des tranches et calcule leur
  **médiane** (plutôt que la moyenne, qui serait biaisée par les valeurs
  aberrantes). Une tranche ne compte comme *petite* que si elle est
  significativement plus courte que la médiane : le seuil est la médiane
  divisée par 2. Le sous-patch produit la liste des adresses des tranches
  retenues.
- `process-below` accouple chaque petite tranche à la tranche qui la suit,
  et `[bach.subs]` remplace le couple par un seul roll dans la liste des
  tranches.

## 4. Traitement des grandes tranches

Sous-patch : `process-above`.

- La médiane est recalculée (le nombre de tranches a pu changer), et une
  tranche compte comme *grande* si sa durée dépasse la médiane × 1,5.
- De longs traits de vitesse quasi constante peuvent se cacher dans une
  grande tranche. Pour ramener les grandes tranches à une taille comparable à
  la médiane, `process-above` les analyse pour y trouver des **grands sauts
  mélodiques** (intervalles équivalents ou supérieurs à la quarte juste), y
  ajoute des marqueurs et divise la tranche en plusieurs rolls. Découper aux
  sauts prépare la réduction mélodique et évite les simplifications trop
  grossières des longs traits.

## 5. Réduction mélodique

Sous-patchs : `>2-notes`, et la comparaison des profils.

- `>2-notes` sépare les rolls comportant moins de deux notes (notes tenues
  simples, avec ou sans appoggiature) de ceux contenant de véritables traits —
  seuls ces derniers nécessitent une simplification notationnelle.
- Les traits ont généralement un profil global ascendant ou descendant, ou
  composite. Pour trancher, chaque trait est comparé à des **profils de
  parcours mélodique idéalisés**, adaptés des classes de profils mélodiques du
  TARSOM de Pierre Schaeffer, elles-mêmes basées sur les neumes élémentaires
  du chant grégorien :
  - **clivis** (ascendant),
  - **podatus** (descendant),
  - **torculus** (ascendant, puis descendant),
  - **porrectus** (descendant, puis ascendant).
- `[zsa.dist]` exige des listes de longueurs équivalentes : les profils sont
  donc **générés dynamiquement** — des séries arithmétiques allant de 0.0 à
  1.0 (clivis, podatus) et de −1.0 à 1.0 (torculus, porrectus), remises à
  l'échelle au nombre de notes de la tranche — puis `[zsa.dist]` choisit le
  modèle gagnant.
- Pendant la comparaison, les points saillants sont retenus : **première
  note, nadir, apogée, dernière note**. Ils recomposent la ligne réduite ; le
  dessin des glissandos de bach remplace le trait lui-même, en gardant
  toujours ses première et dernières notes. Seuls *torculus* et *porrectus*
  ont aussi besoin des extrêmes de la ligne.
- L'interprète lit ensuite cette notation spatiale et réimprovise
  par-dessus, relançant le cycle.

## Problèmes connus et feuille de route

L'outil est en stade **alpha**. Les points suivants proviennent de la
discussion de l'essai.

- **Distribution basée sur les attaques** : `[dada.analysis.rhythmdistrib]`
  se réfère aux *attaques* des notes, non à l'espace qu'elles occupent. Une
  représentation tenant compte des durées totales des notes rendrait
  superflus `edge-detection-rhythm` et les corrections de `round2onsets` et
  `add-markers`.
- **Précision de la réduction mélodique** : les quatre profils sont un choix
  plutôt arbitraire. Prévus : un modèle de ligne stationnaire (pour les
  lignes qui tournent autour d'un même pôle harmonique — trilles, lignes
  vacillantes proches du vibrato) et des lignes droites avec montée ou
  descente en première ou deuxième moitié de parcours, qui devraient aider
  `[zsa.dist]` à trouver plus facilement *torculus* et *porrectus*.
- **Ordonnancement des données** : les fusions et suppressions répétées par
  `[bach.subs]` sont coûteuses dans Max et ne fonctionnent pas de manière
  fiable à chaque exécution. Une optimisation globale du patch est requise,
  voire une autre manière de décomposer et recomposer la mélodie que
  `[cage.slice]`.
- **Tranches minuscules de `process-above`** : elles devraient être fusionnées
  à une tranche adjacente ; cette étape n'est pas encore implémentée.
- **Analyses alternatives** : cette itération repose sur une « distribution
  énergétique » des notes ; d'autres méthodes d'analyse de données sont
  envisageables et demanderaient une recherche plus profonde.
- **Détection de motifs** : les motifs répétés et transposés ne sont pas
  encore détectés ; la méthode prévue emprunte à la notation graphique (une
  boîte noire contenant le motif, prolongée par une ligne ascendante,
  descendante ou planante).
- **Modulation de l'analyse en cours de route** : une option pour changer le
  type d'analyse à la volée (ex. un style pointilliste) ajouterait un retour
  numérique dynamique au système.
- **Sauts mélodiques tôt dans le pipeline** : les sauts ne sont utilisés
  que juste avant la réduction mélodique, alors qu'ils me semblent
  signifiants dès les premières étapes d'analyse pour des phrases
  privilégiant le mouvement conjoint.
- **Séquentialité** : fondamentalement, l'ordinateur attend toujours les
  données avant de rendre son résultat ; l'interprète ne peut pas changer
  l'analyse en cours d'exécution. Le dynamisme est technologique (la partition
  comme technologie), pas numérique.
