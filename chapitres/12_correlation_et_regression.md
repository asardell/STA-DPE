# Chapitre 12 – Corrélation et régression linéaire simple

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] poser $H_0$, $H_1$, $\alpha$ et $\beta$ avant de tester une corrélation, et lire la p-value
- [ ] calculer et tester le coefficient de Pearson avec `cor.test()`
- [ ] construire une matrice de corrélation et une matrice de p-values sur les variables numériques
- [ ] reproduire une table de Student et lire une p-value selon le ddl et la statistique $t$
- [ ] calculer les coefficients de Spearman et de Kendall, et les comparer à Pearson
- [ ] choisir entre ces trois coefficients selon la forme du nuage et la présence de valeurs extrêmes
- [ ] ajuster une droite de régression simple avec `lm()` et lire les coefficients, le $R^2$ et les résidus
- [ ] prédire une valeur à partir du modèle, et dire pourquoi il ne faut pas extrapoler

> **Contexte :** en bureau d'études, on cherche souvent à relier une consommation à une caractéristique du bâtiment : la surface, l'isolation, l'année de construction. Une corrélation significative ne suffit pas à établir un lien causal, mais une droite de régression permet d'estimer une consommation attendue et d'identifier les logements qui s'en écartent. Ce chapitre donne les outils pour le faire et pour en présenter les limites.

## Sommaire

