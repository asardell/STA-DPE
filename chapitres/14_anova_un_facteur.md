# Chapitre 14 – ANOVA à un facteur

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] poser $H_0$, $H_1$, $\alpha$ et $\beta$ pour comparer les moyennes de plusieurs groupes
- [ ] mettre les données en format désempilé et réaliser une ANOVA à un facteur avec `aov()`
- [ ] vérifier les conditions de l'ANOVA avec le test de Levene et le test de Bartlett, et choisir lequel croire
- [ ] décider si une comparaison multiple est nécessaire, et la faire avec la correction de Bonferroni ou le test HSD de Tukey
- [ ] appliquer la même démarche à plusieurs couples de variables, et conclure sans sur-interpréter

> **Contexte :** en bureau d'études, on compare souvent une grandeur entre plusieurs groupes : la consommation selon le type d'énergie, le coût selon l'époque de construction, la surface selon le type de logement. Comparer les groupes deux à deux avec des tests t multiplie les risques d'erreur. L'ANOVA répond d'abord à une question globale, puis les comparaisons multiples disent quels groupes diffèrent. La démarche doit être suivie dans l'ordre, sans sauter d'étape.

## Sommaire

- [Chapitre 14 – ANOVA à un facteur](#chapitre-14--anova-à-un-facteur)
  - [Objectifs](#objectifs)
  - [Sommaire](#sommaire)
  - [Préparation : packages, données et format des données](#préparation--packages-données-et-format-des-données)
  - [Démarche en six étapes](#démarche-en-six-étapes)
  - [Exercice 1 – ANOVA à un facteur : consommation par m² selon l'énergie](#exercice-1--anova-à-un-facteur--consommation-par-m²-selon-lénergie)
  - [Exercice 2 – Vérifier l'égalité des variances : Levene et Bartlett](#exercice-2--vérifier-légalité-des-variances--levene-et-bartlett)
  - [Exercice 3 – Comparaisons multiples : Bonferroni et Tukey](#exercice-3--comparaisons-multiples--bonferroni-et-tukey)
  - [Exercice 4 – ANOVA de Welch : variances inégales](#exercice-4--anova-de-welch--variances-inégales)
  - [Exercice 5 – Exercices d'application sur plusieurs couples](#exercice-5--exercices-dapplication-sur-plusieurs-couples)
  - [Liens utiles](#liens-utiles)

## Préparation : packages, données et format des données

```r
install.packages(c("car"))
library(readr)
library(dplyr)
library(ggplot2)
library(car)

dpe <- read_csv("data/dpe_villeurbanne_10000.csv")
```

Pour `aov()`, `leveneTest()` et `bartlett.test()`, les données doivent être en **format désempilé** (aussi dit « long ») : une colonne contient toutes les valeurs, une autre colonne indique le groupe de chaque valeur. Le format empilé, avec une colonne par groupe, ne convient pas à ces fonctions.

```r
# Format désempilé : une ligne par DPE, une colonne de valeur, une colonne de groupe
dpe_an <- dpe |>
  filter(conso_5_usages_ep > 0, !is.na(surface_habitable_logement)) |>
  mutate(
    ln_conso_m2 = log(conso_5_usages_ep / surface_habitable_logement),
    energie = case_when(
      type_energie_n1 %in% c("Électricité", "Gaz naturel") ~ type_energie_n1,
      type_energie_n1 == "Réseau de Chauffage urbain" ~ "Réseau",
      TRUE ~ "Fioul et autres"
    ),
    energie = factor(energie, levels = c("Électricité", "Gaz naturel", "Réseau", "Fioul et autres"))
  )

# Données pour les autres couples : période de construction en cinq groupes
dpe_per <- dpe |>
  mutate(
    ln_conso_m2 = log(conso_5_usages_ep / surface_habitable_logement),
    energie = factor(case_when(
      type_energie_n1 %in% c("Électricité", "Gaz naturel") ~ type_energie_n1,
      type_energie_n1 == "Réseau de Chauffage urbain" ~ "Réseau",
      TRUE ~ "Fioul et autres"
    ), levels = c("Électricité", "Gaz naturel", "Réseau", "Fioul et autres")),
    periode_r = case_when(
      periode_construction == "avant 1948" ~ "avant 1948",
      periode_construction == "1948-1974" ~ "1948-1974",
      periode_construction %in% c("1975-1977", "1978-1982", "1983-1988") ~ "1975-1988",
      periode_construction %in% c("1989-2000", "2001-2005") ~ "1989-2005",
      TRUE ~ "2006 et après"
    )
  )
```

Deux fonctions servent pour chaque couple : l'une trace le boxplot bivarié (une variable quantitative par groupe, avec la moyenne de chaque groupe), l'autre construit le tableau de l'ANOVA à partir des formules de la démarche.

```r
# Boxplot bivarié : une boîte par groupe, losange = moyenne du groupe
boxplot_bivarie <- function(data, y, x, ylim = NULL) {
  d <- data.frame(valeur = data[[y]], groupe = data[[x]])
  p <- ggplot(d, aes(x = groupe, y = valeur, fill = groupe)) +
    geom_boxplot(outlier.shape = NA) +
    stat_summary(fun = mean, geom = "point", shape = 23, size = 3, fill = "white") +
    guides(fill = "none") +
    labs(x = x, y = y)
  if (!is.null(ylim)) p <- p + coord_cartesian(ylim = ylim)
  p
}

# Tableau ANOVA calculé avec les formules : SCE, ddl, CM, F et p-value
tableau_anova <- function(data, y, x) {
  d <- data[!is.na(data[[y]]) & !is.na(data[[x]]), ]
  val <- d[[y]]
  grp <- as.character(d[[x]])
  moy_g <- tapply(val, grp, mean)
  n_g <- tapply(val, grp, length)
  moy <- mean(val)
  sce_inter <- sum(n_g * (moy_g - moy)^2)
  sce_intra <- sum((val - moy_g[grp])^2)
  ddl_inter <- length(n_g) - 1
  ddl_intra <- length(val) - length(n_g)
  f_obs <- (sce_inter / ddl_inter) / (sce_intra / ddl_intra)
  data.frame(
    source = c("inter", "intra", "total"),
    sce = c(sce_inter, sce_intra, sce_inter + sce_intra),
    ddl = c(ddl_inter, ddl_intra, ddl_inter + ddl_intra),
    cm = c(sce_inter / ddl_inter, sce_intra / ddl_intra, NA),
    F = c(f_obs, NA, NA),
    p = c(pf(f_obs, ddl_inter, ddl_intra, lower.tail = FALSE), NA, NA)
  )
}
```

Le logarithme de la consommation par m² est utilisé comme au chapitre 9 : sa distribution est plus proche de la normale que la consommation brute.

> **Note : immeubles sans surface**
> Les 79 DPE de type « immeuble » n'ont pas de surface habitable renseignée. Ils sont donc exclus de tous les couples qui utilisent la surface ou la consommation par m². Le chapitre 14 teste les autres variables et les groupes qui ont assez d'effectif.

| Notion | Notation | Signification |
|--------|----------|---------------|
| Nombre de groupes | $k$ | nombre de modalités du facteur |
| Effectif total | $N$ | nombre total d'observations |
| Moyenne d'un groupe | $\bar{y}_i$ | moyenne des observations du groupe $i$ |
| Carré moyen inter-groupes | CM$_{\text{inter}}$ | variabilité entre les moyennes des groupes |
| Carré moyen intra-groupes | CM$_{\text{intra}}$ | variabilité à l'intérieur des groupes |

## Démarche en six étapes

Pour une ANOVA à un facteur, on pose :

- $H_0$ : $\mu_1 = \mu_2 = \dots = \mu_k$ (toutes les moyennes sont égales) ;
- $H_1$ : au moins une moyenne diffère des autres ;
- $\alpha = 0{,}05$, fixé avant le calcul ;
- $\beta$ dépend de la taille des groupes et de l'écart entre les moyennes : avec plusieurs milliers de DPE par groupe, des écarts très faibles sont détectés.

La statistique de l'ANOVA est le rapport des deux carrés moyens :

$$
F = \frac{\text{CM}_{\text{inter}}}{\text{CM}_{\text{intra}}} = \frac{\dfrac{\text{SCE}_{\text{inter}}}{k - 1}}{\dfrac{\text{SCE}_{\text{intra}}}{N - k}}
$$

Sous $H_0$, $F$ suit une loi de Fisher à $(k - 1)$ et $(N - k)$ degrés de liberté.

### Le tableau de l'ANOVA

Les calculs se résument dans un tableau. Les sommes des carrés se découpent en une part entre groupes et une part à l'intérieur des groupes :

| Source | Somme des carrés (SCE) | ddl | Carré moyen (CM) | F | p-value |
|--------|------------------------|-----|------------------|---|---------|
| Inter-groupes | formule (1) ci-dessous | k − 1 | SCE inter / (k − 1) | CM inter / CM intra | loi de Fisher, ddl (k − 1 ; N − k) |
| Intra-groupes (résiduelle) | formule (2) ci-dessous | N − k | SCE intra / (N − k) | | |
| Total | formule (3) ci-dessous | N − 1 | | | |

Formules des sommes des carrés :

Inter-groupes (1) :

$$
\text{SCE}_{\text{inter}} = \sum_{i=1}^{k} n_i (\bar{y}_i - \bar{y})^2
$$

Intra-groupes (2) :

$$
\text{SCE}_{\text{intra}} = \sum_{i=1}^{k} \sum_{j=1}^{n_i} (y_{ij} - \bar{y}_i)^2
$$

Total (3) :

$$
\text{SCE}_{\text{total}} = \sum_{i}\sum_{j} (y_{ij} - \bar{y})^2
$$

Deux relations permettent de vérifier le calcul : $\text{SCE}_{\text{total}} = \text{SCE}_{\text{inter}} + \text{SCE}_{\text{intra}}$, et $\eta^2 = \text{SCE}_{\text{inter}} / \text{SCE}_{\text{total}}$.

La démarche se déroule dans cet ordre :

1. **Poser les hypothèses** et fixer $\alpha$ et la taille des groupes attendue.
2. **Visualiser et décrire** : boxplot par groupe, effectif, moyenne et écart-type de chaque groupe (chapitre 5).
3. **Vérifier l'égalité des variances** avec le test de Levene (et Bartlett seulement si la normalité est admise).
4. **Faire l'ANOVA** et lire la p-value. Si elle est supérieure à $\alpha$, on s'arrête : aucune comparaison multiple n'est justifiée.
5. **Comparer les groupes deux à deux** avec une correction de Bonferroni ou un test HSD de Tukey, seulement si l'ANOVA est significative.
6. **Conclure** : dire quels groupes diffèrent, dans quel sens, et avec quelle force (la part de variance expliquée).

> **Note : ANOVA et normalité**
> L'ANOVA suppose des résidus normaux de même variance dans chaque groupe. Avec quelques milliers d'observations par groupe, le théorème central limite rend l'ANOVA robuste à une distribution non normale. En revanche, une variance très différente d'un groupe à l'autre change la conclusion : c'est pourquoi l'étape 3 est obligatoire.

## Exercice 1 – ANOVA à un facteur : consommation par m² selon l'énergie

On compare le logarithme de la consommation par m² selon le type d'énergie de chauffage, avec quatre groupes : électricité, gaz naturel, réseau de chauffage urbain, fioul et autres.

> **Contexte :** un bureau d'études qui compare des solutions de chauffage doit savoir si la performance moyenne diffère vraiment selon l'énergie, ou si l'écart observé peut venir de la variabilité des bâtiments. La réponse globale vient avant le détail par paire.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `aov()` | ANOVA à un facteur, format désempilé. | formule `y ~ facteur`, `data` | `aov(ln_conso_m2 ~ energie, data = dpe_an)` |
| `summary()` | Table de l'ANOVA : ddl, somme des carrés, $F$, p-value. | | `summary(modele)` |
| `aggregate()` | Moyenne et écart-type par groupe. | formule, `FUN` | `aggregate(ln_conso_m2 ~ energie, data = dpe_an, FUN = mean)` |
| Part de variance expliquée | Calcul à partir de la table, sans fonction dédiée. | | $\eta^2 = \text{SCE}_{\text{inter}} / \text{SCE}_{\text{total}}$ |

### Exemple

```r
modele <- aov(ln_conso_m2 ~ energie, data = dpe_an)
summary(modele)
#              Df Sum Sq Mean Sq F value Pr(>F)
# energie       3   27.8   9.275   54.55 <2e-16 ***
# Residuals  9895 1682.4   0.170
```

La p-value est inférieure à $\alpha = 0{,}05$ : on rejette $H_0$. Au moins une énergie a une consommation par m² moyenne différente des autres. Le facteur énergie explique environ 1,6 % de la variance : l'effet est statistiquement net, mais faible en valeur.

### Exercices

1. Calculer la moyenne et l'écart-type du logarithme de la consommation par m² pour chaque énergie. Quelle énergie a la moyenne la plus élevée ?
   > **Intérêt :** avant le test, les statistiques par groupe donnent l'ordre de grandeur de l'écart, que la p-value ne donne pas.
<details>
<summary>Correction</summary>

```r
dpe_an |>
  group_by(energie) |>
  summarise(effectif = n(), moyenne = mean(ln_conso_m2), ecart_type = sd(ln_conso_m2))
# Électricité : n = 4 792, moyenne 5,272, écart-type 0,406
# Gaz naturel : n = 4 358, moyenne 5,187, écart-type 0,428
# Réseau : n = 647, moyenne 5,218, écart-type 0,335
# Fioul et autres : n = 102, moyenne 5,568, écart-type 0,471
```
La moyenne la plus élevée est celle du fioul et autres (5,57 sur le logarithme, soit une consommation par m² plus forte). Le groupe fioul et autres est le plus petit : son résultat doit être lu avec prudence.
</details>

2. Lancer l'ANOVA et lire la p-value et le $F$. Calculer la part de variance expliquée $\eta^2$ à partir de la table.
   > **Intérêt :** une p-value très petite ne dit pas si l'effet est important : $\eta^2$ donne l'ordre de grandeur de la part expliquée.
<details>
<summary>Correction</summary>

```r
summary(modele)
# F = 54.55, ddl = (3, 9895), p < 2e-16
# SCE_inter = 27.8 ; SCE_total = 27.8 + 1682.4 = 1710.2
# eta2 = 27.8 / 1710.2 = 0.016
```
On rejette $H_0$. Le facteur énergie explique 1,6 % de la variance du logarithme de la consommation : l'effet est significatif, mais la part expliquée reste faible. D'autres facteurs (isolation, surface, année de construction) pèsent davantage.
</details>

3. Quelle est l'étape suivante, et pourquoi ne pas la faire tout de suite ?
   > **Intérêt :** l'ANOVA dit seulement qu'une différence existe ; elle ne dit pas entre quels groupes. Il faut d'abord vérifier les conditions (exercice 2).
<details>
<summary>Correction</summary>

L'étape suivante est de vérifier l'égalité des variances (exercice 2), puis les comparaisons multiples (exercice 3). Une comparaison multiple sans vérification des variances peut donner un résultat faux : le test de Tukey, par exemple, suppose des variances comparables entre groupes.
</details>

4. Tracer le boxplot bivarié du logarithme de la consommation par m² selon l'énergie, avec la moyenne de chaque groupe. Quels groupes se recouvrent, et lequel est le plus dispersé ?
   > **Intérêt :** le boxplot montre si l'écart entre groupes est net ou noyé dans la dispersion : l'ANOVA résume cette comparaison en un seul nombre.
<details>
<summary>Correction</summary>

```r
boxplot_bivarie(dpe_an, "ln_conso_m2", "energie")
```
Les boîtes de l'électricité et du gaz naturel se recouvrent largement : leurs moyennes sont proches (5,27 et 5,19) et leurs écarts-types voisins (0,41 et 0,43). Le fioul et autres est décalé vers le haut (moyenne 5,57), mais il ne compte que 102 DPE. Le groupe le plus dispersé est le fioul et autres (écart-type 0,47) ; le réseau est le moins dispersé (0,33).
</details>

5. Calculer à la main la SCE inter, la SCE intra, les carrés moyens, le $F$ et la p-value, puis comparer avec la fonction `tableau_anova()`.
   > **Intérêt :** calculer la table une fois permet de voir d'où vient chaque nombre de la sortie `summary()`, au lieu de la lire comme une boîte noire.
<details>
<summary>Correction</summary>

```r
tableau_anova(dpe_an, "ln_conso_m2", "energie")
# source       sce  ddl       cm     F        p
#  inter     27.82    3    9.275 54.55  5.8e-35
#  intra   1682.39 9895    0.170
#  total   1710.21 9898
```
La SCE totale se découpe en 27,8 entre énergies et 1 682,4 à l'intérieur des groupes. Le $F$ est le rapport des carrés moyens : $9{,}275 / 0{,}170 = 54{,}55$. Le facteur énergie explique $\eta^2 = 27{,}8 / 1\,710{,}2 = 1{,}6\,\%$ de la variance.
</details>

## Exercice 2 – Vérifier l'égalité des variances : Levene et Bartlett

L'ANOVA suppose que les variances sont égales dans tous les groupes. On la teste avec deux tests :

- **Levene** : $H_0$ : les variances sont égales. Il compare les écarts absolus à la médiane de chaque groupe, et ne dépend pas de la normalité. C'est le test à privilégier.
- **Bartlett** : $H_0$ : les variances sont égales. Il est plus puissant, mais très sensible à la non-normalité : un résultat significatif peut venir de la forme de la distribution, pas des variances.

Les deux ont la même hypothèse nulle et la même règle de décision : on rejette $H_0$ si la p-value est inférieure à $\alpha$.

> **Contexte :** une variabilité très différente d'un groupe à l'autre est fréquente dans les DPE : certains groupes ont quelques valeurs très élevées. Ce constat change le choix du test et oblige à présenter les résultats par groupe.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `leveneTest()` (package `car`) | Test de Levene, centré sur la médiane par défaut. | formule, `data`, `center` | `leveneTest(ln_conso_m2 ~ energie, data = dpe_an)` |
| `bartlett.test()` | Test de Bartlett, en formule. | formule, `data` | `bartlett.test(ln_conso_m2 ~ energie, data = dpe_an)` |

### Exemple

```r
car::leveneTest(ln_conso_m2 ~ energie, data = dpe_an)
# F = 47.87, df = (3, 9895), Pr(>F) = 1.05e-30

bartlett.test(ln_conso_m2 ~ energie, data = dpe_an)
# K-squared = 68.25, df = 3, p-value = 1.01e-14
```

Les deux tests rejettent l'égalité des variances : les écarts-types vont de 0,335 (réseau) à 0,471 (fioul et autres).

### Exercices

1. Lancer les deux tests sur le couple ln(consommation par m²) selon l'énergie. Les variances sont-elles égales ? Que faut-il faire ?
   > **Intérêt :** une variance différente ne bloque pas toujours l'analyse, mais elle doit être dite dans le rapport, et elle change le choix du test (exercice suivant).
<details>
<summary>Correction</summary>

Levene (p = 1,1e-30) et Bartlett (p = 1,0e-14) rejettent l'égalité. Les variances ne sont donc pas égales. Il faut le signaler, et utiliser une version de l'ANOVA qui n'exige pas l'égalité (ANOVA de Welch, `oneway.test(ln_conso_m2 ~ energie, data = dpe_an, var.equal = FALSE)`), ou rester prudent sur les p-values des comparaisons multiples.
</details>

2. Refaire les deux tests sur le couple surface selon la période de construction (cinq groupes). Les deux tests donnent-ils le même résultat ?
   > **Intérêt :** Levene et Bartlett ne se contredisent pas toujours ; un désaccord indique souvent une distribution non normale.
<details>
<summary>Correction</summary>

```r
leveneTest(surface_habitable_logement ~ periode_r, data = dpe_per)
# Levene : F = 2.12, p = 0,076 : on ne rejette pas l'égalité des variances.
bartlett.test(surface_habitable_logement ~ periode_r, data = dpe_per)
# Bartlett : K2 = 5 495, ddl = 4, p < 2,2e-16 : on rejette l'égalité des variances.
```
Les deux tests ne donnent pas le même résultat. La surface a une distribution très asymétrique (quelques très grands logements), et Bartlett réagit à cette asymétrie, pas à une vraie différence de variances. C'est pourquoi on fait confiance à Levene dans ce cas.
</details>

3. Sur le couple coût total selon la période de construction, Levene et Bartlett donnent-ils le même verdict ?
   > **Intérêt :** un test significatif n'est pas toujours une variance différente : il faut regarder la forme des distributions avant de conclure.
<details>
<summary>Correction</summary>

```r
leveneTest(cout_total_5_usages ~ periode_r, data = dpe_per)
# Levene : F = 1.48, p = 0,21 : on ne rejette pas l'égalité des variances.
bartlett.test(cout_total_5_usages ~ periode_r, data = dpe_per)
# Bartlett : K2 = 2 297, ddl = 4, p < 2,2e-16 : on rejette.
```
Levene ne rejette pas, Bartlett rejette. La distribution du coût est très asymétrique, avec des valeurs extrêmes. On retient Levene : l'égalité des variances est acceptable, et l'ANOVA peut être lue.
</details>

4. Lire dans la table de Fisher la p-value du test de Levene du couple A (surface selon la période), avec $F = 2{,}12$ et les ddl $(4 ;\ 9\,894)$. Comparer la lecture à la valeur exacte.
   > **Intérêt :** la table de Fisher se lit comme celle du Khi² : une ligne par ddl du dénominateur, une colonne par valeur de $F$. Une lecture entre deux colonnes reste une approximation.
<details>
<summary>Correction</summary>

```r
ddl2 <- c(5, 10, 30, 100, 1000, 9894)
F_val <- c(1, 2, 2.5, 3, 3.5, 4, 5, 10)
table_F <- outer(ddl2, F_val, function(d2, f) pf(f, df1 = 4, df2 = d2, lower.tail = FALSE))
dimnames(table_F) <- list(ddl_denominateur = ddl2, F = F_val)
round(table_F, digits = 4)
#                    F
# ddl_denominateur      1      2    2.5      3    3.5      4      5     10
# 9894             0.4061 0.0917 0.0405 0.0174 0.0073 0.0030 0.0005 0.0000

pf(2.12, df1 = 4, df2 = 9894, lower.tail = FALSE)   # 0.0756
```
Pour 9 894 ddl, $F = 2{,}12$ se situe entre les colonnes 2 (p = 0,092) et 2,5 (p = 0,041). L'interpolation donne environ 0,079 ; la valeur exacte est 0,076. Dans les deux cas, la p-value est supérieure à 0,05 : on ne rejette pas l'égalité des variances, ce qui est cohérent avec le test de Levene de l'exercice 2.
</details>

### Vérifier la normalité des résidus

L'ANOVA suppose que les **résidus** sont normaux, de moyenne nulle et de même variance. Le résidu d'une observation est son écart à la moyenne de son groupe : $e_{ij} = y_{ij} - \bar{y}_i$. C'est donc la normalité de ces écarts, et non celle de la variable brute, qu'il faut examiner.

> **Contexte :** une distribution asymétrique ou à queues lourdes peut rendre les conclusions moins fiables, surtout pour les valeurs extrêmes. Le graphique quantile-quantile (QQ-plot) et un test d'ajustement (vu au chapitre 8) donnent deux lectures complémentaires.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `resid()` | Résidus du modèle `aov()`. | | `resid(modele)` |
| `qqnorm()` + `qqline()` | Graphique quantile-quantile des résidus. | | `qqnorm(residus); qqline(residus)` |
| `nortest::ad.test()` | Test d'Anderson-Darling, sans limite de taille. | | `nortest::ad.test(residus)` |
| `shapiro.test()` | Test de Shapiro-Wilk, limité à 5 000 valeurs. | | `shapiro.test(residus)` |

### Exemple

```r
modele <- aov(ln_conso_m2 ~ energie, data = dpe_an)
residus <- resid(modele)

qqnorm(residus); qqline(residus)
nortest::ad.test(residus)
# A = 4.39, p-value < 0,001
```

### Exercices

1. Lire le QQ-plot des résidus. Où s'écartent-ils de la droite ?
   > **Intérêt :** un QQ-plot montre où la distribution diverge, alors qu'une p-value dit seulement qu'elle diverge.
<details>
<summary>Correction</summary>

Les points suivent la droite au centre, et s'en écartent aux deux extrémités. Trois mesures le confirment : l'asymétrie vaut 0,09 (faible), l'aplatissement en excès vaut 0,38 (queues un peu plus lourdes que la normale), et 0,58 % des résidus dépassent 3 écarts-types contre 0,27 % attendus. La distribution est donc à peu près symétrique, avec des valeurs extrêmes plus nombreuses que la normale.
</details>

2. Le test d'Anderson-Darling donne une p-value inférieure à 0,001. Faut-il abandonner l'ANOVA ?
   > **Intérêt :** avec près de 10 000 résidus, un test détecte de petits écarts qui n'ont pas de conséquence pratique ; la décision se fonde sur l'ampleur de l'écart, pas seulement sur la p-value.
<details>
<summary>Correction</summary>

Non. Le test rejette la normalité stricte, mais l'écart est faible et les queues restent proches de la normale. Avec un grand effectif par groupe, le théorème central limite rend l'ANOVA robuste à ce type d'écart. Il faut toutefois le signaler dans le rapport, et tenir compte des valeurs extrêmes, surtout dans les groupes peu nombreux (exercice 4 sur Welch).
</details>

3. Pourquoi ne peut-on pas appliquer `shapiro.test()` directement aux 9 899 résidus ?
   > **Intérêt :** chaque test a son domaine de validité : une fonction qui refuse les données signale une limite à connaître, pas un résultat.
<details>
<summary>Correction</summary>

`shapiro.test()` n'accepte pas plus de 5 000 valeurs : R renvoie une erreur. On utilise alors Anderson-Darling (`nortest::ad.test()`), qui n'a pas cette limite, ou le QQ-plot. Tirer un sous-échantillon de 5 000 résidus pour appliquer Shapiro n'est pas une bonne solution : la conclusion dépendrait du tirage.
</details>

## Exercice 3 – Comparaisons multiples : Bonferroni et Tukey

Quand l'ANOVA est significative, on compare les groupes deux à deux. Avec $k$ groupes, il y a $m = k(k-1)/2$ comparaisons. Sans correction, le risque de trouver au moins une différence fausse grimpe avec $m$.

- **Bonferroni** : chaque p-value est multipliée par $m$, avec $m = k(k-1)/2$ le nombre de comparaisons, et plafonnée à 1 :

$$
p_{\text{corrigée}} = \min(p_{\text{initiale}} \times m,\ 1)
$$

C'est simple et prudent : la p-value corrigée est plus grande que la p-value initiale, donc moins de paires sont déclarées significatives.

> **Exemple de calcul :** coût total selon la période, avec $k = 5$ groupes, donc $m = 5 \times 4 / 2 = 10$ comparaisons. Pour la paire « avant 1948 » contre « 1989-2005 », la p-value initiale (test t avec écart-type poolé) vaut 0,0306. La p-value corrigée est :
>
> $$p_{\text{corrigée}} = \min(0{,}0306 \times 10,\ 1) = 0{,}306$$
>
> Le test HSD de Tukey donne environ 0,19 pour la même paire. Les deux méthodes concluent à l'absence de différence, mais Bonferroni est ici plus sévère : il multiplie chaque p-value par $m$, alors que Tukey tient compte de l'étendue des moyennes.
>
> ```r
> # p-value initiale, sans correction
> pairwise.t.test(dpe_per$cout_total_5_usages, dpe_per$periode_r, p.adjust.method = "none")
>
> # p-value corrigée de Bonferroni : même résultat que pairwise.t.test(..., "bonferroni")
> m <- 5 * (5 - 1) / 2       # 10
> min(0.0306 * m, 1)         # 0.306
> ```
- **HSD de Tukey** : il utilise la loi de l'étendue studentisée et est conçu pour toutes les paires. Il est en général moins prudent que Bonferroni quand $k$ est grand, et il suppose des effectifs comparables.

> **Attention : ne pas choisir la méthode selon le résultat**
> La méthode se choisit avant le calcul. Si Bonferroni et Tukey donnent des conclusions différentes pour une même paire, on le signale : c'est une zone fragile.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `pairwise.t.test()` | Comparaisons deux à deux avec correction. | `x`, `g`, `p.adjust.method` | `pairwise.t.test(x, g, p.adjust.method = "bonferroni")` |
| `TukeyHSD()` | Test HSD de Tukey sur une ANOVA ajustée avec `aov()`. | `x`, `conf.level` | `TukeyHSD(modele)` |

### Exemple

```r
pairwise.t.test(dpe_per$cout_total_5_usages, dpe_per$periode_r,
                p.adjust.method = "bonferroni")

TukeyHSD(aov(cout_total_5_usages ~ periode_r, data = dpe_per))
```

### Exercices

1. Sur le couple coût total selon la période (ANOVA p = 0,009), lire les comparaisons de Bonferroni. Quelles paires diffèrent ?
   > **Intérêt :** une ANOVA significative ne désigne pas le groupe responsable : c'est la comparaison multiple qui le fait.
<details>
<summary>Correction</summary>

```r
aov_cout <- aov(cout_total_5_usages ~ periode_r, data = dpe_per)
summary(aov_cout)           # F = 3.37, p = 0,009
pairwise.t.test(dpe_per$cout_total_5_usages, dpe_per$periode_r,
                p.adjust.method = "bonferroni")
```
Une seule paire est significative avec Bonferroni : « avant 1948 » contre « 2006 et après » (différence de 566 € en moyenne, p ajustée = 0,021). Les autres paires, dont « 1975-1988 » contre « 2006 et après » (p ajustée = 0,056), ne le sont pas.
</details>

2. Refaire la comparaison avec le test HSD de Tukey. La paire « 1975-1988 » contre « 2006 et après » devient-elle significative ? Quel test donne la conclusion la plus prudente ?
   > **Intérêt :** comparer deux méthodes sur une même paire montre leur niveau de prudence, et justifie le choix annoncé à l'avance.
<details>
<summary>Correction</summary>

```r
TukeyHSD(aov_cout)
```
Avec Tukey, la paire « 1975-1988 » contre « 2006 et après » a une p-value ajustée d'environ 0,044 : elle devient significative. Bonferroni, avec p = 0,056, est plus prudent sur cette paire. La conclusion sûre est : « avant 1948 » se distingue de « 2006 et après » avec les deux méthodes ; l'écart entre « 1975-1988 » et « 2006 et après » reste fragile.
</details>

3. Sur le couple ln(consommation par m²) selon la période, toutes les paires sont-elles significatives ? Lire l'ordre des moyennes.
   > **Intérêt :** quand toutes les paires sont significatives, le classement des moyennes devient la vraie conclusion.
<details>
<summary>Correction</summary>

```r
modele_per <- aov(ln_conso_m2 ~ periode_r, data = dpe_per)
summary(modele_per)                # F = 912, p < 2,2e-16
pairwise.t.test(dpe_per$ln_conso_m2, dpe_per$periode_r, p.adjust.method = "bonferroni")
```
Les dix paires sont significatives. Les moyennes du logarithme diminuent avec l'époque de construction : 5,49 (avant 1948), 5,39 (1948-1974), 5,13 (1975-1988), 5,00 (1989-2005), 4,94 (2006 et après). Les bâtiments récents consomment moins par m².
</details>

4. Pour le couple coût total selon la période, tracer le boxplot bivarié, zoomé jusqu'à 5 000 €, puis construire le tableau ANOVA. Quelle part de la variance le facteur période explique-t-il ?
   > **Intérêt :** un boxplot zoomé masque les valeurs extrêmes : il peut donner une impression d'écarts importants alors que le tableau montre un effet très faible.
<details>
<summary>Correction</summary>

```r
boxplot_bivarie(dpe_per, "cout_total_5_usages", "periode_r", ylim = c(0, 5000))
tableau_anova(dpe_per, "cout_total_5_usages", "periode_r")
```

| Source | SCE | ddl | CM | $F$ | p-value |
|--------|-----|-----|----|-----|---------|
| Inter-groupes | $4{,}17 \times 10^{8}$ | 4 | $1{,}04 \times 10^{8}$ | 3,37 | 0,009 |
| Intra-groupes | $3{,}10 \times 10^{11}$ | 9 995 | $3{,}10 \times 10^{7}$ | | |
| Total | $3{,}10 \times 10^{11}$ | 9 999 | | | |

Le facteur période explique $\eta^2 = 0{,}13\,\%$ de la variance du coût. Les moyennes sont proches (de 1 000 à 1 586 €) et la dispersion à l'intérieur des groupes est très forte : l'écart significatif (p = 0,009) reste faible en valeur.
</details>

## Exercice 4 – ANOVA de Welch : variances inégales

Quand l'égalité des variances est rejetée (exercice 2), l'ANOVA classique ne se lit plus sans réserve : elle suppose une variance commune à tous les groupes. L'ANOVA de Welch ne fait pas cette hypothèse. Elle pondère chaque groupe par l'inverse de sa variance et corrige les degrés de liberté du dénominateur.

Les hypothèses sont les mêmes que pour l'ANOVA classique : $H_0$ : toutes les moyennes sont égales ; $H_1$ : au moins une diffère ; $\alpha = 0{,}05$.

Pour $k$ groupes de tailles $n_i$, de moyennes $\bar{y}_i$ et de variances $s_i^2$, on pose $w_i = n_i / s_i^2$ et $W = \sum_i w_i$. La moyenne pondérée et les statistiques de Welch s'écrivent :

$$
\bar{y}_w = \frac{\sum_i w_i \bar{y}_i}{W}, \qquad
A = \sum_i \frac{(1 - w_i / W)^2}{n_i - 1}
$$

$$
F_{\text{Welch}} = \frac{\sum_i w_i (\bar{y}_i - \bar{y}_w)^2 / (k - 1)}{1 + \dfrac{2(k-2)}{k^2 - 1} A}, \qquad \text{ddl} = (k - 1) \text{ et } \frac{k^2 - 1}{3A}
$$

Le second ddl est généralement non entier et plus petit que $N - k$ : c'est la correction qui protège contre une variance mal estimée dans un petit groupe.

> **Contexte :** dans un audit de consommation, les groupes n'ont pas la même dispersion (quelques bâtiments très atypiques dans un groupe). Présenter une ANOVA classique dans ce cas peut surestimer la précision du résultat : Welch est le choix prudent.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `oneway.test()` | ANOVA à un facteur, Welch par défaut. | formule, `data`, `var.equal` | `oneway.test(ln_conso_m2 ~ energie, data = dpe_an)` |
| `var.equal = TRUE` | Revient à l'ANOVA classique. | `var.equal` | `oneway.test(ln_conso_m2 ~ energie, data = dpe_an, var.equal = TRUE)` |

### Exemple

```r
oneway.test(ln_conso_m2 ~ energie, data = dpe_an)
# F = 48.4, num df = 3, denom df = 448.7, p-value = 4.0e-27
```

### Exercices

1. Comparer l'ANOVA classique et l'ANOVA de Welch sur le logarithme de la consommation par m² selon l'énergie. Le $F$ et les ddl changent-ils la conclusion ?
   > **Intérêt :** quand deux versions donnent la même conclusion, le choix de la méthode ne change pas le message ; il faut tout de même savoir laquelle est la plus fiable.
<details>
<summary>Correction</summary>

| Méthode | $F$ | ddl | p-value |
|---|---|---|---|
| ANOVA classique | 54,55 | 3 ; 9 895 | 5,8e-35 |
| ANOVA de Welch | 48,42 | 3 ; 448,7 | 4,0e-27 |

La conclusion ne change pas : l'énergie a un effet sur la consommation par m². Le second ddl chute de 9 895 à 448,7 : Welch pondère moins le groupe « fioul et autres » (102 DPE), dont la variance est la plus forte (0,222 contre 0,165 pour l'électricité), et il corrige la précision de l'estimation en conséquence.
</details>

2. Refaire les deux tests sur le coût total selon l'énergie. Les deux méthodes concluent-elles de la même façon, et avec la même force ?
   > **Intérêt :** une différence entre l'ANOVA classique et Welch révèle souvent des valeurs extrêmes concentrées dans un groupe.
<details>
<summary>Correction</summary>

| Méthode | $F$ | ddl | p-value |
|---|---|---|---|
| ANOVA classique | 7,92 | 3 ; 9 996 | 2,9e-5 |
| ANOVA de Welch | 15,68 | 3 ; 473,8 | 9,8e-10 |

Les deux concluent à un effet de l'énergie, mais la force change : le groupe réseau de chauffage urbain a un écart-type de 16 086 € (contre 3 424 € pour l'électricité), ce qui gonfle la variance commune de l'ANOVA classique. Welch pondère les groupes par leur précision et donne un $F$ plus élevé. Ce contraste est un signal : il faut vérifier les valeurs extrêmes du réseau avant de présenter un résultat.
</details>

3. Quelle ANOVA présenter dans un rapport où le test de Levene est significatif ? Et quelle comparaison multiple utiliser à la suite ?
   > **Intérêt :** le choix de la méthode se fait avant de regarder le résultat, à partir des conditions de validité.
<details>
<summary>Correction</summary>

On présente l'ANOVA de Welch, parce que son hypothèse de variances égales n'est pas vérifiée. Les comparaisons multiples de Tukey ou de Bonferroni supposent elles aussi des variances comparables : pour les variances inégales, on utilise une comparaison de Welch par paires, ou un test adapté (par exemple celui de Games-Howell, hors programme de ce chapitre). Dans le rapport, on écrit explicitement la méthode choisie et la raison.
</details>

## Exercice 5 – Exercices d'application sur plusieurs couples

Les couples suivants sont à tester avec la même démarche en six étapes. Pour chacun, on remplit : les statistiques par groupe, Levene, l'ANOVA, et les paires significatives si l'ANOVA est significative.

| Couple | Variable quantitative | Facteur (groupes) |
|--------|-----------------------|-------------------|
| A | surface habitable | période de construction (5) |
| B | coût total sur 5 usages | type d'énergie (4) |
| C | ln(consommation par m²) | période de construction (5) |
| D | besoin de chauffage | période de construction (5) |
| E | ln(consommation par m²) | étiquette DPE (7) |

> **Attention : couples piège**
> Le couple E est circulaire : l'étiquette DPE est calculée à partir de la consommation. Une ANOVA significative est alors attendue et ne prouve rien. Le couple D porte des valeurs de besoin de chauffage très élevées, à vérifier avant toute conclusion (voir le chapitre 12).

### Exemple

```r
# Modèle type pour un couple : remplacer la variable et le facteur
modele_A <- aov(surface_habitable_logement ~ periode_r, data = dpe_per)
summary(modele_A)
leveneTest(surface_habitable_logement ~ periode_r, data = dpe_per)
pairwise.t.test(dpe_per$surface_habitable_logement, dpe_per$periode_r,
                p.adjust.method = "bonferroni")
```

### Exercices

1. Appliquer la démarche aux couples A, B, C, D et E. Pour chacun, dire si l'ANOVA est significative, si les variances sont égales (Levene), et quelles paires diffèrent.
   > **Intérêt :** la démarche doit être identique pour chaque couple : c'est ce qui rend la conclusion rigoureuse, même quand le résultat paraît évident.
<details>
<summary>Correction</summary>

| Couple | ANOVA (p) | Levene (p) | Paires significatives (Bonferroni) |
|--------|-----------|------------|------------------------------------|
| A : surface ~ période | 2,2e-37 | 0,076 : égalité acceptable | avant 1948 contre 1948-1974, 1975-1988, 1989-2005 ; 1948-1974 contre 1975-1988 et 2006 et après ; 1975-1988 contre 1989-2005 et 2006 et après ; 1989-2005 contre 2006 et après |
| B : coût ~ énergie | 2,9e-05 | 2,2e-04 : variances différentes | électricité contre réseau ; gaz contre réseau |
| C : ln(conso/m²) ~ période | < 2,2e-16 | 7,9e-28 : variances différentes | toutes les paires |
| D : besoin ~ période | 6,8e-112 | 1,2e-112 : variances différentes | résultat à vérifier : données aberrantes |
| E : ln(conso/m²) ~ étiquette | < 2,2e-16 | 2,3e-127 : variances différentes | toutes les paires (couple circulaire) |

Pour le couple A, l'égalité des variances est acceptable, donc l'ANOVA classique est lue sans réserve. Pour B, C, D et E, les variances diffèrent : il faut le signaler et privilégier l'ANOVA de Welch, ou lire les résultats avec prudence. Le couple E n'a pas de valeur explicative : c'est une vérification de la construction de l'étiquette.
</details>

2. Pour le couple B (coût total selon l'énergie), expliquer pourquoi la comparaison entre électricité et gaz n'est pas significative, alors que celle entre électricité et réseau l'est.
   > **Intérêt :** une différence non significative n'est pas une égalité : elle dit que la différence n'est pas établie avec ces données.
<details>
<summary>Correction</summary>

Les moyennes du coût sont proches entre électricité (1 157 €) et gaz (1 318 €), et la dispersion est forte (écarts-types de 3 400 à 4 300 €). Avec la correction de Bonferroni, la différence de 161 € n'est pas établie (p ajustée = 0,99). Le réseau a une moyenne plus élevée (2 196 €), mais son écart-type est très grand (16 086 €) : quelques valeurs extrêmes tirent la moyenne. La comparaison réseau contre électricité est significative, mais elle repose sur des extrêmes à vérifier.
</details>

3. Rédiger, pour le couple A, une conclusion en trois phrases : le résultat global, les groupes qui diffèrent, et la limite de l'analyse.
   > **Intérêt :** une conclusion doit dire ce qui est établi, dans quel sens, et ce qui ne l'est pas.
<details>
<summary>Correction</summary>

Exemple de rédaction : « La surface moyenne des logements diffère selon la période de construction (ANOVA, F = 44,8, p < 0,001). Les logements construits entre 1975 et 1988 sont en moyenne les plus grands (72 m²), et ceux construits après 2006 les plus petits (46 m²). La liaison est faible (1,8 % de la variance expliquée) et ne dit rien de la consommation : elle décrit seulement la taille des logements. »
</details>

4. Pour chacun des couples A à E, tracer le boxplot bivarié avec la moyenne de chaque groupe. Quels couples montrent un écart visible entre groupes, et lesquels ne le montrent pas ?
   > **Intérêt :** un graphique par couple est la première étape de la démarche : il permet de repérer les effets évidents, les extrêmes et les groupes trop petits avant tout calcul.
<details>
<summary>Correction</summary>

```r
boxplot_bivarie(dpe_per, "surface_habitable_logement", "periode_r", ylim = c(0, 200))   # A
boxplot_bivarie(dpe_per, "cout_total_5_usages", "energie", ylim = c(0, 5000))           # B
boxplot_bivarie(dpe_per, "ln_conso_m2", "periode_r")                                     # C
boxplot_bivarie(dpe_per, "besoin_chauffage", "periode_r", ylim = c(0, 100000))          # D
boxplot_bivarie(dpe_per, "ln_conso_m2", "etiquette_dpe")                                 # E
```
Le couple E montre les écarts les plus nets : les boîtes se décalent à chaque étiquette, parce que l'étiquette est construite à partir de la consommation (couple circulaire). Le couple C montre aussi une baisse régulière de la consommation par m² avec l'époque de construction. Pour A et B, les boîtes se recouvrent davantage ; pour B, les zooms à 5 000 € cachent des valeurs très élevées, notamment dans le réseau. Pour D, le zoom à 100 000 kWh masque les valeurs aberrantes : le graphique ne doit pas être présenté sans vérifier ces données.
</details>

5. Construire le tableau ANOVA de chaque couple avec `tableau_anova()`, puis remplir un tableau de synthèse : SCE inter, CM inter, SCE intra, CM intra, $F$ et p-value.
   > **Intérêt :** un tableau de synthèse permet de comparer l'ampleur des effets entre couples, au-delà de la seule significativité.
<details>
<summary>Correction</summary>

```r
tableau_anova(dpe_per, "surface_habitable_logement", "periode_r")   # A
tableau_anova(dpe_per, "cout_total_5_usages", "energie")            # B
tableau_anova(dpe_per, "ln_conso_m2", "periode_r")                  # C
tableau_anova(dpe_per, "besoin_chauffage", "periode_r")             # D
tableau_anova(dpe_per, "ln_conso_m2", "etiquette_dpe")              # E
```

| Couple | k | N | SCE inter (ddl) | CM inter | SCE intra (ddl) | CM intra | $F$ | p-value | $\eta^2$ |
|--------|---|---|-----------------|----------|-----------------|----------|-----|---------|----------|
| A : surface ~ période | 5 | 9 899 | $6{,}40 \times 10^{5}$ (4) | $1{,}60 \times 10^{5}$ | $3{,}53 \times 10^{7}$ (9 894) | 3 569 | 44,84 | 2,2e-37 | 1,8 % |
| B : coût ~ énergie | 4 | 10 000 | $7{,}35 \times 10^{8}$ (3) | $2{,}45 \times 10^{8}$ | $3{,}09 \times 10^{11}$ (9 996) | $3{,}09 \times 10^{7}$ | 7,92 | 2,9e-5 | 0,24 % |
| C : ln(conso/m²) ~ période | 5 | 9 899 | 460,7 (4) | 115,2 | 1 250 (9 894) | 0,126 | 911,9 | < 2,2e-16 | 26,9 % |
| D : besoin ~ période | 5 | 9 987 | $1{,}15 \times 10^{13}$ (4) | $2{,}88 \times 10^{12}$ | $2{,}14 \times 10^{14}$ (9 982) | $2{,}14 \times 10^{10}$ | 134,2 | 6,8e-112 | 5,1 % |
| E : ln(conso/m²) ~ étiquette | 7 | 9 899 | 1 224 (6) | 204,0 | 486,4 (9 892) | 0,0492 | 4 148,7 | < 2,2e-16 | 71,6 % |

Les couples C et E expliquent une part importante de la variance (27 % et 72 %), ce qui est cohérent avec la construction de l'étiquette. Le couple A est significatif mais explique 1,8 % de la variance de la surface. Le couple D est significatif mais repose sur des valeurs de besoin à vérifier : son $\eta^2$ n'est pas interprétable tant que les données ne sont pas nettoyées.
</details>

## Liens utiles

- [Documentation de `aov()` (R)](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/aov.html)
- [Documentation de `TukeyHSD()` (R)](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/TukeyHSD.html)
- [Documentation de `pairwise.t.test()` (R)](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/pairwise.t.test.html)
- [Documentation du package `car` (`leveneTest`)](https://cran.r-project.org/package=car)
