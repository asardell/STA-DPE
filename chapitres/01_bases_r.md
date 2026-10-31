# Chapitre 1 – Bases de R et RStudio

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] comprendre ce qu'est R et comment il se situe par rapport à Python
- [ ] prendre en main RStudio, les scripts et les commentaires
- [ ] créer des objets et des vecteurs, et accéder à leurs éléments
- [ ] importer un fichier CSV avec l'assistant de RStudio
- [ ] explorer un dataframe : dimensions, colonnes, valeurs manquantes
- [ ] calculer des statistiques simples sur une colonne
- [ ] créer un dataframe à partir de vecteurs

## Sommaire

- [Introduction : R, RStudio et Python](#introduction--r-rstudio-et-python)
- [Préparation : installer et charger des packages](#préparation--installer-et-charger-des-packages)
- [Exercice 1 – Objets et affectation](#exercice-1--objets-et-affectation)
- [Exercice 2 – Vecteurs et accès aux éléments](#exercice-2--vecteurs-et-accès-aux-éléments)
- [Exercice 3 – Importer les DPE de Villeurbanne](#exercice-3--importer-les-dpe-de-villeurbanne)
- [Exercice 4 – Explorer le dataframe](#exercice-4--explorer-le-dataframe)
- [Exercice 5 – Valeurs manquantes et statistiques simples](#exercice-5--valeurs-manquantes-et-statistiques-simples)
- [Exercice 6 – Créer un dataframe à partir de vecteurs](#exercice-6--créer-un-dataframe-à-partir-de-vecteurs)

## Introduction : R, RStudio et Python

**R** est un langage conçu pour la statistique et l'analyse de données. Il manipule nativement des tableaux de données et produit des analyses et des graphiques. **Python** est un langage généraliste qui s'est fortement développé en data science avec `pandas`, `numpy` ou `scikit-learn`. Sur les données, les deux langages font sensiblement la même chose : les concepts (vecteurs, tableaux, filtres, agrégations, tests) se retrouvent dans les deux.

Dans cette formation, on utilise **RStudio**, un environnement qui regroupe :
- un éditeur de scripts (en haut à gauche) ;
- une console où s'exécutent les commandes (en bas à gauche) ;
- l'environnement, qui liste les objets créés (en haut à droite) ;
- les fichiers, les graphiques et l'aide (en bas à droite).

Pour exécuter une ligne de script, on place le curseur dessus et on appuie sur `Ctrl + Entrée`.

Quelques différences utiles quand on vient de Python :

| Sujet | R | Python |
|-------|---|--------|
| Affectation | `x <- 5` (ou `x = 5`) | `x = 5` |
| Position du premier élément | 1 | 0 |
| Liste de valeurs | vecteur, créé avec `c()` | liste `[]` ou tableau `numpy` |
| Tableau de données | `data.frame` (ou `tibble`) | `pd.DataFrame` |
| Accéder à une colonne | `df$colonne` | `df["colonne"]` |
| Valeur manquante | `NA` | `NaN`, `None`, `pd.NA` |
| Installer un package | `install.packages("nom")` | `pip install nom` |
| Charger un package | `library(nom)` | `import nom` |
| Aide sur une fonction | `?mean` | `help(mean)` |

Deux points à retenir dès le début :
- **R est sensible à la casse** : `Surface` et `surface` sont deux objets différents.
- **Les opérations sont vectorisées** : `surfaces * 2` multiplie chaque élément du vecteur, sans boucle.

Les exemples portent sur les **DPE (diagnostics de performance énergétique) de l'ADEME pour Villeurbanne**, le fil rouge de la formation. Les valeurs tapées à la main sont reprises de ce jeu de données.

## Préparation : installer et charger des packages

R ne contient que les fonctions de base. Les autres fonctions viennent de **packages**, à installer une fois puis à charger dans chaque script.

| Nom de la commande | Description | Exemple |
|------------------|-------------|---------|
| `install.packages()` | Télécharge et installe un package (une seule fois). | `install.packages("readr")` |
| `library()` | Charge un package dans la session (à chaque script). | `library(readr)` |

Pour ce chapitre, on a besoin du package `readr` (import de fichiers texte) et du package `dplyr` (manipulation de données, chapitre 3) :

```r
install.packages(c("readr", "dplyr"))
library(readr)
library(dplyr)
```

> **Note : `::` pour utiliser une fonction sans charger le package**
> Écrire `readr::read_csv()` appelle la fonction `read_csv()` du package `readr` sans le charger avec `library()`. Cette écriture évite les conflits de noms entre packages (par exemple `filter()` existe dans `dplyr` et dans la base de R). Dans un script, on la retrouve souvent.

## Exercice 1 – Objets et affectation

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `<-` | Affecte une valeur à un objet. | | `surface <- 21.3` |
| `c()` | Concatène des valeurs pour créer un vecteur. | | `c("A", "B", "C")` |
| `print()` | Affiche un objet dans la console. | | `print(surface)` |
| `rm()` | Supprime des objets de l'environnement. | `list = ls()` supprime tous les objets | `rm(surface)` |
| `class()` | Retourne le type d'un objet. | | `class(surface)` |

### Exemple

```r
# Affecter une valeur à un objet : une surface de 21,3 m² relevée dans le fichier DPE
surface_1 <- 21.3

# Afficher l'objet (taper son nom suffit dans la console)
print(surface_1)

# Le type de l'objet
class(surface_1)
# [1] "numeric"

# R est sensible à la casse : ce sont deux objets différents
Surface_1 <- 80
surface_1
Surface_1
```

### Exercices

1. Créer un objet `surface_2` avec la valeur `62.7`, puis un objet `surface_totale` égal à la somme de `surface_1` et `surface_2`. Afficher `surface_totale`.
<details>
<summary>Correction</summary>

```r
surface_2 <- 62.7
surface_totale <- surface_1 + surface_2
print(surface_totale)
# [1] 84
```
</details>

2. Créer un objet `Surface_2` avec la valeur `56.8`. Afficher `surface_2` puis `Surface_2`. Que remarque-t-on ?
<details>
<summary>Correction</summary>

```r
Surface_2 <- 56.8
surface_2  # 62.7 : la valeur créée à la question 1
Surface_2  # 56.8 : un autre objet, car R distingue les majuscules
```
</details>

3. Supprimer les objets `Surface_1`, `Surface_2` et `Surface_2` (dans cet ordre, en vérifiant d'abord avec `print()` qu'ils existent). Puis supprimer tous les objets d'un coup.
<details>
<summary>Correction</summary>

```r
rm(Surface_1)
rm(Surface_2)
rm(list = ls())  # supprime tous les objets restants
```
</details>

> **Note : l'affectation**
> `<-` est la façon habituelle d'affecter une valeur en R, et `=` fonctionne aussi. Dans un script, on préfère `<-` : c'est le style de la communauté R, et il évite de confondre l'affectation avec les arguments d'une fonction.

## Exercice 2 – Vecteurs et accès aux éléments

Un **vecteur** est une suite de valeurs de même type. On le crée avec `c()`. Les positions commencent à **1** en R (contrairement à Python où elles commencent à 0).

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `c()` | Crée un vecteur à partir de valeurs. | | `c(21.3, 62.7, 56.8)` |
| `:` | Génère une séquence avec un pas de 1. | | `8:15` |
| `seq()` | Génère une séquence avec un pas choisi. | `from`, `to`, `by` | `seq(from = 0, to = 100, by = 20)` |
| `rep()` | Répète les éléments d'un vecteur. | `times` | `rep("E", times = 3)` |
| `length()` | Retourne le nombre d'éléments. | | `length(surfaces)` |
| `class()` | Retourne le type du vecteur. | | `class(etiquettes)` |
| `[ ]` | Accède aux éléments par leur position. | | `surfaces[2]` |
| `[-n]` | Accède à tous les éléments sauf la position `n`. | | `surfaces[-1]` |

### Exemple

```r
# Les étiquettes DPE, de la meilleure (A) à la moins bonne (G)
etiquettes <- c("A", "B", "C", "D", "E", "F", "G")

# Les surfaces habitables des quatre premiers logements du fichier
surfaces <- c(21.3, 62.7, 56.8, 76.6)

class(etiquettes)   # "character" : du texte
class(surfaces)     # "numeric" : des nombres
length(surfaces)    # 4

# Accès aux éléments : position 1 = premier élément
surfaces[1]         # 21.3
surfaces[c(1, 3)]   # 21.3 56.8
surfaces[-1]        # tout sauf le premier : 62.7 56.8 76.6
```

### Exercices

1. Afficher le type de `etiquettes` et de `surfaces` avec `class()`. Pourquoi les deux types sont-ils différents ?
<details>
<summary>Correction</summary>

```r
class(etiquettes)  # "character" : les valeurs sont entre guillemets
class(surfaces)    # "numeric" : les valeurs sont des nombres
```
Les guillemets indiquent du texte : `"A"` est une chaîne de caractères, `21.3` est un nombre.
</details>

2. Générer la séquence des nombres de 0 à 100 avec un pas de 20 (avec `seq()`), puis la séquence de 8 à 15 (avec `:`).
<details>
<summary>Correction</summary>

```r
seq(from = 0, to = 100, by = 20)
# [1]   0  20  40  60  80 100
8:15
# [1]  8  9 10 11 12 13 14 15
```
</details>

3. Répéter la lettre `"E"` trois fois avec `rep()`. Calculer ensuite le nombre de logements dans le vecteur `surfaces` avec `length()`.
<details>
<summary>Correction</summary>

```r
rep("E", times = 3)
# [1] "E" "E" "E"
length(surfaces)
# [1] 4
```
</details>

4. Afficher le troisième élément de `surfaces`, puis tous les éléments sauf le dernier.
<details>
<summary>Correction</summary>

```r
surfaces[3]     # 56.8
surfaces[-4]    # 21.3 62.7 56.8
```
</details>

5. Créer un vecteur `lettres_repetees` qui contient les étiquettes `"A"` et `"B"`, chacune répétée deux fois, puis afficher sa longueur.
<details>
<summary>Correction</summary>

```r
lettres_repetees <- rep(c("A", "B"), times = 2)
lettres_repetees  # "A" "B" "A" "B"
length(lettres_repetees)  # 4
```
Le paramètre `times` répète le vecteur entier. Pour répéter chaque élément, il faut `each = 2`, ce qui donne `"A" "A" "B" "B"`.
</details>

> **Note : `rep()` répète le vecteur, `each` répète chaque élément**
> `rep(c("A", "B"), times = 2)` donne `"A" "B" "A" "B"`. `rep(c("A", "B"), each = 2)` donne `"A" "A" "B" "B"`. Confondre les deux est une erreur fréquente.

## Exercice 3 – Importer les DPE de Villeurbanne

Le fichier `dpe_villeurbanne_10000.csv` se trouve dans le dossier `data/` du dépôt. Il contient **10 000 DPE** de Villeurbanne (code postal 69100) et **22 colonnes**, issues de l'API de l'ADEME. Chaque ligne est un diagnostic.

### Mémo

| Élément | Description |
|---------|-------------|
| `File > Import Dataset > From Text (readr)` | Ouvre l'assistant d'import d'un fichier CSV. |
| `read_csv()` | Fonction du package `readr` qui lit un fichier CSV. Elle détecte les types de colonnes automatiquement. |

### Exemple

Le plus simple est l'assistant de RStudio, qui génère le code de lecture :

1. Dans RStudio, ouvrir **File > Import Dataset > From Text (readr)**.
2. Choisir le fichier `data/dpe_villeurbanne_10000.csv`.
3. Vérifier l'aperçu : les colonnes sont séparées par des virgules et les accents s'affichent correctement.
4. Conserver le nom proposé (`dpe_villeurbanne_10000`) et cliquer sur **Import**.
5. La console affiche le code exécuté. Le copier dans le script pour le conserver.

```r
library(readr)
dpe_villeurbanne_10000 <- read_csv("data/dpe_villeurbanne_10000.csv")
```

Le fichier est ensuite raccourci pour la suite du chapitre :

```r
dpe <- dpe_villeurbanne_10000
```

### Exercices

1. Importer le fichier avec l'assistant, puis vérifier le nombre de lignes et de colonnes du dataframe.
<details>
<summary>Correction</summary>

```r
dim(dpe)
# [1] 10000    22
```
Le fichier compte 10 000 DPE sur 22 colonnes.
</details>

2. Dans la console, vérifier le type de la colonne `date_reception_dpe`, qui est au format `AAAA-MM-JJ`. Que remarque-t-on ?
<details>
<summary>Correction</summary>

```r
class(dpe$date_reception_dpe)
# [1] "Date"
```
`read_csv()` reconnaît le format `AAAA-MM-JJ` et crée une colonne de dates. Dans un fichier où les dates sont mal reconnues, on verra au chapitre 4 comment les convertir.
</details>

3. (Facultatif) Relancer l'import sans l'assistant, en tapant directement la commande `read_csv()`. Le résultat doit être identique.
<details>
<summary>Correction</summary>

```r
dpe_bis <- read_csv("data/dpe_villeurbanne_10000.csv")
dim(dpe_bis)
# [1] 10000    22
```
</details>

> **Note : chemin relatif**
> `"data/dpe_villeurbanne_10000.csv"` est un chemin relatif : R le cherche depuis le dossier de travail. Vérifier ce dossier avec `getwd()`. Si le fichier n'est pas trouvé, ouvrir le projet RStudio situé à la racine du dépôt, ou donner le chemin complet.

## Exercice 4 – Explorer le dataframe

Un **dataframe** est un tableau dont chaque colonne est un vecteur. Une colonne se récupère avec `$` et devient un vecteur. On peut aussi sélectionner des lignes et des colonnes avec `[ , ]`.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `$` | Accède à une colonne sous forme de vecteur. | | `dpe$etiquette_dpe` |
| `[ , ]` | Sélectionne des lignes et des colonnes. | `lignes`, `colonnes` | `dpe[1:3, c("etiquette_dpe", "type_energie_n1")]` |
| `dim()` | Retourne le nombre de lignes et de colonnes. | | `dim(dpe)` |
| `nrow()`, `ncol()` | Retournent le nombre de lignes ou de colonnes. | | `nrow(dpe)` |
| `colnames()` | Retourne les noms des colonnes. | | `colnames(dpe)` |
| `head()` | Affiche les premières lignes. | `n` (6 par défaut) | `head(dpe, n = 3)` |
| `summary()` | Résume chaque colonne. | | `summary(dpe)` |
| `View()` | Ouvre le dataframe dans une fenêtre. | | `View(dpe)` |
| `unique()` | Retourne les valeurs distinctes d'un vecteur. | | `unique(dpe$etiquette_dpe)` |
| `table()` | Compte les effectifs de chaque valeur. | | `table(dpe$etiquette_dpe)` |
| `prop.table()` | Convertit des effectifs en proportions. | | `prop.table(table(dpe$etiquette_dpe))` |
| `sort()` | Trie un vecteur. | `decreasing` (FALSE par défaut) | `sort(x, decreasing = TRUE)` |

### Exemple

```r
# Les noms des colonnes, puis les trois premières lignes
colnames(dpe)
head(dpe, n = 3)

# Une colonne, sous forme de vecteur : les dix premières étiquettes
dpe$etiquette_dpe[1:10]
# [1] "E" "C" "C" "E" "C" "C" "C" "C" "C" "C"

# Sélection de lignes et de colonnes : les trois premières lignes de deux colonnes
dpe[1:3, c("etiquette_dpe", "type_energie_n1")]

# Effectifs et proportions des étiquettes
table(dpe$etiquette_dpe)
prop.table(table(dpe$etiquette_dpe))
```

### Exercices

1. Afficher le nombre de lignes, le nombre de colonnes et les noms des colonnes de `dpe`.
<details>
<summary>Correction</summary>

```r
nrow(dpe)     # 10000
ncol(dpe)     # 22
colnames(dpe)
```
</details>

2. Afficher les 5 premières valeurs de la colonne `surface_habitable_logement`, puis la valeur de la 5e ligne.
<details>
<summary>Correction</summary>

```r
dpe$surface_habitable_logement[1:5]
# [1] 21.3 62.7 56.8 76.6 61.9
dpe$surface_habitable_logement[5]
# [1] 61.9
```
</details>

3. Afficher les étiquettes DPE distinctes, puis leurs effectifs du plus fréquent au moins fréquent.
<details>
<summary>Correction</summary>

```r
unique(dpe$etiquette_dpe)
# [1] "A" "B" "C" "D" "E" "F" "G"
sort(table(dpe$etiquette_dpe), decreasing = TRUE)
```
</details>

4. Afficher les effectifs de `type_energie_n1` en proportions. Quelle est l'énergie la plus fréquente ?
<details>
<summary>Correction</summary>

```r
prop.table(table(dpe$type_energie_n1))
sort(prop.table(table(dpe$type_energie_n1)), decreasing = TRUE)
# L'énergie la plus fréquente est lue en premier dans le tableau trié.
```
</details>

5. Sélectionner les colonnes `numero_dpe` et `etiquette_dpe` pour les 3 premières lignes, puis afficher le résumé de tout le dataframe avec `summary()`.
<details>
<summary>Correction</summary>

```r
dpe[1:3, c("numero_dpe", "etiquette_dpe")]
summary(dpe)
```
</details>

> **Note : `head()` et `View()`**
> `head()` affiche quelques lignes dans la console. `View()` ouvre le tableau complet dans une fenêtre et permet de trier et de filtrer à la souris. Les deux servent à regarder les données, sans les modifier.

## Exercice 5 – Valeurs manquantes et statistiques simples

Une valeur manquante est notée **`NA`** en R. Elle se propage : une moyenne calculée sur une colonne qui contient un `NA` vaut `NA`, sauf si on demande d'ignorer les valeurs manquantes avec `na.rm = TRUE`.

Les statistiques détaillées (quantiles, variance, écart interquartile) sont reprises au chapitre 4. Ici, on prend en main les fonctions de base sur une colonne.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `is.na()` | Indique quelles valeurs sont manquantes (`TRUE`/`FALSE`). | | `is.na(dpe$surface_habitable_logement)` |
| `sum()` | Calcule la somme, ou compte les `TRUE` d'un vecteur logique. | `na.rm` | `sum(is.na(dpe$surface_habitable_logement))` |
| `mean()`, `median()` | Moyenne et médiane. | `na.rm` (FALSE par défaut) | `mean(dpe$surface_habitable_logement, na.rm = TRUE)` |
| `min()`, `max()` | Minimum et maximum. | `na.rm` (FALSE par défaut) | `max(dpe$surface_habitable_logement, na.rm = TRUE)` |
| `sd()`, `var()` | Écart type et variance. | `na.rm` (FALSE par défaut) | `sd(dpe$surface_habitable_logement, na.rm = TRUE)` |
| `quantile()` | Quantiles. | `probs`, `na.rm` | `quantile(x, probs = c(0.25, 0.5, 0.75), na.rm = TRUE)` |
| `round()` | Arrondit les nombres. | `digits` (0 par défaut) | `round(55.93, digits = 1)` |

### Exemple

```r
surface <- dpe$surface_habitable_logement

# Sans na.rm, une seule valeur manquante suffit pour obtenir NA
mean(surface)
# [1] NA

# Combien de valeurs manquantes ?
sum(is.na(surface))
# [1] 101

# Avec na.rm = TRUE, R ignore les valeurs manquantes
mean(surface, na.rm = TRUE)
# [1] 55.93...
round(mean(surface, na.rm = TRUE), digits = 1)
# [1] 55.9
```

### Exercices

1. Calculer la moyenne de `surface_habitable_logement` sans `na.rm`, puis avec `na.rm = TRUE`. Expliquer la différence.
<details>
<summary>Correction</summary>

```r
mean(dpe$surface_habitable_logement)                # NA
mean(dpe$surface_habitable_logement, na.rm = TRUE)  # environ 55,9
# Sans na.rm, la présence d'un NA rend le résultat NA. Avec na.rm = TRUE,
# les DPE sans surface sont ignorés dans le calcul.
```
</details>

2. Compter les valeurs manquantes de `surface_habitable_logement`, puis celles de `annee_construction`.
<details>
<summary>Correction</summary>

```r
sum(is.na(dpe$surface_habitable_logement))  # 101
sum(is.na(dpe$annee_construction))          # 3187
```
Plus de 30 % des DPE n'ont pas d'année de construction renseignée.
</details>

3. Calculer la médiane, le minimum et le maximum de `surface_habitable_logement` (valeurs manquantes ignorées). Le maximum vous paraît-il plausible pour un logement ?
<details>
<summary>Correction</summary>

```r
median(dpe$surface_habitable_logement, na.rm = TRUE)  # 54
min(dpe$surface_habitable_logement, na.rm = TRUE)     # 8.5
max(dpe$surface_habitable_logement, na.rm = TRUE)     # 3330
# Un logement de 3 330 m² est très improbable pour un logement résidentiel.
# Cette valeur extrême sera traitée au chapitre 4.
```
</details>

4. Calculer l'écart type et la somme de `surface_habitable_logement`, en ignorant les valeurs manquantes. Arrondir l'écart type à une décimale.
<details>
<summary>Correction</summary>

```r
sd(dpe$surface_habitable_logement, na.rm = TRUE)  # 60.27
sum(dpe$surface_habitable_logement, na.rm = TRUE) # 553644.1
round(sd(dpe$surface_habitable_logement, na.rm = TRUE), digits = 1)  # 60.3
```
</details>

5. Calculer les quartiles 1, 2 et 3 de `surface_habitable_logement` avec `quantile()`.
<details>
<summary>Correction</summary>

```r
quantile(dpe$surface_habitable_logement, probs = c(0.25, 0.5, 0.75), na.rm = TRUE)
#   25%   50%   75%
# 35.0  54.0  70.9
```
</details>

> **Note : `NA` et les fonctions de calcul**
> Les fonctions `mean()`, `median()`, `sd()`, `sum()`, `min()`, `max()` et `quantile()` ont toutes un argument `na.rm`. Son oubli est une erreur fréquente : le résultat est `NA` sans message d'erreur. Toujours vérifier le nombre de valeurs manquantes avant un calcul.

## Exercice 6 – Créer un dataframe à partir de vecteurs

Un dataframe se construit aussi à partir de vecteurs de **même longueur**, avec `data.frame()`. En Python, on utilise `pd.DataFrame()` avec un dictionnaire de listes.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `data.frame()` | Crée un tableau à partir de vecteurs de même longueur. | Un argument par colonne, `nom = vecteur` | `data.frame(a = 1:3, b = c("x", "y", "z"))` |

### Exemple

```r
# Trois vecteurs de même longueur : les 4 premiers logements
etiquette <- dpe$etiquette_dpe[1:4]
surface <- dpe$surface_habitable_logement[1:4]
energie <- dpe$type_energie_n1[1:4]

# Assemblage en un dataframe : chaque argument devient une colonne
dpe_extrait <- data.frame(
  etiquette_dpe = etiquette,
  surface_habitable_logement = surface,
  type_energie_n1 = energie
)

dpe_extrait
dim(dpe_extrait)  # 4 3
class(dpe_extrait)  # "data.frame"
```

*Équivalent Python :*
```python
import pandas as pd

dpe_extrait = pd.DataFrame({
    "etiquette_dpe": etiquette,
    "surface_habitable_logement": surface,
    "type_energie_n1": energie,
})
```

### Exercices

1. Créer un dataframe `dpe_dix` à partir des 10 premières lignes des colonnes `etiquette_dpe`, `surface_habitable_logement` et `type_energie_n1`. Vérifier ses dimensions.
<details>
<summary>Correction</summary>

```r
dpe_dix <- data.frame(
  etiquette_dpe = dpe$etiquette_dpe[1:10],
  surface_habitable_logement = dpe$surface_habitable_logement[1:10],
  type_energie_n1 = dpe$type_energie_n1[1:10]
)
dim(dpe_dix)  # 10 3
```
</details>

2. Dans `dpe_dix`, sélectionner uniquement la colonne `surface_habitable_logement` avec `$`, puis la calculer la moyenne.
<details>
<summary>Correction</summary>

```r
dpe_dix$surface_habitable_logement
mean(dpe_dix$surface_habitable_logement)
# [1] 53.47
```
</details>

3. Essayer de créer un dataframe avec deux vecteurs de longueurs différentes (`c("A", "B", "C")` et `c(1, 2)`). Que se passe-t-il ?
<details>
<summary>Correction</summary>

```r
data.frame(lettre = c("A", "B", "C"), nombre = c(1, 2))
# Erreur : arguments imply differing number of rows: 3, 2
```
R refuse de construire un tableau dont les colonnes n'ont pas la même longueur.
</details>

## Liens utiles

- [Données DPE de l'ADEME](https://data.ademe.fr/datasets/dpe03existant)
- [Documentation de R (manuels officiels)](https://cran.r-project.org/manuals.html)
