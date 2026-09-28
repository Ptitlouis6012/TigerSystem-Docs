---
sourceHash: b276fd9b17e276371d05b73a8ea59cc00ea622ada212c56e384580b7a1e04203
sourcePath: docs/developers/material-swatch.md
---

# La pastille de matière — la convention officielle

La couleur d'une matière est stockée sous forme de données, pas d'image. La
**pastille** est le dessin de ces données : la forme colorée qui représente la
matière dans une liste, sur une carte, dans un emplacement de rack, à côté de
son nom. Cette page est la **convention officielle TigerSystem** pour la
produire, afin que la même matière affiche la même pastille partout — dans
Tiger Studio, dans l'application mobile, sur le web, dans votre propre
intégration.

**« Matière », et non « bobine », est un choix délibéré.** Un TigerTag
identifie une bobine de filament, mais aussi un accessoire, une pièce détachée,
une résine — et les types de produits que le protocole n'a pas encore. La
convention est écrite pour `id_type` en général : rien de ce qui suit ne lit le
type de produit, si bien qu'une pastille se produit de la même façon quelle que
soit la matière.

Elle est normative. Un filament bicolore qui affiche un dégradé fondu dans une
application et une séparation diagonale franche dans une autre est un bug dans
celle qui s'est écartée de cette page — ce n'est pas une affaire de goût. Si
vous ne pouvez pas reproduire une règle à l'identique (une plateforme sans
dégradé conique, par exemple), implémentez l'équivalent le plus proche décrit
dans [Plateformes non-CSS](#plateformes-non-css) et dites-le ; n'inventez pas une
autre image.

- **Version de la convention :** 1.2 — le bicolore et le tricolore sont un **dégradé à 135°** (la 1.1 dessinait le bicolore en séparation diagonale franche et le tricolore en camembert ; la 1.0 séparait le bicolore verticalement)
- **Moteur de rendu de référence :** [`material-swatch-playground.html`](./material-swatch-playground.html) —
 ouvrez-le dans n'importe quel navigateur, sans serveur ni dépendance. Tous les
 cas, toutes les formes de boîte, des sélecteurs de couleur en direct, et le CSS
 exact qu'il produit.

---

## Trois formes, et trois seulement

| Forme | Quand | Géométrie |
|---|---|---|
| **Dégradé** | **Deux ou trois** couleurs à bords francs — bicolore, tricolore — plus rainbow et le type `gradient` déclaré par le catalogue | Un dégradé linéaire lisse à **135°**, sans arête franche — orienté vers le bas à droite, la première couleur se trouvant donc en haut à gauche |
| **Camembert** (part de tarte) | **Quatre couleurs ou plus** à bords francs — toute liste de N ≥ 4 | N secteurs coniques égaux, la première couleur commençant à **midi**, balayage **dans le sens horaire** |

Dit simplement : **deux ou trois couleurs franches se fondent en un dégradé,
quatre ou plus forment un camembert** — et il existe un seul angle dans tout le
système, 135°, partagé par tous les dégradés.

**Le bicolore et le tricolore sont un dégradé — la même forme qu'un dégradé
déclaré.** Même axe, même ordre (première couleur en haut à gauche), aucune
arête franche nulle part : 2 ou 3 couleurs franches se fondent exactement comme
un `gradient` de catalogue de même longueur. Implémentez-le une seule fois,
dans la fonction qui traite une liste de couleurs (N ≤ 3 → dégradé, N ≥ 4 →
camembert), pour que tous les appelants en profitent ; ne le dessinez jamais en
miroir.

Pourquoi un dégradé pour deux ou trois couleurs : une arête franche se lisait
encore comme deux ou trois objets distincts — sur une vignette, et plus encore
dans le cadre coloré autour d'une photo produit, où la photo masquait le centre
et ne laissait que des barres de couleur disjointes. Un dégradé se lit comme une
seule matière multi-teinte, quelle que soit la forme de la boîte. Pourquoi un
camembert à partir de quatre couleurs : un dégradé sur quatre couleurs ou plus
devient boueux, alors que les frontières de secteurs restent angulaires et
reconnaissables — mesurées depuis le centre de la boîte, donc N secteurs
restent lisibles quel que soit le rapport d'aspect de la boîte. Pourquoi 135° :
c'est l'unique angle du système, partagé par tous les dégradés, qu'ils aient 2,
3 arrêts ou plus.

---

## D'où viennent les données

Deux couches alimentent le rendu, et ce ne sont pas la même chose.

### La puce — trois couleurs au maximum

Un TigerTag porte **trois emplacements de couleur et un aspect**. Rien
d'autre : il n'y a pas de type de dégradé sur la puce, et il n'y en a jamais eu.

| Champ | Type | Signification |
|---|---|---|
| `color_r` / `color_g` / `color_b` | `int 0-255` | Emplacement 1 |
| `color_r2` / `color_g2` / `color_b2` | `int 0-255` | Emplacement 2 |
| `color_r3` / `color_g3` / `color_b3` | `int 0-255` | Emplacement 3 |
| `color_a` | `int 0-255` | Alpha — **ignoré pour le rendu** ; ne mélangez jamais une couleur de matière |
| `id_aspect1` / `id_aspect2` | `int` | L'un ou l'autre emplacement peut porter l'aspect de coloration |

Trois identifiants d'aspect changent la forme (leur table de référence porte
également un `color_count` faisant autorité) :

