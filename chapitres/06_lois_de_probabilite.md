# Chapitre 6 – Génération et représentation de variables aléatoires

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] définir une variable aléatoire, sa loi, sa densité et sa fonction de répartition
- [ ] générer un échantillon suivant une loi uniforme et une loi normale avec R
- [ ] calculer des probabilités, des quantiles et les paramètres théoriques d'une loi
- [ ] représenter un échantillon simulé et le comparer à la densité théorique
- [ ] observer l'effet de la taille de l'échantillon sur la moyenne et sur la forme de la distribution
- [ ] juger si une variable réelle, comme la surface des DPE, suit une loi normale

## Sommaire

- [Chapitre 6 – Génération et représentation de variables aléatoires](#chapitre-6--génération-et-représentation-de-variables-aléatoires)
  - [Objectifs](#objectifs)
  - [Sommaire](#sommaire)
  - [Préparation : packages, données et notations](#préparation--packages-données-et-notations)
    - [Notations](#notations)
  - [Exercice 1 – La loi uniforme](#exercice-1--la-loi-uniforme)
    - [Mémo](#mémo)
    - [Exemple](#exemple)
    - [Exercices](#exercices)
  - [Exercice 2 – La loi normale](#exercice-2--la-loi-normale)
    - [Mémo](#mémo-1)
    - [Exemple](#exemple-1)
    - [Exercices](#exercices-1)
  - [Exercice 3 – Effet de la taille de l'échantillon](#exercice-3--effet-de-la-taille-de-léchantillon)
    - [Exemple](#exemple-2)
    - [Exercices](#exercices-2)
  - [Exercice 4 – Les moyennes d'échantillons tendent vers la loi normale](#exercice-4--les-moyennes-déchantillons-tendent-vers-la-loi-normale)
    - [Exemple](#exemple-3)
    - [Exercices](#exercices-3)
  - [Exercice 5 – La surface des DPE suit-elle une loi normale ?](#exercice-5--la-surface-des-dpe-suit-elle-une-loi-normale-)
    - [Exemple](#exemple-4)
    - [Exercices](#exercices-4)
  - [Exercice 6 – Quantiles et graphique quantile-quantile](#exercice-6--quantiles-et-graphique-quantile-quantile)
    - [Mémo](#mémo-2)
    - [Exemple](#exemple-5)
    - [Exercices](#exercices-5)
  - [Exercice 7 – Histogramme et densité avec `plotly`](#exercice-7--histogramme-et-densité-avec-plotly)
    - [Exemple](#exemple-6)
    - [Exercices](#exercices-6)
  - [Liens utiles](#liens-utiles)

## Préparation : packages, données et notations

```r
library(readr)
library(dplyr)
library(ggplot2)
library(plotly)

dpe <- read_csv("data/dpe_villeurbanne_10000.csv")
surface <- dpe$surface_habitable_logement
```

Dans ce chapitre, les données sont **simulées** : elles servent à comprendre une loi. On fixe la graine avec `set.seed()`, pour qu'un même code donne toujours les mêmes nombres. Les valeurs affichées dans les corrections sont des ordres de grandeur : elles peuvent différer légèrement d'un poste à l'autre.

> **Note : les données réelles sont utilisées à l'exercice 5**
> Les simulations servent à comprendre une loi. L'exercice 5 confronte une loi théorique à une variable réelle, la surface des DPE.

### Notations

Une **variable aléatoire** $X$ prend des valeurs au hasard, selon une loi de probabilité. On note $P(X \le x)$ la probabilité que $X$ soit inférieure ou égale à $x$.

| Notion | Notation | Signification |
|--------|----------|---------------|
| Fonction de répartition | $F(x) = P(X \le x)$ | probabilité d'être sous $x$ |
| Densité (loi continue) | $f(x)$ | hauteur de la courbe ; $\int_a^b f(x)\,dx = P(a \le X \le b)$ |
| Quantile d'ordre $p$ | $x_p = F^{-1}(p)$ | valeur telle que $P(X \le x_p) = p$ |
| Espérance | $E(X)$ ou $\mu$ | valeur moyenne théorique |
| Variance | $\mathrm{Var}(X)$ ou $\sigma^2$ | dispersion théorique autour de $\mu$ |
| Loi de $X$ | $X \sim \mathcal{L}$ | $X$ suit la loi $\mathcal{L}$ |

Pour chaque loi, R fournit quatre fonctions, dont le préfixe indique l'usage :

| Préfixe | Rôle | Exemple normal | Exemple uniforme |
|---------|------|----------------|------------------|
| `d` | densité $f(x)$ | `dnorm(x, mean, sd)` | `dunif(x, min, max)` |
| `p` | probabilité $F(x)$ | `pnorm(x, mean, sd)` | `punif(x, min, max)` |
| `q` | quantile $F^{-1}(p)$ | `qnorm(p, mean, sd)` | `qunif(p, min, max)` |
| `r` | tirage aléatoire | `rnorm(n, mean, sd)` | `runif(n, min, max)` |

> **Note : `sd` et variance**
> Dans `rnorm()` et `dnorm()`, l'argument est l'écart type $\sigma$, pas la variance $\sigma^2$. Une loi $\mathcal{N}(\mu, \sigma^2)$ s'écrit donc `rnorm(n, mean = mu, sd = sigma)`.

## Exercice 1 – La loi uniforme

La loi uniforme $X \sim \mathcal{U}(a, b)$ donne la même densité à toutes les valeurs entre $a$ et $b$ :

$$
f(x) = \frac{1}{b-a} \quad \text{si } a \le x \le b, \qquad E(X) = \frac{a+b}{2}, \qquad \mathrm{Var}(X) = \frac{(b-a)^2}{12}
$$

et la fonction de répartition est $F(x) = \dfrac{x-a}{b-a}$ pour $a \le x \le b$.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `runif()` | Tire $n$ valeurs suivant $\mathcal{U}(\text{min}, \text{max})$. | `n`, `min`, `max` | `runif(1000, min = 30, max = 150)` |
| `punif()` | $P(X \le x)$. | `q`, `min`, `max` | `punif(50, min = 30, max = 150)` |
| `qunif()` | Quantile $x_p$. | `p`, `min`, `max` | `qunif(0.5, min = 30, max = 150)` |
| `set.seed()` | Fixe la graine aléatoire. | `seed` | `set.seed(42)` |

### Exemple

```r
set.seed(42)

# 1000 valeurs suivant U(30, 150)
x_unif <- runif(1000, min = 30, max = 150)

# Moyenne théorique (a + b) / 2 = 90 : la moyenne simulée est proche
mean(x_unif)

# Probabilité théorique P(X <= 50) = (50 - 30) / (150 - 30) = 1/6
punif(50, min = 30, max = 150)
# [1] 0.1666667

# Médiane théorique : quantile d'ordre 0,5
qunif(0.5, min = 30, max = 150)
# [1] 90

# Histogramme de l'échantillon et densité théorique
ggplot(data.frame(x = x_unif), aes(x = x)) +
  geom_histogram(aes(y = after_stat(density)), bins = 20,
                 fill = "steelblue", colour = "white") +
  stat_function(fun = dunif, args = list(min = 30, max = 150),
                colour = "red", linewidth = 1) +
  labs(x = "x", y = "Densité")
```

### Exercices

1. Tirer 10 000 valeurs suivant $\mathcal{U}(0, 1)$. Comparer leur moyenne et leur variance à $E(X) = 0{,}5$ et $\mathrm{Var}(X) = 1/12 \approx 0{,}0833$.
<details>
<summary>Correction</summary>

```r
set.seed(42)
x <- runif(10000, min = 0, max = 1)
mean(x)  # proche de 0.5
var(x)   # proche de 0.0833
```
Avec 10 000 valeurs, la moyenne et la variance simulées sont très proches des valeurs théoriques.
</details>

2. Calculer $P(X \le 0{,}25)$ pour $X \sim \mathcal{U}(0, 1)$, avec `punif()`. Vérifier avec la formule $F(x) = x$.
<details>
<summary>Correction</summary>

```r
punif(0.25, min = 0, max = 1)  # 0.25
# Avec F(x) = (x - a) / (b - a) = (0.25 - 0) / (1 - 0) = 0.25
```
</details>

3. Pour $X \sim \mathcal{U}(30, 150)$, calculer $P(60 \le X \le 120)$. Puis retrouver le quantile $x_{0{,}9}$.
<details>
<summary>Correction</summary>

```r
punif(120, min = 30, max = 150) - punif(60, min = 30, max = 150)
# (120 - 60) / 120 = 0.5

qunif(0.9, min = 30, max = 150)
# 30 + 0.9 * 120 = 138
```
</details>

4. Représenter l'histogramme de 5 000 valeurs de $\mathcal{U}(0, 1)$, avec la densité théorique en rouge. Quelle forme attendez-vous pour l'histogramme ?
<details>
<summary>Correction</summary>

```r
set.seed(42)
x <- runif(5000)

ggplot(data.frame(x = x), aes(x = x)) +
  geom_histogram(aes(y = after_stat(density)), bins = 20, fill = "steelblue", colour = "white") +
  stat_function(fun = dunif, args = list(min = 0, max = 1), colour = "red", linewidth = 1)
# L'histogramme doit être à peu près plat, à la hauteur 1 : la densité est constante.
```
</details>

> **Note : pourquoi `after_stat(density)`**
> Pour superposer un histogramme à une densité, l'axe vertical doit être une densité et non un effectif. `after_stat(density)` demande à `ggplot2` de convertir les effectifs en densité, dont l'aire totale vaut 1.

## Exercice 2 – La loi normale

La loi normale $X \sim \mathcal{N}(\mu, \sigma^2)$ a une densité en forme de cloche, symétrique autour de $\mu$ :

$$
f(x) = \frac{1}{\sigma\sqrt{2\pi}} \exp\left(-\frac{(x-\mu)^2}{2\sigma^2}\right), \qquad E(X) = \mu, \qquad \mathrm{Var}(X) = \sigma^2
$$

Quelques probabilités à connaître, pour une loi normale quelconque :

$$
P(\mu - \sigma \le X \le \mu + \sigma) \approx 0{,}683, \qquad P(\mu - 2\sigma \le X \le \mu + 2\sigma) \approx 0{,}954, \qquad P(X \le \mu + 1{,}96\,\sigma) \approx 0{,}975
$$

Pour la loi normale centrée réduite $\mathcal{N}(0, 1)$, la densité est notée $\varphi$ et la fonction de répartition $\Phi$.

![Loi normale](https://thumb.wikimedia.org/wikipedia/commons/thumb/7/74/Normal_Distribution_PDF.svg/langfr-330px-Normal_Distribution_PDF.svg.png)

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `rnorm()` | Tire $n$ valeurs suivant $\mathcal{N}(\mu, \sigma^2)$. | `n`, `mean`, `sd` | `rnorm(1000, mean = 55, sd = 20)` |
| `dnorm()` | Densité $f(x)$. | `x`, `mean`, `sd` | `dnorm(0, mean = 0, sd = 1)` |
| `pnorm()` | $P(X \le x) = F(x)$. | `q`, `mean`, `sd` | `pnorm(1.96)` |
| `qnorm()` | Quantile $x_p$. | `p`, `mean`, `sd` | `qnorm(0.975)` |

### Exemple

```r
# Probabilités de la loi normale centrée réduite
pnorm(1) - pnorm(-1)    # 0.6827
pnorm(2) - pnorm(-2)    # 0.9545
qnorm(0.975)            # 1.96

# Une loi N(mu = 50, sigma = 10) : P(X <= 60) = Phi((60 - 50) / 10)
pnorm(60, mean = 50, sd = 10)  # 0.8413
```

### Exercices

1. Vérifier avec `pnorm()` que $P(-1{,}96 \le X \le 1{,}96) \approx 0{,}95$ pour $X \sim \mathcal{N}(0, 1)$.
<details>
<summary>Correction</summary>

```r
pnorm(1.96) - pnorm(-1.96)  # 0.9500
```
</details>

2. Pour $X \sim \mathcal{N}(100, 15^2)$, calculer $P(X > 130)$ avec `pnorm()`. Vérifier avec la formule $P(X > x) = 1 - F(x)$.
<details>
<summary>Correction</summary>

```r
1 - pnorm(130, mean = 100, sd = 15)   # 0.0228
pnorm(130, mean = 100, sd = 15, lower.tail = FALSE)  # même résultat
```
L'argument `lower.tail = FALSE` donne directement $P(X > x)$.
</details>

3. Calculer le quantile $x_{0{,}05}$ de $X \sim \mathcal{N}(100, 15^2)$, et le quantile $x_{0{,}95}$. Pourquoi sont-ils symétriques autour de 100 ?
<details>
<summary>Correction</summary>

```r
qnorm(0.05, mean = 100, sd = 15)  # 75.3
qnorm(0.95, mean = 100, sd = 15)  # 124.7
```
La loi est symétrique autour de $\mu = 100$ : les deux quantiles sont à la même distance, $1{,}645\,\sigma \approx 24{,}7$, de part et d'autre de 100.
</details>

4. Tirer 2 000 valeurs de $\mathcal{N}(50, 10^2)$, puis vérifier que la moyenne et l'écart type sont proches de 50 et 10.
<details>
<summary>Correction</summary>

```r
set.seed(42)
x <- rnorm(2000, mean = 50, sd = 10)
mean(x)  # proche de 50
sd(x)    # proche de 10
```
</details>

## Exercice 3 – Effet de la taille de l'échantillon

La moyenne d'un échantillon est elle-même une variable aléatoire. Si $X_1, \dots, X_n$ sont indépendantes de même loi, de moyenne $\mu$ et de variance $\sigma^2$, alors :

$$
\bar{X}_n = \frac{1}{n}\sum_{i=1}^{n} X_i, \qquad E(\bar{X}_n) = \mu, \qquad \mathrm{Var}(\bar{X}_n) = \frac{\sigma^2}{n}
$$

L'écart type de la moyenne, $\sigma/\sqrt{n}$, diminue quand $n$ augmente. C'est pourquoi une grande taille d'échantillon donne une estimation plus précise.

### Exemple

```r
set.seed(42)

# Un échantillon de taille 10, puis de taille 1000 : même loi N(50, 10^2)
petit <- rnorm(10, mean = 50, sd = 10)
grand <- rnorm(1000, mean = 50, sd = 10)

ggplot(data.frame(x = grand), aes(x = x)) +
  geom_histogram(aes(y = after_stat(density)), bins = 30, fill = "steelblue", colour = "white") +
  stat_function(fun = dnorm, args = list(mean = 50, sd = 10), colour = "red", linewidth = 1)
```

### Exercices

1. Tirer 1000 moyennes d'échantillons de taille $n = 10$, puis 1000 moyennes de taille $n = 100$, toutes issues de $\mathcal{N}(50, 10^2)$. Comparer l'écart type de chaque série à $\sigma/\sqrt{n}$.
<details>
<summary>Correction</summary>

```r
set.seed(42)
moy_10  <- replicate(1000, mean(rnorm(10,  mean = 50, sd = 10)))
moy_100 <- replicate(1000, mean(rnorm(100, mean = 50, sd = 10)))

sd(moy_10)   # proche de 10 / sqrt(10)  = 3.16
sd(moy_100)  # proche de 10 / sqrt(100) = 1.00
```
`replicate()` répète l'expression plusieurs fois. Quand $n$ est multiplié par 10, l'écart type de la moyenne est divisé par $\sqrt{10} \approx 3{,}16$.
</details>

2. Calculer la moyenne de chaque série de moyennes. Pourquoi est-elle proche de 50 dans les deux cas ?
<details>
<summary>Correction</summary>

```r
mean(moy_10)   # proche de 50
mean(moy_100)  # proche de 50
```
$E(\bar{X}_n) = \mu$ quelle que soit la taille de l'échantillon : seule la dispersion change.
</details>

3. Représenter l'histogramme des deux séries de moyennes sur le même graphique, avec `geom_density()` ou un histogramme. Quelle série est la plus concentrée ?
<details>
<summary>Correction</summary>

```r
df <- data.frame(
  moyenne = c(moy_10, moy_100),
  taille  = rep(c("n = 10", "n = 100"), each = 1000)
)

ggplot(df, aes(x = moyenne, fill = taille)) +
  geom_density(alpha = 0.4)
# La série n = 100 est plus concentrée autour de 50.
```
</details>

> **Note : la moyenne et l'écart type de la moyenne**
> L'écart type de $\bar{X}_n$, soit $\sigma/\sqrt{n}$, s'appelle aussi l'**erreur standard**. C'est elle qui sert à calculer les intervalles de confiance, au chapitre 7.

## Exercice 4 – Les moyennes d'échantillons tendent vers la loi normale

Le **théorème central limite** (TCL) dit que, pour un $n$ assez grand, la moyenne de $n$ variables indépendantes suit approximativement une loi normale, **quelle que soit la loi de départ** :

$$
\frac{\bar{X}_n - \mu}{\sigma / \sqrt{n}} \ \xrightarrow{\ \text{approx.}\ }\ \mathcal{N}(0, 1)
$$

Pour une loi uniforme $\mathcal{U}(0, 1)$, on a $\mu = 0{,}5$ et $\sigma^2 = 1/12$.

### Exemple

```r
set.seed(42)

# 5000 moyennes de 30 valeurs uniformes
moy_unif <- replicate(5000, mean(runif(30)))

ggplot(data.frame(m = moy_unif), aes(x = m)) +
  geom_histogram(aes(y = after_stat(density)), bins = 40, fill = "steelblue", colour = "white") +
  stat_function(fun = dnorm, args = list(mean = 0.5, sd = sqrt(1 / 12 / 30)),
                colour = "red", linewidth = 1)
```

### Exercices

1. Calculer l'écart type théorique de la moyenne de 30 valeurs $\mathcal{U}(0,1)$ : $\sqrt{1/(12 \times 30)}$. Comparer avec `sd(moy_unif)`.
<details>
<summary>Correction</summary>

```r
sqrt(1 / 12 / 30)  # 0.0527
sd(moy_unif)       # proche de 0.0527
```
</details>

2. Recommencer avec des moyennes de 2 valeurs seulement, puis de 30. Représenter les deux histogrammes côte à côte. La forme normale apparaît-elle à 2 valeurs ?
<details>
<summary>Correction</summary>

```r
set.seed(42)
moy_2  <- replicate(5000, mean(runif(2)))
moy_30 <- replicate(5000, mean(runif(30)))

df <- data.frame(
  moyenne = c(moy_2, moy_30),
  taille  = rep(c("n = 2", "n = 30"), each = 5000)
)

ggplot(df, aes(x = moyenne)) +
  geom_histogram(bins = 40, fill = "steelblue", colour = "white") +
  facet_wrap(~ taille, scales = "free")
# À n = 2, la distribution est triangulaire ; à n = 30, elle est presque normale.
```
</details>

> On on n'a pas besoin de connaître la loi de la population pour utiliser une loi normale sur la moyenne, à condition que $n$ soit assez grand.

## Exercice 5 – La surface des DPE suit-elle une loi normale ?

On compare maintenant une variable réelle à une loi normale. On ajuste une loi $\mathcal{N}(\bar{x}, s^2)$ à la surface, avec $\bar{x}$ la moyenne et $s$ l'écart type observés, puis on regarde si la loi ajustée décrit bien les données.

### Exemple

```r
moyenne <- mean(surface, na.rm = TRUE)   # 55.93
ecart   <- sd(surface, na.rm = TRUE)     # 60.27

# Probabilité qu'une surface soit négative, selon la loi normale ajustée
pnorm(0, mean = moyenne, sd = ecart)     # 0.177

ggplot(dpe, aes(x = surface_habitable_logement)) +
  geom_histogram(aes(y = after_stat(density)), bins = 40, fill = "steelblue", colour = "white") +
  stat_function(fun = dnorm, args = list(mean = moyenne, sd = ecart),
                colour = "red", linewidth = 1) +
  coord_cartesian(xlim = c(0, 200)) +
  labs(x = "Surface (m²)", y = "Densité")
```

### Exercices

1. Calculer la probabilité que la surface soit négative selon la loi normale ajustée. Que pensez-vous de ce résultat ?
<details>
<summary>Correction</summary>

```r
pnorm(0, mean = moyenne, sd = ecart)  # 0.177
```
Une loi normale ajustée donne 17,7 % de surfaces négatives. Une surface ne peut pas être négative : la loi normale n'est pas un bon modèle pour cette variable.
</details>

2. Calculer la part des surfaces dans l'intervalle $[\bar{x} - s ; \bar{x} + s]$, puis la comparer à la valeur théorique 0,683. Que constate-t-on ?
<details>
<summary>Correction</summary>

```r
mean(abs(surface - moyenne) <= ecart, na.rm = TRUE)  # 0.979
```
Environ 98 % des surfaces sont dans cet intervalle, contre 68 % pour une loi normale : la distribution n'est pas symétrique et les valeurs extrêmes gonflent l'écart type. La loi normale ne s'ajuste donc pas bien à la surface brute.
</details>

3. Refaire l'histogramme avec le logarithme de la surface, `log(surface_habitable_logement)`, et ajuster une loi normale à cette variable. Comparer les pourcentages dans $[\mu \pm s]$ et $[\mu \pm 2s]$ avec 68,3 % et 95,4 %.
<details>
<summary>Correction</summary>

```r
log_surface <- log(surface)
mu_log <- mean(log_surface, na.rm = TRUE)   # 3.873
s_log  <- sd(log_surface, na.rm = TRUE)     # 0.553

mean(abs(log_surface - mu_log) <= s_log, na.rm = TRUE)      # 0.705
mean(abs(log_surface - mu_log) <= 2 * s_log, na.rm = TRUE)  # 0.967

ggplot(data.frame(x = log_surface), aes(x = x)) +
  geom_histogram(aes(y = after_stat(density)), bins = 40, fill = "steelblue", colour = "white") +
  stat_function(fun = dnorm, args = list(mean = mu_log, sd = s_log), colour = "red", linewidth = 1)
```
Sur le logarithme, les pourcentages sont proches de 68,3 % et 95,4 % : la loi log-normale, c'est-à-dire la loi dont le logarithme est normal, décrit mieux la surface. La transformation de Box-Cox et les tests d'ajustement sont au chapitre 8.
</details>

> **Note : `exp(mu_log)` et la médiane**
> `exp(mu_log)` vaut environ 48 m² : c'est la médiane de la loi log-normale ajustée. Elle est proche de la médiane observée de 54 m², ce qui confirme que la loi s'ajuste bien au centre de la distribution.

## Exercice 6 – Quantiles et graphique quantile-quantile

Un **graphique quantile-quantile** (QQ-plot) compare les quantiles observés aux quantiles d'une loi théorique. Si les points sont alignés sur la droite, la loi théorique convient.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `qqnorm()` | Trace un QQ-plot contre la loi normale. | `y` | `qqnorm(x)` |
| `qqline()` | Ajoute la droite de référence. | `y` | `qqline(x)` |
| `stat_qq()`, `stat_qq_line()` | Équivalents `ggplot2`. | | `ggplot(df, aes(sample = x)) + stat_qq() + stat_qq_line()` |

### Exemple

```r
# QQ-plot de la surface brute, puis de son logarithme
qqnorm(surface, main = "Surface brute")
qqline(surface, col = "red")
qqnorm(log(surface), main = "Logarithme de la surface")
qqline(log(surface), col = "red")
```

### Exercices

1. Tracer le QQ-plot de la surface brute, puis celui du logarithme. Lequel est le plus aligné sur la droite ?
<details>
<summary>Correction</summary>

```r
par(mfrow = c(1, 2))
qqnorm(surface, main = "Surface brute"); qqline(surface, col = "red")
qqnorm(log(surface), main = "Logarithme"); qqline(log(surface), col = "red")
par(mfrow = c(1, 1))
# La surface brute s'écarte fortement de la droite aux extrémités ;
# le logarithme s'en rapproche nettement.
```
</details>

2. Simuler 500 valeurs de $\mathcal{N}(0, 1)$, puis tracer leur QQ-plot. Que doit-on voir ?
<details>
<summary>Correction</summary>

```r
set.seed(42)
x <- rnorm(500)
qqnorm(x); qqline(x, col = "red")
# Les points sont alignés sur la droite, sauf peut-être quelques points aux extrémités.
```
Un QQ-plot sur des données réellement normales est aligné : c'est la référence pour lire les autres.
</details>

3. Tracer le QQ-plot d'une loi uniforme $\mathcal{U}(0, 1)$ de 500 valeurs. Pourquoi la courbe n'est-elle pas une droite ?
<details>
<summary>Correction</summary>

```r
set.seed(42)
u <- runif(500)
qqnorm(u)
qqline(u, col = "red")
# En forme de S : la loi uniforme a des queues plus courtes que la loi normale.
```
</details>

## Exercice 7 – Histogramme et densité avec `plotly`

`plotly` permet de superposer un histogramme et une courbe de densité interactive. On utilise `add_trace()` pour ajouter la densité théorique.

### Exemple

```r
set.seed(42)
x <- rnorm(2000, mean = 50, sd = 10)

# Points de la courbe théorique, entre 10 et 90
xs <- seq(10, 90, by = 0.5)

plot_ly(x = ~x, type = "histogram", histnorm = "probability density",
        name = "Échantillon") |>
  add_trace(x = xs, y = dnorm(xs, mean = 50, sd = 10),
            type = "scatter", mode = "lines", name = "N(50, 10²)")
```

### Exercices

1. Reproduire le graphique avec `plotly`, puis survoler la courbe : quelle densité vaut-elle en $x = 50$ ?
<details>
<summary>Correction</summary>

```r
dnorm(50, mean = 50, sd = 10)  # 0.0399 : la densité au centre de la loi N(50, 10²)
```
La densité est le pic de la cloche : $\frac{1}{10\sqrt{2\pi}} \approx 0{,}0399$.
</details>

2. Tracer la densité de la surface logarithmique des DPE (`log(surface)`) avec sa loi normale ajustée, en interactif.
<details>
<summary>Correction</summary>

```r
log_surface <- log(surface[!is.na(surface)])
mu_log <- mean(log_surface)
s_log  <- sd(log_surface)

xs <- seq(min(log_surface), max(log_surface), length.out = 300)

plot_ly(x = ~log_surface, type = "histogram", histnorm = "probability density",
        name = "log(surface)") |>
  add_trace(x = xs, y = dnorm(xs, mean = mu_log, sd = s_log),
            type = "scatter", mode = "lines", name = "Loi normale ajustée")
```
</details>

> Sans `histnorm`, l'histogramme affiche des effectifs, alors que la courbe $f(x)$ est une densité. Les deux échelles sont différentes : il faut que l'aire de l'histogramme soit égale à 1, ce que fait `histnorm = "probability density"`.

## Liens utiles

- [Documentation de `rnorm()` et des fonctions de loi (R)](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/Normal.html)
- [Documentation de `runif()` (R)](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/Uniform.html)
- [Documentation de `ggplot2`](https://ggplot2.tidyverse.org/)
- [Documentation de `plotly` pour R](https://plotly.com/r/)
