# Backtesting de la VaR sur le marché belge (GARCH multivarié et VAR-GARCH)

Les rendements du marché américain améliorent-ils la prévision du risque du marché belge ? Ce projet compare un GARCH multivarié à moyenne constante et un VAR(1)-GARCH, puis teste la VaR à 95 % à un jour de chacun sur les 100 derniers jours.

Période : novembre 2025 – janvier 2026, en binôme avec Marc-Allan Monin. Outils : WinRATS (estimation), Python (VaR, backtesting, graphiques), Excel (implémentation de contrôle).

## Données

Rendements logarithmiques journaliers d'un indice belge et d'un indice américain, du 2 janvier 1990 au 23 juillet 2021 pour l'estimation, puis jusqu'au 9 décembre 2021 pour le test. Les colonnes sont `BEL-DS`, `US-DS` et `USD-BIT` (cette dernière n'est pas utilisée ici).

Le fichier `Test.xlsx` n'est pas dans ce dépôt, car les séries peuvent être soumises à une licence. Pour relancer les notebooks, place-le à la racine avec les colonnes `Name` (date), `BEL-DS`, `US-DS` et `USD-BIT`.

## Méthode

1. **Estimation sous WinRATS** (`estimations_winrats.pdf`, code et sorties) :
   - `GARCH(P=1,Q=1)` multivarié sur les deux rendements (variances et covariance de type DVECH), moyenne constante ;
   - `GARCH(P=1,Q=1,MV=VECH,REGRESSORS)` avec une moyenne VAR(1) (rendements retardés des deux marchés) et une matrice de variance VECH complète.
2. **Reconstruction de la variance conditionnelle sous Python** à partir des coefficients estimés : `h(t) = C + A·ε(t-1)ε(t-1)' + B·h(t-1)`, avec la variance inconditionnelle empirique comme valeur initiale.
3. **VaR à 95 %, un jour** : `VaR(t) = V(t-1) × (−μ(t) + 1,645 × √h11(t))` pour un portefeuille de 100 000 €, où μ(t) est la moyenne conditionnelle prévue.
4. **Backtesting** sur les 100 jours du 23 juillet au 9 décembre 2021. Une violation est une perte supérieure à la VaR. Le test de Kupiec (dernière cellule de chaque notebook) compare le taux de violations à 5 %.

## Notebooks

| Fichier | Modèle |
|---|---|
| `Bel_US-Const.ipynb` | GARCH multivarié DVECH, moyenne constante |
| `Bel_US-VAR_DVech.ipynb` | VAR(1)-GARCH, variance DVECH |
| `Bel_US-VaR.ipynb` | VAR(1)-GARCH, variance VECH complète, plus l'analyse exploratoire des séries |

## Résultats

- L'indice américain de la veille explique significativement le rendement belge du lendemain : dans la spécification VECH, le coefficient est d'environ 0,27 avec une statistique t d'environ 22.
- Sur les 100 jours testés, la VaR est dépassée 3 fois avec le modèle à moyenne constante, 4 fois avec le VAR-GARCH à variance DVECH et 3 fois avec la variance VECH complète, pour 5 attendus.
- Le test de Kupiec ne rejette aucun modèle à 5 % (p-values de 0,32 pour 3 violations et de 0,64 pour 4).
- Une implémentation Excel indépendante (non publiée, car elle contient les données) retrouve exactement les mêmes violations pour le modèle constant et pour le modèle VECH complet.
- Les modèles se valent sur cette période : la volatilité propre domine la prévision et la période est calme.

## Limites

- 100 jours de test sont trop peu pour départager les modèles ; une période de crise pourrait changer le classement.
- La VaR suppose des rendements conditionnellement normaux (quantile 1,645).
- Les coefficients viennent d'une estimation faite sous WinRATS, puis recopiés dans les notebooks.

## Reproduire

1. `pip install -r requirements.txt`
2. Place `Test.xlsx` à la racine (voir la section Données).
3. Lance les notebooks dans cet ordre : `Bel_US-Const.ipynb`, `Bel_US-VAR_DVech.ipynb`, `Bel_US-VaR.ipynb`. Chacun se termine par le décompte des violations et le test de Kupiec.

Les coefficients des modèles sont ceux estimés sous WinRATS (`estimations_winrats.pdf`) et saisis en début de notebook.