| id | label | `color_count` | Forme |
|---|---|---|---|
| `252` | Bicolor | 2 | **Dégradé**, 135° |
| `24` | Tricolor | 3 | **Dégradé**, 135° |
| `145` | Rainbow | 3 | Dégradé, 135° |

Tout autre aspect (`Silk`, `Matt`, `Glitter`, …) a un `color_count` ≤ 1 et
n'affecte pas la forme. **Faites la correspondance sur l'id, pas sur le
libellé** — les libellés sont des chaînes d'affichage et peuvent être traduits.

> **Ne comptez jamais les emplacements pour deviner le nombre de couleurs.** Un
> document de puce porte toujours les trois emplacements : les composantes
> absentes sont stockées à `0`, si bien que les emplacements 2 et 3 se lisent
> comme du noir pur sur une matière monochrome. **Le nombre de couleurs vient
> de l'aspect, jamais des emplacements.**

### Le catalogue — une description plus riche, uniquement dans le cloud

Un produit du catalogue officiel peut décrire sa couleur plus précisément
qu'une puce ne le peut. Ces deux champs existent **uniquement** dans les
données cloud/produit — ils ne sont jamais écrits sur une puce :

| Champ | Type | Signification |
|---|---|---|
| `online_color_list` | `string[]` | Couleurs ordonnées, `RRGGBB` ou `RRGGBBAA`, `#` facultatif. L'ordre a du sens : l'index 0 est le premier secteur / le premier arrêt. |
| `online_color_type` | `string` | Instruction de rendu : `mono`, `multi`, `gradient`, `conic_gradient`. Toute autre valeur, ou son absence, est traitée comme `multi`. |

Lorsque les deux couches sont présentes, **le catalogue l'emporte** — c'est la
description la plus précise du même produit.

---

## L'échelle de décision

Évaluez **dans cet ordre, la première correspondance l'emporte**. L'ordre
encode une priorité : le catalogue prime sur la puce, un type de couleur
explicite prime sur une supposition, et l'aspect ne parle que lorsqu'il n'y a
pas de liste de couleurs en ligne.

