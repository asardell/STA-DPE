# Chapitre 7 – Intervalles de confiance

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] définir un intervalle de confiance et le niveau de confiance $1 - \alpha$
- [ ] relier le risque $\alpha$ à la largeur et à la fiabilité de l'intervalle
- [ ] calculer un intervalle de confiance de la moyenne avec la loi normale (test Z) et avec la loi de Student (test t)
- [ ] calculer un intervalle de confiance d'une proportion
- [ ] observer l'effet de la taille de l'échantillon sur la largeur de l'intervalle
- [ ] vérifier par simulation qu'un intervalle à 95 % contient la vraie valeur environ 95 % du temps
- [ ] lire un intervalle de confiance sur un graphique

> **Contexte :** dans un rapport ou une présentation à l'ADEME, on ne donne jamais une moyenne seule : un chiffre sans fourchette paraît plus certain qu'il ne l'est. L'intervalle de confiance montre la précision de l'estimation, et permet de dire si un écart entre deux valeurs est plausible ou non.

## Sommaire

- [Chapitre 7 – Intervalles de confiance](#chapitre-7--intervalles-de-confiance)
  - [Objectifs](#objectifs)
  - [Sommaire](#sommaire)
  - [Préparation : packages, données et notations](#préparation--packages-données-et-notations)
    - [Notations](#notations)
  - [Exercice 1 – Intervalle de confiance de la moyenne (test Z)](#exercice-1--intervalle-de-confiance-de-la-moyenne-test-z)
    - [Mémo](#mémo)
    - [Exemple](#exemple)
    - [Exercices](#exercices)
  - [Exercice 2 – Niveau de confiance et risque α](#exercice-2--niveau-de-confiance-et-risque-α)
    - [Exemple](#exemple-1)
    - [Exercices](#exercices-1)
  - [Exercice 3 – Effet de la taille de l'échantillon](#exercice-3--effet-de-la-taille-de-léchantillon)
    - [Exemple](#exemple-2)
    - [Exercices](#exercices-2)
  - [Exercice 4 – Petit échantillon : loi de Student (test t)](#exercice-4--petit-échantillon--loi-de-student-test-t)
    - [Mémo](#mémo-1)
    - [Exemple](#exemple-3)
    - [Exercices](#exercices-3)
  - [Exercice 5 – Vérifier le niveau de confiance par simulation](#exercice-5--vérifier-le-niveau-de-confiance-par-simulation)
    - [Exemple](#exemple-4)
    - [Exercices](#exercices-4)
  - [Exercice 6 – Intervalle de confiance d'une proportion](#exercice-6--intervalle-de-confiance-dune-proportion)
    - [Exemple](#exemple-5)
    - [Mémo](#mémo-2)
    - [Exercices](#exercices-5)
  - [Exercice 7 – Représenter des intervalles avec `ggplot2` et `plotly`](#exercice-7--représenter-des-intervalles-avec-ggplot2-et-plotly)
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
surface_ok <- surface[!is.na(surface)]   # 9 899 surfaces renseignées
```

### Notations

Soit un échantillon de taille $n$, de moyenne $\bar{x}$ et d'écart type $s$. On cherche un intervalle qui contient la vraie moyenne $\mu$ de la population avec une probabilité donnée.

| Notion | Notation | Signification |
|--------|----------|---------------|
| Taille de l'échantillon | $n$ | nombre d'observations |
| Moyenne observée | $\bar{x}$ | estimation de $\mu$ |
| Écart type observé | $s$ | estimation de $\sigma$ |
| Niveau de confiance | $1 - \alpha$ | probabilité que l'intervalle contienne la vraie valeur |
| Risque | $\alpha$ | probabilité que l'intervalle **ne** contienne **pas** la vraie valeur |
| Quantile normal | $z_{1-\alpha/2}$ | $\approx 1{,}96$ pour $\alpha = 0{,}05$ |
| Quantile de Student | $t_{n-1;\,1-\alpha/2}$ | quantile de la loi de Student à $n - 1$ degrés de liberté |

> **Note : le risque α pour un intervalle de confiance**
> Pour un intervalle de confiance, $\alpha$ n'est pas le risque de rejeter une hypothèse : c'est la probabilité que l'intervalle **rate** la vraie valeur. Avec $\alpha = 0{,}05$, un intervalle à 95 % rate la vraie moyenne 5 fois sur 100 si on répète l'échantillonnage.

> **Note : le lien avec les tests**
> Un test bilatéral au seuil $\alpha$ rejette $H_0 : \mu = \mu_0$ si et seulement si $\mu_0$ est en dehors de l'intervalle à $1 - \alpha$. Les intervalles de confiance et les tests sont donc deux lectures du même calcul. Les tests sont traités en détail à partir du chapitre 8.

## Exercice 1 – Intervalle de confiance de la moyenne (test Z)

Quand la taille $n$ est grande, la moyenne suit approximativement une loi normale (théorème central limite, chapitre 6). L'intervalle de confiance de la moyenne au niveau $1 - \alpha$ est :

$$
\bar{x} \pm z_{1-\alpha/2} \cdot \frac{s}{\sqrt{n}}
$$

Le terme $s/\sqrt{n}$ est l'**erreur standard** de la moyenne. Avec un écart type connu $\sigma$, on remplacerait $s$ par $\sigma$ : c'est le cadre du test Z.

> **Contexte :** le chiffre le plus demandé est souvent une moyenne, comme la surface ou la consommation. L'intervalle de confiance dit avec quelle précision on la connaît à partir de l'échantillon, ce qui conditionne la confiance qu'on peut accorder au chiffre présenté.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `qnorm()` | Quantile de la loi normale $z_{1-\alpha/2}$. | `p` | `qnorm(0.975)` |
| `sqrt()` | Racine carrée, pour $\sqrt{n}$. | | `sqrt(length(x))` |
| `mean()`, `sd()` | Moyenne et écart type observés. | `na.rm` | `mean(x, na.rm = TRUE)` |

### Exemple

```r
x <- surface_ok
n <- length(x)
xbar <- mean(x)
s <- sd(x)

alpha <- 0.05
z <- qnorm(1 - alpha / 2)     # 1.96

marge <- z * s / sqrt(n)
c(inf = xbar - marge, sup = xbar + marge)
#   inf    sup
# 54.74  57.12
```

L'intervalle à 95 % pour la surface moyenne des logements de Villeurbanne est donc $[54{,}74 ;\ 57{,}12]$ m².

### Exercices

1. Calculer l'intervalle de confiance à 95 % de la surface moyenne. Peut-on dire que la surface moyenne est exactement 55,93 m² ?
   > **Intérêt :** un chiffre seul, comme 55,9 m², peut être pris pour exact : l'intervalle rappelle l'incertitude à transmettre au lecteur.
<details>
<summary>Correction</summary>

```r
# Intervalle : [54,74 ; 57,12]
# Non : 55,93 m² est l'estimation ponctuelle. L'intervalle dit que la vraie moyenne de la population est probablement dans cette fourchette, à 95 % de confiance.
```
</details>

2. Calculer l'intervalle à 95 % avec `t.test()`, et comparer le résultat à celui de la formule.
   > **Intérêt :** savoir qu'un résultat est fiable à 95 % évite des conclusions trop fortes lors d'un comité de pilotage.
<details>
<summary>Correction</summary>

```r
t.test(x)$conf.int
# [1] 54.74 57.12
# attr(,"conf.level")
# [1] 0.95
```
Sur 9 899 observations, la loi de Student et la loi normale donnent presque le même quantile (1,96) : les deux intervalles coïncident.
</details>

> **Note : `t.test()` et `$conf.int`**
> `t.test()` calcule l'intervalle de confiance de la moyenne au niveau 95 % par défaut, et le renvoie dans `$conf.int`. Son argument `conf.level` change le niveau : `t.test(x, conf.level = 0.99)`.

## Exercice 2 – Niveau de confiance et risque α

Plus le niveau de confiance est élevé, plus l'intervalle est large : on augmente la probabilité de contenir la vraie valeur en acceptant un intervalle moins précis. Le compromis est donc entre **fiabilité** (niveau élevé) et **précision** (intervalle étroit).

> **Contexte :** le niveau de confiance est un choix de l'équipe. 95 % est une convention, mais un enjeu financier important (travaux, aides) peut justifier 99 %, au prix d'un intervalle plus large. Ce choix se documente dans la méthode.

### Exemple

```r
for (niveau in c(0.90, 0.95, 0.99)) {
  alpha <- 1 - niveau
  z <- qnorm(1 - alpha / 2)
  marge <- z * sd(x) / sqrt(n)
  cat("Niveau", niveau, " z =", round(z, 3), " marge =", round(marge, 2), "\n")
}
# Niveau 0.9   z = 1.645  marge = 1
# Niveau 0.95  z = 1.96   marge = 1.19
# Niveau 0.99  z = 2.576  marge = 1.56
```

### Exercices

1. Calculer l'intervalle à 90 % et à 99 %. Quelle est la largeur de chaque intervalle ?
   > **Intérêt :** montrer le compromis permet au commanditaire de choisir le niveau de confiance en connaissance de cause.
<details>
<summary>Correction</summary>

```r
# 90 % : [54.93 ; 56.93], largeur 1.99
# 99 % : [54.37 ; 57.49], largeur 3.12
```
La largeur passe de 1,99 m² (90 %) à 3,12 m² (99 %) : gagner en fiabilité coûte en précision.
</details>

2. Quelle valeur de $\alpha$ correspond à un niveau de confiance de 99 % ? Et quel quantile normal utiliser ?
   > **Intérêt :** le lien entre α et le quantile normal est la base de tous les calculs suivants du cours.
<details>
<summary>Correction</summary>

$\alpha = 1 - 0{,}99 = 0{,}01$, donc le quantile est $z_{1-0{,}005} = z_{0{,}995} \approx 2{,}576$.

```r
qnorm(1 - 0.01 / 2)  # 2.576
```
</details>

3. Un collègue dit : « mon intervalle à 95 % contient la moyenne, donc il y a 95 % de chances que la moyenne soit dans cet intervalle. » Pourquoi cette phrase est-elle imprécise ?
   > **Intérêt :** cette formulation est une erreur courante dans les présentations : la corriger évite de surestimer la certitude devant un décideur.
<details>
<summary>Correction</summary>

La moyenne $\mu$ est une valeur fixe : elle est dedans ou dehors. C'est l'**intervalle** qui est aléatoire, puisqu'il dépend de l'échantillon. La bonne lecture : si on répétait l'échantillonnage un grand nombre de fois, 95 % des intervalles construits de cette façon contiendraient $\mu$.
</details>

> **Note : la phrase de lecture**
> Dans un rapport, on écrit : « La surface moyenne est estimée à 55,9 m², avec un intervalle de confiance à 95 % de [54,7 ; 57,1] m². » On ne dit pas « il y a 95 % de chances que la moyenne soit dans l'intervalle ».

## Exercice 3 – Effet de la taille de l'échantillon

La marge d'erreur $z \cdot s/\sqrt{n}$ diminue avec $\sqrt{n}$ : pour diviser la marge par 2, il faut multiplier la taille de l'échantillon par 4.

> **Contexte :** avant une enquête, la question est « combien de DPE faut-il pour une précision donnée ? ». La marge varie en $1/\sqrt{n}$ : diviser l'incertitude par deux coûte quatre fois plus de dossiers. C'est un argument de dimensionnement budgétaire.

### Exemple

```r
s <- sd(surface_ok)
for (taille in c(100, 1000, nrow(dpe))) {
  cat("n =", taille, " marge =", round(1.96 * s / sqrt(taille), 2), "\n")
}
# n = 100    marge = 11.81
# n = 1000   marge = 3.74
# n = 9899   marge = 1.19   (surfaces renseignées)
```

### Exercices

1. Calculer la marge d'erreur à 95 % pour un échantillon de 400 DPE tirés au hasard. Comparer avec la marge obtenue pour 9 899 DPE.
   > **Intérêt :** une marge de 1,19 m² pour 9 899 DPE contre 11,81 pour 100 est l'argument chiffré pour justifier la taille d'un échantillon.
<details>
<summary>Correction</summary>

```r
set.seed(42)
echantillon_400 <- sample(surface_ok, 400)
marge_400 <- 1.96 * sd(echantillon_400) / sqrt(400)
marge_400
# Ordre de grandeur : environ 5,9 m² (avec un écart type proche de 60) ; la marge pour 9 899 DPE est de 1,19 m².
```
Le chiffre exact dépend du tirage et de son écart type. Le calcul de référence est $1{,}96 \cdot 60{,}27/\sqrt{400} \approx 5{,}9$ m². Le point clé : la marge est divisée par 5 quand $n$ est multiplié par 25.
</details>

> **Note : vérifier le calcul à la main**
> Le tirage aléatoire donne un résultat différent d'un poste à l'autre si on ne fixe pas la graine. Avec `set.seed(42)`, le résultat est reproductible sur votre poste, mais pas forcément identique à celui du formateur.

2. Pour diviser la marge d'erreur par 10, de combien faut-il multiplier la taille de l'échantillon ?
   > **Intérêt :** la règle de la racine carrée permet d'estimer rapidement le coût d'une amélioration de précision, avant de lancer une campagne.
<details>
<summary>Correction</summary>

Il faut multiplier $n$ par $10^2 = 100$ : la marge varie en $1/\sqrt{n}$. Concrètement, passer de 99 à 9 900 DPE divise la marge par 10 : $\sqrt{100} = 10$.
</details>

## Exercice 4 – Petit échantillon : loi de Student (test t)

Quand $n$ est petit et que $\sigma$ est inconnu, la loi normale sous-estime l'incertitude. On utilise la **loi de Student** à $n - 1$ degrés de liberté.
$$
\bar{x} \pm t_{n-1;\,1-\alpha/2} \cdot \frac{s}{\sqrt{n}}
$$

Plus $n$ est grand, plus $t_{n-1}$ se rapproche de $z = 1{,}96$.


Cela signifie que la loi de Student attribue davantage de probabilité aux valeurs extrêmes que la loi normale.

Autrement dit :
- Avec la loi normale, les valeurs très éloignées de la moyenne sont considérées comme très improbables.
- Avec la loi de Student, ces valeurs extrêmes sont considérées comme un peu plus probables.

![Image](https://toussile.quarto.pub/data-science/posts/tests-statistiques/03-t-test_files/figure-html/normal-vs-student-output-1.png)

> Quand on a un petit échantillon (par exemple 5 ou 10 observations) et qu'on ne connaît pas l'écart-type réel de la population ($\sigma$), on a davantage d'incertitude.
> Conséquence sur l'intervalle de confiance : L'intervalle de confiance devient généralement plus large qu'avec la loi normale.

> **Contexte :** un petit échantillon (une commune, un audit ponctuel, une campagne de contrôle) est fréquent. Utiliser la loi normale dans ce cas donne un intervalle trop étroit : la loi de Student est la bonne référence, et un rapport doit le justifier.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `qt()` | Quantile de la loi de Student. | `p`, `df` (degrés de liberté) | `qt(0.975, df = 29)` |
| `t.test()` | Intervalle de confiance de la moyenne. | `x`, `conf.level` | `t.test(x, conf.level = 0.95)$conf.int` |

### Exemple

```r
# Un petit échantillon : les 30 premières surfaces renseignées du fichier
petit <- surface_ok[1:30]
n_petit <- length(petit)

qt(0.975, df = n_petit - 1)   # 2.045 (contre 1.96 pour la loi normale)
qnorm(0.975)                  # 1.960

t.test(petit)$conf.int
# [1] 45.12 61.72
```

### Exercices

1. Calculer manuellement l'intervalle de confiance à 95 % du petit échantillon avec `qt()`, et comparer à `t.test()`.
   > **Intérêt :** la contre-vérification à la main est une habitude à présenter avec le résultat : elle rassure sur un chiffre qui sera repris.
<details>
<summary>Correction</summary>

```r
xbar <- mean(petit)     # 53.42
s    <- sd(petit)       # 22.22
t_q  <- qt(0.975, df = 29)  # 2.045
marge <- t_q * s / sqrt(30)
c(inf = xbar - marge, sup = xbar + marge)
# inf 45.12 ; sup 61.72 : identique à t.test()
```
</details>

2. Calculer le même intervalle en remplaçant $t$ par $z = 1{,}96$. De combien l'intervalle est-il sous-estimé ?
   > **Intérêt :** utiliser 1,96 avec 30 observations donne une fausse impression de précision : c'est une erreur à éviter dans un rapport.
<details>
<summary>Correction</summary>

```r
marge_z <- qnorm(0.975) * s / sqrt(30)
c(inf = mean(petit) - marge_z, sup = mean(petit) + marge_z)
# inf 45.47 ; sup 61.37
```
L'intervalle avec $z$ est plus étroit de quelques dixièmes de m² : la loi normale donne une fausse impression de précision quand $n$ est petit.
</details>

3. Comparer les quantiles $t_{n-1}$ pour $n = 5$, $n = 30$ et $n = 1000$. À partir de quelle taille la différence avec 1,96 devient-elle négligeable ?
   > **Intérêt :** savoir à partir de quelle taille la loi normale convient permet de décider quel quantile utiliser, sans débat.
<details>
<summary>Correction</summary>

```r
qt(0.975, df = 4)     # 2.776 (n = 5)
qt(0.975, df = 29)    # 2.045 (n = 30)
qt(0.975, df = 999)   # 1.962 (n = 1000)
```
À $n = 100$, $t = 1{,}984$ : l'écart avec 1,96 est encore d'environ 0,02. À $n = 200$, $t = 1{,}972$ : la différence devient inférieure à 0,02, et la loi normale est alors une bonne approximation.
</details>

> **Note : Z ou t ?**
> Dans la pratique, on utilise la loi de Student (`t.test()`) dans tous les cas : pour les grands échantillons, elle donne le même résultat que la loi normale. Le test Z ne sert que si $\sigma$ est réellement connu, ce qui est rare.

## Exercice 5 – Vérifier le niveau de confiance par simulation

Le niveau $1 - \alpha$ est une propriété de la **procédure** : si on répète l'échantillonnage, la fraction d'intervalles qui contiennent la vraie valeur doit être proche de $1 - \alpha$. On vérifie cela sur des données simulées, où la vraie moyenne est connue.

> **Contexte :** la simulation rend concret le sens du niveau de confiance. Elle montre que 95 % des intervalles construits de cette façon contiennent la vraie valeur. Un statisticien l'utilise pour expliquer à un non-spécialiste ce que promet réellement un intervalle.

### Exemple

```r
set.seed(42)
mu <- 50

ics <- replicate(1000, {
  x <- rnorm(50, mean = mu, sd = 10)
  marge <- qt(0.975, df = 49) * sd(x) / sqrt(50)
  c(inf = mean(x) - marge, sup = mean(x) + marge)
})

# Part des intervalles qui contiennent mu = 50
mean(ics["inf", ] <= mu & ics["sup", ] >= mu)
# proche de 0.95
```

### Exercices

1. Exécuter la simulation ci-dessus. La couverture est-elle proche de 95 % ?
   > **Intérêt :** vérifier la couverture par simulation fournit une preuve reproductible, utile en formation comme en audit de méthode.
<details>
<summary>Correction</summary>

```r
mean(ics["inf", ] <= mu & ics["sup", ] >= mu)
# Une valeur voisine de 0,95 : environ 950 intervalles sur 1000 contiennent 50.
```
Le résultat exact varie avec la graine ; sur 1000 répétitions, il reste proche de 0,95.
</details>

2. Refaire la simulation avec un niveau de 90 % (quantile $t_{49;\,0{,}95}$). Quelle couverture observe-t-on ?
   > **Intérêt :** un niveau de 90 % doit couvrir environ 90 % du temps : vérifier ce point confirme que le niveau est une propriété de la méthode.
<details>
<summary>Correction</summary>

```r
set.seed(42)
ics90 <- replicate(1000, {
  x <- rnorm(50, mean = mu, sd = 10)
  marge <- qt(0.95, df = 49) * sd(x) / sqrt(50)
  c(inf = mean(x) - marge, sup = mean(x) + marge)
})
mean(ics90["inf", ] <= mu & ics90["sup", ] >= mu)
# proche de 0,90
```
</details>

3. Dans la simulation à 95 %, combien d'intervalles ratent la vraie valeur, en moyenne, sur 1000 ? Relier ce nombre à $\alpha$.
   > **Intérêt :** relier les intervalles ratés à α rend concrète une définition du risque souvent mal comprise.
<details>
<summary>Correction</summary>

Environ $1000 \times \alpha = 50$ intervalles ratent $\mu$. C'est exactement la définition du risque $\alpha$ : la fraction d'intervalles qui ratent la vraie valeur.
</details>

> **Note : `replicate()` avec plusieurs valeurs**
> `replicate()` renvoie une matrice : chaque colonne est le résultat d'une répétition. `ics["inf", ]` extrait la ligne des bornes inférieures, sur toutes les répétitions.

## Exercice 6 – Intervalle de confiance d'une proportion

Pour une proportion $\hat{p}$ calculée sur $n$ observations, l'intervalle de confiance au niveau $1 - \alpha$ est :

$$
\hat{p} \pm z_{1-\alpha/2} \sqrt{\frac{\hat{p}(1 - \hat{p})}{n}}
$$

Cette formule suppose que $n\hat{p} \ge 5$ et $n(1 - \hat{p}) \ge 5$. Pour des effectifs plus faibles, on utilise `prop.test()` ou une méthode exacte.

> **Contexte :** les proportions (part de passoires, de logements très peu performants, d'un type de bâtiment) sont les indicateurs les plus communiqués. L'intervalle donne la précision de ce pourcentage, à présenter avec la valeur.

### Exemple

On cherche la part des **passoires énergétiques**, définies ici comme les DPE en étiquette F ou G (cf. chapitre 3).

```r
n <- nrow(dpe)
k <- sum(dpe$etiquette_dpe %in% c("F", "G"))   # 830
p <- k / n                                     # 0.083

marge <- qnorm(0.975) * sqrt(p * (1 - p) / n)
c(inf = p - marge, sup = p + marge)
#   inf    sup
# 0.0776 0.0884
```

Les passoires représentent 8,3 % des DPE, avec un intervalle à 95 % de [7,8 % ; 8,8 %].

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `prop.test()` | Intervalle de confiance d'une proportion, avec correction de continuité. | `x` (succès), `n`, `conf.level` | `prop.test(830, 10000)$conf.int` |

### Exercices

1. Calculer l'intervalle de confiance à 95 % de la proportion de DPE en étiquette `A` ou `B`. Vérifier les conditions $n\hat{p} \ge 5$ et $n(1 - \hat{p}) \ge 5$.
   > **Intérêt :** vérifier les conditions de la formule évite de publier un intervalle faux pour un petit groupe : c'est un contrôle à faire systématiquement.
<details>
<summary>Correction</summary>

```r
k <- sum(dpe$etiquette_dpe %in% c("A", "B"))   # 251
p <- k / n                                     # 0.0251
c(n * p, n * (1 - p))                          # 251 et 9749 : conditions remplies
marge <- qnorm(0.975) * sqrt(p * (1 - p) / n)
c(inf = p - marge, sup = p + marge)            # environ [0,022 ; 0,028]
```
</details>

2. Calculer l'intervalle à 95 % de la proportion de logements de type `maison` avec `prop.test()`. Comparer avec la formule à la main.
   > **Intérêt :** présenter les deux méthodes montre que le choix n'influence pas la conclusion sur de grands effectifs.
<details>
<summary>Correction</summary>

```r
k_maison <- sum(dpe$type_batiment == "maison")   # 329
prop.test(k_maison, n)$conf.int

p_maison <- k_maison / n                          # 0.0329
marge <- qnorm(0.975) * sqrt(p_maison * (1 - p_maison) / n)
c(p_maison - marge, p_maison + marge)
```
Les deux intervalles sont très proches. `prop.test()` utilise une méthode plus précise (correction de continuité), et donne un intervalle légèrement différent.
</details>

3. Un collègue propose d'estimer la part des passoires sur seulement 50 DPE. Calculer les conditions de la formule, puis conclure sur l'usage de la formule.
   > **Intérêt :** un échantillon de 50 logements est une situation fréquente en contrôle : la formule ne convient plus, il faut une méthode exacte.
<details>
<summary>Correction</summary>

Avec $n = 50$ et $\hat{p} \approx 0{,}083$ : $n\hat{p} \approx 4{,}2$, en dessous de 5. La formule de Wald n'est pas fiable à cette taille ; il faut utiliser `prop.test()` ou un intervalle exact (`binom.test()`, chapitre 10).
</details>

## Exercice 7 – Représenter des intervalles avec `ggplot2` et `plotly`

Un intervalle se lit mieux sur un graphique : une barre d'erreur autour de l'estimation. On compare ici les moyennes de surface par étiquette DPE, chacune avec son intervalle à 95 %.

> **Contexte :** un graphique avec barres d'erreur est le support le plus courant pour comparer des groupes (étiquettes, zones, années). Il montre en un coup d'œil si les écarts sont nets ou noyés dans le bruit.

### Exemple

```r
resume <- dpe |>
  filter(!is.na(surface_habitable_logement)) |>
  group_by(etiquette_dpe) |>
  summarise(
    n = n(),
    moyenne = mean(surface_habitable_logement),
    ecart = sd(surface_habitable_logement),
    marge = qt(0.975, df = n - 1) * ecart / sqrt(n),
    inf = moyenne - marge,
    sup = moyenne + marge
  )

ggplot(resume, aes(x = etiquette_dpe, y = moyenne)) +
  geom_point(size = 3) +
  geom_errorbar(aes(ymin = inf, ymax = sup), width = 0.2) +
  labs(x = "Étiquette DPE", y = "Surface moyenne (m²)",
       title = "Surface moyenne avec intervalle à 95 %")
```

### Exercices

1. Reproduire le graphique et lire l'étiquette dont l'intervalle est le plus large. Pourquoi ?
   > **Intérêt :** les intervalles larges signalent les groupes où une conclusion serait fragile : un message à donner au commanditaire.
<details>
<summary>Correction</summary>

Les intervalles les plus larges sont ceux des étiquettes à faible effectif, comme A (19 DPE) : la marge est proportionnelle à $1/\sqrt{n}$, et $\sigma$ reste élevé. Un petit effectif donne donc une estimation imprécise.
</details>

2. Convertir le graphique avec `ggplotly()`, puis survoler la moyenne de l'étiquette B. Quelles valeurs apparaissent ?
   > **Intérêt :** le survol sert à retrouver une valeur exacte pour une présentation, sans relancer le calcul.
<details>
<summary>Correction</summary>

```r
p <- ggplot(resume, aes(x = etiquette_dpe, y = moyenne)) +
  geom_point(size = 3) +
  geom_errorbar(aes(ymin = inf, ymax = sup), width = 0.2)

ggplotly(p)
# Le survol de B affiche la moyenne d'environ 92,7 m².
```
</details>

> **Note : chevauchement et différence**
> Deux intervalles qui se chevauchent ne prouvent pas l'absence de différence. Pour comparer deux moyennes, on utilise un test (au chapitre 11). L'intervalle est une estimation, pas une décision.

## Liens utiles

- [Documentation de `t.test()` (R)](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/t.test.html)
- [Documentation de `prop.test()` (R)](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/prop.test.html)
- [Documentation de `ggplot2`](https://ggplot2.tidyverse.org/)
