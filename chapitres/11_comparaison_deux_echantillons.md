# Chapitre 11 – Comparaison de deux échantillons

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] poser $H_0$ et $H_1$ pour comparer deux variances ou deux moyennes, avec $\alpha$ et $\beta$ fixés avant le calcul
- [ ] tester l'égalité de deux variances avec le test de Fisher et le test de Levene
- [ ] choisir entre le test t de Student et le test de Welch selon l'égalité des variances
- [ ] réaliser un test t sur des données appariées, et expliquer pourquoi il diffère du test sur échantillons indépendants
- [ ] comparer deux proportions sur deux échantillons indépendants
- [ ] calculer la taille d'échantillon nécessaire pour comparer deux proportions

> **Contexte :** en entreprise ou à l'ADEME, on compare souvent deux groupes : deux zones géographiques, deux types de bâtiments, un avant et un après travaux, deux méthodes de collecte. Une conclusion « il y a une différence » ne vaut que si le test est bien choisi. Ce chapitre donne la démarche de décision, de la variance au test de moyennes ou de proportions, pour que le choix soit justifiable devant un relecteur.

## Sommaire

- [Chapitre 11 – Comparaison de deux échantillons](#chapitre-11--comparaison-de-deux-échantillons)
  - [Objectifs](#objectifs)
  - [Sommaire](#sommaire)
  - [Préparation : packages, données et notations](#préparation--packages-données-et-notations)
    - [Notations](#notations)
    - [Le cadre des tests](#le-cadre-des-tests)
  - [Exercice 1 – Test de Fisher sur les variances](#exercice-1--test-de-fisher-sur-les-variances)
    - [Mémo](#mémo)
    - [Exemple](#exemple)
    - [Exercices](#exercices)
  - [Exercice 2 – Test de Levene sur les variances](#exercice-2--test-de-levene-sur-les-variances)
    - [Mémo](#mémo-1)
    - [Exemple](#exemple-1)
    - [Exercices](#exercices-1)
  - [Exercice 3 – Test t de Student (variances égales)](#exercice-3--test-t-de-student-variances-égales)
    - [Mémo](#mémo-2)
    - [Exemple](#exemple-2)
    - [Exercices](#exercices-2)
  - [Exercice 4 – Test de Welch (variances inégales)](#exercice-4--test-de-welch-variances-inégales)
    - [Exemple](#exemple-3)
    - [Exercices](#exercices-3)
  - [Exercice 5 – Test t sur données appariées](#exercice-5--test-t-sur-données-appariées)
    - [Mémo](#mémo-3)
    - [Exemple](#exemple-4)
    - [Exercices](#exercices-4)
  - [Exercice 6 – Comparer deux proportions](#exercice-6--comparer-deux-proportions)
    - [Mémo](#mémo-4)
    - [Exemple](#exemple-5)
    - [Exercices](#exercices-5)
  - [Exercice 7 – Taille d'échantillon pour deux proportions](#exercice-7--taille-déchantillon-pour-deux-proportions)
    - [Mémo](#mémo-5)
    - [Exemple](#exemple-6)
    - [Exercices](#exercices-6)
  - [Exercice 8 – Visualiser les comparaisons](#exercice-8--visualiser-les-comparaisons)
    - [Exemple](#exemple-7)
    - [Exercices](#exercices-7)
  - [Exercice 9 – Quel test choisir ?](#exercice-9--quel-test-choisir-)
  - [Exercice 10 – Électricité contre réseau de chauffage urbain](#exercice-10--électricité-contre-réseau-de-chauffage-urbain)
    - [Mémo](#mémo-6)
    - [Préparation](#préparation)
    - [Exercices](#exercices-8)
  - [Liens utiles](#liens-utiles)

## Préparation : packages, données et notations

```r
install.packages(c("car", "pwr", "ggplot2", "plotly"))
library(readr)
library(dplyr)
library(ggplot2)
library(plotly)
library(car)
library(pwr)

dpe <- read_csv("data/dpe_villeurbanne_10000.csv")

# Logarithme de la consommation par m², pour les appartements et les maisons
dpe_ln <- dpe |>
  filter(!is.na(surface_habitable_logement), conso_5_usages_ep > 0) |>
  mutate(ln_conso_m2 = log(conso_5_usages_ep / surface_habitable_logement))

ln_appart <- dpe_ln$ln_conso_m2[dpe_ln$type_batiment == "appartement"]   # 9 570 valeurs
ln_maison <- dpe_ln$ln_conso_m2[dpe_ln$type_batiment == "maison"]        # 329 valeurs
```

### Notations

| Notion | Notation | Signification |
|--------|----------|---------------|
| Moyennes vraies | $\mu_1$, $\mu_2$ | moyennes des deux groupes |
| Variances vraies | $\sigma_1^2$, $\sigma_2^2$ | dispersions des deux groupes |
| Variances estimées | $s_1^2$, $s_2^2$ | estimations observées |
| Proportions vraies | $p_1$, $p_2$ | parts dans les deux groupes |

### Le cadre des tests

Pour comparer deux groupes, on pose toujours une hypothèse d'égalité :

- $H_0 : \mu_1 = \mu_2$ (ou $\sigma_1^2 = \sigma_2^2$, ou $p_1 = p_2$) ;
- $H_1$ : l'égalité n'est pas vraie (bilatéral, sauf indication contraire) ;
- $\alpha = 0{,}05$, fixé avant le calcul ;
- le risque $\beta$ dépend de l'écart réel, de la variabilité et des tailles des deux échantillons.

Avant de comparer deux moyennes, il faut **choisir le test** selon la nature des données :

| Situation | Test à utiliser |
|-----------|-----------------|
| Deux groupes indépendants, variances égales | test t de Student (`var.equal = TRUE`) |
| Deux groupes indépendants, variances différentes ou inconnues | test de Welch (par défaut dans `t.test()`) |
| Mêmes unités mesurées deux fois, ou appariées | test t apparié (`paired = TRUE`) |
| Deux proportions indépendantes | `prop.test()` avec deux groupes |

## Exercice 1 – Test de Fisher sur les variances

Le test de Fisher compare deux variances à l'aide du rapport $F = s_1^2 / s_2^2$, qui suit une loi $F$ sous $H_0$.

- $H_0 : \sigma_1^2 = \sigma_2^2$ ;
- $H_1 : \sigma_1^2 \ne \sigma_2^2$ ;
- $\alpha = 0{,}05$.

> **Contexte :** avant de comparer deux moyennes, on vérifie si les dispersions sont comparables. Une variabilité différente change le test à utiliser. Ici, les appartements et les maisons n'ont pas la même dispersion de consommation par m² : c'est un constat à faire avant toute comparaison.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `var.test()` | Test de Fisher d'égalité de deux variances. | `x`, `y`, `alternative` | `var.test(ln_appart, ln_maison)` |

### Exemple

```r
var(ln_appart)     # 0,169
var(ln_maison)     # 0,257
var(ln_appart) / var(ln_maison)   # F ≈ 0,658
```

Avant le test, on visualise la dispersion de chaque groupe avec un boxplot :

```r
dpe_ln |>
  filter(type_batiment %in% c("appartement", "maison")) |>
  ggplot(aes(x = type_batiment, y = ln_conso_m2, fill = type_batiment)) +
  geom_boxplot(outlier.shape = NA) +
  guides(fill = "none") +
  labs(x = "Type de bâtiment", y = "ln(conso / m²)")
```

> **Intérêt :** le boxplot montre la dispersion de chaque groupe avant tout calcul : une boîte plus haute pour les maisons annonce déjà des variances différentes, ce que le test de Fisher va confirmer.

### Exercices

1. Calculer les deux variances et leur rapport. Quelle variance est la plus grande ?
   > **Intérêt :** la variance est le premier diagnostic à faire : c'est elle qui détermine le test de moyennes à utiliser.
<details>
<summary>Correction</summary>

La variance des maisons ($0{,}257$) est plus grande que celle des appartements ($0{,}169$) : le rapport $s_1^2 / s_2^2 \approx 0{,}66$ est inférieur à 1.
</details>

2. Lancer `var.test(ln_appart, ln_maison)`. Quelle est la décision au seuil de 5 % ?
   > **Intérêt :** un rejet d'égalité des variances est une information en soi, à transmettre avant la comparaison des moyennes.
<details>
<summary>Correction</summary>

```r
var.test(ln_appart, ln_maison)
# La p-value est très petite : on rejette H0. Les variances ne sont pas égales.
```
</details>

3. Le test de Fisher suppose des données normales. Expliquer pourquoi c'est un problème ici, et quel test est plus robuste.
   > **Intérêt :** justifier le choix d'un test robuste évite qu'un lecteur conteste la méthode après coup.
<details>
<summary>Correction</summary>

Le test de Fisher est très sensible à l'écart à la normalité (chapitre 8) : une légère asymétrie peut suffire à produire un rejet. Le test de Levene, qui utilise les écarts absolus à la médiane, est plus robuste. C'est l'objet de l'exercice suivant.
</details>

## Exercice 2 – Test de Levene sur les variances

Le test de Levene compare les variances à partir des écarts absolus à la médiane de chaque groupe. Il est moins sensible à la non-normalité que le test de Fisher.

- $H_0 : \sigma_1^2 = \sigma_2^2$ ;
- $H_1 : \sigma_1^2 \ne \sigma_2^2$ ;
- $\alpha = 0{,}05$.

> **Contexte :** le test de Levene est celui qu'on présente par défaut dans un rapport sur des données réelles, qui sont rarement parfaitement normales. Il dit si les variances diffèrent, ce qui conditionne la suite de l'analyse.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `leveneTest()` (package `car`) | Test de Levene d'égalité de variances entre groupes. | formule `y ~ groupe`, `data`, `center` | `leveneTest(ln_conso_m2 ~ type_batiment, data = dpe_ln)` |

### Exemple

```r
# Ne garder que les deux types étudiés
dpe_2 <- dpe_ln |> filter(type_batiment %in% c("appartement", "maison"))
leveneTest(ln_conso_m2 ~ type_batiment, data = dpe_2)
```

### Exercices

1. Lancer `leveneTest()`. La décision est-elle la même qu'avec le test de Fisher ?
   > **Intérêt :** quand deux tests donnent la même conclusion, on peut l'affirmer avec plus de confiance devant un commanditaire.
<details>
<summary>Correction</summary>

La sortie R donne une p-value de $1{,}563 \times 10^{-11}$ (marquée `***`). Elle est très inférieure à 0,001 et à 0,05 : on rejette $H_0$, comme avec le test de Fisher. Les deux tests concordent.
</details>

2. Conclure sur le choix du test de comparaison des moyennes : quel test faut-il utiliser ensuite ?
   > **Intérêt :** la décision sur les variances conditionne tout le reste : c'est elle qui dit quel test de moyennes appliquer.
<details>
<summary>Correction</summary>

Les variances diffèrent : on utilise le test de Welch (exercice 4), et non le test t de Student (exercice 3).
</details>

3. Refaire le test de Levene sur les 30 premières valeurs de chaque groupe seulement. Que devient la décision ?
   > **Intérêt :** avec un petit échantillon, le test ne détecte plus une différence de variances pourtant réelle sur la population entière. La taille de l'échantillon conditionne donc la sensibilité du test : avec plusieurs milliers de DPE, un écart minime devient significatif ; avec 30 par groupe, il passe inaperçu.
<details>
<summary>Correction</summary>

```r
petit <- data.frame(
  y = c(ln_appart[1:30], ln_maison[1:30]),
  g = rep(c("appartement", "maison"), each = 30)
)
leveneTest(y ~ g, data = petit)
# F ≈ 0,52 ; p-value ≈ 0,47 : on ne rejette pas H0.
```
Sur 60 observations, le test ne rejette pas l'égalité des variances, alors que sur les données complètes il la rejette fortement (question 2). Les variances estimées restent différentes (0,215 contre 0,268 sur ces 30 premières valeurs), mais le test n'a pas la puissance de le démontrer. Ne pas rejeter $H_0$ n'est donc pas une preuve d'égalité : c'est une limite de l'échantillon.
</details>

## Exercice 3 – Test t de Student (variances égales)

Le test t de Student suppose des variances égales. Il combine les deux variances dans une variance commune, la variance pondérée $s_p^2$ :

$$
t = \frac{\bar{x}_1 - \bar{x}_2}{s_p \sqrt{\dfrac{1}{n_1} + \dfrac{1}{n_2}}}, \qquad s_p^2 = \frac{(n_1 - 1)s_1^2 + (n_2 - 1)s_2^2}{n_1 + n_2 - 2}
$$

sous $H_0 : \mu_1 = \mu_2$, avec $n_1 + n_2 - 2$ degrés de liberté.

> **Contexte :** le test t de Student est le test de moyennes classique, très utilisé dans les rapports. Il ne doit être appliqué que si les variances sont comparables, ce que l'exercice 2 a déjà contesté. Cet exercice montre ce qui arrive quand on l'applique malgré tout.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `t.test()` | Test t de comparaison de deux moyennes. | `x`, `y`, `var.equal` | `t.test(ln_appart, ln_maison, var.equal = TRUE)` |

### Exemple

```r
t.test(ln_appart, ln_maison, var.equal = TRUE)
# t ≈ -6,83 ; df = 9897 ; p-value très petite
```

### Exercices

1. Lancer le test t de Student. Quelle est la décision ?
   > **Intérêt :** une conclusion qui dépend du test choisi doit être justifiée : c'est pourquoi on ne présente pas seulement le test le plus connu.
<details>
<summary>Correction</summary>

On rejette $H_0$ : les moyennes du logarithme de la consommation par m² diffèrent entre appartements et maisons.
</details>

2. Expliquer pourquoi ce test n'est pas le plus approprié, malgré le rejet, au vu de l'exercice 2.
   > **Intérêt :** montrer qu'un test peut donner un résultat correct pour de mauvaises raisons est un message clé pour un relecteur ou un auditeur.
<details>
<summary>Correction</summary>

Sa condition (variances égales) n'est pas remplie : la variance commune $s_p^2$ est dominée par la variance des appartements (9 570 observations), et la statistique n'a pas la loi de Student attendue. Le résultat peut donc être trompeur, même si la conclusion est la même ici.
</details>

## Exercice 4 – Test de Welch (variances inégales)

Le test de Welch ne suppose pas l'égalité des variances. L'erreur standard est calculée séparément pour chaque groupe :

$$
t = \frac{\bar{x}_1 - \bar{x}_2}{\sqrt{\dfrac{s_1^2}{n_1} + \dfrac{s_2^2}{n_2}}}
$$

avec des degrés de liberté ajustés. C'est le test par défaut de `t.test()`.

> **Contexte :** le test de Welch est celui à présenter par défaut quand on compare deux groupes indépendants. Il reste fiable même quand les variances et les tailles diffèrent, situation fréquente dans les données réelles, comme ici avec 9 570 appartements contre 329 maisons.

### Exemple

```r
t.test(ln_appart, ln_maison , var.equal = FALSE)
# t ≈ -5,62 ; df ajustés ; p-value ≈ 3.9 × 10^-8
# 95 percent confidence interval: -0,2143 -0,1035
```

### Exercices

1. Lancer le test de Welch. Quelle est la décision, et quel est l'intervalle de confiance de la différence $\mu_1 - \mu_2$ ?
   > **Intérêt :** l'intervalle de confiance de la différence dit l'ampleur de l'écart, ce qui est souvent la question posée par un décideur.
<details>
<summary>Correction</summary>

On rejette $H_0$. L'intervalle à 95 % de la différence $\mu_1 - \mu_2$ est $[-0{,}214 ; -0{,}104]$ : il ne contient pas 0, cohérent avec le rejet. La consommation par m² des appartements est plus basse que celle des maisons.
</details>

2. Comparer la statistique de Welch ($-5{,}62$) à celle de Student ($-6{,}83$). Pourquoi diffèrent-elles ?
   > **Intérêt :** comprendre pourquoi deux statistiques diffèrent évite de présenter un chiffre sans savoir ce qu'il mesure.
<details>
<summary>Correction</summary>

Les deux statistiques utilisent des erreurs standard différentes : Welch pondère chaque variance par sa propre taille d'échantillon. Ici, les tailles et les variances sont très inégales, donc les deux résultats s'écartent.
</details>

> **Note : Welch par défaut**
> Quand on hésite entre Student et Welch, on utilise Welch : il donne des résultats proches de Student quand les variances sont égales, et reste valable quand elles ne le sont pas.

## Exercice 5 – Test t sur données appariées

Les données sont **appariées** quand chaque observation d'un groupe correspond à une observation de l'autre : même logement, même personne, même mesure répétée. On teste alors la moyenne des différences $d_i = x_{1,i} - x_{2,i}$.

- $H_0 : \mu_d = 0$ ;
- $H_1 : \mu_d \ne 0$ ;
- $\alpha = 0{,}05$.

Ici, on compare la consommation de chauffage et la consommation d'eau chaude sanitaire (ECS) pour les mêmes 10 000 DPE.

> **Contexte :** dans les données de consommation, un même logement a plusieurs mesures : chauffage, eau chaude, avant et après travaux. Comparer deux groupes indépendants dans ce cas gâche de l'information et peut conduire à une conclusion fausse. Le test apparié compare chaque logement à lui-même.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `t.test(paired = TRUE)` | Test t apparié. | `x`, `y`, `paired = TRUE` | `t.test(chauffage, ecs, paired = TRUE)` |

### Exemple

```r
chauffage <- dpe$conso_chauffage_ep
ecs <- dpe$conso_ecs_ep

t.test(chauffage, ecs, paired = TRUE)
# moyenne des différences ≈ 5 878 kWh EP/an ; t ≈ 14,27 ; p-value < 2,2e-16
```

### Exercices

1. Poser $H_0$ et $H_1$, puis lancer le test apparié. Quelle est la décision ?
   > **Intérêt :** pour un même logement, le rejet indique que les deux consommations diffèrent : une information utile pour cibler des travaux.
<details>
<summary>Correction</summary>

On rejette $H_0$ : la consommation de chauffage est en moyenne plus élevée que celle de l'ECS pour un même logement.
</details>

2. Calculer la moyenne et l'écart type des différences à la main, puis retrouver la statistique $t$.
   > **Intérêt :** retrouver $t$ à la main permet de vérifier un résultat avant de le présenter.
<details>
<summary>Correction</summary>

```r
d <- chauffage - ecs
mean(d)                         # environ 5 878
t_manuel <- mean(d) / (sd(d) / sqrt(length(d)))
t_manuel                        # environ 14,27 : identique à t.test(paired = TRUE)
```
</details>

3. Expliquer pourquoi un test sur échantillons indépendants serait faux ici.
   > **Intérêt :** un test indépendant sur des données appariées peut faire disparaître un effet réel : c'est une erreur classique en évaluation de travaux.
<details>
<summary>Correction</summary>

Les deux consommations viennent des mêmes logements : elles sont liées. Un test indépendant ignore cette liaison, sous-estime la précision de la comparaison, et peut donner une p-value fausse. Le test apparié utilise la variabilité des différences, plus pertinente.
</details>

## Exercice 6 – Comparer deux proportions

On compare la part de passoires (étiquettes F ou G) entre appartements et maisons, sur deux échantillons indépendants.

- $H_0 : p_1 = p_2$ ;
- $H_1 : p_1 \ne p_2$ ;
- $\alpha = 0{,}05$.

> **Contexte :** comparer deux proportions sert à quantifier un écart entre deux groupes, par exemple la part de logements très peu performants selon le type de bâtiment. C'est une base pour cibler une politique de rénovation et justifier le ciblage devant un décideur.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `prop.test()` | Test de comparaison de proportions sur plusieurs groupes. | `x` (succès par groupe), `n` (effectifs), `alternative` | `prop.test(c(731, 89), c(9592, 329))` |

### Exemple

```r
passoires <- function(type) {
  sub <- dpe |> filter(type_batiment == type)
  sum(sub$etiquette_dpe %in% c("F", "G"))
}

k_app <- passoires("appartement")    # 731
k_mai <- passoires("maison")         # 89
n_app <- sum(dpe$type_batiment == "appartement")   # 9 592
n_mai <- sum(dpe$type_batiment == "maison")        # 329

prop.test(c(k_app, k_mai), c(n_app, n_mai))
# proportions : 7,6 % (appartements) et 27,1 % (maisons) ; p-value < 2,2e-16
```

### Exercices

1. Lancer le test. Quelle est la décision ?
   > **Intérêt :** un écart de plus de 19 points est une information directe pour un chargé de politique publique : le test confirme qu'il ne s'agit pas d'un effet du hasard.
<details>
<summary>Correction</summary>

On rejette $H_0$ : la part des passoires est nettement plus élevée chez les maisons (27 %) que chez les appartements (7,6 %).
</details>

2. Calculer l'intervalle de confiance à 95 % de la différence $p_1 - p_2$ avec `prop.test()`.
   > **Intérêt :** l'intervalle donne l'ampleur de l'écart, à présenter avec le résultat du test.
<details>
<summary>Correction</summary>

```r
prop.test(c(k_app, k_mai), c(n_app, n_mai))$conf.int
# L'intervalle est entièrement négatif (appartements moins touchés) et ne contient pas 0.
```
</details>

3. Le groupe « maison » ne compte que 329 DPE. Calculer la condition $n\hat{p} \ge 5$ pour ce groupe, puis commenter la fiabilité de l'approximation.
   > **Intérêt :** un petit groupe peut rendre l'approximation fragile : il faut vérifier les conditions et le dire explicitement.
<details>
<summary>Correction</summary>

$329 \times 0{,}27 \approx 89 \ge 5$ et $329 \times 0{,}73 \approx 240 \ge 5$ : la condition est remplie. L'approximation reste fiable, même si le groupe est petit.
</details>

## Exercice 7 – Taille d'échantillon pour deux proportions

Pour détecter un écart $p_1 - p_2 = \delta$ avec une puissance $1 - \beta$, la taille par groupe est :

$$
n = \left(\frac{z_{1-\alpha/2}\sqrt{2\bar{p}(1-\bar{p})} + z_{1-\beta}\sqrt{p_1(1-p_1) + p_2(1-p_2)}}{p_1 - p_2}\right)^2, \qquad \bar{p} = \frac{p_1 + p_2}{2}
$$

> **Contexte :** avant une enquête comparative, on dimensionne les deux groupes. Un petit groupe ne permet pas de détecter un écart réaliste ; ce constat doit être fait avant de dépenser le budget, pas après.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `pwr.2p.test()` (package `pwr`) | Puissance ou taille par groupe pour deux proportions. | `h`, `n`, `power`, `sig.level` | `pwr.2p.test(h = ES.h(0.27, 0.22), power = 0.80)` |

### Exemple

```r
pwr.2p.test(h = ES.h(0.27, 0.22), power = 0.80, sig.level = 0.05)
# n ≈ 1 160 par groupe
```

### Exercices

1. Calculer la taille par groupe pour détecter $p_1 = 0{,}27$ face à $p_2 = 0{,}22$, avec une puissance de 0,80. Comparer à la taille de la maison dans les données.
   > **Intérêt :** un groupe de 329 logements ne suffit pas pour détecter un écart de 5 points : c'est un argument pour élargir l'échantillon.
<details>
<summary>Correction</summary>

Il faut environ 1 160 DPE par groupe. Le groupe « maison » en compte 329 : il est trop petit pour détecter un écart de 5 points avec une puissance de 0,80.
</details>

2. Calculer la puissance avec 329 maisons contre 9 592 appartements, pour le même écart.
   > **Intérêt :** le déséquilibre des groupes diminue la puissance : une stratégie d'échantillonnage plus équilibrée peut être plus efficace.
<details>
<summary>Correction</summary>

```r
pwr.2p2n.test(h = ES.h(0.27, 0.22), n1 = 9592, n2 = 329, sig.level = 0.05)
# Puissance inférieure à 0,80 : le petit groupe « maison » limite la détection.
```
</details>

## Exercice 8 – Visualiser les comparaisons

Un boxplot par groupe montre l'écart de position et la dispersion, avant tout test.

> **Contexte :** le boxplot est le graphique à produire avant le test. Il montre si les groupes se distinguent, et il est lisible par un non-statisticien, ce qui en fait un support de dialogue avec les équipes métier.

### Exemple

```r
ggplot(dpe_2, aes(x = type_batiment, y = ln_conso_m2, fill = type_batiment)) +
  geom_boxplot(outlier.shape = NA) +
  guides(fill = "none") +
  labs(x = "Type de bâtiment", y = "ln(conso / m²)")
```

### Exercices

1. Reproduire le graphique. Les boîtes se recouvrent-elles ?
   > **Intérêt :** le graphique doit confirmer ou nuancer le test : si les boîtes se distinguent peu, un rejet mérite d'être présenté avec prudence.
<details>
<summary>Correction</summary>

Les boîtes se recouvrent en partie, mais la médiane des maisons est plus haute, cohérente avec la différence de moyennes du test de Welch.
</details>

2. Convertir le graphique avec `plotly` (`plot_ly(type = "box")`), puis survoler la médiane de chaque groupe.
   > **Intérêt :** le survol permet de vérifier les valeurs médianes avant de les citer.
<details>
<summary>Correction</summary>

```r
plot_ly(dpe_2, x = ~type_batiment, y = ~ln_conso_m2, type = "box",
        color = ~type_batiment) |>
  layout(showlegend = FALSE)
```
Le survol affiche la médiane et les quartiles de chaque type.
</details>

3. Représenter avec `ggplot2` la part de passoires (F ou G) entre maisons et appartements, avec un diagramme empilé en pourcentage (`position = "fill"`). Quelle différence visuelle observe-t-on entre les deux types ?
   > **Intérêt :** le diagramme empilé en pourcentage compare directement les proportions, indépendamment des effectifs très différents (9 570 appartements contre 329 maisons). C'est le graphique qui accompagne le test de comparaison de proportions (exercice 6).
<details>
<summary>Correction</summary>

```r
dpe_2 |>
  mutate(categorie = if_else(etiquette_dpe %in% c("F", "G"), "F ou G", "A à E")) |>
  ggplot(aes(x = type_batiment, fill = categorie)) +
  geom_bar(position = "fill") +
  scale_y_continuous(labels = scales::percent) +
  labs(x = "Type de bâtiment", y = "Part des DPE", fill = "Étiquette")
```
La part orange (F ou G) est nettement plus haute pour les maisons (environ 27 %) que pour les appartements (environ 8 %) : c'est l'écart que le test de l'exercice 6 confirme.
</details>

## Exercice 9 – Quel test choisir ?

Pour chacune des situations suivantes, indiquer le test à utiliser, puis la décision.

> **Contexte :** le choix du test est souvent la partie la moins bien justifiée dans un rapport. Cet exercice rappelle la démarche à suivre : nature des données, indépendance des groupes, égalité des variances, puis choix du test.

1. Comparer la consommation de chauffage et la consommation d'ECS sur les mêmes 10 000 DPE.
   > **Intérêt :** une mesure répétée sur un même logement n'est pas un groupe indépendant : c'est la première question à se poser.
<details>
<summary>Correction</summary>

Test t apparié (`paired = TRUE`) : les deux mesures viennent des mêmes logements.
</details>

2. Comparer le logarithme de la consommation par m² entre appartements et maisons, en vérifiant d'abord les variances.
   > **Intérêt :** l'enchaînement variance puis moyenne est la démarche à expliquer dans une méthode.
<details>
<summary>Correction</summary>

Test de Levene (ou Fisher), puis Welch, car les variances diffèrent (exercice 2).
</details>

3. Comparer la part de passoires entre deux types de bâtiment.
   > **Intérêt :** les proportions appellent un test différent : confondre les deux est une erreur fréquente.
<details>
<summary>Correction</summary>

`prop.test()` sur deux groupes indépendants (exercice 6).
</details>

4. Comparer la moyenne de la surface des DPE à la valeur 55 m² (un seul échantillon).
   > **Intérêt :** savoir distinguer un échantillon de deux échantillons évite de mal choisir le test.
<details>
<summary>Correction</summary>

Ce n'est pas un test à deux échantillons : il relève du chapitre 9 (test t à un échantillon).
</details>

## Exercice 10 – Électricité contre réseau de chauffage urbain

On compare le logarithme de la consommation par m² entre deux énergies de chauffage : l'électricité et le réseau de chauffage urbain (libellé `Réseau de Chauffage urbain` dans les données). Les deux groupes ont des effectifs très différents, et des dispersions différentes : c'est un bon exemple pour choisir le test.

> **Contexte :** un bureau d'études qui compare deux solutions de chauffage pour un parc doit savoir si l'écart de performance est réel, et quel test permet de le conclure sans supposer ce qui n'est pas vérifié. Le réseau de chauffage urbain est souvent présenté comme une solution performante : le test doit le confirmer ou l'infirmer sur les données.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `var.test()` | Test F de Fisher d'égalité de deux variances. | `x`, `y` | `var.test(elec, reseau)` |
| `car::leveneTest()` | Test de Levene, centré sur la médiane par défaut. | formule, `data` | `leveneTest(ln_conso_m2 ~ type_energie_n1, data = dpe_ener)` |
| `t.test()` | Test t de Student (`var.equal = TRUE`) ou de Welch (par défaut). | `var.equal` | `t.test(elec, reseau, var.equal = FALSE)` |

### Préparation

```r
dpe_ener <- dpe |>
  filter(surface_habitable_logement > 0, conso_5_usages_ep > 0,
         type_energie_n1 %in% c("Électricité", "Réseau de Chauffage urbain")) |>
  mutate(ln_conso_m2 = log(conso_5_usages_ep / surface_habitable_logement))

elec   <- dpe_ener$ln_conso_m2[dpe_ener$type_energie_n1 == "Électricité"]
reseau <- dpe_ener$ln_conso_m2[dpe_ener$type_energie_n1 == "Réseau de Chauffage urbain"]
```

### Exercices

1. Calculer l'effectif, la moyenne et l'écart-type de chaque groupe. Quelle énergie a la moyenne la plus élevée, et quel est l'écart sur le logarithme ?
   > **Intérêt :** avant tout test, les statistiques par groupe donnent l'ordre de grandeur de l'écart et la dispersion de chaque groupe.
<details>
<summary>Correction</summary>

| Énergie | Effectif | Moyenne | Écart-type |
|---------|----------|---------|------------|
| Électricité | 4 792 | 5,272 | 0,406 |
| Réseau de chauffage urbain | 647 | 5,218 | 0,335 |

L'électricité a la moyenne la plus élevée : l'écart est de 0,054 sur le logarithme, soit un rapport d'environ 1,055 entre les consommations par m² (environ 5 % de plus). Le réseau est le groupe le moins dispersé.
</details>

2. Tester l'égalité des variances avec le test F de Fisher et le test de Levene. Faut-il supposer des variances égales ?
   > **Intérêt :** le test de Student suppose des variances égales : on le vérifie avant de l'utiliser, comme pour les autres comparaisons de ce chapitre.
<details>
<summary>Correction</summary>

```r
var.test(elec, reseau)
# F = 1.4685, ddl = (4791 ; 646), p-value = 7.4e-10

car::leveneTest(ln_conso_m2 ~ type_energie_n1, data = dpe_ener)
# F = 24.0, p < 0,0001
```
Les deux tests rejettent l'égalité des variances. On n'utilise donc pas le test t de Student à variances égales, mais le test de Welch.
</details>

3. Comparer les moyennes avec le test de Student (variances égales) et le test de Welch. Quelle est la conclusion, et quel test retenez-vous ?
   > **Intérêt :** le test de Welch ajuste les degrés de liberté aux dispersions différentes : il est le choix sûr quand l'égalité des variances est rejetée.
<details>
<summary>Correction</summary>

```r
t.test(elec, reseau, var.equal = TRUE)    # Student : t = 3.22, ddl = 5 437, p = 0,0013
t.test(elec, reseau, var.equal = FALSE)   # Welch   : t = 3.72, ddl = 922,7, p = 0,0002
#  IC à 95 % de la différence : [0,025 ; 0,082]
```
Les deux tests concluent à une différence significative. Nous retenons Welch : l'égalité des variances est rejetée (question 2), et les ddl de Student (5 437) ne correspondent pas à la réalité des dispersions. L'intervalle de confiance de la différence ne contient pas 0.
</details>

4. Rédiger en trois phrases la conclusion pour un lecteur non statisticien : l'écart, le test utilisé, et sa limite.
   > **Intérêt :** une conclusion doit dire ce qui est établi, dans quel sens, et ce qui ne l'est pas.
<details>
<summary>Correction</summary>

Exemple de rédaction : « La consommation par m² est plus élevée en électricité qu'avec le réseau de chauffage urbain, avec un écart moyen d'environ 5 % (test de Welch, p = 0,0002). Les dispersions diffèrent fortement, ce qui impose ce test et réduit la précision de l'écart (intervalle de 2,5 % à 8,5 %). L'analyse ne tient pas compte du type de bâtiment ni de l'âge des logements : l'écart peut en partie venir de ces facteurs. »
</details>

## Liens utiles

- [Documentation de `t.test()` (R)](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/t.test.html)
- [Documentation de `prop.test()` (R)](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/prop.test.html)
- [Documentation du package `car` (test de Levene)](https://cran.r-project.org/package=car)
- [Documentation du package `pwr`](https://cran.r-project.org/package=pwr)
