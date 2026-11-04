# Chapitre 16 – Plans particuliers : imbriqué, carré latin et ANCOVA

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] réaliser une ANOVA imbriquée avec `aov()`
- [ ] réaliser une ANOVA en carré latin
- [ ] réaliser une ANCOVA avec une covariable


> **Contexte :** en bureau d'études, les données ne sont pas toujours croisées proprement. Des DPE sont faits par des chargés d'études, eux-mêmes rattachés à des bureaux : les chargés sont imbriqués dans les bureaux. Un essai de traitement d'isolation peut être réparti entre des chantiers et des équipes, à croiser avec le type d'isolant. Une consommation dépend aussi de la surface du logement : c'est une covariable. Ces trois situations demandent trois plans différents.

## Sommaire

- [Chapitre 16 – Plans particuliers : imbriqué, carré latin et ANCOVA](#chapitre-16--plans-particuliers--imbriqué-carré-latin-et-ancova)
  - [Objectifs](#objectifs)
  - [Sommaire](#sommaire)
  - [Préparation : packages et données](#préparation--packages-et-données)
  - [Démarche : choisir le plan](#démarche--choisir-le-plan)
  - [Un exemple pour comprendre : la croissance des plantes](#un-exemple-pour-comprendre--la-croissance-des-plantes)
    - [Plan imbriqué : les champs dans les régions](#plan-imbriqué--les-champs-dans-les-régions)
    - [Carré latin : contrôler la position et le jour](#carré-latin--contrôler-la-position-et-le-jour)
    - [ANCOVA : tenir compte de la taille initiale](#ancova--tenir-compte-de-la-taille-initiale)
  - [Exercice 1 – ANOVA imbriquée](#exercice-1--anova-imbriquée)
    - [Mémo](#mémo)
    - [Exemple](#exemple)
    - [Exercices](#exercices)
  - [Exercice 2 – ANOVA en carré latin](#exercice-2--anova-en-carré-latin)
    - [Mémo](#mémo-1)
    - [Exemple](#exemple-1)
    - [Exercices](#exercices-1)
  - [Exercice 3 – ANCOVA : consommation et surface](#exercice-3--ancova--consommation-et-surface)
    - [Mémo](#mémo-2)
    - [Exemple](#exemple-2)
    - [Exercices](#exercices-2)
  - [Exercice 4 – Choisir le plan](#exercice-4--choisir-le-plan)
  - [Liens utiles](#liens-utiles)

## Préparation : packages et données

```r
library(readr)
library(dplyr)
library(ggplot2)
library(tibble)

dpe <- read_csv("data/dpe_villeurbanne_10000.csv")
```

Les exemples 1 et 2 utilisent de **petits jeux de données fictifs**, écrits à la main. Ils servent à comprendre la structure du plan : les valeurs ne décrivent pas de vrais bâtiments. L'exemple 3 utilise les vrais DPE.

> **Note : données fictives**
> Les valeurs des exemples 1 et 2 sont inventées pour la démonstration. Elles ne doivent pas être citées comme résultats sur le parc de Villeurbanne.

## Démarche : choisir le plan

| Plan | Question | Structure | Test |
|------|----------|-----------|------|
| Imbriqué (hiérarchique) | Le facteur fin (chargé) est-il lié au facteur global (bureau) ? | chargé à l'intérieur du bureau | bureau contre chargé imbriqué ; chargé contre résiduelle |
| Carré latin | L'isolant a-t-il un effet, une fois contrôlés le chantier et l'équipe ? | 3 traitements, 3 lignes, 3 colonnes, chaque traitement une fois par ligne et par colonne | chaque facteur contre la résiduelle |
| ANCOVA | La période a-t-elle un effet, une fois la surface prise en compte ? | 1 facteur + 1 covariable numérique | covariable, facteur et interaction contre la résiduelle |

Trois règles communes :

- on pose $H_0$ et $H_1$ pour chaque effet, avec $\alpha = 0{,}05$ fixé avant le calcul ;
- on vérifie d'abord les hypothèses propres au plan (indiquées dans chaque exercice) ;
- la p-value se lit ensuite dans la table de Fisher (chapitre 14) ou se calcule avec `pf()`.

> **Note : le dénominateur change selon le plan**
> Dans un plan imbriqué, le facteur global ne se teste pas contre la résiduelle, mais contre le facteur imbriqué. C'est l'erreur la plus fréquente, et elle donne une p-value trop petite.

## Un exemple pour comprendre : la croissance des plantes

Avant les données de l'ADEME, on suit un exemple simple : la croissance de plantes (en cm, après 30 jours) selon le type d'engrais. Les données sont fictives et volontairement petites, pour pouvoir refaire chaque calcul à la main.

### Plan imbriqué : les champs dans les régions

**Question :** deux facteurs ont-ils un effet, sachant que les modalités de l'un sont propres aux modalités de l'autre ?

Des plantes poussent dans deux régions (A et B), avec trois champs par région, et trois plantes par champ. Attention : le champ 1 de la région A n'est pas le même que le champ 1 de la région B. Les champs sont donc **imbriqués** dans les régions : on les note A1, A2, A3, B1, B2, B3.

| Région | Champ | Croissance (cm) | Moyenne du champ |
|---|---|---|---|
| A | 1 | 12 ; 14 ; 13 | 13,0 |
| A | 2 | 16 ; 15 ; 17 | 16,0 |
| A | 3 | 11 ; 12 ; 13 | 12,0 |
| B | 1 | 18 ; 19 ; 17 | 18,0 |
| B | 2 | 15 ; 16 ; 14 | 15,0 |
| B | 3 | 20 ; 19 ; 21 | 20,0 |

Les moyennes des régions sont 13,7 (A) et 17,7 (B) ; la moyenne générale vaut 15,7.

La variabilité se découpe ainsi :

**Entre régions** (1 ddl) :

$$
\text{SCE}_{\text{région}} = 9 \times [(13{,}7 - 15{,}7)^2 + (17{,}7 - 15{,}7)^2] = 72{,}0
$$

**Entre champs, à l'intérieur d'une région** (4 ddl, soit 2 × (3 − 1)) :

$$
\text{SCE}_{\text{champ}} = 3 \times \sum (\bar{y}_{\text{champ}} - \bar{y}_{\text{région}})^2 = 64{,}0
$$

**Résiduelle**, c'est-à-dire les plantes dans un même champ (12 ddl, soit 6 × 2) :

$$
\text{SCE}_E = 12{,}0
$$

Le total vaut $72 + 64 + 12 = 148$, pour 17 ddl. Les carrés moyens sont : région 72,0 ; champ 16,0 ; résiduelle 1,0.

Chaque facteur se teste contre celui qui le contient.

**Région** (ddl $(1 ; 4)$, p = 0,10 : pas significatif) :

$$
F = \frac{\text{CM}_{\text{région}}}{\text{CM}_{\text{champ}}} = \frac{72}{16} = 4{,}5
$$

**Champ** (ddl $(4 ; 12)$, p = 9,4e-5 : significatif) :

$$
F = \frac{\text{CM}_{\text{champ}}}{\text{CM}_E} = \frac{16}{1} = 16
$$

**Ce que l'exemple montre :** les plantes poussent très différemment d'un champ à l'autre, mais la différence entre régions n'est pas établie avec seulement deux régions. Le piège est de tester la région contre la résiduelle : on obtiendrait $F = 72$, une p-value minuscule, et une conclusion fausse, puisque les champs ne sont pas les mêmes d'une région à l'autre.

> **Hypothèses à vérifier** (non vérifiées dans cet exemple, pour ne pas alourdir le chapitre) : les résidus sont normaux et de même variance dans chaque champ.

### Carré latin : contrôler la position et le jour

**Question :** comment comparer plusieurs traitements tout en contrôlant deux autres sources de variation ?

On teste 4 engrais (A, B, C, D) sur des plantes. Mais on sait que la position dans la serre et le jour de mesure influencent aussi la croissance. Le carré latin range les engrais de sorte que chacun apparaisse une fois par position et une fois par jour.

| | Jour 1 | Jour 2 | Jour 3 | Jour 4 |
|---|---|---|---|---|
| Position 1 | A : 12 | B : 15 | C : 18 | D : 14 |
| Position 2 | B : 16 | C : 19 | D : 13 | A : 11 |
| Position 3 | C : 17 | D : 12 | A : 10 | B : 14 |
| Position 4 | D : 13 | A : 9 | B : 15 | C : 16 |

Les moyennes par engrais sont A 10,5 ; B 15,0 ; C 17,5 ; D 13,0 ; la moyenne générale vaut 14,0.

| Source | SCE | ddl | CM | $F$ | p-value |
|---|---|---|---|---|---|
| Engrais | 106,0 | 3 | 35,3 | 60,6 | 7e-5 |
| Position | 9,0 | 3 | 3,0 | 5,1 | 0,043 |
| Jour | 1,5 | 3 | 0,5 | 0,9 | 0,51 |
| Résiduelle | 3,5 | 6 | 0,58 | | |
| Total | 120,0 | 15 | | | |

Chaque $F$ se calcule avec le carré moyen résiduel : $F = \text{CM}_{\text{engrais}} / \text{CM}_E = 35{,}3 / 0{,}58 = 60{,}6$.

**Ce que l'exemple montre :** l'engrais a un effet net. La position a aussi un effet, faible mais significatif : sans le carré latin, cet effet se mélangerait à celui des engrais. Le jour n'a pas d'effet détectable. L'hypothèse du carré latin (pas d'interaction entre engrais et position, ou entre engrais et jour) n'est pas testable ici : elle se pose.

> **Hypothèses à vérifier** (non vérifiées dans cet exemple, pour ne pas alourdir le chapitre) : additivité (pas d'interaction entre engrais et position, ni entre engrais et jour) ; résidus normaux et de même variance ; chaque engrais apparaît une fois par position et une fois par jour ; mesures indépendantes.

### ANCOVA : tenir compte de la taille initiale

**Question :** les groupes ont-ils des moyennes différentes lorsqu'on tient compte d'une autre variable quantitative ?

On compare la croissance de plantes avec trois engrais. Mais les plantes n'ont pas la même taille au départ, et la taille initiale $x$ influence la taille finale $y$. On veut comparer les engrais à taille initiale égale.

| Engrais | Taille initiale $x$ (cm) | Taille finale $y$ (cm) |
|---|---|---|
| E1 | 5 ; 6 ; 7 ; 8 ; 9 ; 10 | 17 ; 19 ; 20 ; 22 ; 23 ; 25 |
| E2 | 6 ; 7 ; 8 ; 9 ; 10 ; 11 | 20 ; 21 ; 23 ; 25 ; 26 ; 28 |
| E3 | 4 ; 5 ; 6 ; 7 ; 8 ; 9 | 16 ; 18 ; 19 ; 21 ; 22 ; 24 |

Les moyennes brutes de $y$ sont 21,0 (E1), 23,8 (E2) et 20,0 (E3). Mais la taille initiale moyenne n'est pas la même : 7,5 cm pour E1, 8,5 cm pour E2, 6,5 cm pour E3. La moyenne générale de $x$ vaut 7,5 cm.

Le modèle ajuste une droite $y = \beta_0 + \beta_1 x + \beta_E$ (un effet par engrais), avec une pente commune $\beta_1 = 1{,}57$ (chaque centimètre de départ donne environ 1,6 cm à l'arrivée). La moyenne ajustée d'un groupe se calcule à $x$ égal, ici $\bar{x} = 7{,}5$ cm :

$$
\bar{y}_{\text{ajustée}} = \bar{y}_{\text{groupe}} - \beta_1 (\bar{x}_{\text{groupe}} - \bar{x})
$$

| Source | SCE | ddl | CM | $F$ | p-value |
|---|---|---|---|---|---|
| Taille initiale $x$ | 172,6 | 1 | 172,6 | 2 029 | < 2e-16 |
| Engrais (ajusté) | 4,5 | 2 | 2,3 | 26,6 | 2e-5 |
| Résiduelle | 1,2 | 14 | 0,085 | | |
| Total | 178,3 | 17 | | | |

Les moyennes ajustées sont : E1 21,0 ; E2 22,3 ; E3 21,6.

**Ce que l'exemple montre :** la taille initiale explique presque toute la variabilité (97 %). Une fois cette taille prise en compte, l'effet des engrais reste significatif, mais l'écart entre E2 et E1 passe de 2,8 cm à 1,3 cm : une partie de l'avantage de E2 venait de ses plantes plus grandes au départ. L'ANCOVA suppose que la pente est la même pour les trois engrais. Ici, cette condition est vérifiée par construction ; sur de vraies données, elle se teste avant d'interpréter les moyennes ajustées.

> **Hypothèses à vérifier** (non vérifiées dans cet exemple, pour ne pas alourdir le chapitre) : relation linéaire entre taille initiale et taille finale ; pentes parallèles (pas d'interaction entre la taille initiale et l'engrais) ; taille initiale mesurée avant le traitement, donc non affectée par lui ; résidus normaux et de même variance ; plantes indépendantes.

## Exercice 1 – ANOVA imbriquée

Deux bureaux (B1 et B2) mesurent la consommation de logements après un diagnostic. Chaque bureau a deux chargés d'études, qui n'existent que dans leur bureau : le chargé est imbriqué dans le bureau. Chaque chargé fait 3 DPE. Les données sont fictives.

| Bureau | Chargé | Consommations (kWh/m²/an) |
|--------|--------|---------------------------|
| B1 | 1 | 120 ; 130 ; 125 |
| B1 | 2 | 150 ; 160 ; 155 |
| B2 | 1 | 100 ; 105 ; 110 |
| B2 | 2 | 115 ; 120 ; 118 |

Les hypothèses sont :

- **Effet du bureau (A)** : $H_0$ : les bureaux ont la même moyenne ; $H_1$ : au moins un bureau diffère.
- **Effet du chargé dans le bureau (B(A))** : $H_0$ : les chargés d'un même bureau ont la même moyenne ; $H_1$ : au moins un chargé diffère.

Avec $a$ bureaux, $b$ chargés par bureau et $n$ DPE par chargé, le tableau s'écrit :

| Source | SCE | ddl | CM | F | p-value |
|--------|-----|-----|----|---|---------|
| Bureau (A) | formule (1) ci-dessous | a − 1 | SCE A / (a − 1) | CM A / CM B(A) | loi de Fisher, ddl (a − 1 ; a(b − 1)) |
| Chargé dans le bureau B(A) | formule (2) ci-dessous | a(b − 1) | SCE B(A) / [a(b − 1)] | CM B(A) / CM E | loi de Fisher, ddl (a(b − 1) ; ab(n − 1)) |
| Résiduelle | formule (3) ci-dessous | ab(n − 1) | SCE E / [ab(n − 1)] | | |

Formules des sommes des carrés :

Bureau (1) :

$$
\text{SCE}_A = bn \sum_i (\bar{y}_{i\cdot\cdot} - \bar{y})^2
$$

Chargé dans le bureau B(A) (2) :

$$
\text{SCE}_{B(A)} = n \sum_{i,j} (\bar{y}_{ij\cdot} - \bar{y}_{i\cdot\cdot})^2
$$

Résiduelle (3) :

$$
\text{SCE}_E = \sum_{i,j,k} (y_{ijk} - \bar{y}_{ij\cdot})^2
$$

Ici $a = 2$, $b = 2$, $n = 3$ : les ddl sont $(1)$, $(2)$ et $(8)$.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `aov()` avec `bureau:charge` | ANOVA imbriquée : le chargé est testé dans chaque bureau. | formule | `aov(conso ~ bureau + bureau:charge, data = bureaux)` |
| `pf()` | p-value d'une statistique $F$. | `q`, `df1`, `df2`, `lower.tail` | `pf(3.10, 1, 2, lower.tail = FALSE)` |

### Exemple

```r
bureaux <- tibble(
  bureau = rep(c("B1", "B2"), each = 6),
  charge = rep(c("1", "1", "1", "2", "2", "2"), 2),   # numéro local du chargé dans son bureau
  conso  = c(120, 130, 125, 150, 160, 155, 100, 105, 110, 115, 120, 118)
)

aov_imb <- aov(conso ~ bureau + bureau:charge, data = bureaux)
summary(aov_imb)
#               Df Sum Sq Mean Sq F value   Pr(>F)
# bureau         1   2465  2465.3  121.2  < 2e-16     <- à NE PAS retenir : testé contre la résiduelle
# bureau:charge  2   1591   795.3   39.1  7.4e-05
# Residuals      8    163    20.3
```

La table de `summary()` teste le bureau contre la résiduelle : c'est faux, car le chargé est une source de variabilité entre les bureaux. Le bon test du bureau utilise le carré moyen du chargé dans le bureau.

### Exercices

1. Calculer la SCE et les ddl de chaque source à partir des données, et vérifier que la somme des SCE égale la SCE totale (4 218,7 ; 11 ddl).
   > **Intérêt :** la décomposition imbriquée est une identité : si la somme ne tombe pas juste, une observation est mal classée.
<details>
<summary>Correction</summary>

Bureau : SCE = 2 465,3 (1 ddl). Chargé dans le bureau : SCE = 1 590,7 (2 ddl). Résiduelle : SCE = 162,7 (8 ddl).

Somme :

$$
2\,465{,}3 + 1\,590{,}7 + 162{,}7 = 4\,218{,}7
$$

pour 11 ddl. La décomposition est juste.
</details>

2. Tester le bureau : calculer le $F$ correct (contre le chargé dans le bureau), puis lire la p-value dans la table de Fisher pour $F$ et les ddl $(1 ; 2)$. Comparer avec la valeur donnée par `summary()`.
   > **Intérêt :** le même effet donne deux conclusions selon le dénominateur choisi : le choix du test se fait avant le calcul, à partir du plan.
<details>
<summary>Correction</summary>

$$
F_A = \dfrac{\text{CM}_A}{\text{CM}_{B(A)}} = \dfrac{2\,465{,}3}{795{,}3} = 3{,}10
$$

avec $(1 ; 2)$ ddl. La p-value vaut `pf(3.10, 1, 2, lower.tail = FALSE)` = 0,22 : on ne rejette pas $H_0$. Avec 2 ddl seulement, la puissance est très faible, et une différence de 30 kWh/m² entre bureaux n'est pas établie.

Le test contre la résiduelle donnerait $F = 2\,465{,}3 / 20{,}3 = 121{,}2$, p très petite : c'est la conclusion fausse fournie par `summary()`.
</details>

3. Tester le chargé dans le bureau : calculer le $F$ (carré moyen du chargé contre résiduelle), avec ddl $(2 ; 8)$. Quelle est la conclusion ?
   > **Intérêt :** un facteur imbriqué se teste contre la résiduelle : c'est le seul test qui a le bon dénominateur.
<details>
<summary>Correction</summary>

$F_{B(A)} = 795{,}3 / 20{,}3 = 39{,}1$, avec $(2 ; 8)$ ddl : `pf(39.1, 2, 8, lower.tail = FALSE)` = 7,4e-5. On rejette $H_0$ : les chargés d'un même bureau ne mesurent pas la même consommation. La variabilité vient surtout des chargés, pas des bureaux.
</details>

## Exercice 2 – ANOVA en carré latin

Un essai compare 3 isolants (A, B, C) sur 3 chantiers et 3 équipes. Dans un carré latin, chaque isolant apparaît une fois par chantier et une fois par équipe : on contrôle ainsi deux sources de variation sans multiplier les mesures. Les données sont fictives.

| Chantier | Équipe 1 | Équipe 2 | Équipe 3 |
|----------|----------|----------|----------|
| 1 | A : 102 | B : 121 | C : 139 |
| 2 | B : 119 | C : 142 | A : 98 |
| 3 | C : 137 | A : 104 | B : 122 |

Avec $k = 3$ traitements, le plan a $k^2 = 9$ observations, $k^2 - 1 = 8$ ddl au total, et $(k-1)(k-2) = 2$ ddl résiduels. Chaque facteur (isolant, chantier, équipe) a $k - 1 = 2$ ddl.

Les hypothèses sont, pour l'isolant : $H_0$ : les trois isolants ont la même moyenne de consommation ; $H_1$ : au moins un diffère. Même forme pour le chantier et l'équipe, à tester séparément.

> **Attention : hypothèse d'additivité**
> Le carré latin suppose qu'il n'y a pas d'interaction entre les facteurs : l'effet de l'isolant est le même sur tous les chantiers et toutes les équipes. Avec 9 observations, cette hypothèse ne peut pas être testée. Elle se pose, ne se vérifie pas.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `aov()` | Plan à trois facteurs, sans interaction. | formule additive | `aov(conso ~ isolant + chantier + equipe, data = carre)` |

### Exemple

```r
carre <- tibble(
  chantier = c(1, 1, 1, 2, 2, 2, 3, 3, 3),
  equipe   = c(1, 2, 3, 1, 2, 3, 1, 2, 3),
  isolant  = c("A", "B", "C", "B", "C", "A", "C", "A", "B"),
  conso    = c(102, 121, 139, 119, 142, 98, 137, 104, 122)
)

summary(aov(conso ~ isolant + chantier + equipe, data = carre))
#             Df Sum Sq Mean Sq F value  Pr(>F)
# isolant      2 2166.2  1083.1 128.263 0.00774 **
# chantier     2    2.9     1.4   0.171 0.85400
# equipe       2   16.2     8.1   0.961 0.51001
# Residuals    2   16.9     8.4
```

### Exercices

1. Calculer la moyenne de chaque isolant, puis la SCE de l'isolant à la main. Vérifier avec la table.
   > **Intérêt :** la SCE du traitement vient de l'écart entre les moyennes de chaque isolant : c'est la base du test.
<details>
<summary>Correction</summary>

Moyennes : A = 101,3 ; B = 120,7 ; C = 139,3 ; moyenne générale = 120,4.

$$
\text{SCE}_{\text{isolant}} = 3 \times [(101{,}3 - 120{,}4)^2 + (120{,}7 - 120{,}4)^2 + (139{,}3 - 120{,}4)^2] = 2\,166{,}2
$$

avec 2 ddl.
</details>

2. Lire la p-value de l'isolant (F = 128,3, avec 2 et 2 ddl). Pourquoi la table de Fisher de l'exercice 4 du chapitre 14 ne suffit-elle pas ici ?
   > **Intérêt :** une table a des bornes ; quand la valeur est au-delà, il faut calculer.
<details>
<summary>Correction</summary>

`pf(128.263, 2, 2, lower.tail = FALSE)` = 0,0077 : on rejette $H_0$ au seuil de 5 %. La table de Fisher pour 2 ddl au dénominateur ne va pas jusqu'à $F = 128$ : la colonne la plus haute donne $F = 15$, avec p = 0,0625. Il faut donc `pf()`.
</details>

3. Le chantier et l'équipe sont-ils significatifs ? Quelle est la limite de ce plan avec 2 ddl résiduels ?
   > **Intérêt :** un plan qui contrôle plus de sources garde peu de ddl pour l'erreur : une absence d'effet ne se distingue pas d'un manque de puissance.
<details>
<summary>Correction</summary>

Chantier : $F = 0{,}171$, p = 0,85. Équipe : $F = 0{,}961$, p = 0,51. Aucun n'est significatif, mais le dénominateur n'a que 2 ddl : une vraie différence de chantier ou d'équipe pourrait passer inaperçue. On conclut seulement que l'isolant est le facteur qui explique la consommation dans cet essai.
</details>

## Exercice 3 – ANCOVA : consommation et surface

Une ANCOVA combine un facteur et une covariable numérique. Ici, on étudie la consommation totale (logarithme) selon la période de construction, en tenant compte de la surface (logarithme). Les données sont les vrais DPE.

L'ANCOVA suppose deux choses avant toute interprétation :
- **pentes parallèles** : la relation entre surface et consommation a la même pente dans chaque période. C'est l'hypothèse d'interaction entre covariable et facteur ;
- **covariable indépendante du traitement** : la surface ne doit pas dépendre de la période au point de devenir un effet de la période (ce qui est partiellement le cas ici).

On teste d'abord les pentes parallèles : $H_0$ : les pentes sont égales ; $H_1$ : au moins une pente diffère. Si $H_0$ est rejetée, l'ANCOVA classique ne se lit pas comme une moyenne ajustée unique.

> **Contexte :** une comparaison de consommation entre périodes, à surface égale, est une question fréquente. L'ANCOVA la traite, mais seulement si la relation surface / consommation est la même dans chaque période.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `aov()` avec covariable | ANCOVA : la covariable est entrée en premier, le facteur ensuite. | ordre des termes | `aov(ln_conso ~ ln_surface + periode3, data = dpe_anc)` |
| `aov()` avec `*` | Test des pentes parallèles (interaction covariable × facteur). | `*` | `aov(ln_conso ~ ln_surface * periode3, data = dpe_anc)` |
| `ggplot()` + `geom_smooth()` | Droites par groupe, pour voir si les pentes diffèrent. | `method = "lm"` | `geom_smooth(method = "lm", se = FALSE)` |

### Exemple

```r
dpe_anc <- dpe |>
  filter(!is.na(surface_habitable_logement), conso_5_usages_ep > 0) |>
  mutate(
    ln_conso = log(conso_5_usages_ep),
    ln_surface = log(surface_habitable_logement),
    periode3 = factor(case_when(
      periode_construction %in% c("avant 1948", "1948-1974") ~ "avant 1975",
      periode_construction %in% c("1975-1977", "1978-1982", "1983-1988",
                                  "1989-2000", "2001-2005") ~ "1975-2005",
      TRUE ~ "2006 et après"
    ), levels = c("avant 1975", "1975-2005", "2006 et après"))
  )

# 1. pentes parallèles
summary(aov(ln_conso ~ ln_surface * periode3, data = dpe_anc))
#                       Df Sum Sq Mean Sq F value Pr(>F)
# ln_surface:periode3    2     47    23.4  198.6 <2e-16   <- pentes différentes

# 2. ANCOVA (covariable d'abord)
summary(aov(ln_conso ~ ln_surface + periode3, data = dpe_anc))
```

Le graphique montre les droites par période : si elles ne sont pas parallèles, le test d'interaction le confirme.

```r
ggplot(dpe_anc, aes(x = ln_surface, y = ln_conso, colour = periode3)) +
  geom_point(alpha = 0.05) +
  geom_smooth(method = "lm", se = FALSE) +
  labs(x = "ln(surface)", y = "ln(consommation totale)", colour = "Période")
```

### Exercices

1. Lire le test des pentes parallèles. Faut-il interpréter l'ANCOVA classique ?
   > **Intérêt :** tester l'hypothèse de l'ANCOVA avant de lire ses résultats évite de présenter des moyennes ajustées qui n'ont pas de sens.
<details>
<summary>Correction</summary>

Le test d'interaction donne $F = 198{,}6$ avec 2 ddl, p = 2,7e-85 : on rejette les pentes parallèles. La relation surface / consommation n'a pas la même pente dans les trois périodes. L'ANCOVA classique ne se lit donc pas comme « la période, à surface égale ». Il faut soit modéliser les pentes séparément, soit présenter les résultats par période.
</details>

2. Si l'on ignore le test précédent, la table de l'ANCOVA donne l'effet de la période ajustée. Lire le $F$ et la p-value. Calculer la SCE résiduelle, et vérifier que les trois SCE somment à la SCE totale.
   > **Intérêt :** la décomposition permet de voir la part de la surface dans la consommation : c'est le point de départ d'une ANCOVA.
<details>
<summary>Correction</summary>

| Source | SCE | ddl | CM | $F$ | p-value |
|--------|-----|-----|----|-----|---------|
| ln(surface) | 2 229,9 | 1 | 2 229,9 | 18 212,9 | < 2,2e-16 |
| Période (ajustée) | 438,1 | 2 | 219,1 | 1 789,2 | < 2,2e-16 |
| Résiduelle | 1 211,5 | 9 895 | 0,122 | | |
| Total | 3 879,5 | 9 898 | | | |

Somme : $2\,229{,}9 + 438{,}1 + 1\,211{,}5 = 3\,879{,}5$. La surface explique l'essentiel de la variance ; la période, à surface égale, a un effet significatif mais petit en proportion. Ce résultat ne vaut que si l'hypothèse de pentes parallèles tient, ce qui n'est pas le cas (question 1).
</details>

3. Comparer les moyennes brutes et les moyennes ajustées à la surface moyenne ($\ln$ surface = 3,873, soit environ 48 m²). Pourquoi diffèrent-elles ?
   > **Intérêt :** l'ajustement change la lecture : une période qui paraît meilleure peut l'être seulement parce que ses logements sont plus petits.
<details>
<summary>Correction</summary>

| Période | Moyenne brute | Moyenne ajustée (à surface moyenne) |
|---------|---------------|-------------------------------------|
| avant 1975 | 9,279 | 9,299 |
| 1975-2005 | 9,081 | 8,949 |
| 2006 et après | 8,620 | 8,789 |

Les moyennes ajustées ne sont que des valeurs de référence : le modèle n'a pas de pente commune. Elles montrent que l'écart entre les périodes récentes et anciennes se réduit une fois la surface prise en compte : les logements récents sont plus petits, ce qui fait baisser leur consommation totale brute.
</details>

## Exercice 4 – Choisir le plan

Pour chaque situation, choisir le plan (imbriqué, carré latin, ANCOVA, ou un plan à deux facteurs du chapitre 15) et justifier avant de calculer.

1. Un bureau d'études a 4 chargés, chacun dans un seul bureau, et chaque chargé fait 5 DPE. On veut comparer les bureaux.
   > **Intérêt :** une source de variation qui n'existe que dans un seul niveau est imbriquée : la traiter comme croisée fausse le test.
<details>
<summary>Correction</summary>

Plan imbriqué : les chargés ne sont présents que dans un bureau. Le bureau se teste contre le chargé dans le bureau, et non contre la résiduelle.
</details>

2. Un essai teste 4 isolants, avec 4 chantiers et 4 équipes, chaque isolant apparaissant une fois par chantier et une fois par équipe.
   > **Intérêt :** le carré latin économise des mesures, mais il suppose l'absence d'interaction : c'est un choix à justifier.
<details>
<summary>Correction</summary>

Carré latin d'ordre 4 : 16 observations, $(k-1)(k-2) = 6$ ddl résiduels. Il contrôle les chantiers et les équipes, et suppose l'additivité. Il faut annoncer cette hypothèse dans le rapport.
</details>

3. On veut comparer la consommation entre trois périodes en tenant compte de la surface, mais la relation surface / consommation pourrait différer par période.
   > **Intérêt :** la covariable seule ne suffit pas : il faut vérifier l'hypothèse de pentes parallèles avant de présenter un résultat ajusté.
<details>
<summary>Correction</summary>

ANCOVA, précédée du test des pentes parallèles. Si les pentes diffèrent, on modélise séparément chaque période, ou on présente les résultats par groupe sans moyenne ajustée commune.
</details>

## Liens utiles

- [Documentation de `aov()` (R)](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/aov.html)
- [Documentation de `pf()` (R)](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/Fdist.html)
