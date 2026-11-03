# Chapitre 5 – Liaisons entre deux variables

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] calculer la covariance et le coefficient de corrélation linéaire entre deux variables quantitatives
- [ ] construire une matrice de corrélation et la lire
- [ ] représenter un nuage de points, seul ou avec une droite de tendance
- [ ] croiser deux variables qualitatives avec un tableau croisé, puis calculer les profils lignes et colonnes
- [ ] représenter un tableau croisé avec des barres groupées ou empilées
- [ ] comparer une variable quantitative selon les modalités d'une variable qualitative, avec les moyennes par groupe et les boxplots par groupe
- [ ] distinguer une liaison d'une relation de cause à effet

## Sommaire

- [Chapitre 5 – Liaisons entre deux variables](#chapitre-5--liaisons-entre-deux-variables)
  - [Objectifs](#objectifs)
  - [Sommaire](#sommaire)
  - [Préparation : packages, données et couleurs](#préparation--packages-données-et-couleurs)
  - [Exercice 1 – Covariance entre deux variables quantitatives](#exercice-1--covariance-entre-deux-variables-quantitatives)
    - [Mémo](#mémo)
    - [Exemple](#exemple)
    - [Exercices](#exercices)
  - [Exercice 2 – Coefficient de corrélation linéaire](#exercice-2--coefficient-de-corrélation-linéaire)
    - [Mémo](#mémo-1)
    - [Exemple](#exemple-1)
    - [Exercices](#exercices-1)
  - [Exercice 3 – Corrélation et valeurs extrêmes](#exercice-3--corrélation-et-valeurs-extrêmes)
    - [Exemple](#exemple-2)
    - [Exercices](#exercices-2)
  - [Exercice 4 – Matrice de corrélation](#exercice-4--matrice-de-corrélation)
    - [Mémo](#mémo-2)
    - [Exemple](#exemple-3)
  - [Exercice 5 – Nuage de points](#exercice-5--nuage-de-points)
    - [Mémo](#mémo-3)
    - [Exemple](#exemple-4)
    - [Exercices](#exercices-3)
  - [Exercice 6 – Tableau croisé de deux variables qualitatives](#exercice-6--tableau-croisé-de-deux-variables-qualitatives)
    - [Mémo](#mémo-4)
    - [Exemple](#exemple-5)
    - [Exercices](#exercices-4)
  - [Exercice 7 – Profils lignes et profils colonnes](#exercice-7--profils-lignes-et-profils-colonnes)
    - [Mémo](#mémo-5)
    - [Exemple](#exemple-6)
    - [Exercices](#exercices-5)
  - [Exercice 8 – Barres groupées et barres empilées](#exercice-8--barres-groupées-et-barres-empilées)
    - [Mémo](#mémo-6)
    - [Exemple](#exemple-7)
    - [Exercices](#exercices-6)
  - [Exercice 9 – Variable quantitative selon une variable qualitative](#exercice-9--variable-quantitative-selon-une-variable-qualitative)
    - [Mémo](#mémo-7)
    - [Exemple](#exemple-8)
    - [Exercices](#exercices-7)
  - [Exercice 10 – Boxplots par groupe et graphiques interactifs](#exercice-10--boxplots-par-groupe-et-graphiques-interactifs)
    - [Mémo](#mémo-8)
    - [Exemple](#exemple-9)
    - [Exercices](#exercices-8)
  - [Liens utiles](#liens-utiles)

## Préparation : packages, données et couleurs

```r
library(readr)
library(dplyr)
library(ggplot2)
library(plotly)
library(esquisse)

dpe <- read_csv("data/dpe_villeurbanne_10000.csv")

couleurs_dpe <- c(A = "#319834", B = "#33cc31", C = "#cbfc0c", D = "#fff200",
                  E = "#fcaf17", F = "#ef7d1a", G = "#e2001a")
```

Ce chapitre s'appuie sur trois types de couples de variables :

| Couple | Exemple dans les DPE | Outil principal |
|--------|----------------------|-----------------|
| quantitative – quantitative | surface habitable et consommation d'énergie | covariance, corrélation, nuage de points |
| qualitative – qualitative | étiquette DPE et type de bâtiment | tableau croisé, profils, barres |
| qualitative – quantitative | étiquette DPE et surface | moyennes par groupe, boxplots par groupe |

> **Note : une liaison n'est pas une cause**
> Deux variables peuvent être liées sans que l'une cause l'autre. La surface et la consommation totale augmentent ensemble, mais la consommation par m² est un meilleur indicateur de la performance d'un logement. Ce chapitre décrit les liaisons ; il ne permet pas de conclure à une cause.
> Pour s'en convaincre, la collection de [corrélations fallacieuses de Tyler Vigen](https://www.tylervigen.com/spurious-correlations) associe des séries sans aucun lien réel, qui évoluent pourtant ensemble sur une même période.

## Exercice 1 – Covariance entre deux variables quantitatives

La **covariance** mesure si deux variables varient dans le même sens : positive si elles augmentent ensemble, négative si l'une augmente quand l'autre diminue. Sa valeur dépend des unités : elle n'est pas comparable d'une paire de variables à l'autre.

Pour deux variables $X$ et $Y$ observées sur $n$ DPE, la covariance s'écrit :

$$
\operatorname{cov}(X, Y) = \frac{1}{n - 1} \sum_{i=1}^{n} (x_i - \bar{x})(y_i - \bar{y})
$$

R utilise le diviseur $n - 1$, comme pour la variance.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `cov()` | Covariance entre deux vecteurs. | `x`, `y`, `use` | `cov(x, y, use = "complete.obs")` |
| `use = "complete.obs"` | Ignore les lignes où une des deux valeurs est manquante. | | |

> **Note : `na.rm` n'existe pas pour `cov()` et `cor()`**
> Ces fonctions prennent un argument `use` au lieu de `na.rm`. Sans lui, un seul `NA` rend le résultat `NA`.

### Exemple

```r
surface <- dpe$surface_habitable_logement
conso <- dpe$conso_5_usages_ep

# Covariance entre la surface et la consommation totale
cov(surface, conso, use = "complete.obs")
# [1] 525758

# La covariance change avec les unités : même paire, en MWh au lieu de kWh
cov(surface, conso / 1000, use = "complete.obs")
# [1] 525.8
```

### Exercices

1. Calculer la covariance entre `surface_habitable_logement` et `cout_total_5_usages`. Son signe est-il cohérent avec le nuage attendu ?
<details>
<summary>Correction</summary>

```r
cov(dpe$surface_habitable_logement, dpe$cout_total_5_usages, use = "complete.obs")
# [1] 37890.3
```
La covariance est positive : un logement plus grand coûte plus cher à chauffer, éclairer, etc.
</details>

2. Calculer la covariance entre `surface_habitable_logement` et `conso_5_usages_ep` en divisant la surface par 10 (en dizaines de m²). La valeur change-t-elle ? Pourquoi ?
<details>
<summary>Correction</summary>

```r
cov(dpe$surface_habitable_logement / 10, dpe$conso_5_usages_ep, use = "complete.obs")
# [1] 52575.8 (dix fois plus petite)
```
La covariance est multipliée par le même facteur que la surface : elle dépend des unités. Pour comparer des liaisons, on utilise la corrélation (exercice 2).
</details>

3. Calculer la covariance entre `deperditions_murs` et `besoin_chauffage`. Combien de lignes sont ignorées à cause des valeurs manquantes ?
<details>
<summary>Correction</summary>

```r
cov(dpe$deperditions_murs, dpe$besoin_chauffage, use = "complete.obs")
sum(is.na(dpe$deperditions_murs) | is.na(dpe$besoin_chauffage))
# 13 lignes ignorées
```
</details>

## Exercice 2 – Coefficient de corrélation linéaire

Le **coefficient de corrélation de Pearson** est la covariance rendue sans unité : il vaut entre −1 et +1. Plus sa valeur est proche de ±1, plus la liaison linéaire est forte. Proche de 0, il n'y a pas de liaison linéaire (mais il peut quand même en exister une non linéaire).

Le coefficient s'obtient en divisant la covariance par le produit des écarts-types :

$$
r_{X,Y} = \frac{\operatorname{cov}(X, Y)}{s_X \times s_Y}
$$

où $s_X$ et $s_Y$ sont les écarts-types de $X$ et $Y$. Les unités se compensent entre le numérateur et le dénominateur : $r$ n'a pas d'unité. Diviser la surface par 10 ne change donc pas $r$, alors que la covariance est multipliée par 0,1 (exercice 1, question 2).

| Valeur absolue | Lecture indicative |
|----------------|--------------------|
| 0 à 0,2 | liaison linéaire très faible |
| 0,2 à 0,5 | liaison faible à modérée |
| 0,5 à 0,8 | liaison forte |
| 0,8 à 1 | liaison très forte |

Ces seuils sont indicatifs : ils dépendent du domaine.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `cor()` | Coefficient de corrélation. | `x`, `y`, `use`, `method` | `cor(x, y, use = "complete.obs")` |
| `method = "spearman"` | Corrélation des rangs, moins sensible aux valeurs extrêmes. | `method` | `cor(x, y, method = "spearman", use = "complete.obs")` |

### Exemple

```r
cor(surface, conso, use = "complete.obs")
# [1] 0.77

# Corrélation de Spearman : même couple, calculée sur les rangs
cor(surface, conso, method = "spearman", use = "complete.obs")
# [1] 0.68
```

### Exercices

1. Calculer le coefficient de corrélation entre `surface_habitable_logement` et `conso_5_usages_ep`. Dans quelle catégorie du tableau de lecture se situe-t-il ?
<details>
<summary>Correction</summary>

```r
cor(dpe$surface_habitable_logement, dpe$conso_5_usages_ep, use = "complete.obs")
# [1] 0.776
```
0,776 : liaison forte. Plus un logement est grand, plus sa consommation totale est élevée.
</details>

2. Calculer le coefficient de corrélation entre `deperditions_murs` et `besoin_chauffage`. Que signifie une valeur aussi élevée ?
<details>
<summary>Correction</summary>

```r
cor(dpe$deperditions_murs, dpe$besoin_chauffage, use = "complete.obs")
# [1] 0.978
```
0,978 : liaison très forte. Les deux variables décrivent la même réalité thermique du bâtiment : la corrélation est forte par construction, et ne prouve pas que l'une cause l'autre.
</details>

3. Calculer la corrélation entre `surface_habitable_logement` et `besoin_chauffage`. Pourquoi est-elle proche de 0 alors que la surface influence la consommation ?
<details>
<summary>Correction</summary>

```r
cor(dpe$surface_habitable_logement, dpe$besoin_chauffage, use = "complete.obs")
# [1] -0.009
```
Le besoin de chauffage dépend de la qualité thermique du bâtiment (isolation, déperditions), pas seulement de sa surface : une grande surface bien isolée peut avoir un besoin faible. Une corrélation proche de 0 ne veut pas dire « aucun lien », seulement « aucune liaison linéaire ».
</details>

4. Calculer la corrélation de Spearman entre `surface_habitable_logement` et `conso_5_usages_ep`, et la comparer à la corrélation de Pearson.
<details>
<summary>Correction</summary>

```r
cor(surface, conso, method = "spearman", use = "complete.obs")
# [1] 0.681
```
Si les deux valeurs sont proches, la liaison est à peu près monotone et peu influencée par les extrêmes.
</details>

> **Note : Pearson et Spearman**
> Pearson mesure une liaison **linéaire** et réagit fortement aux valeurs extrêmes. Spearman travaille sur les rangs : il mesure une liaison **monotone** (qui va dans un seul sens), et résiste mieux aux extrêmes. Le choix dépend de la forme du nuage, qu'on regarde toujours avant de calculer.

> **Attention : une corrélation proche de 0 ne veut pas dire « aucun lien »**
> Le coefficient de Pearson ne mesure que la liaison **linéaire**. Une relation forte mais courbe, en U ou en cloche par exemple, peut donner une corrélation proche de 0. Le calcul ne signale rien : seul le nuage de points le montre.

```r
x <- seq(-3, 3, length.out = 101)
y <- x^2          # liaison parfaite, mais non linéaire
cor(x, y)
# [1] environ 0
```

![Exemples de nuages de points : une liaison non linéaire peut donner une corrélation nulle](https://miro.medium.com/v2/resize:fit:2000/format:webp/1*FsDn_bkTV5Gaobd_o45OZg.png)

Avant de calculer un coefficient, on regarde donc toujours le nuage de points (exercice 5).

## Exercice 3 – Corrélation et valeurs extrêmes

Une seule valeur extrême peut modifier fortement un coefficient de Pearson. On le vérifie en calculant la corrélation sur deux sous-ensembles.

### Exemple

```r
# Corrélation sur tous les DPE dont la surface est renseignée
cor(surface, conso, use = "complete.obs")                  # 0.776

# Corrélation sans les surfaces au-dessus de 300 m²
cor(surface[surface <= 300], conso[surface <= 300], use = "complete.obs")  # 0.655
```

### Exercices

1. Calculer la corrélation entre la surface et la consommation pour les logements de 300 m² ou moins. Combien de DPE sont concernés ?
<details>
<summary>Correction</summary>

```r
surface_300 <- surface <= 300
cor(surface[surface_300], conso[surface_300], use = "complete.obs")
# [1] 0.655
sum(surface_300, na.rm = TRUE)  # 9893 DPE
```
</details>

2. Trouver les lignes qui ont une surface supérieure à 300 m² et une consommation renseignée. Combien y en a-t-il, et que se passe-t-il pour la corrélation si on les retire ?
<details>
<summary>Correction</summary>

```r
grands <- dpe |> filter(surface_habitable_logement > 300)
nrow(grands)  # 6
```
Retirer six logements fait passer la corrélation de 0,776 à 0,655 : quelques points très éloignés suffisent à donner une impression de liaison forte. C'est pourquoi on regarde toujours le nuage de points avant d'interpréter un coefficient.
</details>

## Exercice 4 – Matrice de corrélation

Une **matrice de corrélation** rassemble les corrélations de toutes les paires d'un ensemble de variables quantitatives. Sa diagonale vaut toujours 1.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `cor()` sur un dataframe | Matrice de corrélation des colonnes numériques. | `x`, `use` | `cor(dpe[, c("x", "y")], use = "pairwise.complete.obs")` |
| `as.data.frame(as.table())` | Transforme la matrice en table longue, pour `ggplot2`. | | `as.data.frame(as.table(mat))` |
| `geom_tile()` | Carte de couleurs. | | `geom_tile(aes(fill = Freq))` |
| `corrplot()` (package `corrplot`) | Représentation de la matrice de corrélation, avec les coefficients affichés. | `method`, `type`, `addCoef.col` | `corrplot(matrice, method = "color", type = "upper", addCoef.col = "black")` |

### Exemple

```r
variables <- c("surface_habitable_logement", "conso_5_usages_ep",
               "deperditions_murs", "besoin_chauffage", "cout_total_5_usages")

matrice <- cor(dpe[, variables], use = "pairwise.complete.obs")
round(matrice, digits = 2)

# Représentation : une case par paire, colorée selon la corrélation
matrice_df <- as.data.frame(as.table(matrice))

ggplot(matrice_df, aes(x = Var1, y = Var2, fill = Freq)) +
  geom_tile() +
  geom_text(aes(label = round(Freq, 2)), size = 3) +
  scale_fill_gradient2(low = "blue", mid = "white", high = "red", limits = c(-1, 1)) +
  labs(x = NULL, y = NULL, fill = "Corrélation") +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))

# Même matrice avec le package corrplot (install.packages("corrplot"))
library(corrplot)
corrplot(matrice, method = "color", type = "upper", addCoef.col = "black",
         tl.col = "black")
```

> **Note : `corrplot`**
> Le package `corrplot` dessine la matrice en un seul appel. `type = "upper"` n'affiche que la moitié supérieure, puisque la matrice est symétrique. `addCoef.col` écrit les coefficients dans les cases.

> **Note : `pairwise.complete.obs`**
> Avec `"complete.obs"`, une ligne est écartée dès qu'une variable de la matrice est manquante. Avec `"pairwise.complete.obs"`, chaque paire utilise toutes les lignes où ses deux variables sont renseignées. Le second est plus juste, car il perd moins de données.

## Exercice 5 – Nuage de points

Le **nuage de points** place chaque DPE selon ses deux valeurs. C'est le graphique à faire avant tout calcul de corrélation : il montre la forme de la liaison, les groupes et les valeurs extrêmes.

### Mémo

| Outil | Commande | Argument pertinent |
|-------|----------|--------------------|
| Base R | `plot(x, y, pch = 16, col = rgb(0, 0, 0, 0.2))` | `pch` : type de point ; `col` avec transparence |
| Base R | `abline(lm(y ~ x))` | droite de tendance (visuelle, le modèle est au chapitre 12) |
| `ggplot2` | `ggplot(dpe, aes(x, y)) + geom_point(alpha = 0.2)` | `alpha` : transparence |
| `ggplot2` (zoom) | `+ coord_cartesian(xlim = c(0, 200), ylim = c(0, 50000))` | affiche une plage sans retirer les données |
| `plotly` | `plot_ly(dpe, x = ~x, y = ~y, type = "scatter", mode = "markers")` | nuage interactif |

### Exemple

```r
# Base R : transparence pour voir les zones denses
plot(dpe$surface_habitable_logement, dpe$conso_5_usages_ep,
     pch = 16, col = rgb(0, 0, 0, 0.2),
     xlab = "Surface (m²)", ylab = "Consommation totale (kWh EP/an)")
abline(lm(conso_5_usages_ep ~ surface_habitable_logement, data = dpe), col = "red")

# ggplot2 : zoom sur la zone la plus dense, avec une droite de tendance
ggplot(dpe, aes(x = surface_habitable_logement, y = conso_5_usages_ep)) +
  geom_point(alpha = 0.2) +
  geom_smooth(method = "lm", se = FALSE, colour = "red") +
  coord_cartesian(xlim = c(0, 200), ylim = c(0, 50000)) +
  labs(x = "Surface (m²)", y = "Consommation totale (kWh EP/an)")
```

> **Note : la droite de tendance**
> La droite rouge résume la liaison linéaire, avec la pente estimée par une régression. Elle ne sert ici qu'à lire le graphique : l'estimation du modèle et sa validation font l'objet du chapitre 12.
> Attention au nombre de point qu'on souhaite afficher sur notre graphique, au delà de 10 000 points, la fenêtre graphique devient plus lente voir non fonctionnelle.

### Exercices

1. Tracer le nuage de points de la surface et de la consommation en base R, avec des points transparents. Repérer les points isolés en haut à droite.
<details>
<summary>Correction</summary>

```r
plot(dpe$surface_habitable_logement, dpe$conso_5_usages_ep,
     pch = 16, col = rgb(0, 0, 0, 0.2))
# Les points isolés à droite et en haut sont les grands logements et les
# consommations très élevées, déjà vus dans les exercices du chapitre 4.
```
</details>

2. Refaire le nuage avec `ggplot2`, zoomé jusqu'à 200 m² et 50 000 kWh. Quelle est la forme dominante du nuage ?
<details>
<summary>Correction</summary>

```r
ggplot(dpe, aes(x = surface_habitable_logement, y = conso_5_usages_ep)) +
  geom_point(alpha = 0.2) +
  coord_cartesian(xlim = c(0, 200), ylim = c(0, 50000))
# Le nuage est un faisceau qui monte : plus la surface augmente, plus la consommation augmente,
# avec une dispersion importante autour de la tendance.
```
</details>

3. Ajouter une droite de tendance au nuage `ggplot2`, puis le convertir en graphique interactif avec `ggplotly()`. Survoler un point isolé : quelles valeurs apparaissent ?
<details>
<summary>Correction</summary>

```r
p <- ggplot(dpe, aes(x = surface_habitable_logement, y = conso_5_usages_ep)) +
  geom_point(alpha = 0.2) +
  geom_smooth(method = "lm", se = FALSE, colour = "red") +
  coord_cartesian(xlim = c(0, 200), ylim = c(0, 50000))

ggplotly(p)
```
Le survol affiche la surface et la consommation du point : on identifie ainsi les logements qui s'écartent de la tendance.
</details>

## Exercice 6 – Tableau croisé de deux variables qualitatives

Un **tableau croisé** (ou tableau de contingence) compte les DPE pour chaque couple de modalités : par exemple, le nombre de DPE de type « maison » en étiquette « C ». Les marges donnent les effectifs de chaque variable prise seule.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `table()` | Tableau croisé de deux vecteurs. | `x`, `y` | `table(dpe$etiquette_dpe, dpe$type_batiment)` |
| `xtabs()` | Tableau croisé à partir d'une formule et d'un dataframe. | formule, `data` | `xtabs(~ etiquette_dpe + type_batiment, data = dpe)` |
| `addmargins()` | Ajoute les totaux en ligne et en colonne. | | `addmargins(tableau)` |

### Exemple

```r
tableau <- table(dpe$etiquette_dpe, dpe$type_batiment)
tableau
#     appartement maison immeuble
# A          17      2        0
# B         219      7        6
# C        4162     73       31
# D        3021     87       19
# E        1442     71       13
# F         489     49        7
# G         242     40        3

# Avec les totaux
addmargins(tableau)
```

### Exercices

1. Construire le tableau croisé de `etiquette_dpe` et `type_batiment` avec `xtabs()`. Combien de DPE de type « immeuble » sont en étiquette C ?
<details>
<summary>Correction</summary>

```r
tab <- xtabs(~ etiquette_dpe + type_batiment, data = dpe)
tab["C", "immeuble"]  # 31
```
</details>

2. Ajouter les totaux avec `addmargins()`. Quel est le total de la ligne A et celui de la colonne « maison » ?
<details>
<summary>Correction</summary>

```r
addmargins(tab)
# Ligne A : 19 ; colonne maison : 329 ; total général : 10000
```
</details>

3. Compter les DPE des maisons en étiquette E, F ou G. Combien y en a-t-il au total, et quelle part des maisons cela représente-t-il ?
<details>
<summary>Correction</summary>

```r
maisons_efg <- sum(tab[c("E", "F", "G"), "maison"])  # 71 + 49 + 40 = 160
maisons_efg
round(100 * maisons_efg / sum(tab[, "maison"]), digits = 1)  # 48.6 %
```
Près de la moitié des maisons sont en E, F ou G.
</details>

> **Note : petits effectifs**
> Le type « immeuble » ne compte que 79 DPE, dont 31 en C : les pourcentages calculés sur ce type sont instables. Avant d'interpréter une case, on vérifie son effectif.

## Exercice 7 – Profils lignes et profils colonnes

Un tableau croisé en effectifs est difficile à comparer quand les totaux diffèrent. Les **profils** convertissent les effectifs en pourcentages :
- le **profil ligne** répond à « parmi les DPE de l'étiquette A, quelle part est un appartement ? » (chaque ligne fait 100 %) ;
- le **profil colonne** répond à « parmi les appartements, quelle part est en étiquette A ? » (chaque colonne fait 100 %).

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `prop.table()` | Convertit un tableau en proportions. | `margin` : 1 (lignes), 2 (colonnes) | `prop.table(tab, margin = 1)` |
| `round()` | Arrondit les pourcentages. | `digits` | `round(100 * prop.table(tab, 1), 1)` |

### Exemple

```r
# Profil ligne : chaque ligne somme à 100 %
round(100 * prop.table(tab, margin = 1), digits = 1)
#   appartement maison immeuble
# A        89.5   10.5      0.0
# B        94.4    3.0      2.6
# ...

# Profil colonne : chaque colonne somme à 100 %
round(100 * prop.table(tab, margin = 2), digits = 1)
#   appartement maison immeuble
# A         0.2    0.6      0.0
# B         2.3    2.1      7.6
# C        43.4   22.2     39.2
# ...
```

### Exercices

1. Calculer les profils lignes. Quelle part des DPE en étiquette G sont des appartements ?
<details>
<summary>Correction</summary>

```r
round(100 * prop.table(tab, margin = 1), digits = 1)
# G : appartement 84.9 %
```
</details>

2. Calculer les profils colonnes. Parmi les maisons, quelle est la part en étiquette G ?
<details>
<summary>Correction</summary>

```r
round(100 * prop.table(tab, margin = 2), digits = 1)
# Colonne maison, ligne G : 12.2 %
```
</details>

3. Comparer les deux lectures : la part des maisons en G (question 2) et la part des G qui sont des maisons (profil ligne). Pourquoi sont-elles différentes ?
<details>
<summary>Correction</summary>

```r
round(100 * tab["G", "maison"] / sum(tab["G", ]), digits = 1)  # 14.0 %
round(100 * tab["G", "maison"] / sum(tab[, "maison"]), digits = 1)  # 12.2 %
```
Les deux pourcentages ne répondent pas à la même question : le premier se rapporte aux DPE en G, le second aux maisons. Il faut donc toujours lire l'intitulé du profil.
</details>

4. Calculer le profil ligne du type d'énergie de chauffage (`type_energie_n1`) selon l'étiquette, pour les quatre énergies les plus représentées : électricité, gaz naturel, réseau de chauffage urbain et fioul domestique. Quelle énergie a la plus forte part de DPE en F ou G ?
<details>
<summary>Correction</summary>

```r
dpe_energie <- dpe |>
  filter(type_energie_n1 %in% c("Électricité", "Gaz naturel",
                                "Réseau de Chauffage urbain", "Fioul domestique"))

tab_energie <- xtabs(~ type_energie_n1 + etiquette_dpe, data = dpe_energie)
round(100 * prop.table(tab_energie, margin = 1), digits = 1)
# Profils ligne (%), chaque ligne somme à 100 :
#                              A    B     C     D     E     F     G
# Électricité                0.4  3.7  48.8  25.8  12.3   5.1   3.9
# Gaz naturel                0.0  0.8  37.9  35.2  19.0   5.7   1.5
# Réseau de Chauffage urbain 0.0  2.1  35.0  48.8  10.9   2.7   0.5
# Fioul domestique           0.0  0.0   0.0   5.0  30.0  32.5  32.5

# Part des DPE en F ou G, pour chaque énergie
round(100 * (tab_energie[, "F"] + tab_energie[, "G"]) / rowSums(tab_energie), digits = 1)
# Électricité 9.0 ; Gaz naturel 7.2 ; Réseau 3.2 ; Fioul domestique 65.0
```
Le fioul domestique est nettement le plus défavorable : 65 % de ses DPE sont en F ou G. Il ne repose toutefois que sur 80 DPE, donc le pourcentage est instable. Parmi les énergies les plus fréquentes, l'électricité (9,0 %) devance le gaz naturel (7,2 %) et le réseau (3,2 %).

Cette liaison ne prouve pas que le fioul dégrade l'étiquette : le type de bâtiment ou l'époque de construction peuvent expliquer une partie de l'écart. Vérifier cela demande une analyse à plusieurs variables (chapitres 15 et 16).
</details>

> **Note : le sens du profil**
> Le profil doit correspondre à la question posée. Si on veut savoir quel type de bâtiment est le plus touché par une mauvaise étiquette, on lit le profil ligne. Si on veut savoir comment se répartissent les étiquettes d'un type de bâtiment, on lit le profil colonne.

## Exercice 8 – Barres groupées et barres empilées

Pour représenter un tableau croisé, on dispose les barres côte à côte (**groupées**) pour comparer les effectifs, ou empilées (**empilées**) pour voir la composition de chaque barre. L'empilement en pourcentage (`position = "fill"`) donne les profils colonnes sous forme graphique.

### Mémo

| Outil | Commande | Argument pertinent |
|-------|----------|--------------------|
| Base R | `barplot(tab, beside = TRUE)` | `beside = TRUE` : barres groupées |
| Base R | `barplot(tab, col = couleurs_dpe)` | barres empilées (une par colonne) |
| `ggplot2` | `geom_bar(position = "dodge")` | barres groupées |
| `ggplot2` | `geom_bar(position = "stack")` | barres empilées, en effectifs |
| `ggplot2` | `geom_bar(position = "fill")` | barres empilées, en pourcentage |

### Exemple

```r
# Base R : barres groupées, une barre par étiquette et par type
barplot(tab, beside = TRUE, col = couleurs_dpe,
        legend.text = TRUE, main = "Étiquette selon le type de bâtiment")

# ggplot2 : barres empilées en pourcentage, une barre par type de bâtiment
ggplot(dpe, aes(x = type_batiment, fill = etiquette_dpe)) +
  geom_bar(position = "fill") +
  scale_fill_manual(values = couleurs_dpe) +
  scale_y_continuous(labels = scales::percent) +
  labs(x = "Type de bâtiment", y = "Part des DPE", fill = "Étiquette")
```

> **Note : `beside` et l'ordre des barres**
> Dans `barplot()`, le tableau doit être croisé avec les lignes pour les couleurs (étiquettes) et les colonnes pour les groupes (types de bâtiment). Avec `beside = TRUE`, chaque groupe de barres correspond à une colonne du tableau.

### Exercices

1. Tracer les barres empilées en effectifs de l'étiquette selon le type de bâtiment avec `geom_bar()` (position `"stack"`). Pourquoi le type « appartement » domine-t-il visuellement ?
<details>
<summary>Correction</summary>

```r
ggplot(dpe, aes(x = type_batiment, fill = etiquette_dpe)) +
  geom_bar() +
  scale_fill_manual(values = couleurs_dpe) +
  labs(x = "Type de bâtiment", y = "Effectif", fill = "Étiquette")
# Les appartements sont 9 592 sur 10 000 : la barre est beaucoup plus haute.
```
</details>

2. Passer en pourcentage (`position = "fill"`). Quelle étiquette est la plus représentée chez les maisons, et chez les appartements ?
<details>
<summary>Correction</summary>

```r
ggplot(dpe, aes(x = type_batiment, fill = etiquette_dpe)) +
  geom_bar(position = "fill") +
  scale_fill_manual(values = couleurs_dpe) +
  scale_y_continuous(labels = scales::percent)
# Appartements : C (43,4 %) en tête ; maisons : D (26,4 %) devant C (22,2 %).
```
</details>

3. Tracer les barres groupées des effectifs avec `barplot(tab, beside = TRUE)`. Comparer avec le graphique empilé : quelle lecture est la plus facile pour les petits effectifs ?
<details>
<summary>Correction</summary>

```r
barplot(tab, beside = TRUE, col = couleurs_dpe, legend.text = TRUE)
# Les barres groupées permettent de comparer les petits effectifs (immeubles en A, par exemple),
# que l'empilement cache sous la base du graphique.
```
</details>

## Exercice 9 – Variable quantitative selon une variable qualitative

On compare une variable quantitative selon les modalités d'une variable qualitative : par exemple, la surface selon l'étiquette DPE. On calcule d'abord les **statistiques par groupe**, puis on les représente.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `tapply()` | Applique une fonction à un vecteur, par groupe. | `X`, `INDEX`, `FUN` | `tapply(surface, etiquette, mean, na.rm = TRUE)` |
| `aggregate()` | Applique une fonction par groupe, avec une formule. | formule, `data`, `FUN` | `aggregate(surface ~ etiquette, data = dpe, FUN = mean)` |
| `group_by()` + `summarise()` | Équivalent `dplyr`, pour plusieurs statistiques à la fois. | | voir chapitre 3 |

### Exemple

```r
# Moyenne de la surface par étiquette, en base R
tapply(dpe$surface_habitable_logement, dpe$etiquette_dpe, mean, na.rm = TRUE)

# Même calcul avec dplyr, avec l'effectif et la médiane
dpe |>
  group_by(etiquette_dpe) |>
  summarise(
    effectif = n(),
    surface_moyenne = mean(surface_habitable_logement, na.rm = TRUE),
    surface_mediane = median(surface_habitable_logement, na.rm = TRUE)
  )
```

### Exercices

1. Calculer la surface moyenne par étiquette avec `tapply()`. Quelle étiquette a la surface moyenne la plus élevée ?
<details>
<summary>Correction</summary>

```r
tapply(dpe$surface_habitable_logement, dpe$etiquette_dpe, mean, na.rm = TRUE)
# B : 92.67 m² (la plus élevée) ; G : 45.55 m² (la plus basse)
```
</details>

2. Calculer l'effectif, la surface moyenne et la surface médiane par étiquette avec `dplyr`. La médiane est-elle toujours proche de la moyenne ?
<details>
<summary>Correction</summary>

```r
dpe |>
  group_by(etiquette_dpe) |>
  summarise(
    effectif = n(),
    surface_moyenne = mean(surface_habitable_logement, na.rm = TRUE),
    surface_mediane = median(surface_habitable_logement, na.rm = TRUE)
  )
# Pour B, la moyenne (92.67) est nettement au-dessus de la médiane (70.0) :
# quelques très grands logements tirent la moyenne vers le haut.
```
</details>

3. Calculer, pour chaque étiquette, la consommation par m² moyenne (`conso_5_usages_ep / surface_habitable_logement`), en ignorant les surfaces manquantes. Que constate-t-on quand l'étiquette se dégrade ?
<details>
<summary>Correction</summary>

```r
dpe |>
  filter(!is.na(surface_habitable_logement)) |>
  mutate(conso_m2 = conso_5_usages_ep / surface_habitable_logement) |>
  group_by(etiquette_dpe) |>
  summarise(conso_m2_moyenne = mean(conso_m2)) |>
  arrange(etiquette_dpe)
# A : 65.2 ; B : 93.9 ; C : 144.2 ; D : 205.3 ; E : 277.7 ; F : 363.3 ; G : 518.8
```
La consommation par m² augmente nettement de A à G, alors que la surface moyenne ne suit pas la même progression (exercice 2) : l'indicateur par m² est donc plus parlant pour comparer la performance.
</details>

> **Note : moyenne et médiane**
> Une moyenne très différente de la médiane signale une distribution asymétrique ou des valeurs extrêmes. Pour une liaison quantitative–qualitative, on compare les deux ensemble avant de conclure.

## Exercice 10 – Boxplots par groupe et graphiques interactifs

Le boxplot par groupe montre la distribution complète de la variable dans chaque modalité : médiane, quartiles, valeurs extrêmes. C'est le complément des moyennes de l'exercice 9.

### Mémo

| Outil | Commande | Argument pertinent |
|-------|----------|--------------------|
| Base R | `boxplot(surface ~ etiquette, data = dpe, outline = FALSE)` | formule `y ~ x` |
| `ggplot2` | `ggplot(dpe, aes(x = etiquette_dpe, y = surface)) + geom_boxplot(outlier.shape = NA)` | `outlier.shape = NA` masque les points |
| `ggplot2` (zoom) | `+ coord_cartesian(ylim = c(0, 200))` | zoom sans retirer les données |
| `plotly` | `plot_ly(dpe, x = ~etiquette, y = ~surface, type = "box", color = ~etiquette)` | boxplots interactifs |

### Exemple

```r
# Base R : boxplots par étiquette, sans les points extrêmes
boxplot(surface_habitable_logement ~ etiquette_dpe, data = dpe,
        outline = FALSE, col = couleurs_dpe,
        xlab = "Étiquette", ylab = "Surface (m²)")

# ggplot2 : même graphique, zoom jusqu'à 200 m²
ggplot(dpe, aes(x = etiquette_dpe, y = surface_habitable_logement, fill = etiquette_dpe)) +
  geom_boxplot(outlier.shape = NA) +
  scale_fill_manual(values = couleurs_dpe) +
  guides(fill = "none") +
  coord_cartesian(ylim = c(0, 200)) +
  labs(x = "Étiquette", y = "Surface (m²)")
```

### Exercices

1. Tracer les boxplots de la surface par étiquette en base R, avec `outline = FALSE`. Quelle étiquette a la médiane la plus haute ? Et la plus basse ?
<details>
<summary>Correction</summary>

```r
boxplot(surface_habitable_logement ~ etiquette_dpe, data = dpe,
        outline = FALSE, col = couleurs_dpe)
# Médiane la plus haute : B (70 m²) ; la plus basse : G (37,65 m²).
```
</details>

2. Reproduire le graphique avec `ggplot2`, zoomé jusqu'à 200 m². Les boîtes se recouvrent-elles entre C et D ?
<details>
<summary>Correction</summary>

```r
ggplot(dpe, aes(x = etiquette_dpe, y = surface_habitable_logement, fill = etiquette_dpe)) +
  geom_boxplot(outlier.shape = NA) +
  scale_fill_manual(values = couleurs_dpe) +
  guides(fill = "none") +
  coord_cartesian(ylim = c(0, 200))
# Les médianes des étiquettes C, D, E et F sont proches (entre 50 et 56 m²) :
# la surface seule ne distingue pas bien les étiquettes intermédiaires.
```
</details>

3. Tracer les boxplots interactifs avec `plot_ly()`. Survoler la médiane de l'étiquette G : quelle valeur s'affiche ? Comparer avec l'exercice 9.
<details>
<summary>Correction</summary>

```r
plot_ly(dpe, x = ~etiquette_dpe, y = ~surface_habitable_logement,
        type = "box", color = ~etiquette_dpe, colors = unname(couleurs_dpe)) |>
  layout(showlegend = FALSE,
         yaxis = list(range = c(0, 200), title = "Surface (m²)"))
# Le survol de la médiane de G affiche environ 37,65 : c'est la médiane de l'exercice 9.
```
Le zoom par `layout(yaxis = list(range = ...))` ne retire aucune donnée.
</details>

4. Représenter avec `ggplot2` la consommation par m² selon l'étiquette, avec un boxplot zoomé sur 1 000 kWh/m²/an. Pourquoi la lecture est-elle plus claire qu'avec la surface ?
<details>
<summary>Correction</summary>

```r
dpe |>
  filter(!is.na(surface_habitable_logement)) |>
  mutate(conso_m2 = conso_5_usages_ep / surface_habitable_logement) |>
  ggplot(aes(x = etiquette_dpe, y = conso_m2, fill = etiquette_dpe)) +
  geom_boxplot(outlier.shape = NA) +
  scale_fill_manual(values = couleurs_dpe) +
  guides(fill = "none") +
  coord_cartesian(ylim = c(0, 1000))
# Les boîtes sont décalées vers le haut à mesure que l'étiquette se dégrade :
# la consommation par m² sépare mieux les étiquettes que la surface.
```
</details>

## Liens utiles

- [Documentation de `ggplot2`](https://ggplot2.tidyverse.org/)
- [Documentation de `plotly` pour R](https://plotly.com/r/)
- [Documentation de `esquisse`](https://dreamrs.github.io/esquisse/)
