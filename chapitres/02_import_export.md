# Chapitre 2 – Import et export de données

## Objectifs

Voici les objectifs de ce chapitre :
- [ ] lire un fichier CSV avec les fonctions de R de base et avec `readr`
- [ ] gérer les séparateurs, la décimale et l'encodage d'un fichier
- [ ] lire une feuille de calcul Excel
- [ ] exporter un dataframe en CSV et en Excel
- [ ] vérifier qu'un export puis un import redonnent les mêmes données
- [ ] situer un fichier avec son chemin, et vérifier le dossier de travail

## Sommaire

- [Chapitre 2 – Import et export de données](#chapitre-2--import-et-export-de-données)
  - [Objectifs](#objectifs)
  - [Sommaire](#sommaire)
  - [Préparation : packages et dossier de travail](#préparation--packages-et-dossier-de-travail)
  - [Exercice 1 – Lire un CSV avec `read.csv()`](#exercice-1--lire-un-csv-avec-readcsv)
    - [Mémo](#mémo)
    - [Exemple](#exemple)
    - [Exercices](#exercices)
  - [Exercice 2 – Exporter en CSV](#exercice-2--exporter-en-csv)
    - [Mémo](#mémo-1)
    - [Exemple](#exemple-1)
    - [Exercices](#exercices-1)
  - [Exercice 3 – Lire un fichier Excel](#exercice-3--lire-un-fichier-excel)
    - [Mémo](#mémo-2)
    - [Exemple](#exemple-2)
    - [Exercices](#exercices-2)
  - [Exercice 4 – Exporter en Excel](#exercice-4--exporter-en-excel)
    - [Mémo](#mémo-3)
    - [Exemple](#exemple-3)
    - [Exercices](#exercices-3)
  - [Liens utiles](#liens-utiles)

## Préparation : packages et dossier de travail

Ce chapitre utilise trois packages : `readr` pour les CSV (déjà vu au chapitre 1), `readxl` pour lire les fichiers Excel et `writexl` pour les écrire. `writexl` n'a pas besoin d'Excel installé.

```r
install.packages(c("readr", "readxl", "writexl"))
library(readr)
library(readxl)
library(writexl)
```

Les fichiers sont lus et écrits à partir du **dossier de travail**, qu'on affiche avec `getwd()`. Les chemins sont donnés depuis ce dossier, par exemple `"data/dpe_villeurbanne_10000.csv"`.

| Nom de la commande | Description | Exemple |
|------------------|-------------|---------|
| `getwd()` | Affiche le dossier de travail. | `getwd()` |
| `setwd()` | Change le dossier de travail. | `setwd("C:/Users/Anthony/Documents/GitHub/STA-DPE")` |
| `file.exists()` | Vérifie qu'un fichier existe à ce chemin. | `file.exists("data/dpe_villeurbanne_10000.csv")` |

> **Note : `/` et `\` dans les chemins**
> Sous Windows, R accepte `/` dans les chemins : `"data/fichier.csv"`. Le caractère `\` est réservé par R : il faut l'écrire `\\` (`"data\\fichier.csv"`). Le plus simple est d'utiliser `/`.

## Exercice 1 – Lire un CSV avec `read.csv()`

`read.csv()` est la fonction de base de R pour lire un fichier CSV. Elle ne demande aucun package. Le fichier DPE est encodé en UTF-8 avec une marque d'octets (BOM) en tête : il faut le préciser avec `fileEncoding`, sinon le nom de la première colonne est altéré.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `read.csv()` | Lit un CSV avec la virgule comme séparateur et le point comme décimale. | `file`, `fileEncoding`, `nrows` | `read.csv("data/dpe.csv", fileEncoding = "UTF-8-BOM")` |
| `nrows` | Limite le nombre de lignes lues. | | `nrows = 100` |
| `str()` | Affiche le type de chaque colonne. | | `str(dpe_base)` |

### Exemple

```r
# Lecture complète du fichier avec la fonction de base de R
dpe_base <- read.csv("data/dpe_villeurbanne_10000.csv", fileEncoding = "UTF-8-BOM")

dim(dpe_base)
# [1] 10000    22

# Comparaison avec read_csv() : les dates ne sont pas reconnues
class(dpe_base$date_reception_dpe)
# [1] "character"
```

> **Note : Pourquoi `"UTF-8-BOM"` ? **
> Il est particulièrement utile pour éviter les problèmes d'accents lors de l'ouverture de fichiers CSV dans Excel, notamment sous Windows.


### Exercices

1. Lire le fichier avec `read.csv()` en précisant `fileEncoding = "UTF-8-BOM"`, puis afficher le nom de la première colonne avec `colnames()`.
<details>
<summary>Correction</summary>

```r
dpe_base <- read.csv("data/dpe_villeurbanne_10000.csv", fileEncoding = "UTF-8-BOM")
colnames(dpe_base)[1]
# [1] "numero_dpe"
```
</details>

2. Lire seulement les 100 premières lignes avec `nrows`, puis vérifier les dimensions.
<details>
<summary>Correction</summary>

```r
dpe_100 <- read.csv("data/dpe_villeurbanne_10000.csv",
                    fileEncoding = "UTF-8-BOM", nrows = 100)
dim(dpe_100)
# [1] 100  22
```
Lire un extrait permet de tester une importation rapidement, avant de lire un fichier complet.
</details>

> **Note : `read.csv()` et `read_csv()`**
> `read_csv()` (package `readr`, chapitre 1) reconnaît les dates et est plus rapide sur les gros fichiers. `read.csv()` ne convertit pas les dates : elles restent du texte, comme dans la colonne `date_reception_dpe` ci-dessus. Dans la suite de la formation, on utilise `read_csv()`.

## Exercice 2 – Exporter en CSV

Exporter un dataframe consiste à l'écrire dans un fichier pour le partager ou le réutiliser. On écrit ici un extrait des DPE, dans un dossier `sorties/` créé à la demande.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `write_csv()` | Écrit un dataframe en CSV (package `readr`). | `x`, `file` | `write_csv(extrait, "sorties/extrait.csv")` |
| `write.csv()` | Écrit en CSV avec la fonction de base. | `row.names` (TRUE par défaut) | `write.csv(extrait, "sorties/extrait.csv", row.names = FALSE)` |
| `dir.create()` | Crée un dossier. | `showWarnings` | `dir.create("sorties", showWarnings = FALSE)` |

### Exemple

```r
dpe <- read_csv("data/dpe_villeurbanne_10000.csv")

# Un extrait de 10 lignes et de 3 colonnes
dpe_extrait <- dpe[1:10, c("numero_dpe", "etiquette_dpe", "surface_habitable_logement")]

dir.create("sorties", showWarnings = FALSE)

# Export avec write_csv() : pas de numéro de ligne ajouté
write_csv(dpe_extrait, "sorties/extrait.csv")

# Export avec la fonction de base : row.names = FALSE évite une colonne de numéros
write.csv(dpe_extrait, "sorties/extrait_base.csv", row.names = FALSE)
```

> **Note : `row.names = FALSE`**
> Par défaut, `write.csv()` ajoute une première colonne de numéros de ligne, sans nom. Cette colonne se retrouve ensuite dans le fichier relu, ce qui fausse les données. Avec `row.names = FALSE`, on l'évite.

### Exercices

1. Exporter `dpe_extrait` avec `write_csv()` dans `sorties/extrait.csv`, puis le relire avec `read_csv()` et vérifier ses dimensions.
<details>
<summary>Correction</summary>

```r
write_csv(dpe_extrait, "sorties/extrait.csv")
extrait_relu <- read_csv("sorties/extrait.csv")
dim(extrait_relu)
# [1] 10  3
```
</details>

2. Exporter les 10 000 DPE complets dans `sorties/dpe_complet.csv`, sans numéros de ligne. Comparer le poids du fichier avec celui de `data/dpe_villeurbanne_10000.csv` dans l'explorateur de fichiers.
<details>
<summary>Correction</summary>

```r
write_csv(dpe, "sorties/dpe_complet.csv")
```
Les deux fichiers ont des poids proches : l'export ne change pas le contenu, seulement l'encodage et la mise en forme.
</details>

## Exercice 3 – Lire un fichier Excel

Une feuille Excel peut contenir plusieurs onglets. `read_excel()` lit un onglet choisi par son numéro ou son nom.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `read_excel()` | Lit une feuille d'un fichier `.xlsx` ou `.xls`. | `path`, `sheet` (1 par défaut), `range` | `read_excel("data/dpe.xlsx", sheet = "dpe")` |
| `excel_sheets()` | Liste les noms des onglets d'un fichier. | `path` | `excel_sheets("data/dpe.xlsx")` |

### Exemple

```r
# Créer un petit fichier Excel pour l'exemple (voir l'exercice 5 pour l'écriture)
write_xlsx(dpe_extrait, "sorties/extrait.xlsx")

# Lister les onglets, puis lire le premier
excel_sheets("sorties/extrait.xlsx")
# [1] "Sheet1"

extrait_xlsx <- read_excel("sorties/extrait.xlsx", sheet = 1)
dim(extrait_xlsx)
# [1] 10  3
```

### Exercices

1. Lister les onglets du fichier `sorties/extrait.xlsx`, puis lire le premier onglet avec `read_excel()`.
<details>
<summary>Correction</summary>

```r
excel_sheets("sorties/extrait.xlsx")
extrait_xlsx <- read_excel("sorties/extrait.xlsx")
head(extrait_xlsx, n = 3)
```
</details>

2. Lire seulement les lignes 1 à 4 du tableau, c'est-à-dire l'en-tête et les trois premières lignes de données, avec `range = "A1:C4"`.
<details>
<summary>Correction</summary>

```r
extrait_court <- read_excel("sorties/extrait.xlsx", range = "A1:C4")
dim(extrait_court)
# [1] 3 3
```
Le tableau lu compte 3 lignes de données : la ligne 1 sert d'en-tête.
</details>

3. Lire le fichier Excel avec `read_csv()` ou `read.csv()`. Que se passe-t-il ?
<details>
<summary>Correction</summary>

```r
# Le fichier .xlsx est un fichier compressé, pas un texte : la lecture échoue ou renvoie du texte illisible.
read_csv("sorties/extrait.xlsx")
```
Chaque format a sa fonction : `read_csv()` pour les fichiers texte, `read_excel()` pour les classeurs Excel.
</details>

## Exercice 4 – Exporter en Excel

`write_xlsx()` écrit un dataframe dans un classeur Excel. On peut aussi écrire **plusieurs onglets** en lui passant une liste nommée de dataframes.

### Mémo

| Nom de la commande | Description | Argument pertinent | Exemple |
|------------------|-------------|--------------------|---------|
| `write_xlsx()` | Écrit un dataframe ou une liste nommée de dataframes en `.xlsx`. | `x`, `path` | `write_xlsx(dpe_extrait, "sorties/extrait.xlsx")` |

### Exemple

```r
# Une liste nommée : chaque élément devient un onglet
classeur <- list(
  extrait = dpe_extrait,
  etiquettes = data.frame(etiquette_dpe = c("A", "B", "C", "D", "E", "F", "G"))
)

write_xlsx(classeur, "sorties/classeur.xlsx")
excel_sheets("sorties/classeur.xlsx")
# [1] "extrait"    "etiquettes"
```

### Exercices

1. Écrire `dpe_extrait` dans un fichier `sorties/dpe_extrait.xlsx`, puis le relire et vérifier qu'il a 10 lignes et 3 colonnes.
<details>
<summary>Correction</summary>

```r
write_xlsx(dpe_extrait, "sorties/dpe_extrait.xlsx")
dim(read_excel("sorties/dpe_extrait.xlsx"))
# [1] 10  3
```
</details>

2. Écrire dans un même classeur `sorties/effectifs.xlsx` deux onglets : `effectifs_etiquette` (les effectifs de `etiquette_dpe` sur les 10 000 DPE) et `effectifs_energie` (les effectifs de `type_energie_n1`). Pour un effectif, utiliser `as.data.frame(table(...))`.
<details>
<summary>Correction</summary>

```r
effectifs <- list(
  effectifs_etiquette = as.data.frame(table(dpe$etiquette_dpe)),
  effectifs_energie = as.data.frame(table(dpe$type_energie_n1))
)
write_xlsx(effectifs, "sorties/effectifs.xlsx")
excel_sheets("sorties/effectifs.xlsx")
# [1] "effectifs_etiquette" "effectifs_energie"
```
`table()` produit un objet de type tableau. `as.data.frame()` le transforme en dataframe, que `write_xlsx()` peut écrire.
</details>

> **Note : `table()` et le dataframe**
> `table()` compte les effectifs, mais son résultat n'est pas un dataframe. Pour l'écrire dans un fichier ou le manipuler avec `dplyr`, il faut le convertir avec `as.data.frame()`. Les colonnes obtenues s'appellent `Var1` et `Freq` : on peut les renommer avec `names()`.

## Liens utiles

- [Documentation de `readr`](https://readr.tidyverse.org/)
- [Documentation de `readxl`](https://readxl.tidyverse.org/)
- [Documentation de `writexl`](https://docs.ropensci.org/writexl/)
