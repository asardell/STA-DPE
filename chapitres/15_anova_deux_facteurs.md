# Chapitre 15 – ANOVA à deux facteurs

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] poser les hypothèses de l'ANOVA à deux facteurs : effet de chaque facteur, et effet d'interaction
- [ ] construire un échantillon équilibré, avec le même nombre de DPE dans chaque cellule
- [ ] réaliser une ANOVA à deux facteurs avec répétitions, et lire la table des sommes des carrés
- [ ] réaliser une ANOVA à deux facteurs sans répétition, et expliquer pourquoi l'interaction ne peut pas être testée
- [ ] représenter les effets et l'interaction avec un boxplot bivarié et un graphique des moyennes
- [ ] lire une p-value dans la table de Fisher à partir d'une statistique $F$ et de deux ddl

> **Contexte :** en bureau d'études, une consommation dépend souvent de deux facteurs à la fois : l'énergie de chauffage et l'époque de construction, par exemple. L'ANOVA à deux facteurs sépare l'effet de chacun, et teste si l'effet de l'un dépend de l'autre. Cette question d'interaction est souvent la plus utile : un bon chauffage peut améliorer le bâtiment ancien sans changer le bâtiment récent.

## Sommaire

- [Chapitre 15 – ANOVA à deux facteurs](#chapitre-15--anova-à-deux-facteurs)
  - [Objectifs](#objectifs)
  - [Sommaire](#sommaire)
  - [Préparation : packages, données et échantillon équilibré](#préparation--packages-données-et-échantillon-équilibré)
  - [Démarche : modèle, hypothèses et tableau](#démarche--modèle-hypothèses-et-tableau)
  - [Avec ou sans répétition : la différence](#avec-ou-sans-répétition--la-différence)
  - [Exercice 1 – Moyennes par cellule et graphiques](#exercice-1--moyennes-par-cellule-et-graphiques)
  - [Exercice 2 – ANOVA à deux facteurs avec répétitions](#exercice-2--anova-à-deux-facteurs-avec-répétitions)
  - [Exercice 3 – ANOVA à deux facteurs sans répétition](#exercice-3--anova-à-deux-facteurs-sans-répétition)
  - [Exercice 4 – Lire une p-value dans la table de Fisher](#exercice-4--lire-une-p-value-dans-la-table-de-fisher)
  - [Exercice 5 – Avec ou sans répétitions ?](#exercice-5--avec-ou-sans-répétitions-)
  - [Liens utiles](#liens-utiles)

## Préparation : packages, données et échantillon équilibré

```r
library(readr)
library(dplyr)
library(ggplot2)

dpe <- read_csv("data/dpe_villeurbanne_10000.csv")

# Deux facteurs : énergie (2 modalités) et période en 3 groupes
dpe_2f <- dpe |>
  filter(!is.na(surface_habitable_logement), conso_5_usages_ep > 0,
         type_energie_n1 %in% c("Électricité", "Gaz naturel")) |>
  mutate(
    ln_conso_m2 = log(conso_5_usages_ep / surface_habitable_logement),
    periode3 = case_when(
      periode_construction %in% c("avant 1948", "1948-1974") ~ "avant 1975",
      periode_construction %in% c("1975-1977", "1978-1982", "1983-1988",
                                  "1989-2000", "2001-2005") ~ "1975-2005",
      TRUE ~ "2006 et après"
    ),
    periode3 = factor(periode3, levels = c("avant 1975", "1975-2005", "2006 et après"))
  )

# Échantillon équilibré : les 5 premiers DPE de chaque cellule, dans l'ordre du fichier
ech5 <- dpe_2f |> group_by(type_energie_n1, periode3) |> slice_head(n = 5) |> ungroup()

# Un seul DPE par cellule : sans répétition
ech1 <- dpe_2f |> group_by(type_energie_n1, periode3) |> slice_head(n = 1) |> ungroup()
```

Le tirage par `slice_head()` suit l'ordre du fichier : il est reproductible, sans graine aléatoire. Il donne un échantillon équilibré de 30 DPE (5 par cellule) et de 6 DPE (1 par cellule).

> **Note : pourquoi un échantillon équilibré ?**
> Avec des effectifs très inégaux (2 664 DPE au gaz avant 1975, contre 540 au gaz après 2006), les sommes des carrés dépendent de l'ordre d'entrée des facteurs. Un plan équilibré évite ce problème : chaque cellule a le même nombre d'observations, et chaque effet se lit sans ambiguïté.

## Démarche : modèle, hypothèses et tableau

Le modèle à deux facteurs s'écrit : $y_{ijk} = \mu + \alpha_i + \beta_j + (\alpha\beta)_{ij} + \varepsilon_{ijk}$, où $\alpha_i$ est l'effet de l'énergie $i$, $\beta_j$ l'effet de la période $j$, et $(\alpha\beta)_{ij}$ l'effet d'interaction.

On pose trois tests :

- **Effet du facteur A (énergie)** : $H_0$ : toutes les moyennes par énergie sont égales ; $H_1$ : au moins une diffère.
- **Effet du facteur B (période)** : $H_0$ : toutes les moyennes par période sont égales ; $H_1$ : au moins une diffère.
- **Interaction A × B** : $H_0$ : l'effet de l'énergie est le même dans toutes les périodes (les écarts sont parallèles) ; $H_1$ : l'effet de l'énergie change selon la période.

Pour chaque test, $\alpha = 0{,}05$ est fixé avant le calcul. Le risque $\beta$ dépend de la taille des cellules : avec 5 DPE par cellule, seules des interactions marquées sont détectées.

Avec $a$ modalités de A, $b$ de B et $n$ répétitions par cellule, le tableau s'écrit :

| Source | Somme des carrés (SCE) | ddl | Carré moyen (CM) | F | p-value |
|--------|------------------------|-----|------------------|---|---------|
| Facteur A | formule (1) ci-dessous | a − 1 | SCE A / (a − 1) | CM A / CM E | loi de Fisher, ddl (a − 1 ; ab(n − 1)) |
| Facteur B | formule (2) ci-dessous | b − 1 | SCE B / (b − 1) | CM B / CM E | loi de Fisher, ddl (b − 1 ; ab(n − 1)) |
| Interaction A × B | formule (3) ci-dessous | (a − 1)(b − 1) | SCE AB / [(a − 1)(b − 1)] | CM AB / CM E | loi de Fisher, ddl ((a − 1)(b − 1) ; ab(n − 1)) |
| Résiduelle | formule (4) ci-dessous | ab(n − 1) | SCE E / [ab(n − 1)] | | |
| Total | formule (5) ci-dessous | abn − 1 | | | |

Formules des sommes des carrés :

Facteur A (1) :

$$
\text{SCE}_A = bn \sum_i (\bar{y}_{i\cdot\cdot} - \bar{y}_{\cdot\cdot\cdot})^2
$$

Facteur B (2) :

$$
\text{SCE}_B = an \sum_j (\bar{y}_{\cdot j\cdot} - \bar{y}_{\cdot\cdot\cdot})^2
$$

Interaction A × B (3) :

$$
\text{SCE}_{AB} = n \sum_{i,j} (\bar{y}_{ij\cdot} - \bar{y}_{i\cdot\cdot} - \bar{y}_{\cdot j\cdot} + \bar{y}_{\cdot\cdot\cdot})^2
$$

Résiduelle (4) :

$$
\text{SCE}_E = \sum_{i,j,k} (y_{ijk} - \bar{y}_{ij\cdot})^2
$$

Total (5) :

$$
\text{SCE}_T = \sum_{i,j,k} (y_{ijk} - \bar{y})^2
$$

Les sommes des carrés vérifient :

$$
\text{SCE}_T = \text{SCE}_A + \text{SCE}_B + \text{SCE}_{AB} + \text{SCE}_E
$$

> **Note : l'interaction d'abord**
> Si l'interaction est significative, les effets principaux ne se lisent pas seuls : l'effet de l'énergie dépend de la période. On commence donc toujours par le test d'interaction.

## Avec ou sans répétition : la différence

Une **cellule** est une combinaison d'un niveau de chaque facteur : ici, « gaz naturel, avant 1975 » est une cellule. Une **répétition** est une observation supplémentaire dans la même cellule. Dans l'échantillon équilibré, il y a 5 DPE au gaz avant 1975 : la cellule a donc 5 répétitions.

Ce qui distingue les deux plans est ce qu'on sait de la variabilité à l'intérieur d'une cellule :

- **avec répétitions**, les 5 DPE d'une cellule diffèrent entre eux. Cette dispersion mesure l'erreur : c'est la variabilité qui n'est expliquée ni par l'énergie, ni par la période, ni par leur combinaison ;
- **sans répétition**, il n'y a qu'un DPE par cellule. Aucune dispersion n'est observée à l'intérieur d'une cellule : impossible de savoir si l'écart entre deux cellules vient d'une vraie interaction ou du bruit d'une seule mesure.

> **Note : répétition et doublon**
> Une répétition n'est pas la même mesure copiée deux fois. Ce sont des unités distinctes (des DPE différents) placées dans les mêmes conditions. Deux lignes identiques dans le fichier ne constituent pas une répétition.

Le tableau suivant résume les conséquences pratiques :

| | Avec répétitions | Sans répétition |
|---|---|---|
| Observations par cellule | $n \geq 2$ (ici $n = 5$) | 1 |
| Taille de l'échantillon | $abn$ (ici 30) | $ab$ (ici 6) |
| Source de l'erreur | variabilité à l'intérieur des cellules | aucune mesure directe : l'erreur est confondue avec l'interaction |
| ddl de la résiduelle | $ab(n-1)$ (ici 24) | $(a-1)(b-1)$ (ici 2) |
| Interaction testable | oui | non |
| Hypothèse à poser | aucune sur l'absence d'interaction | additivité : pas d'interaction |
| Puissance des tests | bonne | faible |

### Ce que cela change dans la table

Avec répétitions, la table a une ligne « Résiduelle » qui ne contient que la variabilité interne aux cellules. Elle sert de dénominateur commun à tous les $F$ : c'est pour cela que l'interaction et les deux effets principaux se testent contre la même erreur.

Sans répétition, la ligne « Résiduelle » contient à la fois l'interaction et l'erreur. Elle a $(a-1)(b-1)$ ddl. Ici, elle vaut 0,0856 pour 2 ddl. Ce n'est pas la même quantité que la résiduelle avec répétitions (2,688 pour 24 ddl) : elle est calculée sur 6 valeurs seulement et ne peut pas être comparée directement.

Concrètement, la différence se voit sur le même tableau de cellules :

- avec répétitions, le gaz avant 1975 a 5 valeurs. Leur écart-type donne une idée de l'erreur, et l'écart entre énergies dans chaque période peut être jugé contre cette erreur ;
- sans répétition, le gaz avant 1975 a une seule valeur (5,748). Il n'existe aucune estimation de l'erreur dans cette cellule, et on ne sait pas si l'écart de 0,15 entre les énergies avant 1975 est réel ou du bruit.

> **Règle pratique**
> Si l'on peut mesurer plusieurs DPE par configuration, on choisit le plan avec répétitions : il permet de tester l'interaction et de mesurer l'erreur. Si l'on n'a qu'une valeur par configuration, on choisit le plan sans répétition, et l'on écrit dans le rapport que l'on suppose les effets additifs.

## Exercice 1 – Moyennes par cellule et graphiques

Avant tout test, on calcule la moyenne de chaque cellule et on la représente. La figure est la première lecture de l'interaction : si les droites sont parallèles, il n'y a pas d'interaction.

> **Contexte :** une présentation à un maître d'ouvrage doit montrer les effets sur une figure lisible, avant d'annoncer une conclusion statistique.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `group_by()` + `summarise()` | Moyenne par cellule. | | `group_by(type_energie_n1, periode3)` |
| `geom_boxplot()` | Boîtes par cellule, avec un facteur en couleur. | `fill` | `aes(x = periode3, y = ln_conso_m2, fill = type_energie_n1)` |
| `geom_line()` | Droites des moyennes : le graphique des interactions. | `group`, `colour` | `aes(x = periode3, y = moy, colour = type_energie_n1, group = type_energie_n1)` |

### Exemple

```r
moyennes <- ech5 |>
  group_by(type_energie_n1, periode3) |>
  summarise(moyenne = mean(ln_conso_m2), .groups = "drop")

ggplot(moyennes, aes(x = periode3, y = moyenne, colour = type_energie_n1, group = type_energie_n1)) +
  geom_line() +
  geom_point(size = 3) +
  labs(x = "Période de construction", y = "Moyenne de ln(conso / m²)", colour = "Énergie")
```

### Exercices

1. Calculer la moyenne de chaque cellule (6 valeurs). Pour chaque période, quelle énergie a la consommation par m² la plus élevée ?
   > **Intérêt :** une moyenne par cellule donne l'ordre de grandeur avant le test : c'est elle qui sera décomposée dans le tableau.
<details>
<summary>Correction</summary>

| Période | Électricité | Gaz naturel | Écart (élec. − gaz) |
|---------|-------------|-------------|---------------------|
| avant 1975 | 5,593 | 5,440 | 0,15 |
| 1975-2005 | 5,303 | 4,875 | 0,43 |
| 2006 et après | 5,054 | 4,467 | 0,59 |

L'électricité a une consommation par m² plus élevée dans les trois périodes. L'écart entre énergies augmente avec la période, mais avec 5 DPE par cellule, cet écart peut venir du hasard : le test de l'exercice 2 le vérifie.
</details>

2. Tracer le boxplot bivarié de `ln_conso_m2` par période, avec l'énergie en couleur. Pourquoi un graphique des moyennes est-il plus lisible ici ?
   > **Intérêt :** avec 5 points par boîte, un boxplot ne donne pas de quartiles fiables : la figure des moyennes est plus honnête.
<details>
<summary>Correction</summary>

```r
ggplot(ech5, aes(x = periode3, y = ln_conso_m2, fill = type_energie_n1)) +
  geom_boxplot(outlier.shape = NA) +
  labs(x = "Période de construction", y = "ln(conso / m²)", fill = "Énergie")
```
Avec 5 DPE par boîte, les quartiles et la médiane sont très instables : le boxplot montre surtout la dispersion locale. Le graphique des moyennes (exercice 1, question 1) résume mieux l'écart entre énergies dans chaque période.
</details>

## Exercice 2 – ANOVA à deux facteurs avec répétitions

Le test porte sur l'échantillon de 30 DPE (5 par cellule). `aov()` accepte la formule avec l'interaction `*` : elle ajoute les deux facteurs et leur croisement.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `aov()` avec `*` | ANOVA à deux facteurs et interaction. | formule `y ~ A * B` | `aov(ln_conso_m2 ~ type_energie_n1 * periode3, data = ech5)` |
| `summary()` | Table : SCE, ddl, CM, $F$ et p-value pour chaque source. | | `summary(modele)` |

### Exemple

```r
modele_2f <- aov(ln_conso_m2 ~ type_energie_n1 * periode3, data = ech5)
summary(modele_2f)
#                              Df Sum Sq Mean Sq F value   Pr(>F)
# type_energie_n1               1  1.137   1.137  10.154  0.00397 **
# periode3                      2  2.876   1.438  12.838  0.00016 ***
# type_energie_n1:periode3      2  0.241   0.120   1.075  0.35702
# Residuals                    24  2.688   0.112
```

Les effets de l'énergie et de la période sont significatifs. L'interaction ne l'est pas : rien n'indique que l'écart entre énergies change selon la période, au seuil de 5 %.

### Exercices

1. Vérifier la somme des carrés totale à partir des quatre sources. Quelle part de la variance totale expliquent l'énergie, la période et l'interaction ?
   > **Intérêt :** la décomposition en sources est une identité : si elle ne se vérifie pas, le modèle n'est pas celui qu'on pense.
<details>
<summary>Correction</summary>

$\text{SCE}_T = 1{,}137 + 2{,}876 + 0{,}241 + 2{,}688 = 6{,}942$, ce qui correspond au total de la table (ddl : $1 + 2 + 2 + 24 = 29$). Parts de variance : énergie $1{,}137 / 6{,}942 = 16{,}4\ \%$, période $2{,}876 / 6{,}942 = 41{,}4\ \%$, interaction $0{,}241 / 6{,}942 = 3{,}5\ \%$, résiduelle $38{,}7\ \%$.
</details>

2. Lire la p-value de l'effet de la période. Son $F$ est 12,84 avec 2 et 24 ddl : la table de Fisher donne-t-elle la même conclusion ? (voir exercice 4)
   > **Intérêt :** une p-value affichée par R se vérifie dans une table : c'est la façon de contrôler qu'on lit le bon ddl.
<details>
<summary>Correction</summary>

La p-value est 0,00016 : on rejette $H_0$. Les ddl sont $(2 ; 24)$ : deux pour le numérateur (la période a 3 modalités), 24 pour le dénominateur (la résiduelle). Dans la table de l'exercice 4, la ligne 24 ddl donne une p-value très proche de zéro à $F = 12{,}84$.
</details>

3. Calculer la part de variance de l'énergie et l'$F$ de l'énergie avec ses ddl. Pourquoi son $F$ est-il calculé avec le même carré moyen résiduel que la période ?
   > **Intérêt :** dans un plan équilibré, chaque effet principal se teste contre la même erreur résiduelle : c'est ce qui rend les tests comparables.
<details>
<summary>Correction</summary>

$F_{\text{énergie}} = \dfrac{1{,}137 / 1}{0{,}112} = 10{,}15$, avec $(1 ; 24)$ ddl, p = 0,004. Le carré moyen résiduel $0{,}112$ est le dénominateur commun de tous les tests : c'est la variabilité qui n'est expliquée par aucun facteur.
</details>

## Exercice 3 – ANOVA à deux facteurs sans répétition

Sans répétition, chaque cellule a une seule observation : il n'y a plus de variabilité à l'intérieur des cellules pour mesurer l'erreur. L'interaction se confond alors avec la résiduelle, et ne peut pas être testée.

> **Contexte :** un plan sans répétition est fréquent quand on n'a qu'une mesure par cas (un bâtiment par configuration). On accepte alors de supposer l'absence d'interaction, et on le dit dans le rapport.

### Exemple

```r
modele_1 <- aov(ln_conso_m2 ~ type_energie_n1 + periode3, data = ech1)
summary(modele_1)
#                  Df Sum Sq Mean Sq F value Pr(>F)
# type_energie_n1   1 0.2098  0.2098   4.900  0.157
# periode3          2 1.2226  0.6113  14.276  0.065
# Residuals         2 0.0856  0.0428
```

### Exercices

1. Lire la table. L'énergie est-elle significative ? La période ?
   > **Intérêt :** un échantillon de 6 DPE donne peu de ddl au dénominateur : même un écart net peut ne pas être significatif.
<details>
<summary>Correction</summary>

Aucun effet n'est significatif au seuil de 5 % : énergie $F = 4{,}90$, p = 0,157 ; période $F = 14{,}28$, p = 0,065. Le dénominateur n'a que 2 ddl : la puissance est faible. La période est proche du seuil, mais l'échantillon ne permet pas de conclure.
</details>

2. Expliquer pourquoi le test d'interaction n'apparaît pas dans la table. Quelle hypothèse faut-il poser pour continuer ?
   > **Intérêt :** une hypothèse posée pour pouvoir calculer doit être écrite dans le rapport, avec sa justification.
<details>
<summary>Correction</summary>

Il y a 6 cellules et 6 valeurs : la résiduelle a $(2-1)(3-1) = 2$ ddl et contient l'interaction. On ne peut pas séparer les deux. Pour continuer, on pose que l'interaction est nulle (les effets sont additifs). Ce choix doit être annoncé, et vérifié si possible avec des répétitions (exercice 2).
</details>

## Exercice 4 – Lire une p-value dans la table de Fisher

La table de Fisher se construit comme celle du Khi² : une ligne par ddl du dénominateur, une colonne par valeur de $F$, pour un numérateur fixé. On reproduit ici la table pour $d_1 = 2$ (une source à 3 modalités).

> **Contexte :** les logiciels donnent la p-value directement, mais la table reste utile pour vérifier une sortie et comprendre l'effet des ddl sur la décision.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `pf()` | Probabilité de dépasser $F$ : p-value. | `q`, `df1`, `df2`, `lower.tail = FALSE` | `pf(12.84, df1 = 2, df2 = 24, lower.tail = FALSE)` |
| `outer()` | Table pour toutes les combinaisons de ddl et de $F$. | `X`, `Y`, `FUN` | voir l'exemple |

### Exemple

```r
ddl2 <- c(2, 8, 24, 100)
F_val <- c(1, 2, 3, 5, 10, 15)
table_F2 <- outer(ddl2, F_val, function(d2, f) pf(f, df1 = 2, df2 = d2, lower.tail = FALSE))
dimnames(table_F2) <- list(ddl_denominateur = ddl2, F = F_val)
round(table_F2, digits = 4)
#                     F
# ddl_denominateur      1      2      3      5     10     15
# 2                0.5000 0.3333 0.2500 0.1667 0.0909 0.0625
# 8                0.4096 0.1975 0.1066 0.0390 0.0067 0.0020
# 24               0.3827 0.1573 0.0687 0.0153 0.0007 0.0001
# 100              0.3715 0.1407 0.0543 0.0085 0.0001 0.0000
```

### Exercices

1. Lire la p-value de l'interaction de l'exercice 2 : $F = 1{,}075$, ddl $(2 ; 24)$. Comparer la lecture à `pf()`.
   > **Intérêt :** la lecture entre deux colonnes est une interpolation : on mesure son erreur en la comparant à la valeur exacte.
<details>
<summary>Correction</summary>

Pour 24 ddl, $F = 1{,}075$ se situe entre les colonnes 1 (p = 0,383) et 2 (p = 0,157). L'interpolation donne environ 0,366. La valeur exacte est `pf(1.075, 2, 24, lower.tail = FALSE)` = 0,357. L'interaction n'est pas significative dans les deux cas.
</details>

2. Lire la p-value de l'effet de la période : $F = 12{,}84$, ddl $(2 ; 24)$. Pourquoi la table ne donne-t-elle pas directement cette valeur ?
   > **Intérêt :** une valeur de $F$ au-delà de la dernière colonne ne se lit pas : il faut savoir quand la table ne suffit plus.
<details>
<summary>Correction</summary>

La table s'arrête à $F = 15$ : pour 24 ddl, $F = 12{,}84$ se situe entre les colonnes 10 (p = 0,0007) et 15 (p = 0,0001). La p-value exacte est 0,0002 : elle est très inférieure à 0,05. On rejette $H_0$ : la période a un effet sur la consommation par m².
</details>

## Exercice 5 – Avec ou sans répétitions ?

Le choix du plan se décide avant la collecte, pas après l'analyse. Pour chaque situation, dire quel plan convient et pourquoi.

1. Un audit mesure, dans chaque cellule énergie × période, la consommation de 5 bâtiments. Plan ?
   > **Intérêt :** la répétition est ce qui permet de tester l'interaction.
<details>
<summary>Correction</summary>

Plan avec répétitions (5 par cellule). Les raisons :

- les 5 bâtiments d'une même cellule (même énergie, même période) sont mesurés dans les mêmes conditions : leurs écarts entre eux mesurent la variabilité propre aux bâtiments, c'est-à-dire l'erreur ;
- avec cette erreur, on peut tester trois choses : l'effet de l'énergie, l'effet de la période, et l'interaction entre les deux ;
- le nombre de ddl de la résiduelle est $ab(n - 1) = 6 \times 4 = 24$ : il est suffisant pour un test fiable.

Pour que les 5 mesures soient des répétitions, il faut qu'elles soient indépendantes (pas cinq DPE du même immeuble, par exemple). Sans répétition, on n'aurait que 6 valeurs et aucune façon de mesurer l'erreur.
</details>

2. Une étude mesure une seule consommation par configuration, pour 6 configurations. Plan ?
   > **Intérêt :** un plan sans répétition est un compromis : il faut annoncer l'hypothèse d'additivité.
<details>
<summary>Correction</summary>

Plan sans répétition, avec 6 valeurs seulement :

- les ddl se répartissent ainsi : énergie 1, période 2, résiduelle $(a-1)(b-1) = 1 \times 2 = 2$. Au total, $6 - 1 = 5$ ddl ;
- la résiduelle ne contient plus seulement l'erreur : elle contient aussi l'interaction. On ne peut donc pas les séparer, et le test d'interaction n'existe pas dans ce plan ;
- il faut donc poser l'hypothèse d'**additivité** : l'écart entre les énergies est le même dans les trois périodes. Cette hypothèse n'est pas vérifiée par les données ; elle est choisie. Elle doit être écrite dans le rapport, avec sa justification ;
- les tests des effets principaux restent calculables, mais avec seulement 2 ddl au dénominateur, ils sont peu sensibles. Dans l'exemple du chapitre, la période a $F = 14{,}3$ et une p-value de 0,065 : un effet fort en valeur n'est pas déclaré significatif, faute de ddl.
</details>

3. Dans l'échantillon de l'exercice 2, le nombre de DPE est inégal : 2 664 au gaz avant 1975, 540 au gaz après 2006. Peut-on garder le même plan ?
   > **Intérêt :** un plan déséquilibré demande une méthode de calcul des sommes des carrés explicite, et change l'interprétation des effets.
<details>
<summary>Correction</summary>

Le problème vient du déséquilibre. Dans un plan équilibré, chaque effet a une part de variance bien définie, et la somme des parts donne la variance totale. Dans un plan déséquilibré :

- la somme des carrés totale reste la même, mais sa répartition entre les facteurs dépend de l'ordre dans lequel `aov()` les introduit : `aov(y ~ energie + periode)` n'attribue pas la même part à l'énergie que `aov(y ~ periode + energie)` ;
- les DPE au gaz après 2006 sont environ cinq fois moins nombreux que ceux avant 1975 : les moyennes par cellule sont alors pondérées différemment, et l'effet de l'un des facteurs peut être déformé par le poids de l'autre.

Deux solutions :
1. **rééquilibrer l'échantillon**, comme dans ce chapitre avec `slice_head()` : on garde le même nombre de DPE par cellule (ici 5), au prix d'une perte de données ;
2. **garder toutes les données** et utiliser une méthode de sommes des carrés de type III (`car::Anova()` avec `type = 3`), qui teste chaque effet en ajustant sur l'autre. Dans ce cas, le choix doit être écrit dans le rapport, car il change les résultats.
</details>

## Liens utiles

- [Documentation de `aov()` (R)](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/aov.html)
- [Documentation de `pf()` (R)](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/Fdist.html)
