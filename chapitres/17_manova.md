# Chapitre 17 – MANOVA : comparer des groupes sur plusieurs variables

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] expliquer pourquoi comparer plusieurs variables groupe par groupe, une à une, ne suffit pas
- [ ] construire les matrices de sommes des carrés et des produits $W$, $B$ et $T$, et vérifier $T = W + B$
- [ ] calculer le lambda de Wilks et la statistique $F$ associée, et lire la p-value
- [ ] réaliser une MANOVA avec `manova()` et `summary()`, puis la comparer aux ANOVA de suivi
- [ ] représenter les groupes dans le plan de deux variables réponses

> **Contexte :** en bureau d'études, un même bâtiment se décrit par plusieurs grandeurs : sa consommation par m², ses déperditions, son coût. Comparer les périodes de construction sur chaque grandeur séparément multiplie les tests et ignore la liaison entre les grandeurs. La MANOVA teste d'abord si les groupes diffèrent sur l'ensemble des variables, comme un vecteur. Elle répond à une question globale avant de détailler variable par variable.

## Sommaire

- [Chapitre 17 – MANOVA : comparer des groupes sur plusieurs variables](#chapitre-17--manova--comparer-des-groupes-sur-plusieurs-variables)
  - [Objectifs](#objectifs)
  - [Sommaire](#sommaire)
  - [Préparation : packages et données](#préparation--packages-et-données)
  - [Démarche : hypothèses, matrices et statistique](#démarche--hypothèses-matrices-et-statistique)
  - [Un exemple pour comprendre : la croissance des plantes](#un-exemple-pour-comprendre--la-croissance-des-plantes)
  - [Exercice 1 – Moyennes par groupe et graphique bivarié](#exercice-1--moyennes-par-groupe-et-graphique-bivarié)
  - [Exercice 2 – Matrices $W$, $B$ et $T$](#exercice-2--matrices-w-b-et-t)
  - [Exercice 3 – Lambda de Wilks et test multivarié](#exercice-3--lambda-de-wilks-et-test-multivarié)
  - [Exercice 4 – ANOVA de suivi, variable par variable](#exercice-4--anova-de-suivi-variable-par-variable)
  - [Exercice 5 – Pourquoi tester le vecteur d'abord ?](#exercice-5--pourquoi-tester-le-vecteur-dabord-)
  - [Liens utiles](#liens-utiles)

## Préparation : packages et données

```r
library(readr)
library(dplyr)
library(ggplot2)

dpe <- read_csv("data/dpe_villeurbanne_10000.csv")

# Deux variables réponses : consommation par m² et déperditions des murs (logarithmes)
dpe_mv <- dpe |>
  filter(!is.na(surface_habitable_logement), conso_5_usages_ep > 0, deperditions_murs > 0) |>
  mutate(
    ln_conso_m2 = log(conso_5_usages_ep / surface_habitable_logement),
    ln_deperd = log(deperditions_murs),
    periode3 = factor(case_when(
      periode_construction %in% c("avant 1948", "1948-1974") ~ "avant 1975",
      periode_construction %in% c("1975-1977", "1978-1982", "1983-1988",
                                  "1989-2000", "2001-2005") ~ "1975-2005",
      TRUE ~ "2006 et après"
    ), levels = c("avant 1975", "1975-2005", "2006 et après"))
  )
```

Les logarithmes stabilisent la dispersion, comme au chapitre 9. Le fichier retient 9 886 DPE : ceux qui ont une surface, une consommation et des déperditions renseignées et positives.

> **Note : la MANOVA suppose des variables réponses liées**
> La MANOVA est utile quand les variables réponses sont corrélées : elle exploite cette liaison. Si elles étaient indépendantes, tester chaque variable séparément suffirait.

## Démarche : hypothèses, matrices et statistique

Pour trois groupes ($k = 3$) et deux variables réponses, les hypothèses sont :

- $H_0$ : les vecteurs de moyennes des trois groupes sont égaux, $\boldsymbol{\mu}_1 = \boldsymbol{\mu}_2 = \boldsymbol{\mu}_3$ ;
- $H_1$ : au moins un vecteur de moyennes diffère des autres.

La variabilité se découpe en trois matrices de sommes des carrés et des produits (SCP) :

- $W$ : variabilité à l'intérieur des groupes, $W = \sum_{g} \sum_{i \in g} (\mathbf{y}_i - \bar{\mathbf{y}}_g)(\mathbf{y}_i - \bar{\mathbf{y}}_g)^\top$ ;
- $B$ : variabilité entre les groupes, $B = \sum_{g} n_g (\bar{\mathbf{y}}_g - \bar{\mathbf{y}})(\bar{\mathbf{y}}_g - \bar{\mathbf{y}})^\top$ ;
- $T$ : variabilité totale, $T = W + B$.

Les sommes des carrés sont sur la diagonale, les produits hors diagonale. Le lambda de Wilks compare la variabilité résiduelle à la variabilité totale :

$$
\Lambda = \frac{\det(W)}{\det(T)}
$$

Plus $\Lambda$ est petit, plus les groupes sont séparés. Pour deux variables réponses et $k$ groupes, la transformation suivante donne une statistique $F$ exacte :

$$
F = \frac{1 - \sqrt{\Lambda}}{\sqrt{\Lambda}} \times \frac{N - k - 1}{k - 1}, \qquad \text{ddl} = 2(k - 1) \text{ et } 2(N - k - 1)
$$

Le test est bilatéral et se lit comme une ANOVA : on rejette $H_0$ si la p-value est inférieure à 0,05.

> **Note : Pillai et les autres statistiques**
> `summary()` affiche par défaut la trace de Pillai, $V = \mathrm{tr}(B T^{-1})$, qui est plus robuste aux écarts à la normalité. Les quatre statistiques (Pillai, Wilks, Hotelling-Lawley, Roy) donnent en général la même conclusion ici. On présente le lambda de Wilks, dont la formule est explicite.

## Un exemple pour comprendre : la croissance des plantes

Reprenons l'exemple du chapitre 16. Trois engrais (E1, E2, E3) sont appliqués à 6 plantes chacun. On mesure deux choses sur chaque plante après 30 jours : la **hauteur** (cm) et le **nombre de feuilles**. Les données sont fictives.

| Engrais | Hauteur (cm) | Nombre de feuilles |
|---|---|---|
| E1 | 17 ; 19 ; 20 ; 22 ; 23 ; 25 | 8 ; 9 ; 10 ; 10 ; 11 ; 12 |
| E2 | 20 ; 21 ; 23 ; 25 ; 26 ; 28 | 9 ; 10 ; 11 ; 11 ; 12 ; 13 |
| E3 | 16 ; 18 ; 19 ; 21 ; 22 ; 24 | 12 ; 13 ; 12 ; 14 ; 13 ; 15 |

Moyennes par engrais : E1 (21,0 cm ; 10,0 feuilles), E2 (23,8 cm ; 11,0 feuilles), E3 (20,0 cm ; 13,2 feuilles). Moyenne générale : (21,6 cm ; 11,4 feuilles).

### Première approche : deux ANOVA séparées

La question est la même pour les deux variables : l'engrais a-t-il un effet ? On lance donc une ANOVA sur la hauteur, puis une sur le nombre de feuilles.

| Variable | $F$ | ddl | p-value | Part de variance expliquée |
|---|---|---|---|---|
| Hauteur | 2,72 | (2 ; 15) | 0,098 | 27 % |
| Nombre de feuilles | 8,79 | (2 ; 15) | 0,003 | 54 % |

Avec ces deux tests, la hauteur n'est pas significative, et le nombre de feuilles l'est. Si l'on s'arrêtait là, on dirait « l'engrais agit sur les feuilles, pas sur la hauteur ». Mais on a fait deux tests : le risque d'au moins un faux positif grimpe, et chaque test ignore ce que l'autre mesure.

### Deuxième approche : une MANOVA

La MANOVA pose la question une seule fois : les trois engrais ont-ils le même **vecteur** de moyennes (hauteur ; feuilles) ? Elle compare les matrices de variabilité :

$$
W = \begin{pmatrix} 130{,}8 & 55{,}0 \\ 55{,}0 & 26{,}8 \end{pmatrix}, \qquad
B = \begin{pmatrix} 47{,}4 & -17{,}3 \\ -17{,}3 & 31{,}4 \end{pmatrix}, \qquad
T = W + B = \begin{pmatrix} 178{,}3 & 37{,}7 \\ 37{,}7 & 58{,}3 \end{pmatrix}
$$

Le lambda de Wilks vaut $\Lambda = \det(W) / \det(T) = 485{,}3 / 8\,967{,}2 = 0{,}054$. Il est très petit : la variabilité à l'intérieur des engrais est faible par rapport à la variabilité totale. La statistique $F$ exacte (deux variables) donne $F = 23{,}1$ avec $(4 ; 28)$ ddl, et p = 1,6e-8 : on rejette $H_0$.

### Pourquoi la MANOVA voit ce que les ANOVA ne voient pas

Dans chaque engrais, la hauteur et le nombre de feuilles sont très corrélés : la corrélation intra-groupes vaut 0,93. Une plante qui a plus de feuilles est aussi plus haute, à l'intérieur de son engrais. La MANOVA exploite cette liaison.

Entre engrais, le signe de la covariance est négatif ($-17{,}3$) : E3 a le plus de feuilles (13,2) mais une hauteur plus faible (20,0 cm). L'écart entre les engrais se lit donc dans une **direction** de l'espace des deux variables : plus de feuilles, mais pas plus de hauteur. Aucune des deux ANOVA ne voit cette direction ; la MANOVA la teste en une fois.

> **Note : la MANOVA n'est pas toujours plus puissante**
> Si les deux variables ne sont pas corrélées, la MANOVA n'apporte rien et perd des degrés de liberté. Elle est utile quand les variables réponses sont liées et que l'effet se lit dans leur combinaison.

### Comparaison : ANOVA, plusieurs ANOVA et MANOVA

| | ANOVA (une variable) | Plusieurs ANOVA | MANOVA |
|---|---|---|---|
| Question posée | les moyennes d'une variable sont-elles égales entre groupes ? | la même question, variable par variable | les vecteurs de moyennes sont-ils égaux entre groupes ? |
| Nombre de tests | 1 | 1 par variable (ici 2) | 1 (global) |
| Risque global de faux positif | 5 % | augmente avec le nombre de tests (corriger par Bonferroni) | 5 % pour l'ensemble |
| Prend en compte la liaison entre variables | sans objet | non | oui (matrices $W$, $B$, $T$) |
| Résultat sur l'exemple | hauteur : p = 0,098 | hauteur : p = 0,098 ; feuilles : p = 0,003 | p = 1,6e-8 |
| Ce qu'elle dit | un écart existe ou non sur une variable | où se situe l'écart, variable par variable | qu'il existe un écart sur la combinaison des variables |
| Ce qu'elle ne dit pas | rien sur les autres variables | rien sur la liaison entre variables | quelle variable porte l'écart : il faut les ANOVA de suivi |

> **Hypothèses à vérifier** (non vérifiées dans cet exemple, pour ne pas alourdir le chapitre) : normalité multivariée (chaque variable, et leur combinaison, approximativement normale dans chaque groupe) ; mêmes matrices de covariance dans les groupes (test de Box, à interpréter avec prudence) ; observations indépendantes.

## Exercice 1 – Moyennes par groupe et graphique bivarié

Avant les matrices, on calcule le vecteur des moyennes par groupe et on représente les DPE dans le plan des deux variables.

> **Contexte :** un graphique bivarié montre si les groupes se séparent ou se recouvrent. C'est le premier contrôle avant tout calcul multivarié.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `group_by()` + `summarise()` | Vecteur des moyennes par groupe. | | `summarise(across(c(ln_conso_m2, ln_deperd), mean))` |
| `geom_point()` + `stat_summary()` | Nuage et moyennes de chaque groupe dans le plan. | `alpha`, `fun` | `stat_summary(fun = mean, geom = "point", shape = 23)` |

### Exemple

```r
dpe_mv |>
  group_by(periode3) |>
  summarise(across(c(ln_conso_m2, ln_deperd), mean), effectif = n())
```

### Exercices

1. Calculer le vecteur des moyennes de chaque période de construction. Dans quel sens varient les deux variables quand la période est plus récente ?
   > **Intérêt :** le vecteur des moyennes est l'objet central de la MANOVA : c'est lui que l'on compare entre groupes.
<details>
<summary>Correction</summary>

| Période | Effectif | $\overline{\ln(\text{conso/m}^2)}$ | $\overline{\ln(\text{déperditions})}$ |
|---|---|---|---|
| avant 1975 | 5 252 | 5,429 | 4,052 |
| 1975-2005 | 2 946 | 5,053 | 3,265 |
| 2006 et après | 1 688 | 4,945 | 2,540 |

Les deux variables baissent quand la période est plus récente. La baisse des déperditions est nettement plus forte (1,51 sur le logarithme, contre 0,48 pour la consommation par m²).
</details>

2. Représenter le nuage des DPE dans le plan (déperditions, consommation par m²), coloré par période, avec les moyennes de chaque groupe. Les groupes se recouvrent-ils ?
   > **Intérêt :** un nuage avec 9 886 points devient vite illisible : une transparence faible et les moyennes en losange rendent la lecture possible.
<details>
<summary>Correction</summary>

```r
ggplot(dpe_mv, aes(x = ln_deperd, y = ln_conso_m2, colour = periode3)) +
  geom_point(alpha = 0.05) +
  stat_summary(fun = mean, geom = "point", shape = 23, size = 4, fill = "white") +
  labs(x = "ln(déperditions des murs)", y = "ln(conso / m²)", colour = "Période")
```
Les nuages se recouvrent largement, mais les losanges des moyennes s'écartent dans la direction « récent : moins de déperditions et moins de consommation ». La séparation est donc nette sur les moyennes et faible sur les individus.
</details>

## Exercice 2 – Matrices $W$, $B$ et $T$

On calcule les trois matrices à partir des données. Le contrôle essentiel est $T = W + B$.

### Exemple

```r
modele_manova <- manova(cbind(ln_conso_m2, ln_deperd) ~ periode3, data = dpe_mv)
summary(modele_manova, test = "Wilks")
```

Les matrices se calculent à partir des résidus de chaque groupe (pour $W$) et des écarts des moyennes de groupe à la moyenne générale (pour $B$). Dans un exercice, on les construit directement.

### Exercices

1. Calculer $W$ et $T$ à partir des données, puis $B = T - W$. Vérifier que $B$ est bien la matrice des écarts entre les moyennes de groupe pondérés par les effectifs.
   > **Intérêt :** la décomposition $T = W + B$ est une identité : si elle ne se vérifie pas, une des matrices est fausse.
<details>
<summary>Correction</summary>

Matrices obtenues sur les 9 886 DPE (arrondies à l'unité) :

$$
W = \begin{pmatrix} 1\,270{,}9 & 710{,}9 \\ 710{,}9 & 24\,035{,}0 \end{pmatrix}, \qquad
B = \begin{pmatrix} 436{,}8 & 1\,157{,}7 \\ 1\,157{,}7 & 3\,280{,}0 \end{pmatrix}, \qquad
T = \begin{pmatrix} 1\,707{,}7 & 1\,868{,}6 \\ 1\,868{,}6 & 27\,315{,}0 \end{pmatrix}
$$

Le contrôle $W + B = T$ tient à l'arrondi près. Les termes hors diagonale positifs (710,9 et 1 157,7) indiquent que les deux variables varient dans le même sens à l'intérieur des groupes comme entre eux.
</details>

2. Calculer $\det(W)$, $\det(T)$ et le lambda de Wilks avec R (`det()`).
   > **Intérêt :** $\Lambda$ se lit comme la part de variabilité qui n'est pas expliquée par le groupe.
<details>
<summary>Correction</summary>

```r
det(W) / det(T)
# [1] 0.696
```
Le lambda vaut 0,696 : environ 70 % de la variabilité totale reste à l'intérieur des groupes, et 30 % est expliquée par la période.
</details>

## Exercice 3 – Lambda de Wilks et test multivarié

On calcule la statistique $F$ à partir de $\Lambda$, puis on lit la p-value.

### Exemple

```r
summary(modele_manova, test = "Wilks")
# Wilks = 0.696, approx F = 981, df = 4 and 19764, p-value < 2.2e-16
```

### Exercices

1. Calculer la statistique $F$ à partir de $\Lambda = 0{,}696$, avec $N = 9\,886$ et $k = 3$. Vérifier qu'elle est proche de la valeur de `summary()`.
   > **Intérêt :** une formule exacte permet de contrôler la sortie du logiciel, sans l'accepter comme une boîte noire.
<details>
<summary>Correction</summary>

$F = \dfrac{1 - \sqrt{0{,}696}}{\sqrt{0{,}696}} \times \dfrac{9\,886 - 3 - 1}{3 - 1} \approx 0{,}1986 \times 4\,941 \approx 981$, avec $2(k-1) = 4$ et $2(N-k-1) = 19\,764$ ddl. La valeur est identique à celle de `summary()`.
</details>

2. Conclure au seuil de 5 %. Que dit le test sur les groupes, et que ne dit-il pas ?
   > **Intérêt :** un test multivarié significatif ne dit pas quelle variable porte la différence : il faut les tests de suivi.
<details>
<summary>Correction</summary>

On rejette $H_0$ (p < 2,2e-16) : les trois périodes de construction ne partagent pas le même vecteur de moyennes. Le test ne dit pas laquelle des deux variables porte l'écart, ni si les écarts vont dans le même sens. C'est le rôle des ANOVA de suivi (exercice 4).
</details>

3. Calculer la trace de Pillai $V = \mathrm{tr}(B T^{-1})$, qui est la statistique par défaut de `summary()`. Vérifier qu'elle est comprise entre 0 et 1, et plus grande quand les groupes sont plus séparés.
   > **Intérêt :** la statistique par défaut ne suffit pas à elle seule : savoir ce qu'elle mesure permet de la lire.
<details>
<summary>Correction</summary>

Avec $V = \mathrm{tr}(B T^{-1}) = 0{,}306$ : c'est la part de la variabilité totale expliquée par les groupes, mesurée sur les deux variables à la fois. Cette lecture est cohérente avec $\Lambda = 0{,}696$ (environ 30 % de variabilité expliquée). Dans R, `summary(modele_manova)` sans `test = "Wilks"` affiche $V$ et sa p-value.
</details>

## Exercice 4 – ANOVA de suivi, variable par variable

Quand le test multivarié est significatif, on regarde chaque variable avec une ANOVA. Avec deux variables, on compare les p-values à $0{,}05 / 2 = 0{,}025$ (correction de Bonferroni, chapitre 14).

### Exemple

```r
summary.aov(modele_manova)
```

### Exercices

1. Lire les deux ANOVA de suivi. Pour chaque variable, donner le $F$, les ddl et la p-value.
   > **Intérêt :** les ANOVA de suivi disent quelle variable porte l'écart, mais elles ignorent la liaison entre les variables.
<details>
<summary>Correction</summary>

| Variable | SCE inter (2 ddl) | SCE intra (9 883 ddl) | $F$ | p-value |
|---|---|---|---|---|
| ln(conso / m²) | 436,8 | 1 270,9 | 1 698,5 | < 2,2e-16 |
| ln(déperditions) | 3 280,0 | 24 035,0 | 674,4 | < 2,2e-16 |

Les deux variables diffèrent entre périodes (p-values toutes deux inférieures à 2,2e-16). Le $F$ de la consommation est plus élevé ; la question de la part de variance expliquée est traitée à la question suivante.
</details>

2. Calculer la part de variance expliquée $\eta^2$ pour chaque variable. Laquelle est la plus liée à la période ?
   > **Intérêt :** une p-value très petite ne mesure pas l'importance de la liaison : $\eta^2$ la mesure.
<details>
<summary>Correction</summary>

$\eta^2 = \text{SCE}_{\text{inter}} / (\text{SCE}_{\text{inter}} + \text{SCE}_{\text{intra}})$ :
- ln(conso / m²) : $436{,}8 / (436{,}8 + 1\,270{,}9) = 25{,}6\ \%$ ;
- ln(déperditions) : $3\,280{,}0 / (3\,280{,}0 + 24\,035{,}0) = 12{,}0\ \%$.

La consommation par m² est donc la variable la plus liée à la période, malgré un $F$ plus faible.
</details>

## Exercice 5 – Pourquoi tester le vecteur d'abord ?

> **Contexte :** dans un rapport, présenter cinq ou six ANOVA séparées sans test global multiplie les risques d'interpréter un écart fortuit. La MANOVA répond d'abord à la question générale, puis les ANOVA précisent.

1. Si l'on lançait deux ANOVA sans MANOVA, quel risque apparaît ? Quelle correction s'impose ?
   > **Intérêt :** chaque test a son risque d'erreur : plusieurs tests en même temps accumulent ces risques.
<details>
<summary>Correction</summary>

Deux tests à 5 % donnent environ 10 % de chances d'au moins un faux positif si les tests sont indépendants. Il faut donc une correction (Bonferroni : $\alpha / 2 = 0{,}025$ par test), ou commencer par le test multivarié, qui contrôle le risque global.
</details>

2. Une MANOVA peut être significative alors qu'aucune ANOVA ne l'est. Pourquoi ce cas est-il possible, et que faut-il en conclure ?
   > **Intérêt :** la liaison entre variables peut porter un signal que chaque variable seule ne montre pas.
<details>
<summary>Correction</summary>

Quand les variables sont corrélées, un écart faible sur chacune peut devenir net dans la direction qui les combine. Le résultat global doit alors être interprété avec les moyennes par groupe, pas seulement avec les p-values : c'est la direction de l'écart qui compte.
</details>

## Liens utiles

- [Documentation de `manova()` (R)](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/manova.html)
- [Documentation de `summary.manova()` (R)](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/summary.manova.html)
