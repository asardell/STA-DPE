# Chapitre 3 – Manipulation de dataframes avec dplyr

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] sélectionner des lignes et des colonnes avec les fonctions de base de R
- [ ] trier un dataframe avec `order()` et `arrange()`
- [ ] choisir des colonnes avec `select()` et des lignes avec `filter()`
- [ ] créer de nouvelles variables avec `mutate()`
- [ ] calculer des effectifs et des moyennes par groupe avec `group_by()` et `summarise()`
- [ ] enchaîner plusieurs étapes avec le pipe `|>`
- [ ] rattacher une table de libellés à une autre avec `left_join()`

## Sommaire

- [Préparation : packages et données](#préparation--packages-et-données)
- [Exercice 1 – Sélectionner avec `[ , ]` et `subset()`](#exercice-1--sélectionner-avec--et-subset)
- [Exercice 2 – Trier avec `order()` et `arrange()`](#exercice-2--trier-avec-order-et-arrange)
- [Exercice 3 – Choisir des colonnes avec `select()`](#exercice-3--choisir-des-colonnes-avec-select)
- [Exercice 4 – Choisir des lignes avec `filter()`](#exercice-4--choisir-des-lignes-avec-filter)
- [Exercice 5 – Créer des variables avec `mutate()`](#exercice-5--créer-des-variables-avec-mutate)
- [Exercice 6 – Effectifs et moyennes par groupe](#exercice-6--effectifs-et-moyennes-par-groupe)
- [Exercice 7 – Enchaîner les étapes avec le pipe `|>`](#exercice-7--enchaîner-les-étapes-avec-le-pipe-)
- [Exercice 8 – Rattacher des libellés avec `left_join()`](#exercice-8--rattacher-des-libellés-avec-left_join)

## Préparation : packages et données

`dplyr` est le package de référence pour manipuler des dataframes. Il est installé avec le *tidyverse*, comme `readr` vu au chapitre 2.

```r
install.packages("dplyr")
library(dplyr)
library(readr)

dpe <- read_csv("data/dpe_villeurbanne_10000.csv")
```

Les fonctions de `dplyr` ont une grammaire commune : le premier argument est toujours le dataframe, et les noms des colonnes s'écrivent **sans guillemets**. Chaque fonction ne fait qu'une chose : choisir des colonnes, choisir des lignes, trier, créer une variable, agréger.

> **Note : le pipe `|>`**
> Le symbole `|>` (pipe natif de R, à partir de la version 4.1) envoie le résultat d'une étape dans la suivante : `dpe |> filter(...)` équivaut à `filter(dpe, ...)`. Le package `magrittr` propose le même principe avec `%>%`, très répandu dans les anciens scripts.

## Exercice 1 – Sélectionner avec `[ , ]` et `subset()`

Avant `dplyr`, R sélectionne les lignes et les colonnes avec `[ lignes , colonnes ]`. Une condition placée dans `lignes` garde les lignes qui la vérifient.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `[ , ]` | Sélectionne des lignes et des colonnes. | `lignes`, `colonnes` | `dpe[1:5, c("numero_dpe", "etiquette_dpe")]` |
| `==` | Teste l'égalité. | | `dpe$etiquette_dpe == "A"` |
| `subset()` | Filtre les lignes et choisit les colonnes en une seule commande. | `subset`, `select` | `subset(dpe, etiquette_dpe == "A", select = c(numero_dpe, etiquette_dpe))` |

### Exemple

```r
# Les 5 premières lignes de deux colonnes
dpe[1:5, c("numero_dpe", "etiquette_dpe")]

# Les lignes où l'étiquette vaut "A" : une condition dans la partie lignes
dpe[dpe$etiquette_dpe == "A", c("numero_dpe", "surface_habitable_logement")]

# La même chose avec subset(), où les noms de colonnes s'écrivent sans $
subset(dpe, etiquette_dpe == "A", select = c(numero_dpe, surface_habitable_logement))
```

### Exercices

1. Afficher les colonnes `numero_dpe` et `type_batiment` pour les lignes 100 à 105.
<details>
<summary>Correction</summary>

```r
dpe[100:105, c("numero_dpe", "type_batiment")]
```
</details>

2. Afficher les colonnes `numero_dpe` et `etiquette_dpe` des DPE dont le `type_batiment` est `"maison"`. Combien de lignes obtient-on ?
<details>
<summary>Correction</summary>

```r
maisons <- dpe[dpe$type_batiment == "maison", c("numero_dpe", "etiquette_dpe")]
nrow(maisons)
# [1] 329
```
</details>

3. Refaire la question 2 avec `subset()`.
<details>
<summary>Correction</summary>

```r
maisons <- subset(dpe, type_batiment == "maison", select = c(numero_dpe, etiquette_dpe))
nrow(maisons)
# [1] 329
```
</details>

> **Note : `==` et `=`**
> Dans une condition, on écrit `==` pour tester l'égalité. Un seul `=` ne compare pas : il sert à affecter une valeur ou à donner un argument à une fonction. Écrire `etiquette_dpe = "A"` dans `filter()` n'est donc pas une comparaison.

## Exercice 2 – Trier avec `order()` et `arrange()`

Trier consiste à réordonner les lignes selon les valeurs d'une colonne. R de base utilise `order()`, qui renvoie les positions dans l'ordre demandé. `dplyr` propose `arrange()`, plus lisible.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `order()` | Renvoie les positions triées d'un vecteur. | `decreasing` (FALSE par défaut) | `order(dpe$surface_habitable_logement, decreasing = TRUE)` |
| `arrange()` | Trie les lignes d'un dataframe. | colonnes à trier, `desc()` pour l'ordre décroissant | `arrange(dpe, desc(surface_habitable_logement))` |
| `desc()` | Indique un ordre décroissant dans `arrange()`. | | `desc(surface_habitable_logement)` |

### Exemple

```r
# Base R : on récupère les positions triées, puis on sélectionne les lignes
dpe[order(dpe$surface_habitable_logement, decreasing = TRUE), c("numero_dpe", "surface_habitable_logement")][1:3, ]

# dplyr : même résultat, plus lisible
dpe |>
  arrange(desc(surface_habitable_logement)) |>
  select(numero_dpe, surface_habitable_logement) |>
  head(n = 3)
```

> **Note : les valeurs manquantes**
> Dans `arrange()` comme dans `order()`, les `NA` sont placés à la fin, quel que soit l'ordre. Un tri décroissant ne fait donc pas remonter les DPE sans surface.

### Exercices

1. Trier les DPE par `surface_habitable_logement` croissante et afficher les 3 premiers, avec `numero_dpe` et la surface.
<details>
<summary>Correction</summary>

```r
dpe |>
  arrange(surface_habitable_logement) |>
  select(numero_dpe, surface_habitable_logement) |>
  head(n = 3)
```
</details>

2. Trier par `etiquette_dpe` puis, à étiquette égale, par `surface_habitable_logement` décroissante. Afficher les 5 premières lignes avec `etiquette_dpe` et la surface.
<details>
<summary>Correction</summary>

```r
dpe |>
  arrange(etiquette_dpe, desc(surface_habitable_logement)) |>
  select(etiquette_dpe, surface_habitable_logement) |>
  head(n = 5)
```
Le premier critère est l'étiquette ; le second ne départage que les DPE de même étiquette.
</details>

3. Afficher les 3 DPE dont la surface est la plus grande. Que remarque-t-on sur la valeur la plus haute ?
<details>
<summary>Correction</summary>

```r
dpe |>
  arrange(desc(surface_habitable_logement)) |>
  select(numero_dpe, surface_habitable_logement) |>
  head(n = 3)
```
La plus grande surface est de 3 330 m², ce qui est très improbable pour un logement. Cette valeur extrême est traitée au chapitre 4.
</details>

## Exercice 3 – Choisir des colonnes avec `select()`

`select()` garde les colonnes demandées, dans l'ordre demandé. Des fonctions d'aide permettent de sélectionner des groupes de colonnes sans les nommer une à une.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `select()` | Garde les colonnes indiquées. | noms des colonnes, sans guillemets | `select(dpe, numero_dpe, etiquette_dpe)` |
| `starts_with()` | Colonnes dont le nom commence par un texte. | le texte | `select(dpe, starts_with("conso_"))` |
| `ends_with()` | Colonnes dont le nom finit par un texte. | le texte | `select(dpe, ends_with("_n1"))` |
| `select(-col)` | Retire une colonne. | le nom, précédé de `-` | `select(dpe, -description_installation_chauffage_n1)` |
| `rename()` | Renomme une colonne : `nouveau = ancien`. | | `rename(surface = surface_habitable_logement)` |

### Exemple

```r
# Les colonnes qui décrivent les consommations
dpe |> select(starts_with("conso_")) |> names()

# Toutes les colonnes sauf une
dpe |> select(-description_installation_chauffage_n1) |> ncol()
# [1] 21

# Renommer une colonne pour l'utiliser plus vite
dpe |> rename(surface = surface_habitable_logement) |> select(numero_dpe, surface) |> head(n = 3)
```

### Exercices

1. Garder les colonnes `numero_dpe`, `etiquette_dpe`, `type_batiment` et `annee_construction`, puis afficher les 3 premières lignes.
<details>
<summary>Correction</summary>

```r
dpe |>
  select(numero_dpe, etiquette_dpe, type_batiment, annee_construction) |>
  head(n = 3)
```
</details>

2. Lister les colonnes dont le nom commence par `conso_`. Combien y en a-t-il ?
<details>
<summary>Correction</summary>

```r
dpe |> select(starts_with("conso_")) |> names()
# [1] "conso_5_usages_ep"  "conso_chauffage_ep"  "conso_ecs_ep"
# [4] "conso_refroidissement_ep"  "conso_eclairage_ep"  "conso_auxiliaires_ep"
# 6 colonnes
```
</details>

3. Renommer `deperditions_murs` en `deperdition_murs` et `besoin_chauffage` en `besoin`, puis afficher ces deux colonnes pour les 3 premières lignes. Le renommage modifie-t-il `dpe` ?
<details>
<summary>Correction</summary>

```r
dpe |>
  rename(deperdition_murs = deperditions_murs, besoin = besoin_chauffage) |>
  select(deperdition_murs, besoin) |>
  head(n = 3)

# Non : sans affectation, dpe reste inchangé. Le résultat n'existe que dans la console.
names(dpe)[10]  # "deperditions_murs"
```
</details>

> **Note : les modifications sont visibles seulement si on les affecte**
> `dpe |> select(...)` affiche le résultat sans modifier `dpe`. Pour garder le résultat, il faut l'affecter : `dpe_court <- dpe |> select(...)`. C'est le cas pour toutes les fonctions de `dplyr`.

## Exercice 4 – Choisir des lignes avec `filter()`

`filter()` garde les lignes qui vérifient une condition. On combine les conditions avec `&` (et), `|` (ou) et `%in%` (appartient à une liste).

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `filter()` | Garde les lignes qui vérifient la condition. | une ou plusieurs conditions | `filter(dpe, etiquette_dpe == "G")` |
| `&` | Et : les deux conditions doivent être vraies. | | `etiquette_dpe == "A" & type_batiment == "maison"` |
| `\|` | Ou : au moins une condition doit être vraie. | | `etiquette_dpe == "F" \| etiquette_dpe == "G"` |
| `%in%` | Appartient à une liste de valeurs. | | `etiquette_dpe %in% c("F", "G")` |
| `is.na()` | Vrai si la valeur est manquante. | | `filter(dpe, !is.na(surface_habitable_logement))` |
| `!` | Négation : inverse la condition. | | `!is.na(x)` |

### Exemple

```r
# Les DPE en étiquette G
dpe |> filter(etiquette_dpe == "G") |> nrow()
# [1] 285

# Les DPE en étiquette F ou G
dpe |> filter(etiquette_dpe %in% c("F", "G")) |> nrow()
# [1] 830

# Les DPE sans surface renseignée, puis ceux qui en ont une
dpe |> filter(is.na(surface_habitable_logement)) |> nrow()
# [1] 101
dpe |> filter(!is.na(surface_habitable_logement)) |> nrow()
# [1] 9899
```

### Exercices

1. Combien de DPE sont en étiquette `A` ou `B` ?
<details>
<summary>Correction</summary>

```r
dpe |> filter(etiquette_dpe %in% c("A", "B")) |> nrow()
# [1] 251
```
</details>

2. Garder les maisons (`type_batiment == "maison"`) dont la surface dépasse 100 m². Combien y en a-t-il ?
<details>
<summary>Correction</summary>

```r
dpe |>
  filter(type_batiment == "maison", surface_habitable_logement > 100) |>
  nrow()
```
Une virgule entre deux conditions dans `filter()` équivaut à un `&`.
</details>

3. Garder les DPE en étiquette `E`, `F` ou `G` dont la surface est renseignée. Combien y en a-t-il ?
<details>
<summary>Correction</summary>

```r
dpe |>
  filter(etiquette_dpe %in% c("E", "F", "G"), !is.na(surface_habitable_logement)) |>
  nrow()
```
Les DPE sans surface sont exclus : la condition `!is.na()` est indispensable quand on compare une colonne qui contient des `NA`.
</details>

> **Note : un `NA` dans une condition**
> Une comparaison avec `NA` vaut `NA`, et `filter()` garde uniquement les lignes où la condition vaut `TRUE`. Les lignes avec `NA` sont donc écartées sans message. Pour les garder ou les compter, il faut écrire `is.na()` explicitement.

## Exercice 5 – Créer des variables avec `mutate()`

`mutate()` ajoute une colonne calculée à partir des autres. La colonne est créée une fois pour toutes dans le dataframe affecté.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `mutate()` | Ajoute ou modifie une colonne. | `nom = expression` | `mutate(dpe, conso_m2 = conso_5_usages_ep / surface_habitable_logement)` |
| `round()` | Arrondit une valeur calculée. | `digits` | `round(conso_m2, digits = 1)` |
| Opérateurs arithmétiques | `+`, `-`, `*`, `/`, `^` | | `surface_habitable_logement * 2` |

### Exemple

```r
# Consommation par m² : énergie primaire totale divisée par la surface
dpe_avec_conso <- dpe |>
  mutate(conso_m2 = conso_5_usages_ep / surface_habitable_logement)

dpe_avec_conso |> select(numero_dpe, conso_m2) |> head(n = 3)

# Moyenne de la nouvelle variable, sans les surfaces manquantes
mean(dpe_avec_conso$conso_m2, na.rm = TRUE)
# [1] 204.8...
```

### Exercices

1. Créer la colonne `conso_m2` comme dans l'exemple, puis la colonne `passoire`, qui vaut `TRUE` si l'étiquette est `F` ou `G`. Compter les `TRUE`.
<details>
<summary>Correction</summary>

```r
dpe_avec_conso <- dpe |>
  mutate(
    conso_m2 = conso_5_usages_ep / surface_habitable_logement,
    passoire = etiquette_dpe %in% c("F", "G")
  )

sum(dpe_avec_conso$passoire)
# [1] 830
```
`sum()` d'une colonne logique compte les `TRUE`.
</details>

2. Créer une colonne `surface_m2_arrondie` égale à la surface arrondie à l'entier le plus proche, puis afficher les 3 premières valeurs de `surface_habitable_logement` et de `surface_m2_arrondie`.
<details>
<summary>Correction</summary>

```r
dpe |>
  mutate(surface_m2_arrondie = round(surface_habitable_logement)) |>
  select(surface_habitable_logement, surface_m2_arrondie) |>
  head(n = 3)
```
</details>

3. Dans un même `mutate()`, créer `conso_mwh = conso_5_usages_ep / 1000` (en MWh), puis `conso_mwh` pour les 3 premières lignes. Pourquoi peut-on utiliser cette colonne dans la même commande ?
<details>
<summary>Correction</summary>

```r
dpe |>
  mutate(conso_mwh = conso_5_usages_ep / 1000) |>
  select(numero_dpe, conso_mwh) |>
  head(n = 3)
```
Dans un même `mutate()`, les colonnes créées sont disponibles pour les calculs suivants. On peut donc écrire `mutate(a = x * 2, b = a + 1)`.
</details>

## Exercice 6 – Effectifs et moyennes par groupe

`group_by()` regroupe les lignes selon les valeurs d'une colonne. `summarise()` calcule ensuite une valeur par groupe. `count()` est un raccourci pour compter les effectifs.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `group_by()` | Regroupe les lignes par une ou plusieurs colonnes. | colonnes de regroupement | `group_by(dpe, etiquette_dpe)` |
| `summarise()` | Calcule une valeur par groupe. | `nom = fonction(colonne)` | `summarise(dpe, moyenne = mean(surface_habitable_logement, na.rm = TRUE))` |
| `n()` | Compte les lignes du groupe, dans `summarise()`. | | `summarise(dpe, effectif = n())` |
| `count()` | Compte les effectifs par valeur. | colonne | `count(dpe, etiquette_dpe)` |
| `ungroup()` | Supprime le regroupement. | | `ungroup(dpe)` |

### Exemple

```r
# Effectifs par étiquette
dpe |> count(etiquette_dpe)
#   etiquette_dpe     n
#   A                19
#   B               232
#   C              4266
#   D              3127
#   E              1526
#   F               545
#   G               285

# Même chose avec group_by() et summarise(), plus flexible
dpe |>
  group_by(etiquette_dpe) |>
  summarise(
    effectif = n(),
    surface_moyenne = mean(surface_habitable_logement, na.rm = TRUE)
  )
```

### Exercices

1. Compter les DPE par `type_batiment`. Quel type est le plus fréquent ?
<details>
<summary>Correction</summary>

```r
dpe |> count(type_batiment, sort = TRUE)
# appartement : 9592 ; maison : 329 ; immeuble : 79
```
`sort = TRUE` trie du plus fréquent au moins fréquent.
</details>

2. Calculer, pour chaque `etiquette_dpe`, le nombre de DPE et la surface moyenne, en ignorant les valeurs manquantes. Trier le résultat par étiquette.
<details>
<summary>Correction</summary>

```r
dpe |>
  group_by(etiquette_dpe) |>
  summarise(
    effectif = n(),
    surface_moyenne = mean(surface_habitable_logement, na.rm = TRUE)
  ) |>
  arrange(etiquette_dpe)
```
</details>

3. Calculer le nombre de DPE par `periode_construction` et `etiquette_dpe` à la fois. Combien de groupes obtient-on ?
<details>
<summary>Correction</summary>

```r
dpe |> count(periode_construction, etiquette_dpe)
# Une ligne par couple (période, étiquette) présent dans les données.
nrow(dpe |> count(periode_construction, etiquette_dpe))
```
Le nombre de lignes dépend des couples observés : c'est un résultat à lire dans la console.
</details>

> **Note : `summarise()` et le regroupement**
> Après `summarise()`, `dplyr` supprime le dernier niveau de regroupement et renvoie un tableau sans regroupement. Si on a regroupé par deux colonnes, le résultat reste regroupé par la première : `ungroup()` permet de l'éviter.

## Exercice 7 – Enchaîner les étapes avec le pipe `|>`

Une analyse s'écrit souvent en plusieurs étapes : choisir les lignes, créer une variable, regrouper, résumer. Le pipe `|>` enchaîne ces étapes dans un seul bloc lisible, de haut en bas.

### Exemple

```r
# Pour les maisons et les appartements, la consommation moyenne par m² selon l'étiquette
dpe |>
  filter(type_batiment %in% c("maison", "appartement"), !is.na(surface_habitable_logement)) |>
  mutate(conso_m2 = conso_5_usages_ep / surface_habitable_logement) |>
  group_by(etiquette_dpe) |>
  summarise(
    effectif = n(),
    conso_m2_moyenne = mean(conso_m2)
  ) |>
  arrange(etiquette_dpe)
```

### Exercices

1. Pour les DPE dont la surface dépasse 30 m², calculer l'effectif et la surface moyenne par `type_batiment`. Trier par effectif décroissant.
<details>
<summary>Correction</summary>

```r
dpe |>
  filter(surface_habitable_logement > 30) |>
  group_by(type_batiment) |>
  summarise(
    effectif = n(),
    surface_moyenne = mean(surface_habitable_logement)
  ) |>
  arrange(desc(effectif))
```
</details>

2. Garder les colonnes `numero_dpe`, `etiquette_dpe`, `surface_habitable_logement` et `conso_5_usages_ep` des DPE en étiquette `G`. Ajouter `conso_m2`, puis afficher les 5 DPE qui ont la plus grande consommation par m².
<details>
<summary>Correction</summary>

```r
dpe |>
  filter(etiquette_dpe == "G", !is.na(surface_habitable_logement)) |>
  select(numero_dpe, etiquette_dpe, surface_habitable_logement, conso_5_usages_ep) |>
  mutate(conso_m2 = conso_5_usages_ep / surface_habitable_logement) |>
  arrange(desc(conso_m2)) |>
  head(n = 5)
```
</details>

3. Calculer la surface moyenne des DPE par `periode_construction`, pour les DPE dont la surface est renseignée et inférieure ou égale à 300 m². Trier par surface moyenne décroissante.
<details>
<summary>Correction</summary>

```r
dpe |>
  filter(surface_habitable_logement <= 300) |>
  group_by(periode_construction) |>
  summarise(surface_moyenne = mean(surface_habitable_logement)) |>
  arrange(desc(surface_moyenne))
```
Le filtre `<= 300` écarte les valeurs extrêmes : `NA` est aussi écarté, car la comparaison avec `NA` ne vaut pas `TRUE`.
</details>

## Exercice 8 – Rattacher des libellés avec `left_join()`

Une jointure rattache à chaque ligne d'un tableau les informations d'un autre tableau, à partir d'une colonne commune appelée **clé**. Ici, on rattache une table de libellés à partir de `etiquette_dpe`.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `left_join()` | Garde toutes les lignes du tableau de gauche et ajoute les colonnes de droite quand la clé correspond. | `x`, `y`, `by` | `left_join(dpe, libelles, by = "etiquette_dpe")` |
| `distinct()` | Supprime les lignes en double. | colonnes à comparer | `distinct(dpe, etiquette_dpe)` |

### Exemple

```r
# Une petite table de libellés pédagogiques, une ligne par étiquette
libelles <- data.frame(
  etiquette_dpe = c("A", "B", "C", "D", "E", "F", "G"),
  niveau = c("très performant", "performant", "assez performant", "moyen",
             "peu performant", "très peu performant", "très énergivore")
)

dpe_libelle <- left_join(dpe, libelles, by = "etiquette_dpe")

# Le nombre de lignes ne change pas : chaque DPE a une seule étiquette
nrow(dpe_libelle)
# [1] 10000
```

> **Note : `left_join()` et les doublons**
> Si la table de droite contient deux lignes pour une même clé, la jointure duplique les lignes de gauche : le nombre de lignes augmente. Avant une jointure, on vérifie donc que la clé est unique dans la table de droite, par exemple avec `distinct()` ou en comparant `nrow()` avant et après.

### Exercices

1. Rattacher les libellés à `dpe`, puis compter les DPE par `niveau`.
<details>
<summary>Correction</summary>

```r
dpe_libelle <- left_join(dpe, libelles, by = "etiquette_dpe")
dpe_libelle |> count(niveau, sort = TRUE)
```
</details>

2. Créer une table `libelles_partielle` qui ne contient pas la lettre `G`. Faire la jointure et compter les DPE dont le `niveau` est manquant. Combien en obtient-on ?
<details>
<summary>Correction</summary>

```r
libelles_partielle <- libelles |> filter(etiquette_dpe != "G")
dpe_partiel <- left_join(dpe, libelles_partielle, by = "etiquette_dpe")

dpe_partiel |> filter(is.na(niveau)) |> nrow()
# [1] 285
```
Les 285 DPE en étiquette `G` restent dans le tableau, avec un `niveau` à `NA` : `left_join()` ne supprime jamais de lignes du tableau de gauche.
</details>

3. Vérifier qu'une jointure ne duplique pas les lignes : créer une table `libelles_doubles` qui contient deux fois la lettre `A`, faire la jointure, et comparer le nombre de lignes avant et après.
<details>
<summary>Correction</summary>

```r
libelles_doubles <- rbind(libelles, data.frame(etiquette_dpe = "A", niveau = "autre libellé"))

dpe_double <- left_join(dpe, libelles_doubles, by = "etiquette_dpe")

nrow(dpe)         # 10000
nrow(dpe_double)  # 10019
# Les 19 DPE en étiquette A sont dupliqués : la clé n'est pas unique dans libelles_doubles.
```
</details>

## Liens utiles

- [Documentation de `dplyr`](https://dplyr.tidyverse.org/)
- [Aide-mémoire de `dplyr` (RStudio)](https://posit.co/resources/cheatsheets/)
