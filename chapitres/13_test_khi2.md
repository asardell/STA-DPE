# Chapitre 13 – Test du Khi²

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] poser $H_0$, $H_1$, $\alpha$ et $\beta$ pour un test d'indépendance de deux variables qualitatives
- [ ] construire un tableau croisé avec ses effectifs théoriques, et vérifier les conditions du test
- [ ] regrouper les modalités rares quand les effectifs sont trop petits, en justifiant le regroupement
- [ ] réaliser un test du Khi² d'indépendance avec `chisq.test()` et lire la p-value
- [ ] lire une p-value dans la table du Khi² à partir du nombre de degrés de liberté et de la statistique
- [ ] réaliser un test du Khi² d'ajustement à une distribution de référence

> **Contexte :** en bureau d'études, on se demande souvent si deux caractères du bâtiment sont liés : l'étiquette DPE et la période de construction, ou le type d'énergie et la performance. Le test du Khi² dit si cette liaison est établie ou si la répartition observée peut venir du hasard. Il ne dit pas dans quel sens agit la liaison : cela se lit dans les profils (chapitre 5).

## Sommaire

- [Chapitre 13 – Test du Khi²](#chapitre-13--test-du-khi)
  - [Objectifs](#objectifs)
  - [Sommaire](#sommaire)
  - [Préparation : packages, données et notations](#préparation--packages-données-et-notations)
  - [Exercice 1 – Khi² d'indépendance : étiquette et type de bâtiment](#exercice-1--khi-dindépendance--étiquette-et-type-de-bâtiment)
    - [Mémo](#mémo)
    - [Exemple](#exemple)
    - [Exercices](#exercices)
  - [Exercice 2 – Regrouper les modalités rares](#exercice-2--regrouper-les-modalités-rares)
    - [Mémo](#mémo-1)
    - [Exemple](#exemple-1)
    - [Exercices](#exercices-1)
  - [Exercice 3 – Étiquette et période de construction](#exercice-3--étiquette-et-période-de-construction)
    - [Mémo](#mémo-2)
    - [Exemple](#exemple-2)
    - [Exercices](#exercices-2)
  - [Exercice 4 – Type d'énergie et étiquette](#exercice-4--type-dénergie-et-étiquette)
    - [Exemple](#exemple-3)
    - [Exercices](#exercices-3)
  - [Exercice 5 – Lire une p-value dans la table du Khi²](#exercice-5--lire-une-p-value-dans-la-table-du-khi)
    - [Mémo](#mémo-3)
    - [Exemple](#exemple-4)
    - [Exercices](#exercices-4)
  - [Liens utiles](#liens-utiles)

## Préparation : packages, données et notations

```r
library(readr)
library(dplyr)

dpe <- read_csv("data/dpe_villeurbanne_10000.csv")
```

| Notion | Notation | Signification |
|--------|----------|---------------|
| Effectif observé | $O_{ij}$ | nombre de DPE dans la ligne $i$ et la colonne $j$ |
| Effectif théorique | $E_{ij}$ | effectif attendu si les deux variables sont indépendantes |
| Statistique du Khi² | $\chi^2$ | écart global entre observé et théorique |
| Degrés de liberté | ddl | $(\text{nombre de lignes} - 1) \times (\text{nombre de colonnes} - 1)$ |

Pour un test d'indépendance de deux variables qualitatives :

- $H_0$ : les deux variables sont indépendantes (la répartition des modalités de l'une ne dépend pas de l'autre) ;
- $H_1$ : les deux variables sont liées ;
- $\alpha = 0{,}05$, fixé avant le calcul ;
- $\beta$ et la puissance dépendent de la taille de l'échantillon et de l'intensité de la liaison : avec 10 000 DPE, une liaison faible est détectée facilement ;
- règle de décision : on rejette $H_0$ si la p-value est inférieure à $\alpha$.

Les effectifs théoriques se calculent à partir des totaux de ligne et de colonne :

$$
E_{ij} = \frac{n_{i\cdot} \times n_{\cdot j}}{n}
$$

et la statistique s'écrit :

$$
\chi^2 = \sum_{i,j} \frac{(O_{ij} - E_{ij})^2}{E_{ij}}
$$

Sous $H_0$, $\chi^2$ suit une loi du Khi² à $(\text{lignes} - 1) \times (\text{colonnes} - 1)$ degrés de liberté.

> **Note : condition de validité**
> L'approximation par la loi du Khi² n'est fiable que si les effectifs théoriques sont assez grands : en pratique, aucun $E_{ij}$ ne doit être inférieur à 1, et au plus 20 % des $E_{ij}$ peuvent être inférieurs à 5. Si la condition n'est pas remplie, on regroupe les modalités rares.

## Exercice 1 – Khi² d'indépendance : étiquette et type de bâtiment

On teste si l'étiquette DPE et le type de bâtiment sont liés. On construit d'abord le tableau croisé, puis on vérifie les effectifs théoriques avant de lire la p-value.

> **Contexte :** un bureau d'études qui présente la performance par type de logement doit savoir si la répartition des étiquettes diffère vraiment entre appartements et maisons, ou si l'écart vient de la variabilité d'échantillon.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `table()` | Tableau croisé de deux vecteurs (chapitre 5). | `x`, `y` | `table(dpe$etiquette_dpe, dpe$type_batiment)` |
| `chisq.test()` | Test du Khi² d'indépendance ou d'ajustement. | `x`, `p`, `correct` | `chisq.test(tab)` |
| `$expected` | Effectifs théoriques du test. | | `chisq.test(tab)$expected` |
| `$residuals` | Résidus de Pearson : écart standardisé entre observé et théorique. | | `chisq.test(tab)$residuals` |

### Exemple

```r
tab <- table(dpe$etiquette_dpe, dpe$type_batiment)
chisq.test(tab)
# Warning: Chi-squared approximation may be incorrect
# X-squared = 221.08, df = 12, p-value < 2.2e-16

chisq.test(tab)$expected
```

R affiche un avertissement : plusieurs effectifs théoriques sont inférieurs à 5. La p-value est très petite, mais elle n'est pas fiable dans ces conditions.

### Exercices

1. Construire le tableau croisé `etiquette_dpe` × `type_batiment`, puis lancer `chisq.test()`. Quelle est la conclusion au seuil de 5 %, et quel avertissement apparaît ?
   > **Intérêt :** un test significatif avec un avertissement n'est pas une conclusion : il faut d'abord rendre les conditions valables.
<details>
<summary>Correction</summary>

```r
tab <- table(dpe$etiquette_dpe, dpe$type_batiment)
chisq.test(tab)
# X-squared = 221.08, df = 12, p-value < 2.2e-16
# Warning: Chi-squared approximation may be incorrect
```
On rejette $H_0$ : l'étiquette et le type de bâtiment ne sont pas indépendants. Mais l'avertissement indique que cinq des 21 cases ont un effectif théorique inférieur à 5. Par exemple, le type « immeuble » ne compte que 79 DPE, et son effectif théorique en A est de 0,2.
</details>

2. Afficher les effectifs théoriques avec `$expected`. Combien de cases ont un effectif théorique inférieur à 5 ?
   > **Intérêt :** avant de regrouper, on sait exactement quelles cases posent problème : le regroupement doit cibler les cases fragiles, pas réduire le tableau au hasard.
<details>
<summary>Correction</summary>

```r
round(chisq.test(tab)$expected, digits = 1)
# A : 18.2 ; 0.6 ; 0.2
# B : 222.5 ; 7.6 ; 1.8
# ...
```
Les cinq cases en dessous de 5 sont : maison en A (0,6), immeuble en A (0,2), immeuble en B (1,8), immeuble en F (4,3) et immeuble en G (2,3). Cinq cases sur 21, soit près de 24 %, dépassent la limite de 20 %.
</details>

## Exercice 2 – Regrouper les modalités rares

Quand des modalités ont des effectifs trop petits, on les regroupe avec une modalité voisine qui a du sens. Le regroupement doit être décidé à partir du sens métier, pas du seul résultat du test.

> **Contexte :** les étiquettes A et B ne représentent qu'une fraction très faible des DPE (A : 19, B : 232). Les présenter séparément rend le tableau illisible, et les effectifs théoriques trop petits.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `case_when()` (dplyr) | Recode une variable selon des conditions. | conditions et valeurs | `case_when(etiquette_dpe %in% c("A", "B") ~ "A-B"` |
| `if_else()` (dplyr) | Recode une variable en deux modalités. | condition, valeur si vrai, valeur si faux | `if_else(type_batiment == "appartement", "appartement", "maison ou immeuble")` |

### Exemple

```r
dpe_r <- dpe |>
  mutate(
    etiq_r = case_when(
      etiquette_dpe %in% c("A", "B") ~ "A-B",
      etiquette_dpe %in% c("C", "D", "E") ~ "C-E",
      etiquette_dpe %in% c("F", "G") ~ "F-G"
    ),
    type_r = if_else(type_batiment == "appartement", "appartement", "maison ou immeuble")
  )

tab_r <- table(dpe_r$etiq_r, dpe_r$type_r)
tab_r
chisq.test(tab_r, correct = FALSE)
```

### Exercices

1. Regrouper les étiquettes A et B, puis lancer le test avec le type de bâtiment en trois modalités. Combien de cases restent inférieures à 5 ?
   > **Intérêt :** regrouper une modalité est acceptable quand le regroupement garde un sens (deux classes de haute performance) ; il ne doit pas servir à obtenir la p-value voulue.
<details>
<summary>Correction</summary>

```r
dpe_r <- dpe |>
  mutate(etiq_r = case_when(
    etiquette_dpe %in% c("A", "B") ~ "A-B",
    etiquette_dpe %in% c("C", "D", "E") ~ "C-E",
    etiquette_dpe %in% c("F", "G") ~ "F-G"
  ))

tab1 <- table(dpe_r$etiq_r, dpe$type_batiment)
chisq.test(tab1)
# X-squared = 201.48, df = 8, p-value < 2.2e-16
# Warning: Chi-squared approximation may be incorrect (une case : immeuble en A-B, effectif théorique 2,0)
```
Une seule case reste inférieure à 5. Il faut aller plus loin : le type « immeuble » reste trop petit.
</details>

2. Regrouper « maison » et « immeuble » en une seule modalité « maison ou immeuble », puis refaire le test avec les étiquettes A-B, C, D, E, F-G. Quelle est la conclusion ? Ce regroupement se justifie-t-il ?
   > **Intérêt :** un regroupement se justifie par le sens des modalités (habitat individuel contre collectif) et doit être annoncé dans le rapport, avec ses limites.
<details>
<summary>Correction</summary>

```r
tab2 <- table(dpe_r$etiq_r, dpe_r$type_r)
round(chisq.test(tab2)$expected, digits = 1)
chisq.test(tab2, correct = FALSE)
# X-squared = 174.03, df = 4, p-value < 2.2e-16
```
Toutes les cases ont un effectif théorique supérieur à 5 (minimum 10,2). On rejette $H_0$ : la répartition des étiquettes diffère entre appartements et « maisons ou immeubles ». Le regroupement est justifié par le fait que la maison et l'immeuble sont des habitats non appartements, mais il change la question : on ne compare plus trois types, seulement appartement et non appartement. Ce choix doit être écrit dans le rapport.
</details>

## Exercice 3 – Étiquette et période de construction

La période de construction est une variable qualitative ordonnée : on la regroupe en cinq périodes, car les effectifs de certaines périodes (1975-1977, 1978-1982, 1983-1988) sont trop petits pour être présentés séparément. Les périodes sont regroupées par tranches voisines, en gardant les périodes les plus anciennes séparées.

> **Contexte :** dans un audit de parc, savoir si l'étiquette dépend de l'âge du bâtiment permet de cibler les actions : plus de travaux sur le parc ancien, ou un problème de conception sur des constructions récentes.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `case_when()` | Regroupe les périodes. | conditions | voir l'exemple |
| `prop.table()` (chapitre 5) | Profils lignes pour lire la répartition des étiquettes par période. | `margin = 1` | `prop.table(tab_per, margin = 1)` |

### Exemple

```r
dpe_per <- dpe |>
  mutate(
    etiq_r = case_when(
      etiquette_dpe %in% c("A", "B") ~ "A-B",
      etiquette_dpe %in% c("C", "D", "E") ~ "C-E",
      etiquette_dpe %in% c("F", "G") ~ "F-G"
    ),
    periode_r = case_when(
      periode_construction == "avant 1948" ~ "avant 1948",
      periode_construction == "1948-1974" ~ "1948-1974",
      periode_construction %in% c("1975-1977", "1978-1982", "1983-1988") ~ "1975-1988",
      periode_construction %in% c("1989-2000", "2001-2005") ~ "1989-2005",
      TRUE ~ "2006 et après"
    )
  )

tab_per <- table(dpe_per$periode_r, dpe_per$etiq_r)
chisq.test(tab_per, correct = FALSE)
# X-squared = 4028.5, df = 16, p-value < 2.2e-16
```

### Exercices

1. Tester l'indépendance entre la période de construction regroupée et l'étiquette regroupée. Quelle est la conclusion ?
   > **Intérêt :** une p-value qui s'affiche « < 2,2e-16 » ne dit pas la force de la liaison : il faut la lire dans les profils.
<details>
<summary>Correction</summary>

```r
chisq.test(tab_per, correct = FALSE)
# X-squared = 4028.5, df = 16, p-value < 2.2e-16
```
On rejette $H_0$ : l'étiquette dépend de la période de construction. La liaison est très forte, et toutes les cases ont un effectif théorique suffisant.
</details>

2. Calculer les profils lignes et donner la part des DPE en F ou G pour chaque période. Dans quel sens agit la liaison ?
   > **Intérêt :** le Khi² ne donne pas le sens de la liaison : seul le profil le montre, comme au chapitre 5.
<details>
<summary>Correction</summary>

```r
round(100 * prop.table(tab_per, margin = 1)[, "F-G"], digits = 1)
# avant 1948 : 18.8 ; 1948-1974 : 12.8 ; 1975-1988 : 1.9 ; 1989-2005 : 0.3 ; 2006 et après : 0.3
```
La part de F-G est la plus élevée pour les bâtiments les plus anciens, puis baisse fortement après 1975. Le sens de la liaison est donc : plus le bâtiment est ancien, plus il est mal classé. Les constructions après 1975 ont une part de F-G proche de zéro.
</details>

3. Refaire le test avec les cinq étiquettes d'origine (A à G) sans regroupement. Le test est-il toujours valable ?
   > **Intérêt :** regrouper des modalités n'est pas une option cosmétique : l'exercice montre quelles cases ne tiennent pas sans regroupement.
<details>
<summary>Correction</summary>

```r
tab_per_b <- table(dpe_per$periode_construction, dpe$etiquette_dpe)
chisq.test(tab_per_b)$expected |> round(digits = 1)
```
Avec les 7 étiquettes et les 10 périodes d'origine, 13 cases sur 70 ont un effectif théorique inférieur à 5, soit 19 % : la limite est tout juste respectée, mais certaines cases sont très petites (0,1 pour « après 2021 » en A, avec 31 DPE dans toute cette période). Le test donne encore X² = 4 620 avec 54 ddl, mais la lecture par modalité devient fragile. C'est pourquoi on regroupe.
</details>

## Exercice 4 – Type d'énergie et étiquette

Le type d'énergie de chauffage a de nombreuses modalités très rares : bois, propane, charbon, GPL, électricité renouvelable. On garde les trois énergies principales (électricité, gaz, réseau de chauffage urbain) et le fioul, et on regroupe les autres.

> **Contexte :** savoir si l'énergie de chauffage est liée à l'étiquette est utile pour décider où porter les efforts (conversion du fioul, raccordement au réseau). Les énergies rares ne doivent pas faire disparaître l'analyse, mais elles ne doivent pas non plus fausser le test.

### Exemple

```r
dpe_en <- dpe |>
  mutate(
    energie_r = case_when(
      type_energie_n1 == "Électricité" ~ "Électricité",
      type_energie_n1 == "Gaz naturel" ~ "Gaz naturel",
      type_energie_n1 == "Réseau de Chauffage urbain" ~ "Réseau",
      type_energie_n1 == "Fioul domestique" ~ "Fioul et autres",
      TRUE ~ "Fioul et autres"
    ),
    etiq_r = case_when(
      etiquette_dpe %in% c("A", "B") ~ "A-B",
      etiquette_dpe %in% c("C", "D", "E") ~ "C-E",
      etiquette_dpe %in% c("F", "G") ~ "F-G"
    )
  )

tab_en <- table(dpe_en$energie_r, dpe_en$etiq_r)
chisq.test(tab_en)
```

### Exercices

1. Tester l'indépendance entre le type d'énergie regroupé et l'étiquette regroupée. Quelle est la conclusion, et combien de cases ont un effectif théorique inférieur à 5 ?
   > **Intérêt :** une énergie peu présente ne doit pas être supprimée du test, mais regroupée avec des modalités proches pour garder un effectif exploitable.
<details>
<summary>Correction</summary>

```r
chisq.test(tab_en)
# X-squared = 740.68, df = 12, p-value < 2.2e-16
```
On rejette $H_0$ : l'étiquette dépend du type d'énergie. Une seule case (« Fioul et autres » en A-B, effectif théorique 2,6) reste inférieure à 5, ce qui reste dans la limite des 20 %.
</details>

2. Calculer la part de F-G pour chaque énergie. Quelle énergie a la plus forte part, et pourquoi ce résultat est-il fragile ?
   > **Intérêt :** une part très élevée sur un petit effectif doit toujours être présentée avec son effectif.
<details>
<summary>Correction</summary>

```r
round(100 * prop.table(tab_en, margin = 1)[, "F-G"], digits = 1)
# Électricité : 9.0 ; Gaz naturel : 7.2 ; Réseau : 3.2 ; Fioul et autres : 54.9
```
La catégorie « Fioul et autres » a la part de F-G la plus élevée (54,9 %), mais elle regroupe seulement 102 DPE, dont 80 de fioul. Le résultat est fragile : il faut le présenter avec l'effectif, et le regroupement doit être mentionné.
</details>

## Exercice 5 – Lire une p-value dans la table du Khi²

Quand on n'a pas de logiciel, la p-value se lit dans la table du Khi² : une ligne par degré de liberté, une colonne par valeur de la statistique $\chi^2$. On reproduit cette table, puis on lit une p-value qui n'est pas proche de zéro.

La p-value est la probabilité qu'une loi du Khi² à $d$ ddl dépasse la valeur observée :

$$
p = P(\chi^2_d > \chi^2_{\text{obs}})
$$

> **Contexte :** la table est utile pour vérifier rapidement un résultat dans un rapport, ou pour comprendre pourquoi une valeur de $\chi^2$ devient significative selon le nombre de catégories.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `pchisq()` | Probabilité de dépasser une valeur. | `q`, `df`, `lower.tail = FALSE` | `pchisq(4.93, df = 1, lower.tail = FALSE)` |
| `qchisq()` | Valeur critique au seuil donné. | `p`, `df` | `qchisq(0.95, df = 1)` |
| `outer()` | Tableau complet pour toutes les combinaisons de ddl et de statistiques. | `X`, `Y`, `FUN` | voir l'exemple |

### Exemple

```r
stats <- c(1, 2, 4, 6, 8, 10, 15, 20)
ddls  <- c(1, 2, 4, 6, 8, 12)

table_chi2 <- outer(ddls, stats, function(df, x) pchisq(x, df = df, lower.tail = FALSE))
dimnames(table_chi2) <- list(ddl = ddls, X2 = stats)
round(table_chi2, digits = 4)
#      X2
# ddl     1      2      4      6      8     10     15     20
# 1     0.3173 0.1573 0.0455 0.0143 0.0047 0.0016 0.0001 0.0000
# 2     0.6065 0.3679 0.1353 0.0498 0.0183 0.0067 0.0006 0.0000
# 4     0.9098 0.7358 0.4060 0.1991 0.0916 0.0404 0.0047 0.0005
# 6     0.9856 0.9197 0.6767 0.4232 0.2381 0.1247 0.0203 0.0028
# 8     0.9982 0.9810 0.8571 0.6472 0.4335 0.2650 0.0591 0.0103
# 12    1.0000 0.9994 0.9834 0.9161 0.7851 0.6160 0.2414 0.0671

qchisq(0.95, df = c(1, 2, 4, 6, 8, 12))
# 3.841 5.991 9.488 12.592 15.507 21.026
```

### Exercices

1. Reprendre le tableau 2 × 2 de l'électricité et du gaz selon le type de bâtiment (appartement ou maison). Calculer le Khi² sans correction (`correct = FALSE`), puis lire la p-value dans la table pour 1 ddl. La valeur obtenue est-elle significative ?
   > **Intérêt :** une p-value de 0,03 est une lecture de table ; le calcul exact dit si la lecture était juste, et montre où la table est approximative.
<details>
<summary>Correction</summary>

```r
tab_am <- table(dpe$type_energie_n1[dpe$type_energie_n1 %in% c("Électricité", "Gaz naturel")],
                dpe$type_batiment[dpe$type_energie_n1 %in% c("Électricité", "Gaz naturel")])
tab_am <- tab_am[, c("appartement", "maison")]
chisq.test(tab_am, correct = FALSE)
# X-squared = 4.93, df = 1, p-value = 0.0263
```
Dans la table pour 1 ddl, $\chi^2 = 4{,}93$ se situe entre 4 (p = 0,0455) et 6 (p = 0,0143). Une interpolation donne environ 0,031. La valeur exacte est 0,026 : elle est significative au seuil de 5 %, donc on rejette $H_0$. L'écart avec l'interpolation montre qu'une lecture de table est approximative, et qu'elle dépasse la valeur réelle ici.

> **Note : correction de Yates**
> Pour un tableau 2 × 2, `chisq.test()` applique par défaut une correction de continuité (Yates). Ici on l'a désactivée avec `correct = FALSE` pour retrouver la formule de la table. La correction réduit la statistique, donc la p-value avec correction serait un peu plus élevée.
</details>

2. Lire la p-value pour $\chi^2 = 10$ à 4 ddl dans la table, puis la comparer à `pchisq()`. Quelle est la valeur critique à 5 % pour 4 ddl ?
   > **Intérêt :** la valeur critique indique directement le seuil de décision : une statistique qui le dépasse est significative.
<details>
<summary>Correction</summary>

```r
round(pchisq(10, df = 4, lower.tail = FALSE), digits = 4)   # 0.0404
qchisq(0.95, df = 4)                                         # 9.488
```
La table donne 0,0404 pour $\chi^2 = 10$ et 4 ddl, ce qui est significatif au seuil de 5 %. La valeur critique est 9,488 : toute statistique supérieure est significative.
</details>

## Liens utiles

- [Documentation de `chisq.test()` (R)](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/chisq.test.html)
- [Documentation de `pchisq()` et `qchisq()` (R)](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/Chisquare.html)
