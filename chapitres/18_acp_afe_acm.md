# Chapitre 18 – ACP, AFE et ACM : résumer un jeu de données multidimensionnel

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] réaliser une analyse en composantes principales (ACP)
- [ ] réaliser une analyse factorielle (AFE) sans rotation, puis avec une rotation varimax
- [ ] réaliser une analyse des correspondances multiples (ACM) sur des variables qualitatives

> **Contexte :** un bureau d'études dispose de dizaines de grandeurs par bâtiment : surface, consommations par usage, déperditions, coûts. Présenter chacune séparément ne donne pas une vue d'ensemble. Les méthodes de ce chapitre résument ces données en quelques axes lisibles, qui servent à classer des bâtiments, à repérer des profils types et à choisir les variables d'un modèle.

## Sommaire

- [Chapitre 18 – ACP, AFE et ACM : résumer un jeu de données multidimensionnel](#chapitre-18--acp-afe-et-acm--résumer-un-jeu-de-données-multidimensionnel)
  - [Objectifs](#objectifs)
  - [Sommaire](#sommaire)
  - [Préparation : packages et données](#préparation--packages-et-données)
  - [Démarche : ce que chaque méthode résume](#démarche--ce-que-chaque-méthode-résume)
  - [Exercice 1 – Standardiser et matrice de corrélation](#exercice-1--standardiser-et-matrice-de-corrélation)
    - [Mémo](#mémo)
    - [Exemple](#exemple)
    - [Exercices](#exercices)
  - [Exercice 2 – ACP : valeurs propres et nombre d'axes](#exercice-2--acp--valeurs-propres-et-nombre-daxes)
    - [Mémo](#mémo-1)
    - [Exemple](#exemple-1)
    - [Exercices](#exercices-1)
  - [Exercice 3 – ACP : saturations et sens des axes](#exercice-3--acp--saturations-et-sens-des-axes)
    - [Exemple](#exemple-2)
    - [Exercices](#exercices-2)
  - [Exercice 4 – AFE sans rotation](#exercice-4--afe-sans-rotation)
    - [Exemple](#exemple-3)
    - [Exercices](#exercices-3)
  - [Exercice 5 – AFE avec rotation varimax](#exercice-5--afe-avec-rotation-varimax)
    - [Exemple](#exemple-4)
    - [Exercices](#exercices-4)
  - [Exercice 6 – ACM : inertie et modalités](#exercice-6--acm--inertie-et-modalités)
    - [Mémo](#mémo-2)
    - [Exemple](#exemple-5)
    - [Exercices](#exercices-5)
  - [Liens utiles](#liens-utiles)

## Préparation : packages et données

```r
library(readr)
library(dplyr)
library(ggplot2)
library(FactoMineR)   # pour l'ACM
library(factoextra)   # graphiques de l'ACP et de l'ACM

dpe <- read_csv("data/dpe_villeurbanne_10000.csv")

# Variables quantitatives : logarithmes, pour réduire l'asymétrie
donnees_acp <- dpe |>
  transmute(
    ln_surface  = log(surface_habitable_logement),
    ln_conso    = log(conso_5_usages_ep),
    ln_deperd   = log(deperditions_murs),
    ln_cout     = log(cout_total_5_usages),
    ln_conso_ch = log(conso_chauffage_ep)
  ) |>
  na.omit()
```

Le fichier retient 9 886 DPE qui ont les cinq variables renseignées et positives. Les logarithmes rendent les distributions plus symétriques, comme au chapitre 9.

> **Note : les variables sont mesurées dans des unités différentes**
> La surface est en m², les consommations en kWh, le coût en euros. Sans standardisation, la variable la plus dispersée dominerait l'analyse. C'est pourquoi l'ACP se fait sur les variables centrées réduites, donc sur la matrice de corrélation.

## Démarche : ce que chaque méthode résume

| Méthode | Données | Ce qu'elle résume | Sortie principale |
|---|---|---|---|
| ACP | quantitatives | la variance commune des variables, en axes orthogonaux | valeurs propres, saturations |
| AFE | quantitatives | les corrélations, par des facteurs latents | communalités, saturations |
| ACM | qualitatives | les associations entre modalités | inertie, coordonnées des modalités |

Dans l'ACP, les axes sont des combinaisons linéaires des variables réduites. L'axe $s$ a pour variance la valeur propre $\lambda_s$. Les valeurs propres somment au nombre de variables $p$, puisque chaque variable réduite a une variance de 1. Le pourcentage de variance expliquée par l'axe $s$ est donc $\lambda_s / p$.

Dans l'AFE, chaque variable $j$ s'écrit comme une combinaison de facteurs communs et d'une part propre :

$$
\text{variance}_j = h_j^2 + u_j^2 = 1
$$

où $h_j^2$ est la communalité (part expliquée par les facteurs) et $u_j^2$ l'unicité.

> **Note : ACP et AFE ne sont pas la même chose**
> L'ACP maximise la variance expliquée ; l'AFE cherche à expliquer les corrélations entre variables par un petit nombre de facteurs. Ici, l'AFE est calculée à partir des composantes principales non tournées, puis tournée (varimax) pour faciliter l'interprétation.

## Exercice 1 – Standardiser et matrice de corrélation

> **Contexte :** avant toute analyse multidimensionnelle, on regarde les corrélations : si toutes les variables sont indépendantes, il n'y a rien à résumer.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `scale()` | Centre et réduit les colonnes. | | `scale(donnees_acp)` |
| `cor()` | Matrice de corrélation. | `use` | `cor(donnees_acp)` |

### Exemple

```r
round(cor(donnees_acp), digits = 3)
```

### Exercices

1. Calculer la matrice de corrélation des cinq variables. Quelles paires sont les plus corrélées ?
   > **Intérêt :** des corrélations fortes entre variables signalent des axes communs : c'est ce que l'ACP va résumer.
<details>
<summary>Correction</summary>

| | ln surface | ln conso | ln déperditions | ln coût | ln conso chauffage |
|---|---|---|---|---|---|
| ln surface | 1,000 | 0,758 | 0,117 | 0,723 | 0,568 |
| ln conso | 0,758 | 1,000 | 0,285 | 0,931 | 0,829 |
| ln déperditions | 0,117 | 0,285 | 1,000 | 0,240 | 0,308 |
| ln coût | 0,723 | 0,931 | 0,240 | 1,000 | 0,761 |
| ln conso chauffage | 0,568 | 0,829 | 0,308 | 0,761 | 1,000 |

Les paires les plus corrélées sont la consommation et le coût total (0,931), puis la consommation et la consommation de chauffage (0,829). Les déperditions des murs sont peu corrélées aux autres variables (entre 0,12 et 0,31) : elles porteront un axe à part.
</details>

2. Pourquoi l'ACP se fait-elle sur la matrice de corrélation plutôt que sur la matrice de covariance ici ?
   > **Intérêt :** le choix de la matrice conditionne les axes : le lire évite de comparer des résultats qui ne portent pas sur la même chose.
<details>
<summary>Correction</summary>

Les variables ont des unités différentes. Sur la matrice de covariance, les axes seraient dominés par les variables dont la variance est la plus grande, quelle que soit leur importance. La standardisation donne à chaque variable le même poids : c'est l'ACP normée.
</details>

## Exercice 2 – ACP : valeurs propres et nombre d'axes

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `prcomp()` | ACP par décomposition de la matrice standardisée. | `scale.` | `prcomp(donnees_acp, scale. = TRUE)` |
| `summary()` | Écarts-types, part de variance et cumul par axe. | | `summary(acp)` |
| `fviz_eig()` (`factoextra`) | Éboulis des valeurs propres. | `addlabels` | `fviz_eig(acp, addlabels = TRUE)` |
| `fviz_pca_var()` (`factoextra`) | Cercle des corrélations : saturations des variables sur deux axes. | `axes`, `repel` | `fviz_pca_var(acp, repel = TRUE)` |

### Exemple

```r
acp <- prcomp(donnees_acp, scale. = TRUE)
summary(acp)
acp$sdev^2     # valeurs propres
```

### Exercices

1. Lire les valeurs propres. Quel pourcentage de variance expliquent les deux premiers axes ?
   > **Intérêt :** la part de variance par axe est le critère principal pour décider du nombre d'axes à interpréter.
<details>
<summary>Correction</summary>

| Axe | Valeur propre | Part | Cumul |
|---|---|---|---|
| CP1 | 3,393 | 67,9 % | 67,9 % |
| CP2 | 0,937 | 18,7 % | 86,6 % |
| CP3 | 0,404 | 8,1 % | 94,7 % |
| CP4 | 0,210 | 4,2 % | 98,9 % |
| CP5 | 0,056 | 1,1 % | 100 % |

Les deux premiers axes expliquent 86,6 % de la variance.
</details>

2. Combien d'axes faut-il retenir ? Appliquer le critère de Kaiser (valeur propre supérieure à 1) et regarder l'éboulis.
   > **Intérêt :** aucun critère n'est unique : on croise le critère de Kaiser, l'éboulis et l'interprétation possible.
<details>
<summary>Correction</summary>

Le critère de Kaiser ne retient que CP1 (valeur propre 3,39 ; CP2 vaut 0,94, juste en dessous de 1). L'éboulis montre une cassure après CP2 : les deux premiers axes sont retenus, ce qui résume 86,6 % de l'information. CP2 est proche du seuil de Kaiser, et son interprétation est à conduire avec prudence.

```r
fviz_eig(acp, addlabels = TRUE)   # éboulis : pourcentage de variance par axe
```
</details>

## Exercice 3 – ACP : saturations et sens des axes

Les saturations (corrélations entre une variable et un axe) donnent le sens de chaque axe. Elles s'obtiennent par $\text{saturation} = \text{vecteur propre} \times \sqrt{\lambda}$.

### Exemple

```r
saturations <- sweep(acp$rotation[, 1:2], 2, acp$sdev[1:2], "*")
round(saturations, digits = 3)

# Cercle des corrélations : chaque flèche est une variable, sa position donne ses saturations
fviz_pca_var(acp, repel = TRUE)
```

### Exercices

1. Calculer les saturations sur CP1 et CP2. Quelles variables pèsent le plus sur chaque axe ?
   > **Intérêt :** une saturation élevée en valeur absolue signale qu'une variable est bien représentée par l'axe.
<details>
<summary>Correction</summary>

| Variable | CP1 | CP2 |
|---|---|---|
| ln surface | 0,817 | −0,274 |
| ln conso | 0,971 | −0,062 |
| ln déperditions | 0,363 | 0,919 |
| ln coût | 0,940 | −0,104 |
| ln conso chauffage | 0,876 | 0,056 |

CP1 est pesé par la consommation, le coût, la conso de chauffage et la surface : c'est un axe de **taille et d'énergie consommée**. CP2 est porté presque entièrement par les déperditions des murs : c'est un axe de **qualité de l'enveloppe**, indépendant de la taille.
</details>

2. Le signe de CP1 est arbitraire. Expliquer ce que signifie un DPE au score élevé sur CP1, et un DPE au score élevé sur CP2.
   > **Intérêt :** un axe se nomme d'après ses saturations, pas d'après son signe : les deux sens sont à interpréter ensemble.
<details>
<summary>Correction</summary>

Un score élevé sur CP1 signifie des consommations, un coût et une surface élevés (le signe pourrait être inversé sans changer l'analyse). Un score élevé sur CP2 signifie des déperditions de murs élevées, à consommation comparable. Un logement peut donc être grand et peu déperditif, ou petit et très déperditif.
</details>

## Exercice 4 – AFE sans rotation

L'analyse factorielle (AFE) à partir des deux premières composantes, sans rotation, donne une première lecture : chaque variable est reliée aux deux facteurs, et l'on mesure la part de variance qu'ils expliquent.

### Exemple

```r
h2 <- rowSums(saturations^2)   # communalités
u2 <- 1 - h2                   # unicités
round(cbind(saturations, communalite = h2, unicite = u2), digits = 3)
```

### Exercices

1. Calculer les communalités. Quelle variable est la mieux expliquée par les deux facteurs ?
   > **Intérêt :** une communalité faible signale une variable que les facteurs n'expliquent pas : elle doit être lue à part.
<details>
<summary>Correction</summary>

| Variable | Communalité $h^2$ | Unicité $u^2$ |
|---|---|---|
| ln surface | 0,743 | 0,257 |
| ln conso | 0,947 | 0,053 |
| ln déperditions | 0,976 | 0,024 |
| ln coût | 0,894 | 0,106 |
| ln conso chauffage | 0,770 | 0,230 |

Les déperditions des murs sont les mieux expliquées (0,976), même si elles n'ont pas de saturation élevée sur CP1 : CP2 les porte à elle seule. La surface a l'unicité la plus forte (0,257) : une part de sa variance n'est pas expliquée par les deux axes.
</details>

2. Pourquoi la communalité d'une variable est-elle la somme de ses saturations au carré ?
   > **Intérêt :** la communalité se calcule à partir des saturations : la relation évite de la traiter comme une boîte noire.
<details>
<summary>Correction</summary>

Les axes sont orthogonaux (non corrélés). La variance d'une variable réduite égale 1 : la somme des carrés de ses saturations sur les facteurs retenus donne la part de cette variance expliquée par ces facteurs, c'est la communalité. Le reste, $1 - h^2$, est l'unicité.
</details>

## Exercice 5 – AFE avec rotation varimax

Une rotation ne change pas la part de variance expliquée par les deux facteurs, mais elle redistribue les saturations pour qu'une variable pèse surtout sur un facteur. La rotation **varimax** maximise la variance des carrés des saturations sur chaque facteur, ce qui facilite l'interprétation.

### Exemple

```r
rot <- varimax(saturations)
round(rot$loadings[, 1:2], digits = 3)
```

### Exercices

1. Comparer les saturations avant et après varimax. Que devient le partage entre facteurs ?
   > **Intérêt :** la rotation ne change ni les communalités ni le total expliqué : elle change seulement la lecture.
<details>
<summary>Correction</summary>

| Variable | Facteur 1 (varimax) | Facteur 2 (varimax) |
|---|---|---|
| ln surface | 0,860 | −0,058 |
| ln conso | 0,955 | 0,185 |
| ln déperditions | 0,118 | 0,981 |
| ln coût | 0,935 | 0,137 |
| ln conso chauffage | 0,833 | 0,276 |

Après rotation, quatre variables pèsent sur le facteur 1 et les déperditions seules sur le facteur 2. Le critère varimax passe de 0,192 à 0,245, ce qui confirme une structure plus nette.
</details>

2. Donner un nom à chaque facteur. Les communalités changent-elles après rotation ?
   > **Intérêt :** un facteur sans nom ne sert pas à la décision. Le nom doit venir des variables qui le saturent.
<details>
<summary>Correction</summary>

Facteur 1 : **consommation et taille**. Facteur 2 : **déperditions de l'enveloppe**. Les communalités ne changent pas : la rotation est orthogonale, elle conserve la somme des carrés des saturations pour chaque variable.
</details>

3. Quelle est la différence entre ce résultat et l'ACP des exercices 2 et 3 ?
   > **Intérêt :** l'ACP et l'AFE donnent des axes voisins, mais leur objectif diffère : à présenter comme tel dans un rapport.
<details>
<summary>Correction</summary>

L'ACP cherche des axes qui résument la variance totale, sans modèle. L'AFE suppose des facteurs latents qui expliquent les corrélations. Ici les deux résultats convergent : le premier facteur correspond à CP1 et le second à CP2. Les unicités, elles, n'existent pas dans l'ACP.
</details>

## Exercice 6 – ACM : inertie et modalités

L'analyse des correspondances multiples (ACM) résume des variables qualitatives. Chaque variable se transforme en indicatrices (une colonne par modalité). Les variables sont l'étiquette DPE (7 modalités), le type de bâtiment (3), le type d'énergie regroupé (4) et la période de construction (3) : $J = 17$ modalités pour $Q = 4$ variables.

> **Contexte :** l'ACM permet de voir quelles modalités vont ensemble : par exemple, quelles périodes et quelles énergies accompagnent les étiquettes les plus basses. C'est un outil de description, utile avant de choisir les variables d'un modèle.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `MCA()` (package `FactoMineR`) | Analyse des correspondances multiples. | `X`, `graph` | `MCA(donnees_acm, graph = FALSE)` |
| `$eig` | Valeurs propres et pourcentages d'inertie. | | `res_acm$eig` |
| `$var$coord` | Coordonnées des modalités sur les axes. | | `res_acm$var$coord` |
| `fviz_eig()` (`factoextra`) | Valeurs propres de l'ACM. | | `fviz_eig(res_acm, addlabels = TRUE)` |
| `fviz_mca_var()` (`factoextra`) | Carte des modalités sur deux axes. | `repel` | `fviz_mca_var(res_acm, repel = TRUE)` |

### Exemple

```r
donnees_acm <- dpe |>
  transmute(
    etiquette = factor(etiquette_dpe),
    type      = factor(type_batiment),
    energie   = factor(case_when(
      type_energie_n1 == "Réseau de Chauffage urbain" ~ "Réseau",
      type_energie_n1 %in% c("Électricité", "Gaz naturel") ~ type_energie_n1,
      TRUE ~ "Fioul et autres")),
    periode   = factor(case_when(
      periode_construction %in% c("avant 1948", "1948-1974") ~ "avant 1975",
      periode_construction %in% c("1975-1977", "1978-1982", "1983-1988",
                                  "1989-2000", "2001-2005") ~ "1975-2005",
      TRUE ~ "2006 et après"))
  )

res_acm <- MCA(donnees_acm, graph = FALSE)
res_acm$eig

fviz_eig(res_acm, addlabels = TRUE)
fviz_mca_var(res_acm, repel = TRUE)   # carte des modalités sur les axes 1 et 2
```

### Exercices

1. Vérifier que l'inertie totale vaut $J/Q - 1 = 17/4 - 1 = 3{,}25$. Quel pourcentage d'inertie porte le premier axe ?
   > **Intérêt :** l'inertie totale est une identité : si elle ne vaut pas $J/Q - 1$, la table de données est mal construite.
<details>
<summary>Correction</summary>

Avec 17 modalités et 4 variables, l'inertie totale vaut $17/4 - 1 = 3{,}25$. Le premier axe porte une valeur propre brute de 0,427, soit 13,1 % de l'inertie totale. Les axes 2 à 6 portent entre 7,8 % et 10,3 % chacun : l'inertie est répartie sur plusieurs axes, ce qui est fréquent pour des variables qualitatives à beaucoup de modalités.
</details>

2. Lire les coordonnées des modalités sur les axes 1 et 2. Quelles modalités s'opposent sur l'axe 1 ?
   > **Intérêt :** la lecture d'une ACM porte sur les écarts entre modalités, et sur leur proximité : deux modalités proches sont associées.
<details>
<summary>Correction</summary>

| Modalité | Axe 1 | Axe 2 |
|---|---|---|
| étiquette C | −0,86 | +0,16 |
| étiquette D | +0,42 | −0,63 |
| étiquette E | +0,99 | −0,22 |
| étiquette F | +1,29 | +1,01 |
| étiquette G | +1,37 | +2,70 |
| maison | +1,54 | +2,92 |
| appartement | −0,05 | −0,10 |
| période 2006 et après | −1,20 | +0,45 |
| période avant 1975 | +0,77 | −0,12 |
| gaz naturel | +0,34 | −0,35 |
| électricité | −0,44 | +0,34 |

L'axe 1 oppose les étiquettes C et les DPE récents (négatif) aux étiquettes E, F, G et aux logements anciens (positif). Il lit donc la **performance et l'âge**. Les modalités de faible effectif (immeuble, fioul et autres, étiquette A) ont des coordonnées très instables : elles ne doivent pas être commentées seules.
</details>

3. Les modalités de faible effectif sont-elles fiables sur les axes ? Justifier avec leurs masses.
   > **Intérêt :** une coordonnée extrême peut venir d'un effectif très petit, pas d'une vraie association.
<details>
<summary>Correction</summary>

Les masses (proportions dans le tableau disjonctif) sont de 0,0005 pour l'étiquette A, 0,0020 pour l'immeuble, 0,0026 pour « fioul et autres », contre 0,24 pour les appartements. Une modalité qui représente moins de 1 % des données peut avoir une coordonnée extrême sans que son association soit établie. Il faut le signaler, ou regrouper ces modalités avant l'ACM.
</details>

## Liens utiles

- [Documentation de `prcomp()` (R)](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/prcomp.html)
- [Documentation de `varimax()` (R)](https://stat.ethz.ch/R-manual/R-patched/library/stats/html/varimax.html)
- [Documentation du package `FactoMineR`](https://cran.r-project.org/package=FactoMineR)
- [Documentation du package `factoextra`](https://cran.r-project.org/package=factoextra)
