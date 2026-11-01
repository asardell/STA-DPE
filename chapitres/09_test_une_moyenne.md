# Chapitre 9 – Test sur une moyenne, puissance et taille d'échantillon

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] poser $H_0$ et $H_1$ pour un test bilatéral sur une moyenne, avec $\alpha$ fixé avant le calcul
- [ ] réaliser un test t à un échantillon et lire sa p-value et son intervalle de confiance
- [ ] réaliser un test Z lorsque l'écart type est connu, et le comparer au test t
- [ ] calculer la puissance d'un test Z et d'un test t, et relier $\beta$ à l'écart à détecter
- [ ] calculer la taille d'échantillon nécessaire pour une puissance donnée
- [ ] juger quand le test t est fiable sur une variable asymétrique

> **Contexte :** dans un établissement comme l'ADEME, un bureau d'études ou un service de données, on compare rarement une moyenne « à l'œil ». On veut savoir si un écart observé sur un échantillon de DPE est réel ou dû au hasard, avant de présenter un chiffre à un décideur, de lancer une action ou de dimensionner une enquête. Ce chapitre donne le cadre pour répondre à ces questions de façon reproductible.

## Sommaire

- [Chapitre 9 – Test sur une moyenne, puissance et taille d'échantillon](#chapitre-9--test-sur-une-moyenne-puissance-et-taille-déchantillon)
  - [Objectifs](#objectifs)
  - [Sommaire](#sommaire)
  - [Préparation : packages, données et notations](#préparation--packages-données-et-notations)
    - [Notations](#notations)
    - [Le cadre du test](#le-cadre-du-test)
  - [Exercice 1 – Test t sur un grand échantillon](#exercice-1--test-t-sur-un-grand-échantillon)
    - [Mémo](#mémo)
    - [Exemple](#exemple)
    - [Exercices](#exercices)
  - [Exercice 2 – Test t sur un petit échantillon](#exercice-2--test-t-sur-un-petit-échantillon)
    - [Exemple](#exemple-1)
    - [Exercices](#exercices-1)
  - [Exercice 3 – Test Z lorsque l'écart type est connu](#exercice-3--test-z-lorsque-lécart-type-est-connu)
    - [Mémo](#mémo-1)
    - [Exemple](#exemple-2)
    - [Exercices](#exercices-2)
  - [Exercice 4 – Puissance d'un test Z](#exercice-4--puissance-dun-test-z)
    - [Mémo](#mémo-2)
    - [Exemple](#exemple-3)
    - [Exercices](#exercices-3)
  - [Exercice 5 – Puissance d'un test t](#exercice-5--puissance-dun-test-t)
    - [Mémo](#mémo-3)
    - [Exercices](#exercices-4)
  - [Exercice 6 – Taille d'échantillon](#exercice-6--taille-déchantillon)
    - [Exemple](#exemple-4)
    - [Exercices](#exercices-5)
  - [Exercice 7 – Visualiser le test avec `ggplot2` et `plotly`](#exercice-7--visualiser-le-test-avec-ggplot2-et-plotly)
    - [Exemple](#exemple-5)
    - [Exercices](#exercices-6)
  - [Liens utiles](#liens-utiles)

## Préparation : packages, données et notations

```r
install.packages(c("pwr", "ggplot2", "plotly"))
library(readr)
library(dplyr)
library(ggplot2)
library(plotly)
library(pwr)

dpe <- read_csv("data/dpe_villeurbanne_10000.csv")

# Variable principale : logarithme de la consommation par m², proche de la loi normale (chapitre 8)
conso_m2 <- dpe$conso_5_usages_ep / dpe$surface_habitable_logement
ln_conso_m2 <- log(conso_m2[!is.na(conso_m2)])     # 9 899 valeurs
```

### Notations

| Notion | Notation | Signification |
|--------|----------|---------------|
| Moyenne vraie | $\mu$ | valeur inconnue de la population |
| Valeur de référence | $\mu_0$ | valeur testée dans $H_0$ |
| Écart réel à détecter | $\delta = \mu_1 - \mu_0$ | plus petit écart qu'on veut repérer |
| Écart type | $\sigma$ (connu) ou $s$ (estimé) | dispersion |
| Statistique t | $t = \dfrac{\bar{x} - \mu_0}{s / \sqrt{n}}$ | suit une loi de Student à $n - 1$ ddl sous $H_0$ |
| Statistique Z | $z = \dfrac{\bar{x} - \mu_0}{\sigma / \sqrt{n}}$ | suit une loi $\mathcal{N}(0, 1)$ sous $H_0$ |
| Puissance | $1 - \beta$ | probabilité de rejeter $H_0$ quand $H_1$ est vraie |

### Le cadre du test

Pour un test bilatéral sur une moyenne :

- $H_0 : \mu = \mu_0$ ;
- $H_1 : \mu \ne \mu_0$ ;
- $\alpha = 0{,}05$, fixé avant le calcul ;
- on rejette $H_0$ si $p < \alpha$, ce qui revient à dire que $\mu_0$ n'est pas dans l'intervalle de confiance à $1 - \alpha$ (chapitre 7).

Le risque $\beta$ est la probabilité de ne pas rejeter $H_0$ alors que $\mu$ est en réalité différent de $\mu_0$ d'un écart $\delta$. Il dépend de $\delta$, de $\sigma$, de $n$ et de $\alpha$.

## Exercice 1 – Test t sur un grand échantillon

Avec $n = 9\,899$, la loi de Student et la loi normale sont pratiquement confondues : le test t s'applique sans précaution particulière sur la moyenne du logarithme de la consommation par m².

> **Contexte :** une question typique est « la consommation moyenne du parc de logements de Villeurbanne est-elle celle de notre référence ? ». On compare une moyenne observée à une valeur de référence (un objectif, un bilan antérieur, une moyenne régionale), et le test indique si l'écart peut venir du hasard d'échantillonnage.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `t.test()` | Test t sur une moyenne, avec p-value et intervalle de confiance. | `x`, `mu`, `alternative`, `conf.level` | `t.test(x, mu = 5.23)` |
| `$p.value`, `$statistic`, `$conf.int` | Éléments extraits du résultat. | | `t.test(x, mu = 5.23)$p.value` |

### Exemple

```r
t.test(ln_conso_m2, mu = 5.23)
# t = 0.99, df = 9898, p-value = 0.32
# 95 percent confidence interval: 5.2259 5.2423
```

### Exercices

1. Poser $H_0$ et $H_1$ pour $\mu_0 = 5{,}23$, puis lancer `t.test()`. Quelle est la décision au seuil de 5 % ?
   > **Intérêt :** c'est la lecture de base qu'un chargé d'études doit savoir faire avant de présenter une comparaison à une référence.
<details>
<summary>Correction</summary>

$H_0 : \mu = 5{,}23$ ; $H_1 : \mu \ne 5{,}23$ ; $\alpha = 0{,}05$.

```r
t.test(ln_conso_m2, mu = 5.23)
# p-value ≈ 0,32 > 0,05 : on ne rejette pas H0.
```
Les données ne contredisent pas $\mu = 5{,}23$. Cela ne prouve pas que $\mu = 5{,}23$ : seulement que l'écart observé est compatible avec le hasard.
</details>

2. Refaire le test avec $\mu_0 = 5{,}20$. Quelle est la décision, et pourquoi la p-value est-elle si petite ?
   > **Intérêt :** un écart faible en valeur peut devenir très significatif sur un grand fichier. Il faut donc toujours présenter l'écart lui-même, pas seulement la p-value.
<details>
<summary>Correction</summary>

```r
t.test(ln_conso_m2, mu = 5.20)
# t ≈ 8,2 ; p-value < 2,2e-16 : on rejette H0.
```
L'écart entre la moyenne observée (5,234) et $\mu_0$ est faible en valeur, mais la p-value dépend de $n$ : avec près de 10 000 observations, l'erreur standard est très petite, et un écart de 0,034 devient très significatif. C'est le même effet que pour les tests d'ajustement au chapitre 8.
</details>

3. Vérifier le lien avec l'intervalle de confiance : $\mu_0 = 5{,}23$ est-il dans l'intervalle à 95 %, et $\mu_0 = 5{,}20$ ?
   > **Intérêt :** un intervalle de confiance se présente plus facilement qu'une p-value : il donne une fourchette de valeurs plausibles pour la moyenne.
<details>
<summary>Correction</summary>

```r
t.test(ln_conso_m2)$conf.int
# [5,2259 ; 5,2423]
```
$5{,}23$ est dans l'intervalle, donc on ne rejette pas $H_0$ : $5{,}20$ n'y est pas, donc on rejette. Les deux lectures donnent la même décision.
</details>

> **Note : la p-value ne mesure pas l'importance de l'écart**
> Une p-value très petite dit que l'écart est peu probable sous $H_0$, pas qu'il est important. Pour l'importance pratique, on regarde l'écart lui-même, ici $5{,}234 - 5{,}20 = 0{,}034$ sur le logarithme, soit une consommation environ 3 % plus élevée.

## Exercice 2 – Test t sur un petit échantillon

Avec peu d'observations, on teste les 30 premières valeurs de $\ln(\text{conso/m}^2)$.

> **Contexte :** on ne dispose pas toujours de 10 000 DPE. Une commune de petite taille, une campagne de contrôle ponctuelle ou un audit sur un lot restreint donnent des échantillons de quelques dizaines de dossiers. Ce test montre ce qu'on peut conclure avec peu de données, et ce qu'il faut dire au lecteur.

### Exemple

```r
petit <- ln_conso_m2[1:30]
mean(petit)     # 5,0895
sd(petit)       # 0,464

t.test(petit, mu = 5.20)
# t = -1,3046 ; df = 29 ; p-value = 0,2023
# 95 percent confidence interval: 4,916193 5,262751
# mean of x: 5,089472
```

### Exercices

1. Poser $H_0$ et $H_1$ pour $\mu_0 = 5{,}20$, puis lancer `t.test()`. Quelle est la décision ?
   > **Intérêt :** sur 30 dossiers, « pas de différence détectée » est une conclusion fragile : le rapport doit le préciser.
<details>
<summary>Correction</summary>

$H_0 : \mu = 5{,}20$ ; $H_1 : \mu \ne 5{,}20$ ; $\alpha = 0{,}05$. Le test donne $t = -1{,}3046$ avec 29 degrés de liberté et une p-value de $0{,}2023$. Comme $p > 0{,}05$ (et que $|t|$ est inférieur au quantile $t_{29;\,0{,}975} = 2{,}045$), on ne rejette pas $H_0$.
</details>

2. Calculer l'intervalle à 95 % avec `qt()` et le comparer à celui de `t.test()`.
   > **Intérêt :** un intervalle large sur un petit échantillon est un message utile : il indique qu'une étude complémentaire est nécessaire avant de conclure.
<details>
<summary>Correction</summary>

```r
m <- mean(petit); s <- sd(petit)
m + c(-1, 1) * qt(0.975, df = 29) * s / sqrt(30)
# [4,9162 ; 5,2627] : identique à t.test()
```
</details>

3. Recalculer l'intervalle en remplaçant $t_{29}$ par la valeur normale $1{,}96$. Quelle erreur commet-on ?
   > **Intérêt :** utiliser 1,96 à la place de la loi de Student est une erreur fréquente dans les rapports : elle donne une fausse impression de précision.
<details>
<summary>Correction</summary>

```r
m + c(-1, 1) * qnorm(0.975) * s / sqrt(30)
# Intervalle plus étroit : la loi normale sous-estime l'incertitude avec 30 observations.
```
</details>

> **Note : un test non significatif avec 30 observations**
> La non-décision de la question 1 ne dit pas que la vraie moyenne vaut 5,20 : avec 30 observations, β est grand. Le même écart réel pourrait être détecté avec l'échantillon complet (exercice 1).

## Exercice 3 – Test Z lorsque l'écart type est connu

Le test Z suppose que $\sigma$ est connu. En pratique, on le remplace par l'écart type observé quand $n$ est grand : le test Z et le test t donnent alors presque le même résultat.

> **Contexte :** le test Z s'utilise quand une source fiable fournit l'écart type de référence (une étude antérieure, une norme, un historique très large). Dans la plupart des analyses, l'écart type est estimé à partir des données : le test t est alors le bon choix, et il faut savoir justifier pourquoi le test Z n'est pas retenu.

$$
z = \frac{\bar{x} - \mu_0}{\sigma / \sqrt{n}} \sim \mathcal{N}(0, 1) \text{ sous } H_0, \qquad p = 2\left(1 - \Phi(|z|)\right)
$$

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `pnorm()` | Fonction de répartition normale $\Phi$. | `q`, `mean`, `sd` | `2 * (1 - pnorm(abs(z)))` |
| `qnorm()` | Quantile normal. | `p` | `qnorm(0.975)` |

### Exemple

```r
sigma <- sd(ln_conso_m2)   # écart type supposé connu
n <- length(ln_conso_m2)
z <- (mean(ln_conso_m2) - 5.23) / (sigma / sqrt(n))
z                          # 0,988
2 * (1 - pnorm(abs(z)))    # p-value ≈ 0,32
```

### Exercices

1. Calculer $z$ pour $\mu_0 = 5{,}23$ et la p-value. Comparer avec le test t de l'exercice 1.
   > **Intérêt :** montrer que les deux tests concordent sur un grand échantillon permet de justifier le choix du test t, plus courant dans les rapports.
<details>
<summary>Correction</summary>

$z \approx 0{,}99$ et $p \approx 0{,}32$, quasiment identiques au test t : avec $n = 9\,899$, la différence entre $s$ et $\sigma$ et entre $t$ et $z$ est négligeable.
</details>

2. Pour l'échantillon de 30 observations, calculer $z$ avec $\sigma = 0{,}464$ (écart type observé) et le comparer à $t = -1{,}305$. Pourquoi la conclusion du test Z est-elle trop optimiste ?
   > **Intérêt :** un chargé d'études qui utilise le test Z sur un petit échantillon annonce trop de résultats significatifs, ce qui peut conduire à des décisions coûteuses (travaux, aides) fondées sur du bruit.
<details>
<summary>Correction</summary>

```r
z30 <- (mean(petit) - 5.20) / (sd(petit) / sqrt(30))
z30    # environ -1,3 ; c'est la même valeur que t, mais la loi de référence est différente
qnorm(0.975)  # 1,96 : seuil du test Z ; pour le test t, le seuil est 2,045
```
Le seuil du test Z (1,96) est plus bas que celui du test t (2,045) : avec 30 observations, le test Z rejetterait plus souvent à tort. C'est pourquoi on utilise le test t quand $\sigma$ est estimé.
</details>

## Exercice 4 – Puissance d'un test Z

La puissance est la probabilité de rejeter $H_0$ quand la vraie moyenne est $\mu_1 = \mu_0 + \delta$. Pour un test bilatéral :

$$
1 - \beta = \Phi\!\left(\frac{\delta\sqrt{n}}{\sigma} - z_{1-\alpha/2}\right) + \Phi\!\left(-\frac{\delta\sqrt{n}}{\sigma} - z_{1-\alpha/2}\right)
$$

Le second terme est négligeable sauf si $\delta$ est très grand ou de signe opposé. Plus $\delta$ ou $n$ augmentent, plus la puissance augmente.

> **Note : les trois valeurs à connaître**
> - **Seuil de significativité** ($\alpha$) : 5 %. Risque de rejeter $H_0$ alors qu'elle est vraie.
> - **Niveau de confiance** ($1 - \alpha$) : 95 %. Niveau de confiance d'un intervalle.
> - **Puissance statistique** ($1 - \beta$) : 80 %. Probabilité de détecter un effet réel.
>
> Ce sont des conventions, pas des lois : elles se discutent selon l'enjeu. Un contrôle à fort enjeu peut viser une puissance de 90 %, au prix d'un échantillon plus grand (exercice 6).

> **Contexte :** avant une enquête ou un contrôle, on se demande si l'on repérera un écart réel suffisant pour justifier une action, par exemple une dérive de consommation après des travaux ou une différence entre deux zones. La puissance répond à cette question avant de mobiliser des moyens. Une étude trop petite peut conclure « rien à signaler » alors qu'un écart réel existe.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `pwr.norm.test()` (package `pwr`) | Puissance d'un test Z sur une moyenne. | `d` (écart standardisé), `n`, `sig.level` | `pwr.norm.test(d = 0.12, n = 100, sig.level = 0.05)` |

Ici, l'écart standardisé est $d = \delta / \sigma$.

### Exemple

```r
d <- 0.05 / sigma                 # écart de 0,05 sur le logarithme, en unités d'écart type
pwr.norm.test(d = d, n = 100, sig.level = 0.05)
# power = 0,225 : β ≈ 0,78
```

### Exercices

1. Calculer la puissance pour $n = 100$ et $\delta = 0{,}05$. Que vaut $\beta$ ?
   > **Intérêt :** avec 100 dossiers, on ne détecte qu'un écart sur quatre : si l'étude est trop petite, son résultat « négatif » ne vaut pas grand-chose.
<details>
<summary>Correction</summary>

Puissance $\approx 0{,}225$, donc $\beta \approx 0{,}775$ : avec 100 observations, on ne détecte que 1 fois sur 4 un écart de 0,05.
</details>

2. Calculer la puissance pour $n = 544$. Que constate-t-on ?
   > **Intérêt :** 544 DPE est un ordre de grandeur utile pour dimensionner un échantillon de contrôle, avant de discuter du coût.
<details>
<summary>Correction</summary>

```r
pwr.norm.test(d = d, n = 544, sig.level = 0.05)
# power ≈ 0,80
```
Avec 544 observations, la puissance atteint 0,80 : c'est la valeur conventionnelle.
</details>

3. Calculer la puissance pour $\delta = 0{,}02$ avec $n = 544$. Pourquoi est-elle beaucoup plus faible ?
   > **Intérêt :** il faut décider, avant l'étude, de ce qui constitue un écart réellement important. Un écart plus petit demande beaucoup plus de dossiers.
<details>
<summary>Correction</summary>

```r
pwr.norm.test(d = 0.02 / sigma, n = 544, sig.level = 0.05)
# power nettement plus faible
```
La puissance dépend de $\delta / \sigma$ : un écart plus petit est plus difficile à détecter, et il faut un échantillon beaucoup plus grand.
</details>

## Exercice 5 – Puissance d'un test t

Pour un test t, la puissance se calcule avec la loi de Student non centrée. Le package `pwr` la calcule directement.

> **Contexte :** dans la plupart des enquêtes, l'écart type n'est pas connu d'avance. La puissance du test t est donc la grandeur à calculer pour planifier une étude réelle, et c'est celle que l'on présente à un commanditaire.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `pwr.t.test()` | Puissance ou taille d'échantillon d'un test t à un échantillon. | `d`, `n`, `sig.level`, `type = "one.sample"` | `pwr.t.test(d = 0.12, n = 30, type = "one.sample")` |

### Exercices

1. Calculer la puissance d'un test t pour $n = 30$ et $d = 0{,}12$, puis pour $n = 100$. Comparer aux puissances du test Z de l'exercice 4.
   > **Intérêt :** l'écart entre les deux puissances mesure le coût de l'estimation de $\sigma$. C'est un argument pour prévoir un échantillon un peu plus grand.
<details>
<summary>Correction</summary>

```r
pwr.t.test(d = d, n = 30, sig.level = 0.05, type = "one.sample")
pwr.t.test(d = d, n = 100, sig.level = 0.05, type = "one.sample")
# La puissance du test t est légèrement inférieure à celle du test Z
# pour le même n : la loi de Student est plus étalée.
```
</details>

2. Expliquer pourquoi la puissance d'un test sur 30 observations est faible, même pour un écart réel de $0{,}12\,\sigma$.
   > **Intérêt :** un résultat non significatif sur un petit échantillon n'est pas une preuve d'absence d'effet. C'est un message à rendre explicite dans un rapport.
<details>
<summary>Correction</summary>

Avec 30 observations, l'erreur standard $\sigma/\sqrt{n}$ est grande par rapport à l'écart $\delta$. Le test ne peut pas distinguer $\mu_0$ de $\mu_1$ de façon fiable : β est grand.
</details>

## Exercice 6 – Taille d'échantillon

La taille nécessaire pour une puissance $1 - \beta$ donnée découle de la formule de la puissance. Pour un test Z bilatéral :

$$
n = \left(\frac{(z_{1-\alpha/2} + z_{1-\beta})\,\sigma}{\delta}\right)^2
$$

Avec $\alpha = 0{,}05$ et $1 - \beta = 0{,}80$, on a $z_{1-\alpha/2} = 1{,}96$ et $z_{1-\beta} = 0{,}84$.

> **Contexte :** c'est la question la plus fréquente en planification : combien de DPE faut-il analyser pour répondre à une question avec une précision donnée ? Le résultat dépend surtout de l'écart jugé important, et c'est donc une décision à documenter avec le commanditaire.

### Exemple

```r
za <- qnorm(0.975)
zb <- qnorm(0.80)

ceiling(((za + zb) * sigma / 0.05)^2)   # 543
ceiling(((za + zb) * sigma / 0.02)^2)   # 3 391
```

### Exercices

1. Calculer la taille nécessaire pour détecter $\delta = 0{,}05$ avec une puissance de 0,80. Comparer à l'exercice 4.
   > **Intérêt :** vérifier que deux méthodes de calcul donnent le même ordre de grandeur évite de présenter un chiffre de dimensionnement erroné.
<details>
<summary>Correction</summary>

$n = 543$, cohérent avec la puissance de 0,80 trouvée à $n = 544$ dans l'exercice 4 (écart d'arrondi).
</details>

2. Calculer la taille nécessaire pour $\delta = 0{,}02$, puis pour une puissance de 0,90. Commenter.
   > **Intérêt :** dans un appel d'offres ou un plan de contrôle, le choix de $\delta$ et de la puissance se traduit en coût : c'est une décision de la direction, que le statisticien doit rendre visible.
<details>
<summary>Correction</summary>

```r
ceiling(((za + qnorm(0.90)) * sigma / 0.02)^2)
# environ 4 500
```
Diviser l'écart à détecter par 2,5 multiplie la taille par environ 6 ; passer de 80 % à 90 % de puissance l'augmente aussi. Le choix de $\delta$ est donc la décision la plus importante.
</details>

3. Avec `pwr.t.test()`, calculer la taille pour $d = 0{,}12$ et une puissance de 0,80 (`type = "one.sample"`). Comparer à la formule Z.
   > **Intérêt :** dans un rapport, on indique la méthode et le package utilisés, pour qu'un relecteur puisse refaire le calcul.
<details>
<summary>Correction</summary>

```r
pwr.t.test(d = d, power = 0.80, sig.level = 0.05, type = "one.sample")
# n est proche de la valeur Z, un peu plus élevé : la loi de Student est plus étalée.
```
</details>

> **Note : la taille se planifie avant l'étude**
> On fixe $\delta$ (l'écart qui compte en pratique), $\alpha$ et la puissance, puis on calcule $n$. Calculer la taille après avoir vu les données, pour choisir un $\delta$ qui donne un résultat voulu, est une pratique à éviter.

## Exercice 7 – Visualiser le test avec `ggplot2` et `plotly`

On représente la moyenne observée, la valeur $\mu_0$ et l'intervalle de confiance à 95 %.

> **Contexte :** un graphique accompagne le test dans un rapport. Il montre où se trouve la valeur de référence par rapport à l'estimation, ce qu'un lecteur non statisticien comprend plus vite qu'une p-value. C'est aussi le moyen le plus simple de communiquer une décision à un commanditaire.

### Exemple

```r
ic <- t.test(ln_conso_m2)$conf.int

ggplot(data.frame(x = ln_conso_m2), aes(x = x)) +
  geom_histogram(aes(y = after_stat(density)), bins = 60, fill = "steelblue", colour = "white") +
  geom_vline(xintercept = mean(ln_conso_m2), colour = "black") +
  geom_vline(xintercept = 5.23, colour = "red", linetype = "dashed") +
  annotate("rect", xmin = ic[1], xmax = ic[2], ymin = 0, ymax = Inf, alpha = 0.2, fill = "grey") +
  coord_cartesian(xlim = c(4, 6.5)) +
  labs(x = "ln(conso / m²)", y = "Densité")
```

### Exercices

1. Reproduire le graphique. Où se trouve $\mu_0 = 5{,}23$ par rapport à l'intervalle de confiance ?
   > **Intérêt :** une valeur de référence hors de la zone grise se voit immédiatement : c'est le résumé visuel de la décision.
<details>
<summary>Correction</summary>

La ligne rouge est à l'intérieur de la zone grise : $\mu_0$ est dans l'intervalle, donc on ne rejette pas $H_0$.
</details>

2. Convertir le graphique avec `ggplotly()`, puis survoler la moyenne observée pour lire sa valeur.
   > **Intérêt :** le format interactif sert à explorer les valeurs, par exemple pour vérifier une observation extrême, avant une présentation figée.
<details>
<summary>Correction</summary>

```r
p <- ggplot(data.frame(x = ln_conso_m2), aes(x = x)) +
  geom_histogram(aes(y = after_stat(density)), bins = 60, fill = "steelblue", colour = "white") +
  coord_cartesian(xlim = c(4, 6.5))
ggplotly(p)
```
Le survol affiche les bornes des barres ; la moyenne 5,234 se lit sur le trait noir du graphique `ggplot2`.
</details>

3. Représenter la puissance en fonction de $n$ (de 50 à 1 000) pour $\delta = 0{,}05$, avec `pwr.norm.test()` dans une boucle, puis tracer la courbe avec `plot_ly()`.
   > **Intérêt :** la courbe de puissance est un outil de dialogue avec un commanditaire : elle montre combien de dossiers il faut pour un écart donné.
<details>
<summary>Correction</summary>

```r
ns <- seq(50, 1000, by = 10)
puiss <- sapply(ns, function(k) pwr.norm.test(d = d, n = k, sig.level = 0.05)$power)

plot_ly(x = ~ns, y = ~puiss, type = "scatter", mode = "lines") |>
  layout(xaxis = list(title = "Taille d'échantillon n"),
         yaxis = list(title = "Puissance"))
# La puissance croît avec n et atteint 0,80 vers n = 544.
```
</details>

## Liens utiles

- [Documentation de `t.test()` (R)](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/t.test.html)
- [Documentation du package `pwr`](https://cran.r-project.org/package=pwr)
- [Documentation de `plotly` pour R](https://plotly.com/r/)
