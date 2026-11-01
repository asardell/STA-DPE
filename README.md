# STA-DPE

Ce référentiel regroupe les supports de la formation de trois jours « Statistiques avec R ». Le fil rouge est l'analyse des DPE (diagnostics de performance énergétique) de Villeurbanne, récupérés via l'API de l'ADEME.

Les données sont filtrées sur le code postal de Villeurbanne (`code_postal_ban`). Le fichier utilisé est `data/dpe_villeurbanne_10000.csv` (10 000 DPE).

Les exercices sont réalisés sous R dans RStudio. Chaque chapitre suit les mêmes sections : objectifs, mémo, exercices avec leur correction repliable.

## Démarche commune à tous les tests

Avant chaque test, le chapitre pose le cadre suivant :

- **Hypothèses** : l'hypothèse nulle H0 (ce qu'on suppose vrai par défaut) et l'hypothèse alternative H1 (ce qu'on cherche à démontrer), avec le caractère bilatéral ou unilatéral du test.
- **Risque alpha (α)** : probabilité de rejeter H0 alors qu'elle est vraie (erreur de première espèce). C'est le seuil fixé avant le calcul, en général 5 %.
- **Risque bêta (β)** : probabilité de ne pas rejeter H0 alors qu'elle est fausse (erreur de seconde espèce).
- **Puissance (1 − β)** : probabilité de détecter un écart réel. Elle sert à calculer la taille d'échantillon nécessaire.
- **Règle de décision** : on rejette H0 si la p-value est inférieure à α.

## Objectifs

- [ ] Manipuler des données avec R (import, dataframes, dplyr)
- [ ] Maîtriser les statistiques descriptives : position, dispersion, quantiles, graphiques
- [ ] Analyser le lien entre deux variables, quantitatives ou qualitatives
- [ ] Générer et représenter des variables aléatoires (lois uniforme et normale)
- [ ] Calculer des intervalles de confiance
- [ ] Poser les hypothèses et les risques α et β d'un test, et interpréter la p-value
- [ ] Choisir et réaliser un test sur un, deux ou plusieurs échantillons
- [ ] Calculer la puissance d'un test et la taille d'échantillon nécessaire
- [ ] Tester l'indépendance de deux variables qualitatives (Khi²) et le lien entre deux variables quantitatives (corrélation, régression)
- [ ] Comparer plusieurs groupes avec une ANOVA, ses variantes et ses comparaisons multiples
- [ ] Résumer un jeu de données multidimensionnel (ACP, AFE, ACM)

## Planning

### Jour 1 : Bases de R et statistiques descriptives

1. Chapitre 1 : Bases de R et RStudio (objets, types, vecteurs, dataframes, packages)
2. Chapitre 2 : Import et export de données (CSV, Excel)
3. Chapitre 3 : Manipulation de dataframes avec dplyr
4. Chapitre 4 : Statistiques descriptives d'une variable (moyenne, médiane, quartiles, déciles, centiles, variance, écart-type, étendue, écart interquartile, histogramme, boxplot, effectifs, fréquences, effectifs cumulés, diagramme en barres et circulaire)
5. Chapitre 5 : Liaisons entre deux variables (covariance et corrélation, matrice de corrélation, nuage de points, tableaux croisés et profils lignes et colonnes, diagrammes en barres empilés, statistiques et boxplots par groupe)

### Jour 2 : Lois de probabilité, estimation et tests sur un échantillon