- [Chapitre 12 – Corrélation et régression linéaire simple](#chapitre-12--corrélation-et-régression-linéaire-simple)
  - [Objectifs](#objectifs)
  - [Sommaire](#sommaire)
  - [Préparation : packages, données et notations](#préparation--packages-données-et-notations)
  - [Exercice 1 – Test de Pearson sur une corrélation](#exercice-1--test-de-pearson-sur-une-corrélation)
  - [Exercice 2 – Matrice de corrélation et matrice de p-values](#exercice-2--matrice-de-corrélation-et-matrice-de-p-values)
  - [Exercice 3 – Reproduire une table de Student](#exercice-3--reproduire-une-table-de-student)
  - [Exercice 4 – Spearman et Kendall](#exercice-4--spearman-et-kendall)
  - [Exercice 5 – Régression linéaire simple](#exercice-5--régression-linéaire-simple)
  - [Exercice 6 – Prédiction et résidus](#exercice-6--prédiction-et-résidus)
  - [Liens utiles](#liens-utiles)

## Préparation : packages, données et notations

```r
install.packages(c("pwr", "corrplot"))
library(readr)
library(dplyr)
library(ggplot2)
library(pwr)
library(corrplot)

dpe <- read_csv("data/dpe_villeurbanne_10000.csv")

surface <- dpe$surface_habitable_logement
conso   <- dpe$conso_5_usages_ep
```

| Notion | Notation | Signification |
|--------|----------|---------------|
| Corrélation vraie | $\rho$ | corrélation dans la population |
| Corrélation observée | $r$ | corrélation calculée sur l'échantillon |
| Rho de Spearman | $\rho_S$ | corrélation de Pearson calculée sur les rangs |
| Tau de Kendall | $\tau_b$ | corrélation fondée sur les paires concordantes et discordantes |
| Pente de la droite | $\beta_1$ | variation moyenne de $y$ pour une unité de $x$ |

Pour toutes les corrélations de ce chapitre, les hypothèses sont les mêmes :

- $H_0 : \rho = 0$ (pas de liaison) ;
- $H_1 : \rho \ne 0$ (liaison, test bilatéral) ;
- $\alpha = 0{,}05$, fixé avant le calcul ;
- $\beta$ dépend de la taille de l'échantillon et de la force réelle de la liaison ;
- règle de décision : on rejette $H_0$ si la p-value est inférieure à $\alpha$.

## Exercice 1 – Test de Pearson sur une corrélation

Le coefficient de Pearson $r$ (chapitre 5) est testé avec la statistique suivante, qui suit une loi de Student à $n - 2$ degrés de liberté sous $H_0$ :

$$
t = r \sqrt{\frac{n - 2}{1 - r^2}}
$$

Plus $n$ est grand, plus une petite corrélation devient significative. C'est pourquoi une corrélation de 0,1 peut être significative avec quelques milliers de DPE, et ne pas l'être avec quelques dizaines.

> **Contexte :** avant de présenter une liaison entre la surface et la consommation, il faut dire si elle est statistiquement établie ou si elle peut venir du hasard de l'échantillon.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `cor.test()` | Coefficient de corrélation, IC à 95 % et test de $H_0 : \rho = 0$. | `x`, `y`, `method`, `alternative`, `conf.level` | `cor.test(x, y, method = "pearson")` |
| `pwr.r.test()` (package `pwr`) | Taille d'échantillon ou puissance pour une corrélation. | `r`, `power`, `sig.level` | `pwr.r.test(r = 0.1, power = 0.80, sig.level = 0.05)` |

Contrairement à `cov()` et `cor()`, `cor.test()` retire automatiquement les paires où une valeur est manquante.

### Exemple

```r
cor.test(surface, conso)
# t = 122.4, df = 9897, p-value < 2.2e-16
# alternative hypothesis: true correlation is not equal to 0
# 95 percent confidence interval:
#  0.768 0.784
# sample estimates:
#       cor
# 0.776
```

La p-value est inférieure à $2{,}2 \times 10^{-16}$ : on rejette $H_0$. La corrélation est donc significativement différente de 0, et l'intervalle à 95 % ($[0{,}768 ; 0{,}784]$) est étroit grâce aux 9 899 observations.

### Exercices

1. Tester la corrélation entre `surface_habitable_logement` et `conso_5_usages_ep`. Quelle est la conclusion au seuil de 5 % ?
   > **Intérêt :** une corrélation affichée sans test peut sembler solide alors qu'elle repose sur très peu de logements. Le test dit si la liaison est établie.
<details>
<summary>Correction</summary>

```r
cor.test(surface, conso)
# p-value < 2.2e-16 : on rejette H0. La liaison linéaire entre surface et consommation est significative.
```
</details>

2. Tester la corrélation entre `surface_habitable_logement` et `besoin_chauffage`. Quelle est la p-value, et que conclure ?
   > **Intérêt :** une p-value élevée ne prouve pas l'absence de lien, elle dit seulement que la liaison linéaire n'est pas établie.
<details>
<summary>Correction</summary>

```r
cor.test(dpe$surface_habitable_logement, dpe$besoin_chauffage)
# r = -0.009, t = -0.915, df = 9884, p-value = 0.36
```
La p-value vaut environ 0,36 : on ne rejette pas $H_0$ au seuil de 5 %. L'exercice 4 montre que la liaison apparaît avec une autre mesure.
</details>

3. Combien de DPE faut-il pour détecter une corrélation de $r = 0{,}1$ avec une puissance de 80 % et un $\alpha$ de 5 % ?
   > **Intérêt :** une étude qui ne détecte pas une liaison faible n'est pas forcément une étude qui la contredit : la puissance de l'échantillon compte.
<details>
<summary>Correction</summary>

```r
pwr.r.test(r = 0.1, power = 0.80, sig.level = 0.05)
# n = 783
```
Il faut environ 783 DPE pour détecter une corrélation de 0,1 avec une probabilité de 80 %. Le jeu de 9 899 observations permet donc de détecter des liaisons bien plus faibles.
</details>

## Exercice 2 – Matrice de corrélation et matrice de p-values

Une matrice de corrélation (chapitre 5) donne le coefficient de chaque paire. Une **matrice de p-values** donne, pour chaque paire, la p-value du test $H_0 : \rho = 0$. Les deux se lisent ensemble : une corrélation forte avec une p-value élevée signifie un petit échantillon ou des données incomplètes.

> **Attention : tests multiples**
> La matrice de p-values fait 10 tests sur les 5 variables. Avec $\alpha = 0{,}05$, environ 0,5 test sur 10 serait significatif par hasard seul. On peut corriger ce risque avec la méthode de Bonferroni (chapitre 14).

> **Contexte :** une matrice de corrélations et de p-values est un tableau de synthèse courant dans une étude : elle montre d'un coup d'œil quelles variables sont liées, et évite de présenter une corrélation sans son risque.

### Mémo

| Outil | Commande | Argument pertinent |
|-------|----------|--------------------|
| Matrice de corrélation | `cor(dpe[, variables], use = "pairwise.complete.obs")` | `use` |
| Matrice de p-values | boucle sur les paires avec `cor.test()` | `method` |
| Graphique | `corrplot(matrice, method = "color", type = "upper", addCoef.col = "black")` | voir chapitre 5 |

### Exemple

```r
variables <- c("surface_habitable_logement", "conso_5_usages_ep",
               "deperditions_murs", "besoin_chauffage", "cout_total_5_usages")

# Matrice des coefficients de Pearson
matrice_r <- cor(dpe[, variables], use = "pairwise.complete.obs")

# Matrice des p-values du test de Pearson, paire par paire
matrice_p <- matrix(NA, nrow = length(variables), ncol = length(variables),
                    dimnames = list(variables, variables))
for (i in seq_along(variables)) {
  for (j in seq_along(variables)) {
    if (i != j) {
      matrice_p[i, j] <- cor.test(dpe[[variables[i]]], dpe[[variables[j]]])$p.value
    }
  }
}

round(matrice_r, digits = 3)
```

### Exercices

1. Construire la matrice de p-values de Pearson sur les 5 variables. Quelles paires ne sont pas significatives au seuil de 5 % ?
   > **Intérêt :** une matrice sans p-values laisse croire que toutes les corrélations sont établies ; la matrice de p-values montre les paires qui ne le sont pas.
<details>
<summary>Correction</summary>

| Paire | Pearson r | p-value |
|-------|-----------|---------|
| surface / consommation | 0,776 | < 2,2e-16 |
| surface / déperditions murs | −0,015 | 0,13 |
| surface / besoin de chauffage | −0,009 | 0,36 |
| surface / coût total | 0,768 | < 2,2e-16 |
| consommation / déperditions murs | 0,291 | < 2,2e-16 |
| consommation / besoin de chauffage | 0,214 | < 2,2e-16 |
| consommation / coût total | 0,984 | < 2,2e-16 |
| déperditions murs / besoin de chauffage | 0,978 | < 2,2e-16 |
| déperditions murs / coût total | 0,273 | < 2,2e-16 |
| besoin de chauffage / coût total | 0,208 | < 2,2e-16 |

Seules les deux paires entre la surface et les variables thermiques (déperditions murs, besoin de chauffage) ne sont pas significatives. Les autres sont établies ; les coefficients proches de 1 reflètent surtout que le coût dépend directement de la consommation.
</details>

2. Avec le même tableau, repérer les paires dont la p-value est significative mais dont le coefficient est inférieur à 0,3 en valeur absolue. Que peut-on dire de leur force ?
   > **Intérêt :** une liaison significative peut rester faible : la significativité dit qu'elle existe, la valeur du coefficient dit son importance.
<details>
<summary>Correction</summary>

Les paires concernées sont : consommation / déperditions murs (0,291), consommation / besoin de chauffage (0,214), déperditions murs / coût total (0,273) et besoin de chauffage / coût total (0,208). Elles sont significatives avec près de 10 000 observations, mais leur force est faible : une part réelle mais limitée de la variabilité d'une variable est expliquée par l'autre.
</details>

3. Construire le graphique `corrplot` de la matrice de corrélation, avec les coefficients affichés. Pourquoi la diagonale est-elle toujours égale à 1 ?
   > **Intérêt :** le graphique résume la matrice en une image, mais il ne montre pas les p-values : elles restent dans le tableau.
<details>
<summary>Correction</summary>

```r
corrplot(matrice_r, method = "color", type = "upper", addCoef.col = "black",
         tl.col = "black")
```
La diagonale vaut 1 parce qu'une variable est toujours parfaitement corrélée avec elle-même. Le graphique ne montre pas les p-values : le tableau de l'exercice 1 reste nécessaire pour la significativité.
</details>

## Exercice 3 – Reproduire une table de Student

Avant les logiciels, la p-value d'un test se lisait dans une **table de Student** : une ligne par nombre de degrés de liberté (ddl), une colonne par valeur de la statistique $t$, et la p-value à l'intersection. On la reproduit ici pour voir comment la p-value dépend de deux choses : la statistique $t$ et le ddl.

La p-value bilatérale d'une statistique $t$ à $d$ ddl est :

$$
p = 2 \times P(T_d > |t|)
$$

Plus $|t|$ est grand, plus la p-value baisse. Pour un même $t$, un petit ddl donne une p-value plus élevée : la loi de Student a des queues plus épaisses que la loi normale.

> **Contexte :** dans un rapport, la table permet de vérifier d'un coup d'œil si un résultat est significatif, sans logiciel. C'est aussi un moyen de comprendre pourquoi un test sur un petit échantillon est moins sensible.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `pt()` | Probabilité sous la loi de Student : $P(T \le t)$. | `q`, `df` | `pt(-2, df = 30)` |
| `qt()` | Valeur critique de la loi de Student. | `p`, `df` | `qt(0.975, df = 30)` |
| `outer()` | Calcule une fonction pour toutes les combinaisons de deux vecteurs : un tableau complet. | `X`, `Y`, `FUN` | `outer(dfs, ts, function(df, t) ...)` |

### Exemple

```r
dfs <- c(5, 10, 30, 100, 1000, 9884)
ts  <- c(0.5, 0.9, 1.0, 1.5, 2.0, 2.5, 3.0)

table_p <- outer(dfs, ts, function(df, t) 2 * pt(-t, df = df))
dimnames(table_p) <- list(ddl = dfs, t = ts)
round(table_p, digits = 4)
#        t
# ddl      0.5    0.9    1.0    1.5    2.0    2.5    3.0
# 5      0.6383 0.4094 0.3632 0.1939 0.1019 0.0545 0.0301
# 10     0.6279 0.3893 0.3409 0.1645 0.0734 0.0314 0.0133
# 30     0.6207 0.3753 0.3253 0.1441 0.0546 0.0181 0.0054
# 100    0.6182 0.3703 0.3197 0.1368 0.0482 0.0140 0.0034
# 1000   0.6172 0.3683 0.3176 0.1339 0.0458 0.0126 0.0028
# 9884   0.6171 0.3681 0.3173 0.1336 0.0455 0.0124 0.0027
```

La dernière ligne (9 884 ddl) est presque identique à la loi normale, comme dans le chapitre 7.

### Exercices

1. Lire dans la table la p-value d'une statistique $t = 2$ pour 5 ddl, puis pour 9 884 ddl. Pourquoi la p-value baisse-t-elle quand le ddl augmente ?
   > **Intérêt :** une même statistique ne donne pas la même conclusion selon la taille de l'échantillon : c'est la raison pour laquelle il faut toujours donner le ddl avec le résultat.
<details>
<summary>Correction</summary>

```r
round(2 * pt(-2, df = 5), digits = 4)      # 0.1019
round(2 * pt(-2, df = 9884), digits = 4)   # 0.0455
```
Avec 5 ddl, $t = 2$ n'est pas significatif au seuil de 5 % ; avec 9 884 ddl, il l'est. Plus le ddl est grand, plus la loi de Student ressemble à la loi normale, dont les queues sont plus fines : une même valeur de $t$ est donc moins probable par hasard.
</details>

2. Pour chaque ddl, trouver la valeur de $t$ à partir de laquelle la p-value passe sous 0,05. Comparer avec la valeur 1,96 de la loi normale.
   > **Intérêt :** le seuil de décision dépend du ddl : un petit échantillon demande une statistique plus forte pour conclure, ce que la table montre directement.
<details>
<summary>Correction</summary>

```r
qt(0.975, df = c(5, 10, 30, 100, 1000, 9884))
# 2.571 2.228 2.042 1.984 1.962 1.960
```
Ces valeurs critiques sont celles de la table de Student utilisées au chapitre 7. Elles tendent vers 1,96 quand le ddl augmente.
</details>

3. La p-value de la paire surface / besoin de chauffage (chapitre 12, exercice 1) est lue dans la table avec $t = -0{,}917$ et 9 884 ddl. Lire la valeur dans la table, puis la comparer à la valeur exacte.
   > **Intérêt :** une lecture de table est toujours approximative : il faut savoir quand l'interpolation suffit et quand le calcul exact est nécessaire.
<details>
<summary>Correction</summary>

Dans la table, $|t| = 0{,}917$ se situe entre les colonnes 0,9 (p = 0,368) et 1,0 (p = 0,317). Une interpolation linéaire donne environ 0,36.

```r
2 * pt(-0.9168, df = 9884)   # 0.359
```
La valeur exacte est 0,359 : on ne rejette pas $H_0$. L'interpolation est très proche ici parce que les colonnes sont proches. Elle resterait approximative sur un intervalle plus large, car la courbe n'est pas une droite entre deux colonnes : pour une décision, le calcul exact est préférable.
</details>

4. Lire la p-value de la paire surface / déperditions murs ($t = -1{,}499$, 9 884 ddl). La décision est-elle la même que pour surface / besoin ?
   > **Intérêt :** deux paires non significatives peuvent avoir des p-values très différentes : la décision ne dit pas tout de la force de la preuve.
<details>
<summary>Correction</summary>

```r
2 * pt(-1.4988, df = 9884)   # 0.134
```
La colonne 1,5 de la table donne 0,134 pour 9 884 ddl. La décision est la même que pour surface / besoin (non significative), mais la p-value est plus proche du seuil.
</details>

## Exercice 4 – Spearman et Kendall

Le coefficient de Pearson mesure une liaison **linéaire** et réagit aux valeurs extrêmes. Deux autres coefficients travaillent sur les **rangs** :

- le coefficient de **Spearman** $\rho_S$ est la corrélation de Pearson calculée sur les rangs ; il mesure une liaison monotone, c'est-à-dire qui va toujours dans le même sens sans forcément être une droite. Sans ex-aequo, il s'écrit :

$$
\rho_S = 1 - \frac{6 \sum_{i=1}^{n} d_i^2}{n (n^2 - 1)}
$$

où $d_i$ est la différence entre le rang de $x_i$ et le rang de $y_i$. Pour le test, R utilise l'approximation $t = \rho_S \sqrt{\frac{n - 2}{1 - \rho_S^2}}$ avec $n - 2$ degrés de liberté quand $n \geq 1\,290$ ;

- le coefficient de **Kendall** $\tau_b$ compare toutes les paires d'observations $(i, j)$. Une paire est **concordante** (C) si les deux variables évoluent dans le même sens, **discordante** (D) sinon. Il vaut :

$$
\tau_b = \frac{C - D}{\sqrt{(n_0 - n_1)(n_0 - n_2)}}
$$

avec $n_0 = \dfrac{n(n-1)}{2}$, $n_1$ le nombre de paires ex aequo sur $x$, et $n_2$ le nombre de paires ex aequo sur $y$. Il est en général plus petit que Spearman en valeur absolue, et s'interprète comme une différence de probabilités : $\tau_b$ vaut la probabilité qu'une paire tirée au hasard soit concordante, moins la probabilité qu'elle soit discordante. Pour le test, R utilise une approximation normale : $z = (C - D) / \sqrt{\mathrm{Var}(C - D)}$, car les ex-aequo rendent le calcul exact impossible en pratique.

Les trois tests ont la même forme d'hypothèses : $H_0$ : pas de liaison monotone (ou linéaire pour Pearson), $H_1$ : liaison.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `cor.test(method = "spearman")` | Test de Spearman. | `method` | `cor.test(x, y, method = "spearman")` |
| `cor.test(method = "kendall")` | Test de Kendall, tau-b (gère les ex-aequo). | `method` | `cor.test(x, y, method = "kendall")` |
| `cor()` | Coefficient seul, sans test. | `method` | `cor(x, y, method = "kendall", use = "complete.obs")` |

> **Note : ex-aequo**
> Les données de type DPE contiennent beaucoup de valeurs identiques (surfaces arrondies, consommations répétées). R gère les ex-aequo automatiquement : Spearman utilise des rangs moyens, Kendall utilise la correction tau-b.

### Exemple

```r
# Même couple que l'exercice 1 : la liaison est forte, Pearson et les rangs sont proches
cor.test(surface, conso, method = "spearman")   # rho = 0.682
cor.test(surface, conso, method = "kendall")    # tau = 0.502

# Couple surface / besoin de chauffage : Pearson et les rangs donnent des résultats différents
cor.test(dpe$surface_habitable_logement, dpe$besoin_chauffage)                       # r = -0.009, p = 0.36
cor.test(dpe$surface_habitable_logement, dpe$besoin_chauffage, method = "spearman")  # rho = 0.332, p < 2.2e-16
cor.test(dpe$surface_habitable_logement, dpe$besoin_chauffage, method = "kendall")   # tau = 0.250, p < 2.2e-16
```

### Exercices

1. Comparer Pearson, Spearman et Kendall sur le couple surface / consommation totale. Lequel donne la valeur la plus élevée ? Pourquoi les trois restent-ils proches ?
   > **Intérêt :** trois coefficients qui concordent renforcent la conclusion ; un désaccord entre eux est un signal à examiner sur le nuage de points.
<details>
<summary>Correction</summary>

```r
cor(surface, conso, use = "complete.obs")                        # 0.776 (Pearson)
cor(surface, conso, method = "spearman", use = "complete.obs")   # 0.682
cor(surface, conso, method = "kendall", use = "complete.obs")    # 0.502
```
Pearson est le plus élevé : il est sensible aux logements très grands et très consommateurs, qui pèsent fortement dans le calcul. Les trois coefficients sont positifs et de même ordre de grandeur : la liaison est monotone et à peu près linéaire.
</details>

2. Sur le couple surface / `besoin_chauffage`, comparer les trois coefficients et leurs p-values. Que se passe-t-il ?
   > **Intérêt :** un coefficient proche de 0 ne veut pas dire « aucun lien » ; un changement de mesure peut faire apparaître une liaison monotone qui était cachée.
<details>
<summary>Correction</summary>

| Coefficient | Valeur | p-value |
|-------------|--------|---------|
| Pearson | −0,009 | 0,36 |
| Spearman | 0,332 | < 2,2e-16 |
| Kendall | 0,250 | < 2,2e-16 |

Pearson ne voit pas de liaison, alors que Spearman et Kendall en détectent une, positive et significative. Le coefficient de Pearson reste proche de 0 même après avoir retiré le 1 % de valeurs les plus élevées de `besoin_chauffage` : la liaison ne repose donc pas seulement sur ces valeurs.

> **Attention : valeurs à vérifier.** La variable `besoin_chauffage` contient des valeurs très élevées, jusqu'à 1 677 307 kWh, et 773 DPE dépassent 100 000. Avant d'interpréter cette liaison, il faut vérifier l'unité et l'origine de ces valeurs. Ce chapitre ne les corrige pas.
</details>

3. Calculer Pearson, Spearman et Kendall sur le couple surface / `deperditions_murs`. Que retenir pour une présentation ?
   > **Intérêt :** choisir le coefficient après avoir regardé les résultats est une pratique à éviter : le choix doit être fait avant le calcul, à partir de la forme du nuage.
<details>
<summary>Correction</summary>

```r
cor.test(dpe$surface_habitable_logement, dpe$deperditions_murs)                       # r = -0.015, p = 0.13
cor.test(dpe$surface_habitable_logement, dpe$deperditions_murs, method = "spearman")  # rho = 0.212, p < 2.2e-16
```
Le résultat est le même que pour `besoin_chauffage` : la liaison monotone est significative, la liaison linéaire ne l'est pas. Il faut le dire dans le rapport, et justifier le choix de la méthode.
</details>

## Exercice 5 – Régression linéaire simple

La **régression linéaire simple** ajuste une droite $y = \beta_0 + \beta_1 x + \varepsilon$ qui minimise la somme des carrés des écarts entre les valeurs observées et la droite. Les écarts $\varepsilon$ sont les **résidus**.

- $\beta_1$ est la pente : la variation moyenne de $y$ quand $x$ augmente d'une unité ;
- $\beta_0$ est l'ordonnée à l'origine : la valeur prédite quand $x = 0$, qui n'a pas toujours de sens ;
- le test de $H_0 : \beta_1 = 0$ est équivalent au test de corrélation de l'exercice 1 : sa p-value est la même ;
- le **$R^2$** est la part de la variance de $y$ expliquée par la droite. Pour une régression simple, $R^2 = r^2$.

Le lien avec la corrélation est direct : la pente se calcule à partir de $r$ et des écarts-types de $X$ et de $Y$ :

$$
\beta_1 = r \times \frac{s_Y}{s_X}
$$

Autrement dit, la corrélation donne le **sens** (signe de $r$) et la **force** de la relation (valeur absolue de $r$), tandis que la régression donne une **équation** qui la quantifie : pour une unité de $x$, de combien $y$ varie en moyenne. Le $R^2 = r^2$ indique le **pourcentage de variance de $y$ expliqué** par $x$.

Pour la surface et la consommation : $s_X = 60{,}27$ m², $s_Y = 11\,241{,}5$ kWh EP/an et $r = 0{,}776$, donc $\beta_1 = 0{,}776 \times 11\,241{,}5 / 60{,}27 \approx 144{,}7$ kWh EP/an par m². C'est la même pente que celle estimée par `lm()`.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `lm()` | Régression linéaire. | formule `y ~ x`, `data` | `lm(conso_5_usages_ep ~ surface_habitable_logement, data = dpe)` |
| `summary()` | Coefficients, p-values, $R^2$, erreur résiduelle. | | `summary(modele)` |
| `coef()` | Coefficients seuls. | | `coef(modele)` |
| `resid()` | Résidus du modèle. | | `resid(modele)` |
| `predict()` | Valeurs prédites pour de nouvelles données. | `newdata` | `predict(modele, newdata = data.frame(...))` |

> **Note : `lm()` et les valeurs manquantes**
> `lm()` retire automatiquement les lignes où une variable de la formule est manquante. Le modèle est donc ajusté sur 9 899 DPE ici.

### Exemple

```r
modele <- lm(conso_5_usages_ep ~ surface_habitable_logement, data = dpe)
summary(modele)
# Coefficients :
#                             Estimate Std. Error t value Pr(>|t|)
# (Intercept)                  2821.99      97.23    29.0   <2e-16
# surface_habitable_logement    144.74       1.18   122.4   <2e-16
#
# Residual standard error: 7091 on 9897 degrees of freedom
# Multiple R-squared:  0.6022
# F-statistic: 1.498e+04 on 1 and 9897 DF
```

La droite estimée est $\widehat{\text{conso}} = 2\,822 + 144{,}7 \times \text{surface}$. Chaque m² supplémentaire ajoute en moyenne 144,7 kWh EP par an à la consommation totale. Le $R^2$ vaut 0,602 : la surface explique environ 60 % de la variance de la consommation, le reste dépend d'autres facteurs (isolation, système de chauffage, etc.).

> **Note : $R^2 = r^2$**
> Avec une seule variable explicative, $R^2 = r^2 = 0{,}776^2 = 0{,}602$. La régression et la corrélation de Pearson donnent donc la même information.

### Exercices

1. Ajuster le modèle `lm(conso_5_usages_ep ~ surface_habitable_logement)` et lire la pente, son erreur standard et sa p-value. La pente est-elle significativement différente de 0 ?
   > **Intérêt :** la pente se traduit en langage métier : « 145 kWh de plus par m² » parle davantage à un décideur qu'un coefficient.
<details>
<summary>Correction</summary>

```r
modele <- lm(conso_5_usages_ep ~ surface_habitable_logement, data = dpe)
summary(modele)$coefficients
# pente = 144.74, erreur standard = 1.18, t = 122.4, p < 2e-16
```
La pente est significativement différente de 0. Elle vaut 144,7 kWh EP/an par m².
</details>

2. Vérifier que $R^2 = r^2$ en calculant le $R^2$ du modèle et le carré de la corrélation de Pearson.
   > **Intérêt :** une égalité vérifiée à la main rassure sur le modèle avant de le présenter.
<details>
<summary>Correction</summary>

```r
summary(modele)$r.squared           # 0.6022
cor(surface, conso, use = "complete.obs")^2   # 0.6022
```
Les deux valeurs coïncident : avec une seule variable explicative, le $R^2$ est le carré du coefficient de corrélation.
</details>

3. Tracer le nuage de points avec la droite de régression en `ggplot2`, en zoomant jusqu'à 200 m² et 50 000 kWh. Le modèle semble-t-il adapté à toute la plage ?
   > **Intérêt :** la droite donne une tendance, mais un nuage très dispersé ne permet pas de prédire un logement précis.
<details>
<summary>Correction</summary>

```r
ggplot(dpe, aes(x = surface_habitable_logement, y = conso_5_usages_ep)) +
  geom_point(alpha = 0.2) +
  geom_smooth(method = "lm", se = FALSE, colour = "red") +
  coord_cartesian(xlim = c(0, 200), ylim = c(0, 50000)) +
  labs(x = "Surface (m²)", y = "Consommation totale (kWh EP/an)")
```
La droite suit la tendance générale, mais la dispersion autour d'elle reste importante : deux logements de même surface peuvent avoir des consommations très différentes. C'est ce que mesure l'erreur résiduelle (7 091 kWh).
</details>

## Exercice 6 – Prédiction et résidus

Le modèle permet de prédire une consommation pour une surface donnée. Deux précautions s'imposent : une prédiction reste une moyenne, et il ne faut pas prédire en dehors de la plage observée.

> **Contexte :** un bureau d'études qui doit estimer la consommation d'un logement avant travaux utilise cette prédiction comme ordre de grandeur, pas comme valeur certaine. Le résidu indique l'écart d'un logement réel à la prédiction, et sert à repérer les logements atypiques à vérifier.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `predict()` | Valeur prédite. | `newdata` | `predict(modele, newdata = data.frame(surface_habitable_logement = 100))` |
| `resid()` | Résidu de chaque logement : valeur observée moins valeur prédite. | | `resid(modele)` |
| `plot(modele, which = 1)` | Résidus contre valeurs prédites, pour repérer une forme. | `which` | `plot(modele, which = 1)` |

### Exemple

```r
predict(modele, newdata = data.frame(surface_habitable_logement = c(50, 100, 200)))
#     1     2     3
# 10059 17296 31770

summary(dpe$surface_habitable_logement)
# La plage observée va de 8,5 m² au minimum jusqu'à 3 330 m² au maximum.
```

### Exercices

1. Calculer la consommation prédite pour un logement de 100 m². Pourquoi cette valeur n'est-elle qu'une moyenne ?
   > **Intérêt :** une prédiction ponctuelle, sans sa dispersion, peut être prise pour une estimation exacte ; le résidu moyen typique est de 7 091 kWh.
<details>
<summary>Correction</summary>

```r
predict(modele, newdata = data.frame(surface_habitable_logement = 100))
# 17296 kWh EP/an
```
C'est la consommation moyenne attendue pour un logement de 100 m². Les logements réels s'en écartent : l'erreur résiduelle est de 7 091 kWh, soit environ 65 % de la consommation moyenne (10 917 kWh).
</details>

2. Calculer la moyenne des résidus, puis les cinq résidus les plus grands en valeur absolue. Que vaut la moyenne, et que représentent ces logements ?
   > **Intérêt :** les résidus extrêmes désignent des logements à vérifier avant de conclure : un DPE mal saisi ou un logement atypique.
<details>
<summary>Correction</summary>

```r
mean(resid(modele))                          # environ 0 (propriété de la régression)
head(sort(abs(resid(modele)), decreasing = TRUE), 5)
```
La moyenne des résidus vaut 0 par construction de la droite des moindres carrés. Les plus grands résidus désignent des logements dont la consommation s'écarte fortement de la tendance : ils méritent une vérification du DPE, pas une conclusion directe.
</details>

3. Prédire la consommation pour 1 000 m². Pourquoi le résultat n'est-il pas fiable ?
   > **Intérêt :** prédire hors de la plage observée est une erreur fréquente, et elle se voit rarement dans le chiffre produit.
<details>
<summary>Correction</summary>

```r
predict(modele, newdata = data.frame(surface_habitable_logement = 1000))
# environ 147 560 kWh EP/an
```
Aucun logement de 1 000 m² n'est dans les données, sauf quelques très grands logements. La droite est prolongée sans preuve que la liaison reste linéaire à cette échelle : c'est une extrapolation, à ne pas présenter comme une prédiction.
</details>

## Liens utiles

- [Documentation de `cor.test()` (R)](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/cor.test.html)
- [Documentation de `lm()` (R)](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/lm.html)
- [Documentation du package `pwr`](https://cran.r-project.org/package=pwr)
- [Documentation du package `corrplot`](https://cran.r-project.org/package=corrplot)
