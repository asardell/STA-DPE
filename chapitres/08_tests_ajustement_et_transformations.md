# Chapitre 8 – Démarche d'un test, tests d'ajustement et transformations

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] formuler une hypothèse nulle $H_0$ et une hypothèse alternative $H_1$
- [ ] définir les risques $\alpha$ et $\beta$, la puissance $1 - \beta$ et la p-value
- [ ] lire une décision de test et éviter les conclusions abusives
- [ ] tester l'adéquation d'une variable à une loi normale avec le test de Shapiro-Wilk et le test d'Anderson-Darling
- [ ] transformer une variable avec la transformation de Box-Cox ou une transformation de Johnson
- [ ] évaluer la robustesse d'un test quand ses conditions ne sont pas remplies

> **Contexte :** avant de comparer des groupes ou d'appliquer un test de moyenne, il faut vérifier que les données s'y prêtent. Dans un service de données, une part importante des erreurs d'interprétation vient d'un test appliqué sans cette vérification. Ce chapitre donne la démarche d'un test et les outils pour contrôler la normalité, qui conditionnent la plupart des chapitres suivants.

## Sommaire

- [Chapitre 8 – Démarche d'un test, tests d'ajustement et transformations](#chapitre-8--démarche-dun-test-tests-dajustement-et-transformations)
  - [Objectifs](#objectifs)
  - [Sommaire](#sommaire)
  - [Préparation : packages, données et notations](#préparation--packages-données-et-notations)
    - [Notations](#notations)
    - [La démarche d'un test](#la-démarche-dun-test)
  - [Exercice 1 – Les risques α et β par simulation](#exercice-1--les-risques-α-et-β-par-simulation)
    - [Mémo](#mémo)
    - [Exemple](#exemple)
    - [Exercices](#exercices)
  - [Exercice 2 – Test de Shapiro-Wilk](#exercice-2--test-de-shapiro-wilk)
    - [Mémo](#mémo-1)
    - [Exemple](#exemple-1)
    - [Exercices](#exercices-1)
  - [Exercice 3 – Test d'Anderson-Darling](#exercice-3--test-danderson-darling)
    - [Mémo](#mémo-2)
    - [Exemple](#exemple-2)
    - [Exercices](#exercices-2)
  - [Exercice 4 – La taille de l'échantillon change la décision](#exercice-4--la-taille-de-léchantillon-change-la-décision)
    - [Exemple](#exemple-3)
    - [Exercices](#exercices-3)
  - [Exercice 5 – Transformation de Box-Cox](#exercice-5--transformation-de-box-cox)
    - [Mémo](#mémo-3)
    - [Exemple](#exemple-4)
    - [Exercices](#exercices-4)
  - [Exercice 6 – Transformation de Johnson](#exercice-6--transformation-de-johnson)
    - [Mémo](#mémo-4)
    - [Exemple](#exemple-5)
    - [Exercices](#exercices-5)
  - [Exercice 7 – Choisir la transformation avec `bestNormalize`](#exercice-7--choisir-la-transformation-avec-bestnormalize)
    - [Mémo](#mémo-5)
    - [Exemple](#exemple-6)
      - [Les méthodes comparées par `bestNormalize`](#les-méthodes-comparées-par-bestnormalize)
    - [Exercices](#exercices-6)
  - [Exercice 8 – Robustesse d'un test](#exercice-8--robustesse-dun-test)
    - [Exemple](#exemple-7)
    - [Exercices](#exercices-7)
  - [Exercice 9 – Visualiser les quantiles avec `plotly`](#exercice-9--visualiser-les-quantiles-avec-plotly)
    - [Exemple](#exemple-8)
    - [Exercices](#exercices-8)
  - [Liens utiles](#liens-utiles)

## Préparation : packages, données et notations

```r
install.packages(c("nortest", "MASS", "bestNormalize", "plotly"))
library(readr)
library(dplyr)
library(ggplot2)
library(plotly)

dpe <- read_csv("data/dpe_villeurbanne_10000.csv")
surface_ok <- dpe$surface_habitable_logement[!is.na(dpe$surface_habitable_logement)]
```

`MASS`, `nortest` et `bestNormalize` sont appelés avec `::` dans ce chapitre, pour indiquer d'où vient chaque fonction.

### Notations

| Notion | Notation | Signification |
|--------|----------|---------------|
| Hypothèse nulle | $H_0$ | ce qu'on suppose vrai par défaut |
| Hypothèse alternative | $H_1$ | ce qu'on cherche à établir |
| Risque de première espèce | $\alpha = P(\text{rejeter } H_0 \mid H_0 \text{ vraie})$ | fausse alerte |
| Risque de seconde espèce | $\beta = P(\text{ne pas rejeter } H_0 \mid H_1 \text{ vraie})$ | fausse sécurité |
| Puissance | $1 - \beta$ | probabilité de détecter un écart réel |
| p-value | $p = P(\text{statistique du test au moins aussi extrême} \mid H_0)$ | probabilité, sous $H_0$, d'un résultat au moins aussi éloigné que celui observé |

### La démarche d'un test

Tout test suit les mêmes étapes :

1. **Poser les hypothèses** $H_0$ et $H_1$, en précisant le caractère bilatéral ou unilatéral.
2. **Fixer le seuil** $\alpha$, avant de regarder les données (en général 5 %).
3. **Calculer la statistique du test** et sa p-value.
4. **Décider** : on rejette $H_0$ si $p < \alpha$, sinon on ne la rejette pas.
5. **Conclure** dans le contexte : « il n'y a pas de preuve que… » ou « les données sont incompatibles avec… ».

| Décision | $H_0$ vraie | $H_1$ vraie |
|----------|-------------|-------------|
| Ne pas rejeter $H_0$ | décision correcte, probabilité $1 - \alpha$ | **erreur de seconde espèce** (β) |
| Rejeter $H_0$ | **erreur de première espèce** (α) | décision correcte, probabilité $1 - \beta$ (puissance) |

> **Note : ne pas rejeter n'est pas accepter**
> Un test qui ne rejette pas $H_0$ n'établit pas que $H_0$ est vraie : il dit seulement que les données ne fournissent pas assez de preuves contre elle. Plus l'échantillon est petit, plus β est grand, et plus il est facile de « ne pas rejeter » à tort.

> **Note : la p-value n'est pas la probabilité que $H_0$ soit vraie**
> La p-value répond à « si $H_0$ était vraie, quelle serait la probabilité d'observer un résultat aussi éloigné ? ». Elle ne donne pas la probabilité que $H_0$ soit vraie.

Dans ce chapitre, les tests vérifient une **condition** : la normalité des données. Cette condition sert ensuite aux tests des chapitres 9 à 14. Les hypothèses sont donc inversées par rapport à un test de comparaison : on veut que $H_0$ ne soit pas rejetée, ce qui justifie l'usage de la loi normale.

## Exercice 1 – Les risques α et β par simulation

Le risque α est la fréquence de rejets quand $H_0$ est vraie. On le vérifie sur des données simulées, où $H_0$ est vraie par construction.

> **Contexte :** un test est une décision prise avec un risque. Le risque α (fausse alerte) est fixé par l'équipe ; β (fausse sécurité) dépend de la taille de l'étude. Avant une décision à fort enjeu, comme le déclenchement de travaux, connaître ces deux risques évite de sur-interpréter un résultat.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `t.test()` | Test t à un échantillon, avec la p-value. | `x`, `mu`, `alternative` | `t.test(x, mu = 50)$p.value` |
| `replicate()` | Répète une simulation. | `n`, `expr` | `replicate(2000, t.test(rnorm(20))$p.value)` |

### Exemple

```r
set.seed(42)

# H0 vraie : la moyenne est 50
p_h0 <- replicate(2000, t.test(rnorm(20, mean = 50, sd = 10), mu = 50)$p.value)

# Fréquence de rejet de H0 : proche de alpha = 0,05
mean(p_h0 < 0.05)
```

### Exercices

1. Exécuter la simulation ci-dessus. La fréquence de rejet est-elle proche de 0,05 ?
   > **Intérêt :** montrer que la fréquence de fausses alertes vaut α rend concrète la notion de risque que le décideur doit accepter.
<details>
<summary>Correction</summary>

```r
mean(p_h0 < 0.05)
# Une valeur voisine de 0,05 : environ 5 % des tests rejettent H0 alors qu'elle est vraie.
```
C'est la définition de α : la proportion de fausses alertes sur un grand nombre d'échantillons.
</details>

2. Simuler maintenant $H_1$ vraie : les données suivent $\mathcal{N}(52, 10^2)$, et on teste $H_0 : \mu = 50$ avec $n = 20$. Calculer la proportion de tests qui **ne rejettent pas** $H_0$. Quel est le risque β correspondant ?
   > **Intérêt :** β dépend de la taille de l'étude : un résultat négatif sur un petit échantillon ne doit pas être présenté comme une absence d'effet.
<details>
<summary>Correction</summary>

```r
set.seed(42)
p_h1 <- replicate(2000, t.test(rnorm(20, mean = 52, sd = 10), mu = 50)$p.value)

beta <- mean(p_h1 >= 0.05)
beta            # proportion de tests qui ne rejettent pas H0 à tort
1 - beta        # puissance : proportion de rejets corrects
```
β est la proportion de tests qui ne détectent pas l'écart réel de 2 unités. La puissance, $1 - \beta$, est la probabilité de le détecter. Elle est souvent faible avec $n = 20$ : le chapitre 9 montre comment la calculer et choisir $n$.
</details>

3. Répéter la question 2 avec $n = 200$. Que deviennent β et la puissance ?
   > **Intérêt :** augmenter n fait baisser β sans changer α : c'est l'argument pour dimensionner l'étude avant de la lancer.
<details>
<summary>Correction</summary>

```r
set.seed(42)
p_h1_200 <- replicate(2000, t.test(rnorm(200, mean = 52, sd = 10), mu = 50)$p.value)
mean(p_h1_200 >= 0.05)   # β plus petit
1 - mean(p_h1_200 >= 0.05)  # puissance plus grande
```
Avec un échantillon plus grand, β diminue et la puissance augmente, pour le même écart réel. Le risque α, lui, ne change pas : il est fixé à 0,05 par le choix du seuil.
</details>

> **Note : α se fixe, β se calcule**
> On choisit α (par convention 5 %). β dépend de l'écart réel, de la variabilité et de la taille de l'échantillon : on ne peut le calculer qu'en supposant une valeur pour l'écart. C'est pourquoi la planification d'une étude part de la puissance souhaitée (chapitre 9).

## Exercice 2 – Test de Shapiro-Wilk

Le test de Shapiro-Wilk vérifie si un échantillon suit une loi normale.

- $H_0$ : les données suivent une loi normale ;
- $H_1$ : les données ne suivent pas une loi normale ;
- $\alpha = 0{,}05$.

Si $p < 0{,}05$, on rejette la normalité. Le test accepte entre 3 et **5 000** observations : au-delà, il faut prendre un sous-échantillon.

> **Contexte :** les tests de normalité conditionnent le choix du test de moyennes. Dans un rapport, cette vérification se présente comme une étape documentée, qui justifie la méthode retenue.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `shapiro.test()` | Test de Shapiro-Wilk. Taille entre 3 et 5 000. | `x` | `shapiro.test(x)$p.value` |
| `head()` | Garde les premières valeurs, pour réduire la taille. | `n` | `head(surface_ok, 5000)` |

### Exemple

```r
surface_5000 <- head(surface_ok, 5000)

shapiro.test(surface_5000)

# Même test sur le logarithme de la surface
shapiro.test(log(surface_5000))
```

> **Note : `p-value < 2.2e-16`**
> Quand R affiche `p-value < 2.2e-16`, cela signifie que la p-value est plus petite que $2{,}2 \times 10^{-16}$, la plus petite valeur que R sait représenter. Ce n'est pas la valeur exacte : R ne peut pas la donner. Il ne faut donc pas lire « 0,00…00022 » ni écrire « p = 0 » dans un rapport, mais « $p < 2{,}2 \times 10^{-16}$ ». Pour la décision, cela ne change rien : une p-value aussi petite est inférieure à $\alpha = 0{,}05$, on rejette $H_0$.

### Exercices

1. Lancer `shapiro.test()` sur les 5 000 premières surfaces. Quelle est la décision au seuil de 5 % ?
   > **Intérêt :** un rejet confirme ce que montrent les histogrammes : la surface n'est pas normale, donc les tests de moyenne doivent être justifiés.
<details>
<summary>Correction</summary>

```r
shapiro.test(surface_5000)
# Attendu : p-value très petite (inférieure à 0,001) : H0 est rejetée.
# La surface n'est pas normale, ce qui confirme le constat du chapitre 6.
```
</details>

2. Lancer le test sur le logarithme de la surface. La décision change-t-elle ?
   > **Intérêt :** montrer que le logarithme ne suffit pas à passer le test aide à décider entre transformer la variable et accepter une approximation.
<details>
<summary>Correction</summary>

```r
shapiro.test(log(surface_5000))
# La p-value reste très petite : même le logarithme n'est pas exactement normal.
```
Avec 5 000 observations, le test détecte des écarts très faibles à la normalité. Une décision « rejet » ne dit pas si l'écart est gênant : il faut aussi regarder le graphique (QQ-plot, chapitre 6).
</details>

3. Lancer `shapiro.test()` sur un échantillon de 6 000 observations. Que se passe-t-il ?
   > **Intérêt :** la limite de 5 000 observations est une contrainte d'outil que l'équipe doit connaître, pour éviter une erreur de script en production.
<details>
<summary>Correction</summary>

```r
shapiro.test(surface_ok)
# Erreur : sample size must be between 3 and 5000
```
La limite de 5 000 est imposée par la fonction. Il faut prendre un sous-échantillon (`head()`, ou `sample()` avec une graine fixée).
</details>

4. Rédiger la conclusion complète pour la surface, en citant H0, H1, α, la p-value et la conclusion.
   > **Intérêt :** cette conclusion est le modèle à reprendre dans un rapport : hypothèses, seuil, statistique, décision.
<details>
<summary>Correction</summary>

« Au seuil $\alpha = 0{,}05$, on rejette l'hypothèse de normalité de la surface habitable (statistique $W$ et p-value lues dans la sortie de `shapiro.test()` sur les 5 000 DPE). La distribution est asymétrique : les tests paramétriques sur la moyenne ne sont pas directement applicables sans transformation ou sans grand échantillon. »
</details>

5. Simuler un échantillon normal de taille 50 : `set.seed(42)` puis `x <- rnorm(50, mean = 50, sd = 10)`. Appliquer `shapiro.test()` et noter la p-value. Que peut-on conclure de cette p-value, et que ne peut-on pas conclure ?
   > **Intérêt :** une p-value élevée ne prouve pas la normalité : c'est le point à expliquer à un lecteur qui y voit une validation.
<details>
<summary>Correction</summary>

```r
set.seed(42)
x <- rnorm(50, mean = 50, sd = 10)
shapiro.test(x)$p.value
```

Conclusion : au seuil de 5 %, si $p > 0{,}05$, on **ne rejette pas** $H_0$. Les données ne contredisent pas la normalité. En revanche, on ne peut pas conclure que les données **sont** normales : avec 50 observations, un test a une puissance limitée, et une p-value élevée reste compatible avec un écart modéré (β). Une p-value proche de 1 n'est pas une preuve de normalité, pas plus qu'une p-value proche de 0 n'est une preuve d'anormalité ; il faut la lire avec le QQ-plot.
</details>

## Exercice 3 – Test d'Anderson-Darling

Le test d'Anderson-Darling compare la fonction de répartition empirique à celle de la loi normale, en donnant plus de poids aux valeurs extrèmes. Il n'a pas de limite de taille d'échantillon.

> **Contexte :** Anderson-Darling pèse davantage les extrémités de la distribution. Il est utile quand les valeurs très élevées comptent, comme les consommations exceptionnelles qui coûtent cher à traiter.

Si $x_{(1)} \le \dots \le x_{(n)}$ sont les observations triées, et $F$ la fonction de répartition de la loi normale ajustée (moyenne et écart type estimés) :

$$
A^2 = -n - \frac{1}{n} \sum_{i=1}^{n} (2i - 1) \left[ \ln F(x_{(i)}) + \ln \left(1 - F(x_{(n+1-i)})\right) \right]
$$

Plus $A^2$ est grand, plus l'écart à la loi normale est important. La p-value est calculée à partir de $A^2$.

Hypothèses : $H_0$ : les données suivent une loi normale ; $H_1$ : elles ne la suivent pas ; $\alpha = 0{,}05$.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `ad.test()` (package `nortest`) | Test d'Anderson-Darling de normalité. | `x` | `nortest::ad.test(x)` |

### Exemple

```r
nortest::ad.test(surface_ok)
# A = 1096 ; p-value < 2.2e-16
```

### Exercices

1. Lancer `nortest::ad.test()` sur `surface_ok`, avec les 9 899 surfaces. Quelle est la décision ?
   > **Intérêt :** un test sans limite de taille est utile sur un fichier complet de 9 899 DPE, où Shapiro-Wilk ne s'applique pas directement.
<details>
<summary>Correction</summary>

```r
nortest::ad.test(surface_ok)
# A ≈ 1096, p-value pratiquement nulle : H0 est rejetée.
```
</details>

2. Lancer le test sur le logarithme de la surface. La statistique diminue-t-elle ? La décision change-t-elle ?
   > **Intérêt :** la transformation réduit la statistique mais ne rend pas la distribution normale : la décision reste un rejet, et il faut le dire.
<details>
<summary>Correction</summary>

```r
nortest::ad.test(log(surface_ok))
# A ≈ 165 : la statistique est bien plus petite, mais la p-value reste pratiquement nulle.
```
La transformation réduit nettement l'écart à la normalité, sans le supprimer : avec presque 10 000 observations, le test est très sensible.
</details>

3. Expliquer pourquoi la statistique $A^2$ augmente avec les valeurs extrêmes, en lisant la formule : que représente le terme $\ln(1 - F(x_{(n+1-i)}))$ pour la plus grande valeur ?
   > **Intérêt :** comprendre pourquoi les valeurs extrêmes pèsent dans $A^2$ aide à expliquer un rejet dû à quelques dossiers atypiques.
<details>
<summary>Correction</summary>

Pour la plus grande valeur ($i = 1$), $F(x_{(n)})$ est proche de 1 si la valeur est très haute, donc $1 - F(x_{(n)})$ est très petit et son logarithme très négatif. Le poids $(2i - 1)$ est faible pour $i = 1$, mais les queues restent pénalisées. Une valeur extrême, comme les 3 330 m², fait donc fortement augmenter $A^2$.
</details>

4. Refaire le test d'Anderson-Darling sur les 30 premières surfaces seulement : `nortest::ad.test(surface_ok[1:30])`. Comparer la p-value à celle obtenue sur les 9 899 surfaces (question 1). Que constate-t-on ?
   > **Intérêt :** c'est l'effet de la taille de l'échantillon, abordé à l'exercice 4 : la même variable, qui n'est pas normale sur l'ensemble du fichier, n'est pas rejetée sur 30 observations. Sur un petit échantillon, le test manque l'écart réel.
<details>
<summary>Correction</summary>

```r
nortest::ad.test(surface_ok[1:30])
# A ≈ 0,54 ; p-value ≈ 0,155 : on ne rejette pas H0.
```
Sur 9 899 surfaces, la p-value est pratiquement nulle et H0 est rejetée (question 1). Sur les 30 premières, la p-value est de 0,155 : la même variable paraît normale. Ce n'est pas que la surface est devenue normale, mais que 30 observations ne suffisent pas à détecter son asymétrie.
</details>

5. Reprendre l'échantillon simulé à la question 5 de l'exercice 2 (`x`, de taille 50, normal par construction). Lancer `nortest::ad.test(x)` et comparer la p-value à celle de `shapiro.test(x)`. Les deux tests prennent-ils la même décision ?
   > **Intérêt :** deux tests qui concordent renforcent la conclusion : un rapport gagne en crédibilité en croisant les méthodes.
<details>
<summary>Correction</summary>

```r
set.seed(42)
x <- rnorm(50, mean = 50, sd = 10)

shapiro.test(x)$p.value
nortest::ad.test(x)$p.value
```
Les deux p-values sont en général différentes, car les deux tests ne mesurent pas le même écart. Sur des données normales, les deux décisions sont cependant identiques : on ne rejette pas $H_0$ au seuil de 5 % si les deux p-values sont supérieures à 0,05. Si l'une des deux était inférieure à 0,05 alors que l'autre ne l'est pas, ce serait un signe de faux positif ou d'un écart dans les queues : il faudrait revenir au QQ-plot.
</details>

> **Note : Shapiro-Wilk et Anderson-Darling**
> Les deux tests ont la même hypothèse, mais des sensibilités différentes : Shapiro-Wilk est plus puissant pour détecter une asymétrie modérée, Anderson-Darling pour détecter un écart dans les extrémités de la distribution (les valeurs très petites et très grandes). Par exemple, si tes données contiennent davantage de valeurs extrêmes que ce qu'une loi normale prévoit, Anderson-Darling est particulièrement sensible à cet écart.

## Exercice 4 – La taille de l'échantillon change la décision

Le même écart à la normalité peut donner un rejet avec un grand échantillon et une non-détection avec un petit. C'est le risque β qui augmente quand $n$ diminue.

> **Contexte :** un même écart à la normalité peut être détecté ou non selon la taille du fichier. Il faut donc interpréter un test de normalité en fonction de l'effectif, avant de conclure à une difficulté réelle.

### Exemple

```r
set.seed(42)
asymetrique <- rexp(30, rate = 1)     # loi exponentielle : fortement asymétrique
hist(asymetrique)
shapiro.test(asymetrique)$p.value
```

### Exercices

1. Répéter le test de Shapiro-Wilk 1000 fois sur des échantillons de taille 30 issus de la loi exponentielle. Quelle proportion de tests rejette la normalité ?
   > **Intérêt :** avec 30 observations, un test manque souvent un écart réel : un rapport doit le signaler.
<details>
<summary>Correction</summary>

```r
set.seed(42)
p_30 <- replicate(1000, shapiro.test(rexp(30, rate = 1))$p.value)
mean(p_30 < 0.05)
# Une part non négligeable : avec n = 30, le test ne détecte pas toujours l'asymétrie.
```
La part exacte dépend de la simulation. Le point important : avec 30 observations, une part non négligeable de tests **ne détecte pas** une loi pourtant très asymétrique (β).
</details>

2. Refaire la question 1 avec des échantillons de taille 1000. Que constate-t-on ?
   > **Intérêt :** avec 1 000 observations, presque tout rejette : un rejet ne signifie pas que l'écart est gênant pour la suite de l'analyse.
<details>
<summary>Correction</summary>

```r
set.seed(42)
p_1000 <- replicate(200, shapiro.test(rexp(1000, rate = 1))$p.value)
mean(p_1000 < 0.05)
# Avec n = 1000, presque tous les tests rejettent la normalité.
```
Le test détecte presque toujours l'écart avec un grand échantillon. Le résultat du test dépend donc autant de la taille que de la forme de la distribution.
</details>

3. Un collègue dit : « Shapiro-Wilk ne rejette pas la normalité sur mes 20 observations, donc mes données sont normales. » Que répondre ?
   > **Intérêt :** un test non significatif sur peu de données ne doit pas être présenté comme une validation de la méthode.
<details>
<summary>Correction</summary>

Ne pas rejeter $H_0$ n'est pas accepter $H_0$. Avec 20 observations, β est grand : le test n'a pas la puissance de détecter un écart modéré. La bonne conclusion : « les données ne contredisent pas la normalité, mais l'échantillon est trop petit pour la confirmer ». Le QQ-plot est alors un complément utile.
</details>

## Exercice 5 – Transformation de Box-Cox

Quand une variable est positive et asymétrique, on peut chercher une puissance $\lambda$ qui la rend plus proche d'une loi normale. La transformation de Box-Cox s'écrit :

$$
y(\lambda) = \left\lbrace
\begin{array}{ll}
\dfrac{x^{\lambda} - 1}{\lambda} & \text{si } \lambda \neq 0 \\
\ln(x) & \text{si } \lambda = 0
\end{array}
\right.
$$

Le paramètre $\lambda$ est choisi pour maximiser la vraisemblance. Quand $\lambda = 0$, la transformation est le logarithme.

![](https://feaz-book.com/numeric-boxcox_files/figure-html/fig-boxcox-normal-1.png)

> **Contexte :** transformer une variable permet d'utiliser des méthodes adaptées à la loi normale. En revanche, la transformation change l'unité de lecture : un résultat sur une variable transformée doit être reconverti ou expliqué au lecteur.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `MASS::boxcox()` | Trace la vraisemblance en fonction de $\lambda$ et renvoie le graphique. | formule, `lambda` | `MASS::boxcox(lm(x ~ 1), lambda = seq(-1, 1, 0.02))` |

### Exemple

```r
modele <- lm(surface_ok ~ 1)
bc <- MASS::boxcox(modele, lambda = seq(-0.5, 0.5, by = 0.02))

lambda_hat <- bc$x[which.max(bc$y)]
lambda_hat
# Environ 0,18 : proche de 0, donc du logarithme
```

> **Note : pourquoi `lm(surface_ok ~ 1)` et non le vecteur ?**
> `boxcox()` du package `MASS` est conçu pour un modèle ajusté (un objet `lm`), car il calcule la vraisemblance du modèle pour chaque valeur de $\lambda$ à partir de ses résidus. Un simple vecteur de données n'a pas de résidus. Le modèle `lm(surface_ok ~ 1)` ne contient que la moyenne : c'est le modèle « la variable est une constante plus du bruit », qui correspond exactement à l'hypothèse de normalité de la variable. On obtient ainsi le même $\lambda$ que pour une variable seule.

### Exercices

1. Calculer le $\lambda$ qui maximise la vraisemblance, puis le comparer à 0 (logarithme) et à 1 (pas de transformation).
   > **Intérêt :** estimer λ montre comment la transformation s'adapte aux données, sans choix arbitraire de puissance.
<details>
<summary>Correction</summary>

```r
lambda_hat  # environ 0,18
```
$\lambda$ est plus proche de 0 que de 1 : une transformation proche du logarithme est justifiée. La valeur exacte dépend de la grille utilisée.
</details>

2. Appliquer la transformation avec $\lambda$ estimé et tester la normalité du résultat avec `nortest::ad.test()`.
   > **Intérêt :** le test après transformation vérifie que la transformation a bien fonctionné : c'est un contrôle à faire et à documenter.
<details>
<summary>Correction</summary>

```r
x_bc <- (surface_ok^lambda_hat - 1) / lambda_hat
nortest::ad.test(x_bc)
# La statistique diminue, mais la p-value reste très petite sur 9 899 observations.
```
Comme à l'exercice 3, la transformation améliore la forme sans la rendre parfaitement normale.
</details>

3. Comparer la statistique $A^2$ obtenue avec $\lambda = 0$ (le logarithme) et avec $\lambda$ estimé. Lequel est le plus proche de la normalité ?
   > **Intérêt :** comparer λ = 0 (logarithme) et λ estimé permet de choisir la transformation la plus simple à expliquer à un décideur.
<details>
<summary>Correction</summary>

```r
nortest::ad.test(log(surface_ok))$statistic
nortest::ad.test(x_bc)$statistic
# Les deux valeurs sont proches : λ = 0,18 et λ = 0 donnent des formes voisines.
```
La transformation optimale n'est pas toujours la plus simple à interpréter. Le logarithme, plus lisible, est souvent préféré quand il donne un résultat voisin.
</details>

> **Note : interpréter une variable transformée**
> Après transformation, les résultats (moyennes, intervalles) portent sur $y(\lambda)$, pas sur la surface. Pour revenir à l'échelle d'origine, on inverse la transformation, en précisant qu'il s'agit alors d'une médiane ou d'une moyenne géométrique.

## Exercice 6 – Transformation de Johnson

Les transformations de Johnson rendent une variable plus proche d'une loi normale, y compris quand elle contient des valeurs négatives, cas où Box-Cox ne s'applique pas. On utilise ici la transformation **Yeo-Johnson**, qui appartient à la même famille et est disponible dans `bestNormalize` :

$$
\psi(x, \lambda) = \left\lbrace
\begin{array}{ll}
\dfrac{(x+1)^{\lambda} - 1}{\lambda} & \text{si } x \ge 0,\ \lambda \neq 0 \\
\ln(x+1) & \text{si } x \ge 0,\ \lambda = 0 \\
-\dfrac{(1-x)^{2-\lambda} - 1}{2-\lambda} & \text{si } x < 0,\ \lambda \neq 2 \\
-\ln(1-x) & \text{si } x < 0,\ \lambda = 2
\end{array}
\right.
$$

Pour une variable positive, seules les deux premières lignes s'appliquent. Le paramètre $\lambda$ est estimé par maximum de vraisemblance, comme pour Box-Cox.

![](https://feaz-book.com/numeric-yeojohnson_files/figure-html/fig-yeojohnson-normal-1.png)

> **Contexte :** les transformations de Johnson ouvrent les données comportant des valeurs négatives, fréquentes dans les écarts ou les bilans (par exemple une variation de consommation entre deux années). Box-Cox ne s'y applique pas.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `bestNormalize::yeojohnson()` | Estime $\lambda$ et transforme la variable. | `x` | `bestNormalize::yeojohnson(x)` |
| `$lambda` | Valeur estimée de $\lambda$. | | `yj$lambda` |
| `$x.t` | Valeurs transformées. | | `yj$x.t` |

### Exemple

```r
yj <- bestNormalize::yeojohnson(surface_ok)

# Valeur estimée de lambda
yj$lambda

# Valeurs transformées
x_yj <- yj$x.t

# Des valeurs négatives sont acceptées
bestNormalize::yeojohnson(c(-2, -1, 0, 1, 2))$x.t
```

### Exercices

1. Estimer le $\lambda$ de Yeo-Johnson sur `surface_ok`. Pourquoi ne peut-on pas le comparer directement au $\lambda$ de Box-Cox obtenu à l'exercice 5 ?
   > **Intérêt :** une valeur de λ n'a de sens que dans la paramétrisation choisie : ne pas comparer deux λ issus de méthodes différentes.
<details>
<summary>Correction</summary>

```r
yj <- bestNormalize::yeojohnson(surface_ok)
yj$lambda
```
Yeo-Johnson transforme $x + 1$ au lieu de $x$ : la valeur de $\lambda$ n'a pas la même signification que pour Box-Cox. Les deux valeurs ne se comparent donc pas directement ; on compare plutôt les tests de normalité après transformation.
</details>

2. Tracer l'histogramme de `x_yj` avec `ggplot2`, puis le comparer à celui de la surface brute (chapitre 4).
   > **Intérêt :** un histogramme plus symétrique après transformation montre l'effet de la méthode à un non-statisticien.
<details>
<summary>Correction</summary>

```r
ggplot(data.frame(x = x_yj), aes(x = x)) +
  geom_histogram(bins = 40, fill = "steelblue", colour = "white")
# La distribution transformée est plus symétrique que la surface brute,
# qui est fortement étalée vers la droite.
```
</details>

3. Essayer de calculer la puissance $(-2)^{0{,}5}$ en R. Puis appliquer `bestNormalize::yeojohnson()` à `c(-2, -1, 0, 1, 2)`. Que constatez-vous, et pourquoi Box-Cox ne s'applique-t-il pas aux valeurs négatives ?
   > **Intérêt :** savoir que Box-Cox exige des valeurs positives évite une erreur de calcul sur des données de bilan contenant des négatifs.
<details>
<summary>Correction</summary>

```r
(-2)^0.5
# [1] NaN  (avec un avertissement)

bestNormalize::yeojohnson(c(-2, -1, 0, 1, 2))$x.t
# Toutes les valeurs sont transformées, sans NaN.
```
Box-Cox utilise $x^{\lambda}$ : pour une valeur négative et une puissance non entière, le résultat n'est pas réel. Yeo-Johnson utilise une branche différente pour $x < 0$ et ne produit pas de NaN.
</details>

> **Note : la famille de Johnson au sens strict**
> La famille de Johnson (SU, SB, SL) est un système plus général que Yeo-Johnson : ses paramètres de position et d'échelle s'ajustent aux données, y compris pour des distributions bornées. Elle n'est pas implémentée dans ce chapitre. Pour l'utiliser, il faut un package dédié, comme `JohnsonDistribution`.

> **Note : Box-Cox ou Johnson ?**
> Box-Cox est plus simple et suffit pour une variable positive. Yeo-Johnson (ou Johnson) sert quand les données contiennent des valeurs négatives, ou quand Box-Cox ne normalise pas assez. Dans les deux cas, on vérifie le résultat par un test et par un graphique.

## Exercice 7 – Choisir la transformation avec `bestNormalize`

`bestNormalize` compare plusieurs transformations, dont Yeo-Johnson, et retient la meilleure selon un critère de normalité.

> **Contexte :** quand plusieurs transformations sont possibles, un package peut comparer leurs scores de normalité. Le choix final reste une décision de l'analyste, à documenter dans la méthode.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `bestNormalize::bestNormalize()` | Compare plusieurs transformations (dont Box-Cox, Yeo-Johnson, `orderNorm`) et retient celle au score le plus bas. | `x` | `bestNormalize::bestNormalize(x)` |
| `$chosen_transform` | Transformation retenue. | | `bn$chosen_transform` |

### Exemple

```r
bn <- bestNormalize::bestNormalize(surface_ok)

# Transformation retenue
bn$chosen_transform

# Valeurs transformées
x_transforme <- bn$x.t
```

#### Les méthodes comparées par `bestNormalize`

`bestNormalize` applique chaque transformation de la liste, mesure la normalité de la variable transformée, puis retient celle qui a le **score le plus petit** : plus le score est petit, plus la variable transformée est proche de la loi normale. Les scores ci-dessous sont ceux obtenus pour la variable $x$ de l'exemple.

| Méthode | Principe | Avantages | Inconvénients | Score |
|---------|----------|-----------|---------------|-------|
| `orderNorm` (ORQ) | Remplace chaque valeur par le quantile normal de son rang | Rend presque toujours la variable normale ; aucune hypothèse de forme | Transformation non paramétrique : pas de formule simple à interpréter ni à inverser ; peut surapprendre sur un petit échantillon | **1,264** (retenue) |
| `sqrt(x + a)` | Racine carrée après décalage $a$ | Simple à lire ; adaptée à une asymétrie modérée | Moins efficace sur une forte asymétrie | 7,397 |
| Box-Cox | Puissance $\lambda$ estimée par vraisemblance | Standard ; $\lambda = 0$ correspond au logarithme | Valeurs strictement positives seulement | 8,325 |
| Yeo-Johnson | Box-Cox généralisé aux valeurs négatives et nulles | Accepte toutes les valeurs ; proche de Box-Cox | Moins intuitive qu'un logarithme | 8,301 |
| `log_b(x + a)` | Logarithme après décalage $a$ | Interprétable ; usuel pour une asymétrie à droite | Le décalage $a$ doit être choisi | 10,082 |
| `arcsinh(x)` | Sinus hyperbolique inverse | Accepte les valeurs négatives ; proche du logarithme pour les grandes valeurs | Peu de paramètres à régler ; moins précise qu'une transformation estimée | 10,073 |
| Double reversed log | Logarithme inversé, appliqué deux fois | Pour une asymétrie inverse (queue à gauche) | Inadaptée ici : une queue à droite | 35,632 |
| Center + scale | Centrage et réduction, sans changer la forme | Conserve les unités et la forme | Ne corrige pas l'asymétrie | 25,724 |

> **Note : un score bas ne suffit pas**
> La méthode retenue a le score le plus bas, mais un score bas ne dit pas si la transformation est utile à interpréter. Ici, `orderNorm` gagne sur le score, alors que le logarithme est souvent plus simple à expliquer dans un rapport. Le choix final doit donc combiner le score, le QQ-plot et la facilité d'interprétation.

### Exercices

1. Lancer `bestNormalize()` sur `surface_ok`. Quelle transformation est retenue ?
   > **Intérêt :** le nom de la transformation retenue doit être noté : une mise à jour du package peut changer le résultat.
<details>
<summary>Correction</summary>

```r
bn <- bestNormalize::bestNormalize(surface_ok)
bn$chosen_transform
# Le nom de la transformation retenue s'affiche : à lire dans la console.
```
Le résultat dépend du package et de sa version : on le note dans le compte rendu.
</details>

2. Tracer l'histogramme des valeurs transformées avec `ggplot2`, puis les comparer à celui de la surface brute.
   > **Intérêt :** le graphique confirme le choix automatique : un score ne remplace pas la vérification visuelle.
<details>
<summary>Correction</summary>

```r
ggplot(data.frame(x = x_transforme), aes(x = x)) +
  geom_histogram(bins = 40, fill = "steelblue", colour = "white")
# La distribution transformée est plus symétrique que celle de la surface brute.
```
</details>

## Exercice 8 – Robustesse d'un test

Un test est **robuste** quand son α reste proche de la valeur annoncée même si une condition n'est pas remplie, par exemple la normalité. La robustesse dépend de la taille de l'échantillon : c'est souvent le TCL (chapitre 6) qui la rend possible.

On vérifie ici la robustesse du test t sur une loi exponentielle, très asymétrique, dont la moyenne vraie est 1.

> **Contexte :** dans la pratique, les données ne sont jamais parfaitement normales. La robustesse indique si un test de moyenne reste fiable malgré cet écart, ce qui permet de l'utiliser en connaissance de cause plutôt que de l'abandonner.

### Exemple

```r
set.seed(42)

# H0 vraie : la moyenne est 1. Proportion de rejets de H0 à 5 %
taux_rejet <- function(n) {
  mean(replicate(2000, t.test(rexp(n, rate = 1), mu = 1)$p.value < 0.05))
}

taux_rejet(5)
taux_rejet(100)
```

### Exercices

1. Calculer le taux de rejet pour $n = 5$ et $n = 100$. Lequel est le plus proche de 0,05 ?
   > **Intérêt :** le taux de rejet réel dépend de la taille : c'est l'argument pour exiger un effectif minimum dans une étude.
<details>
<summary>Correction</summary>

```r
taux_rejet(5)
taux_rejet(100)
# Le taux est généralement plus éloigné de 0,05 pour n = 5 que pour n = 100.
```
Avec peu d'observations, le test t s'écarte de son α annoncé quand la loi est asymétrique. Avec $n = 100$, le TCL rend la moyenne presque normale, et le taux se rapproche de 0,05.
</details>

2. Comparer le test t à un test non paramétrique, `wilcox.test()`, sur la même loi avec $n = 5$. Ce test porte sur la **médiane** : pour la loi exponentielle de paramètre 1, la médiane vraie est $\ln 2 \approx 0{,}693$. Quel taux de rejet obtient-il ?
   > **Intérêt :** un test non paramétrique est une alternative à présenter quand la normalité est douteuse et que l'échantillon est petit.
<details>
<summary>Correction</summary>

```r
wilcox_rejet <- function(n) {
  mean(replicate(2000, wilcox.test(rexp(n, rate = 1), mu = log(2))$p.value < 0.05))
}
wilcox_rejet(5)
```
Le test de Wilcoxon ne suppose pas la normalité : il peut être une alternative quand la taille est petite et la loi très asymétrique. Il ne fait pas partie du programme de la formation ; il est présenté ici seulement pour comparer les taux de rejet.
</details>

3. Rédiger la conclusion : quand peut-on utiliser un test paramétrique sur une variable asymétrique, et quand faut-il une alternative ?
   > **Intérêt :** la conclusion sur la robustesse est un message clé à transmettre : elle justifie ou interdit le test choisi.
<details>
<summary>Correction</summary>

Pour un grand échantillon (plusieurs dizaines d'observations, selon l'asymétrie), le test t reste acceptable grâce au TCL. Pour un petit échantillon fortement asymétrique, il faut vérifier la normalité (QQ-plot, test) et envisager une transformation ou un test non paramétrique. La décision se justifie avec les deux : le test de normalité et le graphique.
</details>

> **Note : robustesse et α**
> La robustesse se mesure sur α (le taux de rejet quand $H_0$ est vraie), mais aussi sur la puissance (quand $H_1$ est vraie). Un test peut garder un bon α et perdre en puissance quand la loi est asymétrique.

## Exercice 9 – Visualiser les quantiles avec `plotly`

Le QQ-plot du chapitre 6 se rend interactif avec `plotly` : on identifie les points qui s'écartent de la droite de normalité.

> **Contexte :** le QQ-plot interactif permet de repérer les dossiers atypiques responsables d'un écart à la normalité, avant de décider de les garder ou de les analyser à part.

### Exemple

```r
qq <- qqnorm(log(surface_5000), plot.it = FALSE)
moyenne <- mean(log(surface_5000))
ecart <- sd(log(surface_5000))

plot_ly(x = ~qq$x, y = ~qq$y, type = "scatter", mode = "markers",
        name = "Observations") |>
  add_trace(x = qq$x, y = moyenne + ecart * qq$x,
            type = "scatter", mode = "lines", name = "Droite normale")
```

### Exercices

1. Tracer le QQ-plot interactif du logarithme de la surface. Quels points s'écartent de la droite ?
   > **Intérêt :** identifier les points qui s'écartent oriente l'analyse : il faut vérifier ces dossiers avant de conclure.
<details>
<summary>Correction</summary>

```r
# Les points qui s'écartent sont les valeurs extrêmes, en haut et en bas de la distribution.
# Survoler un point donne ses coordonnées : quantile théorique (x) et valeur observée (y).
```
</details>

2. Refaire le graphique avec la surface brute. Expliquer ce qu'on voit.
   > **Intérêt :** comparer la surface brute et son logarithme montre l'intérêt de la transformation à un décideur.
<details>
<summary>Correction</summary>

```r
qq_brut <- qqnorm(surface_5000, plot.it = FALSE)
moy_brut <- mean(surface_5000)
ec_brut <- sd(surface_5000)

plot_ly(x = ~qq_brut$x, y = ~qq_brut$y, type = "scatter", mode = "markers") |>
  add_trace(x = qq_brut$x, y = moy_brut + ec_brut * qq_brut$x,
            type = "scatter", mode = "lines")
# Les points s'écartent fortement de la droite dans la queue haute : l'asymétrie est nette.
```
</details>

## Liens utiles

- [Documentation de `shapiro.test()` (R)](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/shapiro.test.html)
- [Documentation du package `nortest`](https://cran.r-project.org/package=nortest)
- [Documentation du package `bestNormalize`](https://cran.r-project.org/package=bestNormalize)
- [Documentation de `plotly` pour R](https://plotly.com/r/)