Soit `LIST` = `online_color_list` après [normalisation](#normalisation), `TYPE` =
`online_color_type`, `SLOTS` = les emplacements non nuls de la puce.

| # | Condition | Résultat |
|---|---|---|
| 1 | `LIST ≥ 2` et `TYPE == "conic_gradient"` | Balayage conique lisse, se refermant sur la première couleur |
| 2 | `LIST ≥ 2` et `TYPE == "gradient"` | Dégradé — même forme que `LIST` obtiendrait de toute façon à 2-3 couleurs ; le type déclaré ne change quelque chose qu'à partir de `LIST.length ≥ 4` |
| 3 | `LIST ≥ 2` | **Dégradé** de `LIST` avec 2 ou 3 couleurs, **camembert** de `LIST.length` secteurs avec 4 ou plus |
| 4 | `LIST == 1` | Couleur unie — **prime sur la couleur de la puce** |
| 5 | aspect **Rainbow** *et* **Tricolor** | Dégradé, 3 arrêts |
| 6 | aspect **Rainbow** *et* **Bicolor** | Dégradé, 2 arrêts |
| 7 | aspect **Rainbow** | Dégradé sur `SLOTS` ; 1 emplacement → uni ; 0 emplacement → les 6 couleurs par défaut |
| 8 | aspect **Tricolor** | **Dégradé**, 135°, sur `SLOTS` (emplacement 3 manquant → dégradé à 2 couleurs sur les emplacements 1 et 2, sans répéter l'emplacement 1) |
| 9 | aspect **Bicolor** | **Dégradé**, 135°, sur les emplacements 1 et 2 |
| 10 | sinon | Emplacement 1 uni ; rien du tout → `#1c2030` |

Valeurs par défaut lorsqu'un aspect ne porte aucune couleur utilisable :

| Cas | Valeurs par défaut |
|---|---|
| Rainbow, aucune couleur | `#ff0000 #ff8800 #ffff00 #00cc00 #0000ff #8b00ff` |
| Rainbow + Tricolor | `#ff4d4d #ffd93d #4da3ff` |
| Rainbow + Bicolor | `#ff7a00 #8a2be2` |
| Tricolor | `#cccccc #888888` (emplacement 3 manquant → dégradé à 2 couleurs, l'emplacement 1 n'est pas répété) |
| Bicolor | `#cccccc #ffffff` |
| Rien | `#1c2030` |

---

## Normalisation

Appliquée à chaque entrée de `online_color_list` avant l'évaluation de
l'échelle :

1. Supprimez les espaces autour, retirez le `#` initial.
2. Si 8 caractères (`RRGGBBAA`), **gardez les 6 premiers** — l'alpha est
 abandonné, jamais mélangé.
3. N'acceptez que `^[0-9a-fA-F]{6}$`. **Tout le reste est retiré de la liste**,
 et non remplacé par une valeur par défaut — une entrée malformée ne doit jamais
 devenir noire en silence.
4. Rajoutez le `#` en sortie.

Le retrait a lieu *avant* l'exécution de l'échelle : `["ff0000", "oops"]` est
donc une liste à **une seule couleur** (règle 4), et non un camembert à deux
secteurs.

Les emplacements de la puce se convertissent en `#` + deux chiffres
hexadécimaux par composante, uniquement lorsque les trois composantes sont des
nombres.

---

## Les expressions exactes

Avec `c1…cN` les couleurs normalisées et `step = 360 / N` :

```css
/* Ramp — 2 or 3 hard colours: rule 3 with 2-3 colours, rules 5-9. Same shape
   as a declared gradient, one angle for every ramp in the system. */
linear-gradient(135deg, c1, c2, …)

/* Camembert — four or more hard colours: rule 3 with ≥ 4 colours */
conic-gradient(c1 0deg <step>deg, c2 <step>deg <2·step>deg, …)

/* Catalogue-declared conic gradient — rule 1 (first colour repeated to close) */
conic-gradient(from 0deg, c1, c2, …, c1)

/* Mono — rules 4, 10 */
# RRGGBB
```

### Vecteurs de test

Toute implémentation doit les reproduire exactement.

| Entrée | Attendu |
|---|---|
| `{online_color_list:["FF5722"]}` | `#FF5722` |
| `{online_color_list:["000000FF"]}` | `#000000` |
| `{online_color_list:["e02424","2463e0"]}` | `linear-gradient(135deg, #e02424, #2463e0)` |
| `{online_color_list:["e02424","2463e0","22a06b"]}` | `linear-gradient(135deg, #e02424, #2463e0, #22a06b)` |
| `{online_color_list:["e02424","2463e0","22a06b","f0a020"]}` | `conic-gradient(#e02424 0deg 90deg, #2463e0 90deg 180deg, #22a06b 180deg 270deg, #f0a020 270deg 360deg)` |
| `{online_color_list:["e02424","2463e0"],online_color_type:"gradient"}` | `linear-gradient(135deg, #e02424, #2463e0)` |
| `{color_r:224,color_g:36,color_b:36,color_r2:36,color_g2:99,color_b2:224,id_aspect2:252}` | `linear-gradient(135deg, #e02424, #2463e0)` |
| `{id_aspect1:145}` | `linear-gradient(135deg, #ff0000, #ff8800, #ffff00, #00cc00, #0000ff, #8b00ff)` |
| `{}` | `#1c2030` |

---

## Le filigrane TigerTag

Toute surface qui peint une couleur de matière **sans** photo de produit porte
le logo TigerTag par-dessus, en filigrane.

| Règle | Valeur |
|---|---|
| Position | Coin supérieur droit de la tuile |
| Opacité | **1** — toujours, sur toutes les surfaces |
| Taille | Un **pourcentage** de la tuile, jamais une taille fixe en pixels, pour qu'il s'adapte à la surface |
| Variante | **Fond sombre → le logo BLANC plein. Fond clair → le logo NOIR avec contour.** |

Les deux fichiers de logo ne sont **pas des variantes teintables l'une de
l'autre** — chacun est livré avec son propre remplissage intégré, et la règle
porte sur le *fichier* à utiliser. N'appliquez jamais un filtre CSS, une
couleur de masque ou une opacité pour faire passer l'un pour l'autre : le
dessin avec contour est un autre dessin, pas l'inverse du blanc.

**Choisir la variante** — prenez la luminance relative de la **première
couleur** de l'expression produite :

```
luminance = (0.299·R + 0.587·G + 0.114·B) / 255   // dark when < 0.5
```

Si vous extrayez cette couleur en cherchant le premier `#` hexadécimal d'une
chaîne CSS, cherchez d'abord **8 chiffres**, puis 6, puis 4, puis 3. Sinon un
`#RRGGBBAA` correspond sur ses six premiers chiffres, échoue sur la limite de
mot qui suit, et toute la correspondance est perdue — ce qui se lit comme
« clair » et pose un logo noir sur une bobine noire. Retirez l'alpha et
développez la notation abrégée avant le calcul.

---

## Plateformes non-CSS

Flutter, SwiftUI, Android ou tout moteur de rendu sur canvas implémentent les
trois mêmes formes. Les angles se mesurent toujours à la manière du CSS : **0°
pointe vers le haut, les angles augmentent dans le sens horaire.**

- **Camembert** — un dégradé balayé centré sur la tuile, démarrant à midi, dans
 le sens horaire, avec des arrêts francs à chaque `k · 360/N` degrés. Flutter :
 `SweepGradient` avec `transform: GradientRotation(-pi/2)`. Ne l'approximez pas
 par des parts dessinées comme des tracés, sauf si la tuile est carrée — les
 frontières des secteurs doivent suivre la boîte. Uniquement à partir de
 **quatre** couleurs franches.
- **Dégradé** — un dégradé linéaire du coin **supérieur gauche** au coin
 **inférieur droit**, arrêts régulièrement espacés, sans arête franche. Utilisé
 pour **deux ou trois** couleurs franches (bicolore, tricolore), pour rainbow,
 et pour le type `gradient` déclaré par le catalogue. Flutter :
 `Alignment.topLeft → Alignment.bottomRight`.

---

## Faire évoluer la convention

Cette page fait foi. Un changement ici est un changement sur toutes les
surfaces TigerSystem : modifiez-la d'abord, incrémentez la version de la
convention, puis réalignez les implémentations et vérifiez-les avec le moteur
de rendu de référence.

---

**▲ [Index de la documentation](../../README.md)** · **Voir aussi :** [La puce TigerTag](../concepts/tigertag-chip.md), [Le format `.ttag`](./ttag-format.md), [Vue d'ensemble développeurs](./README.md)
