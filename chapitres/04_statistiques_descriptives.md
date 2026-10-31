# Chapitre 4 – Statistiques descriptives d'une variable

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] calculer les indicateurs de position : moyenne, médiane, quartiles, déciles, centiles
- [ ] calculer les indicateurs de dispersion : variance, écart type, étendue, écart interquartile
- [ ] repérer des valeurs extrêmes avec la règle de l'écart interquartile
- [ ] représenter une variable quantitative avec un histogramme et un boxplot
- [ ] calculer les effectifs, fréquences et effectifs cumulés d'une variable qualitative
- [ ] représenter une variable qualitative avec un diagramme en barres et un diagramme circulaire
- [ ] reproduire un graphique avec les fonctions de base, `ggplot2`, puis `esquisse`
- [ ] rendre un graphique interactif avec `plotly` (survol, zoom) et le convertir depuis `ggplot2`

## Sommaire

- [Chapitre 4 – Statistiques descriptives d'une variable](#chapitre-4--statistiques-descriptives-dune-variable)
  - [Objectifs](#objectifs)
  - [Sommaire](#sommaire)
  - [Préparation : packages, données et couleurs](#préparation--packages-données-et-couleurs)
  - [Exercice 1 – Indicateurs de position](#exercice-1--indicateurs-de-position)
    - [Mémo](#mémo)
    - [Exemple](#exemple)
    - [Exercices](#exercices)
  - [Exercice 2 – Indicateurs de dispersion](#exercice-2--indicateurs-de-dispersion)
    - [Mémo](#mémo-1)
    - [Exemple](#exemple-1)
    - [Exercices](#exercices-1)
  - [Exercice 3 – Valeurs extrêmes](#exercice-3--valeurs-extrêmes)
    - [Mémo](#mémo-2)
    - [Exemple](#exemple-2)
    - [Exercices](#exercices-2)
  - [Exercice 4 – Histogramme d'une variable quantitative](#exercice-4--histogramme-dune-variable-quantitative)
    - [Mémo](#mémo-3)
    - [Exemple](#exemple-3)
    - [Exercices](#exercices-3)
  - [Exercice 5 – Boxplot d'une variable quantitative](#exercice-5--boxplot-dune-variable-quantitative)
    - [Mémo](#mémo-4)
    - [Exemple](#exemple-4)
    - [Exercices](#exercices-4)
  - [Exercice 6 – Effectifs, fréquences et effectifs cumulés](#exercice-6--effectifs-fréquences-et-effectifs-cumulés)
    - [Mémo](#mémo-5)
    - [Exemple](#exemple-5)
    - [Exercices](#exercices-5)
  - [Exercice 7 – Diagramme en barres](#exercice-7--diagramme-en-barres)
    - [Mémo](#mémo-6)
    - [Exemple](#exemple-6)
    - [Exercices](#exercices-6)
  - [Exercice 8 – Diagramme circulaire](#exercice-8--diagramme-circulaire)
    - [Mémo](#mémo-7)
    - [Exemple](#exemple-7)
    - [Exercices](#exercices-7)
  - [Exercice 9 – Reproduire un graphique avec `esquisse`](#exercice-9--reproduire-un-graphique-avec-esquisse)
    - [Mémo](#mémo-8)
    - [Exemple](#exemple-8)
    - [Exercices](#exercices-8)
  - [Exercice 10 – Graphiques interactifs avec `plotly`](#exercice-10--graphiques-interactifs-avec-plotly)
    - [Mémo](#mémo-9)
    - [Exemple](#exemple-9)
    - [Exercices](#exercices-9)
  - [Liens utiles](#liens-utiles)

## Préparation : packages, données et couleurs

Les graphiques se font de quatre façons dans ce chapitre :
- avec les **fonctions de base** de R (`hist()`, `boxplot()`, `barplot()`, `pie()`), disponibles sans package ;
- avec **`ggplot2`**, le package de référence pour les graphiques, qui construit un graphique couche par couche ;
- avec **`esquisse`**, une interface qui permet de construire un graphique à la souris, puis d'en récupérer le code `ggplot2` ;
- avec **`plotly`**, qui produit des graphiques interactifs : on voit la valeur au survol et on peut zoomer.

```r
install.packages(c("ggplot2", "esquisse", "plotly"))
library(readr)
library(dplyr)
library(ggplot2)
library(esquisse)
library(plotly)

dpe <- read_csv("data/dpe_villeurbanne_10000.csv")
surface <- dpe$surface_habitable_logement

# Couleurs indicatives, proches de celles des étiquettes DPE
couleurs_dpe <- c(A = "#319834", B = "#33cc31", C = "#cbfc0c", D = "#fff200",
                  E = "#fcaf17", F = "#ef7d1a", G = "#e2001a")
```

> **Note : les valeurs manquantes dans les graphiques**
> `hist()` et `boxplot()` ignorent les `NA` sans prévenir. `ggplot2` les ignore aussi, mais l'affiche dans la console : `Removed 101 rows containing non-finite values`. Ce message indique qu'il y a 101 surfaces manquantes sur les 10 000 DPE.

## Exercice 1 – Indicateurs de position

Les indicateurs de position donnent le centre et la répartition d'une variable : moyenne, médiane, quartiles, déciles, centiles. Un quantile $p$ est la valeur en dessous de laquelle se trouve une proportion $p$ des observations.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `mean()` | Moyenne. | `na.rm` | `mean(surface, na.rm = TRUE)` |
| `median()` | Médiane (quantile à 50 %). | `na.rm` | `median(surface, na.rm = TRUE)` |
| `quantile()` | Quantiles pour les proportions données. | `probs`, `na.rm` | `quantile(surface, probs = c(0.25, 0.5, 0.75), na.rm = TRUE)` |
| `summary()` | Résumé : minimum, quartiles, moyenne, maximum, nombre de `NA`. | | `summary(surface)` |

Les déciles correspondent à `probs = seq(0.1, 0.9, by = 0.1)`, les centiles à `probs = 0.01` ou `0.99` par exemple.

### Exemple

```r
summary(surface)
#    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's
#    8.50   35.00   54.00   55.93   70.90 3330.00     101

# Quartiles : 25 %, 50 % et 75 % des logements sont en dessous de ces valeurs
quantile(surface, probs = c(0.25, 0.5, 0.75), na.rm = TRUE)
#   25%   50%   75%
# 35.0  54.0  70.9

# Déciles
quantile(surface, probs = seq(0.1, 0.9, by = 0.1), na.rm = TRUE)
```

### Exercices

1. Calculer la moyenne et la médiane de `surface`. Pourquoi sont-elles différentes ?
<details>
<summary>Correction</summary>

```r
mean(surface, na.rm = TRUE)    # 55.93
median(surface, na.rm = TRUE)  # 54
```
La moyenne est tirée vers le haut par les très grandes surfaces (jusqu'à 3 330 m²). La médiane, elle, ne dépend pas des valeurs extrêmes : elle reste donc plus représentative ici.
</details>

2. Calculer les quartiles de `surface`. Interpréter le troisième : que signifie la valeur obtenue ?
<details>
<summary>Correction</summary>

```r
quantile(surface, probs = c(0.25, 0.5, 0.75), na.rm = TRUE)
# 35.0  54.0  70.9
```
Le troisième quartile vaut 70,9 m² : 75 % des logements ont une surface inférieure ou égale à 70,9 m².
</details>

3. Calculer les déciles 1 à 9, puis le 90e centile et le 99e centile. Combien de logements dépassent 130,6 m² ?
<details>
<summary>Correction</summary>

```r
quantile(surface, probs = seq(0.1, 0.9, by = 0.1), na.rm = TRUE)
quantile(surface, probs = c(0.9, 0.99), na.rm = TRUE)
# 90 % : 86.6 ; 99 % : 130.6

sum(surface > 130.6, na.rm = TRUE)  # environ 1 % des logements
```
Le 99e centile est à 130,6 m² : environ 1 % des logements dépassent cette surface.
</details>

4. Afficher le résumé de `surface` avec `summary()`. Combien de valeurs manquantes sont indiquées ?
<details>
<summary>Correction</summary>

```r
summary(surface)
# La dernière ligne indique NA's : 101 valeurs manquantes.
```
</details>

> **Note : `summary()` et `quantile()`**
> `summary()` donne le minimum, les quartiles, la moyenne et le maximum en une seule ligne de commande. `quantile()` permet de choisir les proportions voulues, ce qui est utile pour les déciles ou les centiles.

## Exercice 2 – Indicateurs de dispersion

Les indicateurs de dispersion mesurent l'étalement des valeurs autour du centre : variance, écart type, étendue, écart interquartile (IQR). La variance et l'écart type sont calculés autour de la moyenne ; l'IQR et l'étendue ne dépendent que de quelques valeurs.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `var()` | Variance. | `na.rm` | `var(surface, na.rm = TRUE)` |
| `sd()` | Écart type (racine carrée de la variance). | `na.rm` | `sd(surface, na.rm = TRUE)` |
| `range()` | Minimum et maximum. | `na.rm` | `range(surface, na.rm = TRUE)` |
| `max() - min()` | Étendue : écart entre le maximum et le minimum. | `na.rm` | `max(surface, na.rm = TRUE) - min(surface, na.rm = TRUE)` |
| `IQR()` | Écart interquartile : `Q3 - Q1`. | `na.rm` | `IQR(surface, na.rm = TRUE)` |

### Exemple

```r
var(surface, na.rm = TRUE)   # 3632.45
sd(surface, na.rm = TRUE)    # 60.27

# Étendue : écart entre la plus grande et la plus petite valeur
diff(range(surface, na.rm = TRUE))  # 3321.5

# Écart interquartile : Q3 - Q1
IQR(surface, na.rm = TRUE)   # 35.9
```

### Exercices

1. Calculer la variance et l'écart type de `surface`. Vérifier que l'écart type est bien la racine carrée de la variance.
<details>
<summary>Correction</summary>

```r
v <- var(surface, na.rm = TRUE)
s <- sd(surface, na.rm = TRUE)
sqrt(v)  # 60.27
s        # 60.27 : même valeur
```
</details>

2. Calculer l'étendue de `surface`. Pourquoi cette valeur est-elle peu utile ici ?
<details>
<summary>Correction</summary>

```r
diff(range(surface, na.rm = TRUE))  # 3321.5
```
L'étendue est de 3 321,5 m², alors que 99 % des logements ont une surface inférieure à 130,6 m². Un seul logement très grand suffit à la fausser : l'étendue ne mesure que les deux valeurs extrêmes.
</details>

3. Calculer l'écart interquartile, puis le calculer à la main à partir des quartiles. Les deux résultats doivent être identiques.
<details>
<summary>Correction</summary>

```r
q <- quantile(surface, probs = c(0.25, 0.75), na.rm = TRUE)
q[2] - q[1]               # 35.9
IQR(surface, na.rm = TRUE) # 35.9
```
</details>

4. Calculer l'écart type et l'écart interquartile après avoir retiré les DPE dont la surface dépasse 300 m². Comment évoluent-ils ?
<details>
<summary>Correction</summary>

```r
surface_300 <- surface[surface <= 300]
sd(surface_300, na.rm = TRUE)   # 26.71 : l'écart type passe de 60,27 à 26,71
IQR(surface_300, na.rm = TRUE)  # 35.8 : l'IQR change très peu (35,9 avant)
```
L'écart type est très sensible aux grandes valeurs, alors que l'écart interquartile reste stable. C'est pourquoi on l'utilise souvent avec la médiane.
</details>

## Exercice 3 – Valeurs extrêmes

Une valeur extrême s'écarte nettement du reste des données. On utilise souvent la **règle de l'écart interquartile** : une valeur est suspecte si elle est au-dessus de `Q3 + 1,5 × IQR` ou en dessous de `Q1 − 1,5 × IQR`.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `quantile()` | Calcule Q1 et Q3. | `probs` | `quantile(surface, c(0.25, 0.75), na.rm = TRUE)` |
| `which()` | Renvoie les positions où une condition est vraie. | | `which(surface > 124)` |
| `sum()` | Compte les `TRUE` d'une condition. | | `sum(surface > 124.75, na.rm = TRUE)` |

### Exemple

```r
q <- quantile(surface, probs = c(0.25, 0.75), na.rm = TRUE)
iqr <- IQR(surface, na.rm = TRUE)

borne_inf <- q[1] - 1.5 * iqr   # -18.85
borne_sup <- q[2] + 1.5 * iqr   # 124.75

# Nombre de logements au-dessus de la borne supérieure
sum(surface > borne_sup, na.rm = TRUE)
# [1] 136
```

### Exercices

1. Calculer `borne_sup` et `borne_inf` à partir des quartiles. Y a-t-il des valeurs sous la borne inférieure ?
<details>
<summary>Correction</summary>

```r
borne_sup  # 124.75
borne_inf  # -18.85
sum(surface < borne_inf, na.rm = TRUE)  # 0
```
Aucune surface n'est négative : la borne inférieure n'écarte aucune valeur.
</details>

2. Compter les logements dont la surface dépasse la borne supérieure. Afficher leurs `numero_dpe` et leur surface, triés par surface décroissante, pour les 5 plus grandes.
<details>
<summary>Correction</summary>

```r
dpe |>
  dplyr::filter(surface_habitable_logement > borne_sup) |>
  dplyr::arrange(dplyr::desc(surface_habitable_logement)) |>
  dplyr::select(numero_dpe, surface_habitable_logement) |>
  head(n = 5)
```
Ces logements sont en dehors de la plage habituelle. Ils sont signalés, sans être supprimés : le chapitre 5 montre comment les vérifier dans les données.
</details>

3. Calculer la moyenne de `surface` avec et sans les valeurs au-dessus de `borne_sup`. Quelle valeur est la plus représentative d'un logement « typique » ?
<details>
<summary>Correction</summary>

```r
mean(surface, na.rm = TRUE)                        # 55.93
mean(surface[surface <= borne_sup], na.rm = TRUE)  # 53.48
```
La moyenne sans valeurs extrêmes est plus basse : les 136 grands logements tirent la moyenne vers le haut. Retirer des données est une décision à justifier, pas un réflexe.
</details>

## Exercice 4 – Histogramme d'une variable quantitative

Un histogramme découpe les valeurs en intervalles (classes) et compte les observations de chaque classe. Il montre la forme de la distribution : symétrique, étalée, avec une queue à droite, etc.

### Mémo

| Outil | Commande | Argument pertinent |
|-------|----------|--------------------|
| Base R | `hist(x, breaks = 30, col = "steelblue")` | `breaks` : nombre de classes ; `xlim` : plage affichée |
| `ggplot2` | `ggplot(dpe, aes(x = surface_habitable_logement)) + geom_histogram(bins = 30)` | `bins` : nombre de classes |
| `ggplot2` (zoom) | `+ coord_cartesian(xlim = c(0, 200))` | affiche une plage sans retirer les données |

### Exemple

```r
# Base R : histogramme complet
hist(surface, breaks = 30, col = "steelblue",
     main = "Surface habitable des logements de Villeurbanne",
     xlab = "Surface (m²)", ylab = "Effectif")

# ggplot2 : même graphique, couche par couche
ggplot(dpe, aes(x = surface_habitable_logement)) +
  geom_histogram(bins = 30, fill = "steelblue", colour = "white") +
  labs(x = "Surface (m²)", y = "Effectif")

# Zoom sur les surfaces jusqu'à 200 m² : les données ne sont pas retirées
ggplot(dpe, aes(x = surface_habitable_logement)) +
  geom_histogram(bins = 50, fill = "steelblue", colour = "white") +
  coord_cartesian(xlim = c(0, 200)) +
  labs(x = "Surface (m²)", y = "Effectif")
```

> **Note : `xlim` et `coord_cartesian()`**
> Dans `ggplot2`, `xlim()` (ou `scale_x_continuous(limits = ...)`) retire les valeurs hors plage avant de calculer le graphique. `coord_cartesian()` change seulement la fenêtre d'affichage : les données restent prises en compte. On utilise donc `coord_cartesian()` pour zoomer.

### Exercices

1. Tracer l'histogramme de `surface` avec `hist()`, avec 30 classes. Quelle est la forme de la distribution ?
<details>
<summary>Correction</summary>

```r
hist(surface, breaks = 30, col = "steelblue")
# La distribution est étalée vers la droite : la moitié des logements ont une surface
# entre 35 et 71 m² (Q1 à Q3), avec quelques très grandes valeurs.
```
</details>

2. Refaire l'histogramme avec `ggplot2` et `geom_histogram()`. Quel message s'affiche dans la console à propos des valeurs manquantes ?
<details>
<summary>Correction</summary>

```r
ggplot(dpe, aes(x = surface_habitable_logement)) +
  geom_histogram(bins = 30, fill = "steelblue", colour = "white")
```
</details>

3. Zoomer sur les surfaces jusqu'à 150 m² avec `coord_cartesian()`, avec 60 classes. Comparer avec l'histogramme complet.
<details>
<summary>Correction</summary>

```r
ggplot(dpe, aes(x = surface_habitable_logement)) +
  geom_histogram(bins = 60, fill = "steelblue", colour = "white") +
  coord_cartesian(xlim = c(0, 150))
```
Le zoom montre la forme autour du pic, sans effacer les grandes surfaces du calcul.
</details>

4. Changer le nombre de classes (5, 30, 200 avec `hist()`). Que constate-t-on ?
<details>
<summary>Correction</summary>

```r
hist(surface, breaks = 5)
hist(surface, breaks = 30)
hist(surface, breaks = 200)
```
Avec peu de classes, l'histogramme est trop grossier ; avec beaucoup de classes, il devient bruité. Le nombre de classes change l'impression visuelle : on compare plusieurs valeurs avant de conclure.
</details>

## Exercice 5 – Boxplot d'une variable quantitative

Le boxplot (boîte à moustaches) résume une variable avec cinq valeurs : minimum (hors valeurs extrêmes), Q1, médiane, Q3, maximum (hors valeurs extrêmes). Les valeurs au-delà des bornes de l'exercice 3 sont affichées comme des points.

### Mémo

| Outil | Commande | Argument pertinent |
|-------|----------|--------------------|
| Base R | `boxplot(x, outline = FALSE)` | `outline = FALSE` masque les points extrêmes |
| `ggplot2` | `ggplot(dpe, aes(x = "", y = surface_habitable_logement)) + geom_boxplot()` | `outlier.shape = NA` masque les points |
| `ggplot2` (zoom) | `+ coord_cartesian(ylim = c(0, 200))` | affiche une plage sans retirer les données |

### Exemple

```r
# Base R : boxplot complet, points extrêmes affichés
boxplot(surface, main = "Surface habitable", ylab = "Surface (m²)")

# Base R : sans les points extrêmes, pour lire la boîte
boxplot(surface, outline = FALSE, main = "Surface habitable", ylab = "Surface (m²)")

# ggplot2 : boîte seule, zoom jusqu'à 200 m²
ggplot(dpe, aes(x = "", y = surface_habitable_logement)) +
  geom_boxplot(outlier.shape = NA, fill = "steelblue") +
  coord_cartesian(ylim = c(0, 200)) +
  labs(x = NULL, y = "Surface (m²)")
```

> **Note : `outline = FALSE` et `outlier.shape = NA`**
> Les points extrêmes existent toujours dans les données : ces options les cachent seulement dans le graphique. La boîte devient lisible, mais les extrêmes doivent être mentionnés dans le commentaire du graphique.

### Exercices

1. Tracer le boxplot de `surface` en base R, puis un second avec `outline = FALSE`. Que change le second ?
<details>
<summary>Correction</summary>

```r
boxplot(surface)
boxplot(surface, outline = FALSE)
# Le second ne montre pas les points isolés au-dessus de la moustache :
# la boîte et la moustache restent les mêmes.
```
</details>

2. Reproduire le boxplot avec `ggplot2`, sans les points extrêmes, zoom jusqu'à 200 m². Repérer la médiane et l'écart interquartile sur le graphique.
<details>
<summary>Correction</summary>

```r
ggplot(dpe, aes(x = "", y = surface_habitable_logement)) +
  geom_boxplot(outlier.shape = NA, fill = "steelblue") +
  coord_cartesian(ylim = c(0, 200)) +
  labs(x = NULL, y = "Surface (m²)")
# Le trait au milieu de la boîte est la médiane (54 m²).
# La boîte va de Q1 (35 m²) à Q3 (70,9 m²) : sa hauteur est l'IQR.
```
</details>

3. Vérifier sur le graphique que la moustache supérieure s'arrête près de 124,75 m², la borne de l'exercice 3.
<details>
<summary>Correction</summary>

```r
ggplot(dpe, aes(x = "", y = surface_habitable_logement)) +
  geom_boxplot(outlier.shape = NA, fill = "steelblue") +
  coord_cartesian(ylim = c(0, 200)) +
  geom_hline(yintercept = 124.75, colour = "red", linetype = "dashed") +
  labs(x = NULL, y = "Surface (m²)")
```
La moustache supérieure s'arrête à la dernière valeur non extrême, proche de la ligne rouge.
</details>

## Exercice 6 – Effectifs, fréquences et effectifs cumulés

Une variable qualitative prend des modalités (ici les étiquettes A à G). On compte les **effectifs** de chaque modalité, leurs **fréquences** (part du total, en %), et les **effectifs cumulés** (somme des effectifs jusqu'à la modalité).

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `table()` | Effectifs de chaque modalité. | | `table(dpe$etiquette_dpe)` |
| `prop.table()` | Fréquences, entre 0 et 1. | | `prop.table(table(dpe$etiquette_dpe))` |
| `round()` | Arrondit les fréquences. | `digits` | `round(100 * prop.table(...), digits = 2)` |
| `cumsum()` | Somme cumulée d'un vecteur. | | `cumsum(table(dpe$etiquette_dpe))` |

### Exemple

```r
effectifs <- table(dpe$etiquette_dpe)
effectifs
#    A    B    C    D    E    F    G
#   19  232 4266 3127 1526  545  285

# Fréquences en pourcentage, arrondies à deux décimales
round(100 * prop.table(effectifs), digits = 2)
#     A     B     C     D     E     F     G
#  0.19  2.32 42.66 31.27 15.26  5.45  2.85

# Effectifs cumulés
cumsum(effectifs)
#    A    B    C    D    E    F    G
#   19  251 4517 7644 9170 9715 10000
```

### Exercices

1. Calculer les effectifs et les fréquences des étiquettes. Quelle est l'étiquette la plus fréquente ?
<details>
<summary>Correction</summary>

```r
effectifs <- table(dpe$etiquette_dpe)
round(100 * prop.table(effectifs), digits = 2)
# L'étiquette C est la plus fréquente, avec 42,66 % des DPE.
```
</details>

2. Calculer les effectifs cumulés. Combien de DPE sont en étiquette D ou mieux (A, B, C ou D) ?
<details>
<summary>Correction</summary>

```r
cumsum(effectifs)["D"]
# D : 7644
```
7 644 DPE sont en étiquette D ou mieux.
</details>

3. Calculer la fréquence cumulée (en %) des étiquettes E, F et G, c'est-à-dire la part des DPE les moins performants.
<details>
<summary>Correction</summary>

```r
round(100 * (effectifs["E"] + effectifs["F"] + effectifs["G"]) / sum(effectifs), digits = 2)
# 23.56
```
Les étiquettes E, F et G représentent 23,56 % des DPE.
</details>

> **Note : `table()` n'est pas un dataframe**
> `table()` renvoie un tableau d'effectifs. Pour le manipuler avec `dplyr` ou l'exporter, on le convertit avec `as.data.frame()`, comme vu au chapitre 2.

## Exercice 7 – Diagramme en barres

Un diagramme en barres représente les effectifs (ou les fréquences) de chaque modalité par une barre. C'est le graphique de référence pour une variable qualitative.

### Mémo

| Outil | Commande | Argument pertinent |
|-------|----------|--------------------|
| Base R | `barplot(effectifs, col = couleurs_dpe[names(effectifs)])` | `col` : couleurs des barres ; `horiz = TRUE` : barres horizontales |
| `ggplot2` | `ggplot(dpe, aes(x = etiquette_dpe)) + geom_bar()` | `geom_bar()` compte les effectifs |
| `ggplot2` (pourcentage) | `aes(y = after_stat(prop), group = 1)` | `prop` : fréquence de chaque barre |

### Exemple

```r
# Base R : barres des effectifs, couleurs des étiquettes
effectifs <- table(dpe$etiquette_dpe)
barplot(effectifs, col = couleurs_dpe[names(effectifs)],
        main = "Étiquettes DPE", xlab = "Étiquette", ylab = "Effectif")

# ggplot2 : même graphique
ggplot(dpe, aes(x = etiquette_dpe, fill = etiquette_dpe)) +
  geom_bar() +
  scale_fill_manual(values = couleurs_dpe) +
  guides(fill = "none") +
  labs(x = "Étiquette", y = "Effectif")

# ggplot2 : barres en pourcentage
ggplot(dpe, aes(x = etiquette_dpe, fill = etiquette_dpe)) +
  geom_bar(aes(y = after_stat(prop), group = 1)) +
  scale_fill_manual(values = couleurs_dpe) +
  guides(fill = "none") +
  scale_y_continuous(labels = scales::percent) +
  labs(x = "Étiquette", y = "Fréquence")
```

### Exercices

1. Tracer le diagramme en barres des effectifs avec `barplot()`, puis le même avec `geom_bar()`.
<details>
<summary>Correction</summary>

```r
barplot(table(dpe$etiquette_dpe), col = couleurs_dpe)

ggplot(dpe, aes(x = etiquette_dpe, fill = etiquette_dpe)) +
  geom_bar() +
  scale_fill_manual(values = couleurs_dpe) +
  guides(fill = "none")
```
</details>

2. Tracer le diagramme en barres horizontales des fréquences, avec `barplot()` et `horiz = TRUE`.
<details>
<summary>Correction</summary>

```r
frequences <- prop.table(table(dpe$etiquette_dpe))
barplot(frequences, horiz = TRUE, col = couleurs_dpe[names(frequences)], las = 1,
        main = "Fréquence des étiquettes", xlab = "Fréquence")
```
`las = 1` écrit les étiquettes horizontalement pour les lire.
</details>

3. Tracer le diagramme en barres des effectifs de `type_energie_n1` avec `geom_bar()`. Pourquoi ce graphique est-il difficile à lire ?
<details>
<summary>Correction</summary>

```r
ggplot(dpe, aes(x = type_energie_n1)) +
  geom_bar(fill = "steelblue") +
  coord_flip()
```
Dix modalités, dont plusieurs avec quelques DPE seulement : les petites barres sont illisibles. On regroupe ces modalités à l'exercice suivant.
</details>

## Exercice 8 – Diagramme circulaire

Un diagramme circulaire (camembert) montre la part de chaque modalité dans le total, sous forme d'angle. Il ne convient qu'à **quelques modalités** : au-delà de 4 ou 5 parts, la comparaison devient difficile.

### Mémo

| Commande | Description | Argument pertinent | Exemple |
|----------|-------------|--------------------|---------|
| `pie()` | Trace un diagramme circulaire (base R). | `x`, `col`, `main` | `pie(effectifs, col = couleurs_dpe)` |
| `ifelse()` | Remplace les valeurs selon une condition. | `test`, `yes`, `no` | `ifelse(x %in% c("A", "B"), x, "Autres")` |

### Exemple

```r
# Regroupement des énergies : les trois principales, puis "Autres"
energies_principales <- c("Électricité", "Gaz naturel", "Réseau de Chauffage urbain")

energie_regroupee <- ifelse(dpe$type_energie_n1 %in% energies_principales,
                            dpe$type_energie_n1, "Autres")

effectifs_energie <- table(energie_regroupee)
effectifs_energie

pie(effectifs_energie, main = "Type d'énergie de chauffage",
    col = c("#4c72b0", "#dd8452", "#55a868", "grey80"))
```

### Exercices

1. Tracer le diagramme circulaire des étiquettes DPE avec `pie()`. Pourquoi ce graphique est-il moins adapté qu'un diagramme en barres ?
<details>
<summary>Correction</summary>

```r
pie(table(dpe$etiquette_dpe), col = couleurs_dpe)
# Sept parts : les petites parts (A, B, F, G) sont difficiles à comparer par l'angle.
# Le diagramme en barres donne une lecture plus précise.
```
</details>

2. Reprendre le regroupement de l'exemple, puis calculer les fréquences de chaque groupe en pourcentage. Quelle part représentent les autres énergies ?
<details>
<summary>Correction</summary>

```r
round(100 * prop.table(effectifs_energie), digits = 1)
# Autres : environ 1 % des DPE (102 sur 10 000)
```
</details>

3. Tracer le diagramme en barres des mêmes effectifs regroupés. Quel graphique choisir pour comparer les groupes ?
<details>
<summary>Correction</summary>

```r
barplot(sort(effectifs_energie, decreasing = TRUE), las = 2, col = "steelblue",
        main = "Type d'énergie de chauffage")
```
Le diagramme en barres trié est plus lisible : on compare les hauteurs plutôt que les angles.
</details>

## Exercice 9 – Reproduire un graphique avec `esquisse`

`esquisse` est une interface graphique pour `ggplot2` : on choisit la variable et le type de graphique à la souris, et le graphique s'affiche. Le code `ggplot2` correspondant peut ensuite être copié dans un script, ce qui le rend reproductible.

### Mémo

| Commande | Description |
|----------|-------------|
| `esquisser(dpe)` | Ouvre l'interface `esquisse` sur le dataframe `dpe`. |
| Choix de la variable `x` | Glisser la colonne sur l'axe horizontal. |
| Choix du graphique | Histogramme, boxplot, barres, nuage de points, etc. |
| Code | Le code `ggplot2` généré est récupérable depuis l'interface, pour le copier dans le script. |

### Exemple

```r
# Ouvre l'interface : on glisse surface_habitable_logement sur l'axe x,
# on choisit "Histogram" dans la liste des graphiques.
esquisser(dpe)
```

Une fois le graphique construit, on peut copier le code généré :

### Exercices

1. Ouvrir `esquisser(dpe)`, construire un histogramme de `surface_habitable_logement`, puis copier le code généré dans le script. 
<details>
<summary>Correction</summary>


2. Construire avec `esquisse` un diagramme en barres de `etiquette_dpe`. Copier le code généré, puis lui ajouter la palette `couleurs_dpe` avec `scale_fill_manual()`.
<details>
<summary>Correction</summary>

```r
# Code généré par esquisse, puis personnalisé :
ggplot(data = dpe) +
  aes(x = etiquette_dpe, fill = etiquette_dpe) +
  geom_bar() +
  scale_fill_manual(values = couleurs_dpe) +
  guides(fill = "none") +
  theme_minimal()
```
</details>


> **Note : `esquisse` et la reproductibilité**
> Un graphique construit à la souris ne se relance pas à l'identique si les données changent. Le code généré est la version reproductible : on le garde dans le script, et `esquisse` sert ensuite à explorer, pas à produire le graphique final.

## Exercice 10 – Graphiques interactifs avec `plotly`

`plotly` produit des graphiques interactifs : au survol, la valeur de la barre ou du point s'affiche, et on peut zoomer, déplacer la vue et télécharger l'image. Le graphique s'affiche dans le panneau *Viewer* de RStudio, ou dans le navigateur.

Deux façons de l'utiliser :
- **`plot_ly()`** : on décrit le graphique directement, avec un type (`"histogram"`, `"box"`, `"bar"`, `"pie"`). Les colonnes s'écrivent avec `~`, par exemple `~surface_habitable_logement`.
- **`ggplotly()`** : on convertit un graphique `ggplot2` déjà construit, en une ligne.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `plot_ly()` | Crée un graphique interactif. | `data`, `x`, `y`, `type` | `plot_ly(dpe, x = ~surface_habitable_logement, type = "histogram")` |
| `layout()` | Modifie le titre, les axes et les bornes d'affichage. | `title`, `xaxis`, `yaxis` | `layout(title = "Surfaces", xaxis = list(range = c(0, 200)))` |
| `ggplotly()` | Convertit un graphique `ggplot2` en graphique interactif. | `p` | `ggplotly(p)` |

### Exemple

```r
# Histogramme interactif : survoler une barre affiche son effectif
plot_ly(dpe, x = ~surface_habitable_logement, type = "histogram", nbinsx = 30) |>
  layout(title = "Surface habitable des logements de Villeurbanne",
         xaxis = list(title = "Surface (m²)"),
         yaxis = list(title = "Effectif"))

# Diagramme en barres des étiquettes, avec les couleurs DPE
effectifs_df <- as.data.frame(table(dpe$etiquette_dpe))

plot_ly(effectifs_df, x = ~Var1, y = ~Freq, type = "bar",
        color = ~Var1, colors = unname(couleurs_dpe)) |>
  layout(showlegend = FALSE,
         xaxis = list(title = "Étiquette"),
         yaxis = list(title = "Effectif"))

# Conversion d'un graphique ggplot2 existant
p <- ggplot(dpe, aes(x = surface_habitable_logement)) +
  geom_histogram(bins = 30, fill = "steelblue", colour = "white") +
  labs(x = "Surface (m²)", y = "Effectif")

ggplotly(p)
```

> **Note : `~` devant une colonne**
> Dans `plot_ly()`, `x = ~surface_habitable_logement` indique que `surface_habitable_logement` est une colonne de `dpe`. Sans `~`, R cherche un objet de ce nom dans l'environnement. C'est l'équivalent de `aes(x = ...)` dans `ggplot2`.

### Exercices

1. Tracer l'histogramme interactif de `surface_habitable_logement` avec `plot_ly()`, avec 30 classes. Survoler la classe la plus haute : quel est son effectif ?
<details>
<summary>Correction</summary>

```r
plot_ly(dpe, x = ~surface_habitable_logement, type = "histogram", nbinsx = 30)
# Le survol affiche le nombre de logements de la classe.
# La valeur exacte dépend du découpage en classes : on la lit au survol.
```
</details>

2. Tracer le diagramme en barres des effectifs de `etiquette_dpe` avec `plot_ly()`. Quelle étiquette a le plus grand effectif, et quel est-il ?
<details>
<summary>Correction</summary>

```r
effectifs_df <- as.data.frame(table(dpe$etiquette_dpe))

plot_ly(effectifs_df, x = ~Var1, y = ~Freq, type = "bar",
        color = ~Var1, colors = unname(couleurs_dpe)) |>
  layout(showlegend = FALSE)
# C : 4266 DPE (survol de la barre C).
```
`as.data.frame(table())` donne deux colonnes : `Var1` (les étiquettes) et `Freq` (les effectifs).
</details>

3. Tracer le boxplot interactif de `surface_habitable_logement` avec `plot_ly(type = "box")`. Lire la médiane au survol, puis retrouver la valeur avec `median()`.
<details>
<summary>Correction</summary>

```r
plot_ly(dpe, y = ~surface_habitable_logement, type = "box", name = "Surface") |>
  layout(yaxis = list(title = "Surface (m²)"))

median(dpe$surface_habitable_logement, na.rm = TRUE)  # 54
# Le survol du trait médian affiche 54.
```
Comme pour les graphiques de base, `plotly` ne trace pas les `NA`.
</details>

4. Convertir l'histogramme `ggplot2` de l'exemple avec `ggplotly()`, puis comparer le survol avec celui de la question 1. Que change la conversion ?
<details>
<summary>Correction</summary>

```r
ggplotly(p)
# Le graphique garde la forme de l'histogramme ggplot2, avec le survol en plus.
# La conversion reprend les couleurs et les titres, mais pas toujours tous les
# réglages de thème : on vérifie le rendu après conversion.
```
</details>

5. Zoomer sur les surfaces jusqu'à 200 m² avec `layout(xaxis = list(range = c(0, 200)))`. Pourquoi cette méthode ne retire-t-elle pas de données ?
<details>
<summary>Correction</summary>

```r
plot_ly(dpe, x = ~surface_habitable_logement, type = "histogram", nbinsx = 60) |>
  layout(xaxis = list(range = c(0, 200)))
```
`range` change seulement la fenêtre affichée : les données sont toujours dans le graphique, comme avec `coord_cartesian()` dans `ggplot2`. Pour zoomer, on évite donc `xlim()` ou un filtre.
</details>

6. Tracer le diagramme circulaire interactif des énergies regroupées de l'exercice 8, avec `plot_ly(type = "pie")`. Quelle part représentent les « Autres » ?
<details>
<summary>Correction</summary>

```r
energies_principales <- c("Électricité", "Gaz naturel", "Réseau de Chauffage urbain")
energie_regroupee <- ifelse(dpe$type_energie_n1 %in% energies_principales,
                            dpe$type_energie_n1, "Autres")
effectifs_energie <- table(energie_regroupee)

plot_ly(labels = names(effectifs_energie), values = as.numeric(effectifs_energie),
        type = "pie", textinfo = "label+percent")
# Autres : 1,02 % (102 DPE sur 10 000)
```
</details>

> **Note : interactif ne veut pas dire plus clair**
> Un graphique interactif permet de vérifier des valeurs, mais il ne remplace pas un graphique bien construit. Dans un rapport imprimé ou un document figé, on préfère le graphique `ggplot2` ; `plotly` sert à explorer les données, ou à partager une page HTML.

## Liens utiles

- [Documentation de `ggplot2`](https://ggplot2.tidyverse.org/)
- [Documentation de `plotly` pour R](https://plotly.com/r/)
- [Documentation de `esquisse`](https://dreamrs.github.io/esquisse/)
- [Documentation de `hist()` et `boxplot()` (R de base)](https://stat.ethz.ch/R-manual/R-patched/library/graphics/html/hist.html)
