# Chapitre 10 – Test sur une proportion et test de Poisson

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] poser $H_0$ et $H_1$ pour une proportion, avec $\alpha$ et $\beta$ fixés avant le calcul
- [ ] réaliser un test binomial exact sur une proportion et lire sa p-value
- [ ] calculer un intervalle de confiance exact d'une proportion
- [ ] calculer la taille d'échantillon pour tester une proportion
- [ ] réaliser un test de Poisson sur un nombre d'événements, en bilatéral ou en unilatéral
- [ ] distinguer une hypothèse bilatérale d'une hypothèse unilatérale et leurs conséquences sur la p-value

> **Contexte :** dans un établissement comme l'ADEME ou un service de données, les comptages sont partout : part de logements très peu performants, nombre de dossiers reçus, de contrôles réalisés, d'incidents. Tester une proportion ou un nombre d'événements permet de savoir si ce qu'on observe est compatible avec un objectif ou une référence, et de dimensionner un contrôle avant de le lancer.

## Sommaire

- [Préparation : packages, données et notations](#préparation--packages-données-et-notations)
- [Exercice 1 – Test binomial exact sur une proportion](#exercice-1--test-binomial-exact-sur-une-proportion)
- [Exercice 2 – Intervalle de confiance exact d'une proportion](#exercice-2--intervalle-de-confiance-exact-dune-proportion)
- [Exercice 3 – Taille d'échantillon pour une proportion](#exercice-3--taille-déchantillon-pour-une-proportion)
- [Exercice 4 – Test de Poisson bilatéral](#exercice-4--test-de-poisson-bilatéral)
- [Exercice 5 – Test de Poisson unilatéral](#exercice-5--test-de-poisson-unilatéral)
- [Exercice 6 – Visualiser le test avec `ggplot2` et `plotly`](#exercice-6--visualiser-le-test-avec-ggplot2-et-plotly)

## Préparation : packages, données et notations

```r
install.packages(c("pwr", "ggplot2", "plotly"))
library(readr)
library(dplyr)
library(ggplot2)
library(plotly)
library(pwr)

dpe <- read_csv("data/dpe_villeurbanne_10000.csv")
n <- nrow(dpe)   # 10 000 DPE
```

Ce chapitre utilise deux grandeurs de comptage :
- la **proportion** de passoires énergétiques, définies comme les DPE en étiquette F ou G : $k = 830$ sur $n = 10\,000$, soit $\hat{p} = 0{,}083$ ;
- le **nombre** d'immeubles (79 DPE) et de maisons (329 DPE) sur les 10 000 DPE.

### Notations

| Notion | Notation | Signification |
|--------|----------|---------------|
| Proportion vraie | $p$ | valeur inconnue de la population |
| Proportion de référence | $p_0$ | valeur testée dans $H_0$ |
| Proportion observée | $\hat{p} = k / n$ | estimation de $p$ |
| Taux de Poisson | $\lambda$ | nombre moyen d'événements par unité d'exposition |
| Exposition | $T$ | quantité sur laquelle on compte les événements (ici, nombre de DPE) |

### Le cadre du test

Pour une proportion, sous $H_0$ le nombre de succès $K$ suit une loi binomiale $\mathcal{B}(n, p_0)$ :

$$
P(K = k) = \binom{n}{k} p_0^{\,k} (1 - p_0)^{\,n-k}
$$

Le **test binomial exact** calcule la p-value à partir de cette loi, sans approximation normale. Il est recommandé pour les petits effectifs, et il reste valable pour les grands.

Pour un comptage avec une exposition $T$, sous $H_0$ le nombre d'événements suit une loi de Poisson de paramètre $\lambda_0 T$ :

$$
P(X = k) = e^{-\lambda_0 T} \frac{(\lambda_0 T)^k}{k!}
$$

Pour les deux tests :
- $\alpha = 0{,}05$, fixé avant le calcul ;
- $H_0$ est rejetée si $p < \alpha$ ;
- le risque $\beta$ dépend de l'écart réel ($p_1$ ou $\lambda_1$), de $n$ ou $T$, et de $\alpha$.

## Exercice 1 – Test binomial exact sur une proportion

On teste si la part des passoires énergétiques est égale à 10 %.

- $H_0 : p = 0{,}10$ ;
- $H_1 : p \ne 0{,}10$ ;
- $\alpha = 0{,}05$.

> **Contexte :** les passoires énergétiques (étiquettes F et G) sont une cible des politiques de rénovation. Une collectivité veut savoir si la part observée sur son territoire diffère d'un objectif de référence. La valeur de 10 % est un exemple : dans un vrai dossier, on la remplace par l'objectif ou la moyenne de référence. Le test binomial donne une réponse exacte, utile même sur de petits échantillons de contrôle.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `binom.test()` | Test binomial exact sur une proportion, avec p-value et intervalle exact. | `x` (succès), `n`, `p`, `alternative`, `conf.level` | `binom.test(830, 10000, p = 0.10)` |
| `$p.value`, `$conf.int` | Éléments extraits du résultat. | | `binom.test(830, 10000, p = 0.10)$p.value` |

### Exemple

```r
k <- sum(dpe$etiquette_dpe %in% c("F", "G"))   # 830

binom.test(k, n, p = 0.10)
# x = 830, n = 10000 ; p-value ≈ 6,5e-09
# 95 percent confidence interval: 0,0776 0,0884
```

### Exercices

1. Lancer le test ci-dessus. Quelle est la décision au seuil de 5 % ?
   > **Intérêt :** une décision de type « la part est-elle différente de l'objectif ? » se prend avec la p-value et la valeur de la part, pas avec la seule décision.
<details>
<summary>Correction</summary>

La p-value est d'environ $6{,}5 \times 10^{-9}$, très inférieure à 0,05 : on rejette $H_0$. La part des passoires n'est pas égale à 10 %.
</details>

2. Tester $p_0 = 0{,}083$ (valeur observée). Quelle est la décision, et pourquoi la p-value est-elle proche de 1 ?
   > **Intérêt :** montrer qu'une valeur de référence égale à la valeur observée ne déclenche aucun rejet permet d'expliquer au commanditaire ce que signifie « compatible avec ».
<details>
<summary>Correction</summary>

```r
binom.test(k, n, p = 0.083)
# p-value proche de 1 : on ne rejette pas H0.
```
La proportion observée est exactement celle de $H_0$ : la statistique est au centre de la loi binomiale, donc la p-value est maximale.
</details>

3. Vérifier la décision avec `prop.test()` et la correction de continuité. Les deux p-values sont-elles identiques ?
   > **Intérêt :** savoir quel test a été utilisé est indispensable dans un rapport : une approximation normale et un test exact peuvent différer sur de petits effectifs.
<details>
<summary>Correction</summary>

```r
prop.test(k, n, p = 0.10)$p.value
# Très proche de la valeur du test binomial, mais pas identique.
```
Le test binomial est exact. `prop.test()` utilise une approximation normale : pour les grands effectifs, les deux p-values sont très proches.
</details>

> **Note : la p-value dépend du sens de l'hypothèse**
> Dans cet exercice, $H_1$ est bilatérale : on rejette si $p$ est trop grand **ou** trop petit. Pour tester seulement « plus grand que 10 % », on écrit `alternative = "greater"`. Quand l'observation est loin de $p_0$ du côté testé, la p-value unilatérale est proche de la moitié de la bilatérale ; ce n'est pas une règle générale.

## Exercice 2 – Intervalle de confiance exact d'une proportion

Le test binomial donne aussi l'intervalle de confiance exact, qui correspond au test bilatéral : $p_0$ est dans l'intervalle si et seulement si $H_0$ n'est pas rejetée.

> **Contexte :** l'intervalle de la part de passoires se présente volontiers dans un tableau de bord ou un rapport. Il indique la plage de valeurs plausibles, ce qui est plus utile à un décideur qu'une simple décision « oui ou non ».

### Exemple

```r
binom.test(k, n, conf.level = 0.95)$conf.int
# [1] 0,0776 0,0884
```

### Exercices

1. Calculer l'intervalle à 95 % de la proportion de passoires. Vérifier que $p_0 = 0{,}10$ n'y est pas, et que $p_0 = 0{,}083$ y est.
   > **Intérêt :** le lien entre intervalle et test permet de répondre vite à « cet objectif est-il compatible avec nos données ? » sans relancer le calcul.
<details>
<summary>Correction</summary>

$0{,}10$ est au-dessus de la borne supérieure $0{,}0884$ : on rejette. $0{,}083$ est dans l'intervalle : on ne rejette pas. Les deux lectures sont cohérentes.
</details>

2. Calculer l'intervalle à 99 %. Que devient sa largeur ?
   > **Intérêt :** un intervalle à 99 % plus large montre le prix de la confiance supplémentaire : c'est un choix à expliquer, pas une convention automatique.
<details>
<summary>Correction</summary>

```r
binom.test(k, n, conf.level = 0.99)$conf.int
```
L'intervalle est plus large : on gagne en fiabilité en perdant en précision (chapitre 7).
</details>

## Exercice 3 – Taille d'échantillon pour une proportion

Pour un test de $p_0$ contre une valeur $p_1$ réelle, la taille nécessaire pour une puissance $1 - \beta$ est :

$$
n = \left(\frac{z_{1-\alpha/2}\sqrt{p_0(1 - p_0)} + z_{1-\beta}\sqrt{p_1(1 - p_1)}}{p_1 - p_0}\right)^2
$$

> **Contexte :** avant une campagne de contrôle ou une enquête, il faut savoir combien de dossiers analyser pour détecter un écart donné de la part. Ce calcul se fait avant de mobiliser les équipes, et il justifie la taille de l'échantillon auprès du commanditaire.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `pwr.p.test()` (package `pwr`) | Puissance ou taille d'un test sur une proportion. | `h` (écart), `n`, `power`, `sig.level` | `pwr.p.test(h = ES.h(0.083, 0.10), power = 0.80)` |
| `ES.h()` | Écart $h$ entre deux proportions, transformé par la fonction arcsinus. | `p1`, `p2` | `ES.h(0.083, 0.10)` |

### Exemple

```r
za <- qnorm(0.975)
zb <- qnorm(0.80)
p0 <- 0.10; p1 <- 0.083

ceiling(((za * sqrt(p0 * (1 - p0)) + zb * sqrt(p1 * (1 - p1))) / (p1 - p0))^2)
# environ 2 300
```

### Exercices

1. Calculer la taille nécessaire pour détecter $p_1 = 0{,}083$ face à $p_0 = 0{,}10$, avec une puissance de 0,80. Commenter.
   > **Intérêt :** un écart de 1,7 point sur une part de 10 % demande plusieurs milliers de dossiers : c'est une information à transmettre avant d'engager un budget.
<details>
<summary>Correction</summary>

La taille est d'environ 2 300 DPE : l'écart de 1,7 point est petit par rapport à $p_0$, donc il faut un grand échantillon. L'échantillon des 10 000 DPE dépasse largement ce besoin.
</details>

2. Calculer la même taille avec `pwr.p.test()`, en prenant `h = ES.h(p1, p0)`.
   > **Intérêt :** le package `pwr` fournit un calcul standard et reproductible, à citer dans la méthode d'une étude.
<details>
<summary>Correction</summary>

```r
pwr.p.test(h = ES.h(0.083, 0.10), power = 0.80, sig.level = 0.05)
# n proche de 2 300, avec une différence d'arrondi due à la transformation arcsinus.
```
</details>

3. Calculer la puissance d'un test sur 200 DPE pour le même écart. Que se passe-t-il ?
   > **Intérêt :** avec 200 dossiers, un contrôle a très peu de chances de détecter l'écart. Mieux vaut le savoir avant, pour ne pas conclure à tort qu'il n'y a pas de problème.
<details>
<summary>Correction</summary>

```r
pwr.p.test(h = ES.h(0.083, 0.10), n = 200, sig.level = 0.05)
# La puissance est très faible : β est grand, le test ne détecte pas l'écart.
```
</details>

## Exercice 4 – Test de Poisson bilatéral

Les immeubles représentent 79 DPE sur 10 000. On compare ce nombre à une fréquence de référence $\lambda_0 = 0{,}01$, soit 100 immeubles attendus.

- $H_0 : \lambda = 0{,}01$ (exposition $T = 10\,000$ DPE) ;
- $H_1 : \lambda \ne 0{,}01$ ;
- $\alpha = 0{,}05$.

> **Contexte :** un événement rare (un type de bâtiment peu représenté, un incident, un sinistre) se compte souvent avec une loi de Poisson. La question « ce nombre est-il normal pour nos volumes ? » se pose par exemple pour suivre une catégorie de dossiers particulière, ou repérer une évolution anormale.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `poisson.test()` | Test de Poisson exact sur un nombre d'événements. | `x` (nombre), `T` (exposition), `r` (taux de référence), `alternative` | `poisson.test(79, T = 10000, r = 0.01)` |

### Exemple

```r
nb_immeubles <- sum(dpe$type_batiment == "immeuble")   # 79

poisson.test(nb_immeubles, T = n, r = 0.01)
# p-value ≈ 0,036
```

### Exercices

1. Lancer le test. Quelle est la décision au seuil de 5 % ? La décision serait-elle la même au seuil de 1 % ?
   > **Intérêt :** un résultat proche du seuil doit être présenté avec sa valeur : la décision seule masque l'incertitude.
<details>
<summary>Correction</summary>

La p-value est d'environ 0,036 : on rejette $H_0$ au seuil de 5 %, mais pas au seuil de 1 %. Le résultat est proche du seuil : il faut le présenter avec sa p-value, pas seulement comme « significatif ».
</details>

2. Tester les maisons : 329 sur 10 000, avec $\lambda_0 = 0{,}03$. Quelle est la décision ?
   > **Intérêt :** un chargé d'études peut ainsi distinguer les catégories qui s'écartent de la référence de celles qui ne s'en écartent pas.
<details>
<summary>Correction</summary>

```r
nb_maisons <- sum(dpe$type_batiment == "maison")   # 329
poisson.test(nb_maisons, T = n, r = 0.03)
# p-value ≈ 0,094 : on ne rejette pas H0.
```
</details>

3. Expliquer pourquoi le test de Poisson est adapté à ce comptage, et quelle hypothèse il suppose.
   > **Intérêt :** connaître les hypothèses du test permet de repérer les cas où il ne faut pas l'utiliser, par exemple les dates de réception mensuelles.
<details>
<summary>Correction</summary>

Le comptage porte sur des événements rares, dans une exposition fixe, avec un taux constant. Le test suppose que les événements sont indépendants et que le taux ne change pas pendant la période. Pour un nombre d'événements par mois, comme les dates de réception, ce n'est pas le cas : le taux varie fortement d'un mois à l'autre.
</details>

## Exercice 5 – Test de Poisson unilatéral

Un test unilatéral répond à une question orientée : « le nombre est-il plus petit que prévu ? » ou « est-il plus grand ? ». L'hypothèse alternative ne contient qu'un sens.

> **Contexte :** un test unilatéral correspond à une question d'action. Par exemple : « une mesure de rénovation a-t-elle réduit le nombre de logements très peu performants ? ». On ne s'intéresse alors qu'à une seule direction, et le sens doit être fixé avant de regarder les données.

### Exemple

```r
# H1 : moins d'immeubles que prévu (lambda < 0,01)
poisson.test(nb_immeubles, T = n, r = 0.01, alternative = "less")
# p-value ≈ 0,017
```

### Exercices

1. Poser $H_0$ et $H_1$ pour l'alternative « moins d'immeubles que 1 % ». Lancer le test. Quelle décision ?
   > **Intérêt :** une question orientée a une réponse orientée : le rapport doit énoncer l'hypothèse avant le calcul.
<details>
<summary>Correction</summary>

$H_0 : \lambda = 0{,}01$ ; $H_1 : \lambda < 0{,}01$ ; $\alpha = 0{,}05$. La p-value unilatérale est d'environ 0,017 : on rejette $H_0$.
</details>

2. Comparer avec la p-value bilatérale de l'exercice 4 (0,036). Pourquoi est-elle plus petite ?
   > **Intérêt :** expliquer la différence de p-value évite qu'un lecteur choisisse le test le plus favorable a posteriori.
<details>
<summary>Correction</summary>

Le test unilatéral concentre tout $\alpha$ d'un seul côté : la p-value est plus petite dans le sens testé, et plus grande dans l'autre sens. Un test unilatéral ne doit pas être choisi après avoir vu le sens de l'écart : le sens se décide avant la collecte des données.
</details>

3. Tester maintenant « plus de maisons que 3 % » (`alternative = "greater"`, $\lambda_0 = 0{,}03$). Quelle décision ?
   > **Intérêt :** un résultat juste au-dessus du seuil ne doit pas être présenté comme un succès : c'est un signal à approfondir, pas une conclusion.
<details>
<summary>Correction</summary>

```r
poisson.test(nb_maisons, T = n, r = 0.03, alternative = "greater")
# p-value ≈ 0,052 : on ne rejette pas H0 au seuil de 5 %.
```
La p-value est juste au-dessus du seuil : on donne la valeur exacte et on évite de conclure sur un détail d'arrondi.
</details>

> **Note : unilatéral ou bilatéral ?**
> Le choix se fait avant le calcul, selon la question posée. Une hypothèse unilatérale choisie après avoir vu les données revient à gonfler artificiellement le risque $\alpha$.

## Exercice 6 – Visualiser le test avec `ggplot2` et `plotly`

Sous $H_0$, le nombre de passoires suit une loi binomiale $\mathcal{B}(10\,000 ; 0{,}10)$. On représente cette loi et la valeur observée.

> **Contexte :** ces graphiques aident à expliquer à un non-statisticien pourquoi une valeur est improbable sous l'hypothèse de référence, ou combien de dossiers il faut pour détecter un écart. Ils servent de support à la décision, pas à sa place.

### Exemple

```r
x <- 700:960
loi_h0 <- data.frame(k = x, prob = dbinom(x, size = n, prob = 0.10))

ggplot(loi_h0, aes(x = k, y = prob)) +
  geom_col(fill = "grey70") +
  geom_vline(xintercept = k, colour = "red") +
  labs(x = "Nombre de passoires", y = "Probabilité sous H0",
       title = "Loi binomiale sous H0 (p = 0,10) et valeur observée")
```

### Exercices

1. Reproduire le graphique. Où se trouve la valeur observée (830) par rapport à la loi sous $H_0$ ?
   > **Intérêt :** une observation loin du centre de la loi sous $H_0$ se voit sans calcul : c'est l'argument visuel de la décision.
<details>
<summary>Correction</summary>

La valeur observée est loin de la zone centrale de la loi sous $H_0$ (centrée sur 1 000) : la p-value est très petite. Cela se voit sans calcul.
</details>

2. Convertir le graphique avec `ggplotly()`, puis survoler la barre correspondant à la valeur 830.
   > **Intérêt :** le survol permet de vérifier une probabilité précise avant de la citer.
<details>
<summary>Correction</summary>

```r
p <- ggplot(loi_h0, aes(x = k, y = prob)) + geom_col(fill = "grey70")
ggplotly(p)
```
Le survol affiche la probabilité de chaque nombre de passoires sous $H_0$.
</details>

3. Tracer la courbe de la puissance du test binomial en fonction de $p_1$ (de 0,10 à 0,05) pour $n = 10\,000$, avec `plot_ly()`.
   > **Intérêt :** la courbe de puissance montre à un commanditaire que la précision dépend de l'écart qu'il juge important.
<details>
<summary>Correction</summary>

```r
p1s <- seq(0.10, 0.05, by = -0.001)
puiss <- sapply(p1s, function(p1) pwr.p.test(h = ES.h(p1, 0.10), n = n, sig.level = 0.05)$power)

plot_ly(x = ~p1s, y = ~puiss, type = "scatter", mode = "lines") |>
  layout(xaxis = list(title = "Proportion réelle p1"), yaxis = list(title = "Puissance"))
# La puissance augmente à mesure que p1 s'éloigne de 0,10.
```
</details>

## Liens utiles

- [Documentation de `binom.test()` (R)](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/binom.test.html)
- [Documentation de `poisson.test()` (R)](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/poisson.test.html)
- [Documentation du package `pwr`](https://cran.r-project.org/package=pwr)