6. Chapitre 6 : Génération et représentation de variables aléatoires (loi uniforme, loi normale, influence de la taille de l'échantillon)
7. Chapitre 7 : Intervalles de confiance (moyenne avec le test Z, moyenne avec le test t, proportion)
8. Chapitre 8 : Démarche d'un test et tests d'ajustement (hypothèses, α, β, puissance, p-value ; test de Shapiro-Wilk, test d'Anderson-Darling ; transformations de Box-Cox et de Johnson ; robustesse d'un test quand ses conditions ne sont pas remplies)
9. Chapitre 9 : Tests sur une moyenne (test Z et test t à un échantillon, bilatéraux), puissance d'un test Z et d'un test t, taille d'échantillon d'un test Z et d'un test t
10. Chapitre 10 : Tests sur une proportion et de Poisson (comparaison de proportions sur une même population par test binomial exact, taille d'échantillon d'une proportion à un échantillon, test de Poisson à un échantillon unilatéral et bilatéral)
11. Chapitre 11 : Comparaison de deux échantillons (test de Fisher et test de Levene sur l'égalité des variances, test t homoscédastique, test de Welch, test t pour données appariées, comparaison de deux proportions sur deux échantillons indépendants, taille d'échantillon pour deux proportions)

### Jour 3 : Liaisons, modèles et analyses multidimensionnelles

12. Chapitre 12 : Test de corrélation de Pearson et régression linéaire simple (équation de régression, prédiction, résidus)
13. Chapitre 13 : Test du Khi² (test d'indépendance de deux variables qualitatives, test d'ajustement)
14. Chapitre 14 : ANOVA à un facteur fixe (ANOVA-1 canonique désempilée, test de Levene et test de Bartlett d'égalité des variances, comparaisons multiples de Student avec correction de Bonferroni, test HSD de Tukey)
15. Chapitre 15 : ANOVA à deux facteurs fixes (sans répétitions avec ou sans interaction, avec répétitions avec ou sans interaction)
16. Chapitre 16 : Plans particuliers (ANOVA imbriquée et hiérarchique complète, ANOVA carré latin, ANCOVA : analyse de la covariance)
17. Chapitre 17 : MANOVA (analyse de la variance multivariée)
18. Chapitre 18 : Analyses multidimensionnelles (ACP paramétrique, analyse factorielle exploratoire avec ACP sans rotation puis avec rotation varimax, analyse des correspondances multiples)

Le Jour 3 regroupe huit chapitres : prévoir un rythme soutenu, ou reporter les chapitres 16 et 17 si le temps manque.

## Fonctions abordées

| Chapitre | Fonctions et packages |
|----------|-----------------------|
| 1 | `c()`, `:`, `seq()`, `rep()`, `length()`, `class()`, `print()`, `data.frame()`, `head()`, `dim()`, `summary()`, `install.packages()`, `library()` |
| 2 | `getwd()`, `setwd()`, `file.exists()`, `read.csv()`, `read.csv2()`, `read.table()`, `read_csv()`, `write_csv()`, `write.csv()`, `write.csv2()`, `read_excel()`, `excel_sheets()`, `write_xlsx()`, `dir.create()`, `str()`, `nrow()`, `ncol()`, `colnames()`, `$` |
| 3 | `[ , ]`, `subset()`, `order()`, `select()`, `rename()`, `starts_with()`, `filter()`, `is.na()`, `%in%`, `mutate()`, `arrange()`, `desc()`, `group_by()`, `summarise()`, `count()`, `n()`, `ungroup()`, `left_join()`, `distinct()`, pipe `\|>` |
| 4 | `summary()`, `mean()`, `median()`, `var()`, `sd()`, `min()`, `max()`, `range()`, `diff()`, `quantile()`, `IQR()`, `which()`, `round()`, `table()`, `prop.table()`, `cumsum()`, `hist()`, `boxplot()`, `barplot()`, `pie()`, `ifelse()`, `ggplot()`, `geom_histogram()`, `geom_boxplot()`, `geom_bar()`, `coord_cartesian()`, `scale_fill_manual()`, `esquisser()` (package `esquisse`), `plot_ly()`, `layout()`, `ggplotly()` (package `plotly`) |
| 5 | `cov()`, `cor()` (`use`, `method = "spearman"`), `addmargins()`, `plot()`, `abline()`, `lm()` (droite de tendance, visuelle), `table()`, `xtabs()`, `prop.table()` (`margin` 1 et 2), `barplot()` (`beside`), `tapply()`, `aggregate()`, `boxplot()` (formule), `ggplot()`, `geom_point()`, `geom_smooth()`, `geom_tile()`, `geom_bar()` (`position`), `geom_boxplot()`, `scale_fill_gradient2()`, `plot_ly()`, `ggplotly()`, `esquisser()` |
| 6 | `runif()`, `rnorm()`, `set.seed()`, `replicate()`, `dunif()`, `punif()`, `qunif()`, `dnorm()`, `pnorm()`, `qnorm()`, `quantile()`, `qqnorm()`, `qqline()`, `stat_function()`, `stat_qq()`, `geom_density()`, `facet_wrap()`, `after_stat()`, `add_trace()` (package `plotly`), `histnorm` |
| 7 | `qnorm()`, `qt()`, `sqrt()`, `length()`, `mean()`, `sd()`, `sample()`, `replicate()`, `t.test()` (`$conf.int`, `conf.level`), `prop.test()`, `cat()`, `for`, `geom_point()`, `geom_errorbar()`, `ggplotly()` |
| 8 | `t.test()` (`$p.value`, `mu`), `replicate()`, `shapiro.test()`, `head()`, `ad.test()` (package `nortest`), `lm()`, `MASS::boxcox()`, `which.max()`, `bestNormalize::yeojohnson()` (`$lambda`, `$x.t`), `bestNormalize::bestNormalize()` (`$chosen_transform`, `$x.t`), `wilcox.test()` (comparaison hors programme), `plot_ly()`, `add_trace()`, `qqnorm()` (`plot.it = FALSE`), `qqline()`, `log()`, `rexp()`, `ggplot()` (`geom_histogram()`) |
| 9 | `t.test()` (un échantillon, `$conf.int`, `$p.value`), `pnorm()`, `qnorm()`, `pwr.norm.test()`, `pwr.t.test()`, `replicate()`, `sapply()`, `ggplot()`, `geom_vline()`, `annotate()`, `ggplotly()`, `plot_ly()` (package `pwr` pour les puissances) |
| 10 | `binom.test()`, `poisson.test()` (`T`, `r`, `alternative`), `prop.test()`, `pwr.p.test()`, `ES.h()`, `dbinom()`, `geom_col()`, `ggplotly()`, `plot_ly()` |
| 11 | `var.test()`, `car::leveneTest()`, `t.test()` (`var.equal = TRUE`, `paired = TRUE`, Welch), `prop.test()` (deux échantillons, `$conf.int`), `pwr.2p.test()`, `pwr.2p2n.test()`, `ES.h()`, `geom_boxplot()`, `plot_ly(type = "box")` |
| 12 | `cor.test()`, `lm()`, `coef()`, `predict()`, `summary()`, `resid()`, `fitted()`, `abline()` |
| 13 | `chisq.test()`, `table()`, `xtabs()`, `$expected`, `$residuals` |
| 14 | `aov()`, `summary()`, `TukeyHSD()`, `pairwise.t.test()` (correction Bonferroni), `car::leveneTest()`, `bartlett.test()` |
| 15 | `aov()` avec interaction (`A * B`), `interaction.plot()`, `summary()` |
| 16 | `aov()` avec terme imbriqué (`A / B`) et plan carré latin (`row + col + trt`), `aov()` avec covariable (ANCOVA) |
| 17 | `manova()`, `summary.manova()` |
| 18 | `FactoMineR` (`PCA()`, `MCA()`), `factoextra` (`fviz_eig()`, `fviz_pca_var()`, `fviz_mca_var()`), `psych::fa()` ou `factanal()` (rotation `varimax`) |

## Aller plus loin

Une fois les dix-huit chapitres terminés, les résultats peuvent être partagés sans que le destinataire ait besoin de R :

- **R Markdown** (package `rmarkdown`, avec Quarto comme successeur récent) : un seul fichier mêle texte, code et résultats, puis se transforme en rapport HTML, Word ou PDF. Chaque chiffre du rapport est recalculé à chaque compilation.
- **Shiny** (package `shiny`) : une application web pilotée par R. L'utilisateur choisit une étiquette DPE, une tranche de surface ou une commune, et les tests et graphiques se mettent à jour.

## Liens utiles

- [Données DPE de l'ADEME](https://data.ademe.fr/datasets/dpe03existant)
- [Quel test statistique choisir ?](https://statsandr.com/blog/what-statistical-test-should-i-do/images/overview-statistical-tests-statsandr.svg)
