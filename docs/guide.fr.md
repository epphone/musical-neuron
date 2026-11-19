# Guide d'installation et d'usage

[Français](guide.fr.md) | [English](guide.md)

## Prérequis

| Composante | Rôle | Lien |
|---|---|---|
| Max/MSP (64 bits) | environnement de programmation visuelle | <https://cycling74.com/products/max> |
| Bach, Cage, Dada | librairies de notation et d'analyse musicale | <https://www.bachproject.net/> |
| fiddle~ | suivi de hauteur en temps réel | <https://vboehm.net/downloads/> |
| zsa.descriptors | mesures de distance/similarité | <http://www.e--j.com/index.php/what-is-zsa-descriptors/> |

## 1. Installer Max/MSP

Installez la version 64 bits de Max/MSP depuis Cycling '74. Le patch a été
développé avec Bach/Cage/Dada, qui ciblent les versions récentes de Max.

## 2. Installer les librairies Bach, Cage et Dada

1. Téléchargez le dernier installeur unifié depuis <https://www.bachproject.net/>
   (la famille bach+ cage+ dada+ est distribuée ensemble).
2. Suivez les instructions officielles d'installation pour votre plateforme.
3. Redémarrez Max et vérifiez que les objets apparaissent (ex. tapez
   `bach.roll` dans une nouvelle boîte d'objet).

## 3. Installer fiddle~ et zsa.descriptors

1. **fiddle~** : téléchargez l'externe 64 bits de Volker Böhm depuis
   <https://vboehm.net/downloads/> et placez-la dans le chemin de recherche de
   Max (ex. le dossier `externals` de l'installation de Max, ou un dossier
   listé dans *Options ▸ File Preferences*).
2. **zsa.descriptors** : téléchargez la librairie depuis
   <http://www.e--j.com/index.php/what-is-zsa-descriptors/> et installez-la de
   la même manière.

Redémarrez Max pour que les externes soient détectées, puis vérifiez que
`[fiddle~]` et `[zsa.dist]` s'instancient correctement.

## 4. Ouvrir le patch

Ouvrez le fichier `.maxpat` du dossier [`patch/`](../patch/). Assurez-vous que
les dossiers des librairies Bach/Cage/Dada sont dans le chemin de recherche
de Max.

## 5. Configuration audio et MIDI

- Connectez votre instrument (micro ou capteur) à l'entrée audio du patch.
  Un signal monophonique propre donne la meilleure transcription.
- La transcription est déclenchée manuellement. Dans l'itération actuelle,
  utilisez la **barre d'espace** ; en performance, une **pédale MIDI** est le
  déclencheur recommandé (idéalement pour lancer et arrêter l'analyse).

## Jouer avec l'outil

Le contexte envisagé est une pièce improvisée pour instrument à vent seul,
mais l'outil peut aussi servir à la composition à l'extérieur d'une
performance.

1. **Inspirez, lancez l'analyse** (barre d'espace / pédale) : la transcription
   est en direct.
2. **Improvisez une phrase en un seul souffle**, en suivant le dessin d'une
   ligne correspondant à la variation de hauteur sur toute la durée de
   l'épuisement du souffle.
3. **Respirez** : pendant votre inspiration, l'analyse s'exécute. Elle est
   assez rapide pour se terminer au cours de la respiration, de sorte que
   vous pouvez reprendre immédiatement.
4. **Lisez la ligne réduite** : une version simplifiée de votre phrase
   apparaît en notation spatiale — notes tenues et glissandos.
5. **Réimprovisez par-dessus**, avec le degré de fidélité que vous voulez.
   Ne cherchez pas à la reproduire exactement : le floutage est une
   fonctionnalité, pas un défaut.
6. **Répétez** pour autant d'itérations que vous voulez. La ligne continue
   d'évoluer.

### Conventions d'improvisation (présupposées par l'outil)

L'algorithme s'attend à des phrases qui respectent quelques contraintes
musicales. Elles ne restreignent pas la liberté de l'interprète : elles
définissent simplement le genre de matériel pour lequel l'analyse est conçue :

- des phrases **continues**, sans arrêt ni pause entre les notes ;
- une **alternance de notes longues tenues et de courts passages en traits**
  (de gamme ou de mode, possiblement ornementés), favorisant le **mouvement
  conjoint** (secondes, tierces). Les sauts mélodiques plus grands restent
  tout à fait permis ;
- un **rythme libre** : sans mesure ni tempo précis.

### Paramètres

- **Arrondi des hauteurs** : au demi-ton par défaut ; paramétrable au quart ou
  au huitième de ton.
- **Durées des notes à la transcription** : puisque `[fiddle~]` n'envoie la
  hauteur qu'à l'attaque de la note, chaque note est transcrite avec une durée
  arbitraire (1000 ms), puis mise en legato (`sel all, legato` sur le roll).
- **Nombre de tranches** : la phrase est subdivisée selon sa durée totale
  divisée par la moyenne des durées des notes — approximativement une tranche
  par unité de type mesure.

### Conseils de performance

- **Faites confiance au flou.** La valeur de l'outil réside dans ce qu'il
  oublie. Si vous rejouez la ligne réduite trop fidèlement, la boucle
  humain–machine risque de stagner musicalement (une *boucle d'hystérésis*) :
  variez votre fidélité d'une itération à l'autre.
- L'analyse conçoit ses propres unités mélodiques — attendez-vous à ce que
  les appoggiatures soient absorbées dans la note ou le trait qui les suit.
