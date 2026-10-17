# Énergies renouvelables & conditions météorologiques

Analyse exploratoire de la relation entre les paramètres météorologiques et la production d'énergie renouvelable, sur près de 200 000 mesures horaires. Statistiques descriptives, visualisations, information mutuelle et ACP.

---

## Sommaire

- [Contexte et problématique](#contexte-et-problématique)
- [Structure du dépôt](#structure-du-dépôt)
- [Jeu de données](#jeu-de-données)
- [Démarche d'analyse](#démarche-danalyse)
- [Principaux résultats](#principaux-résultats)
- [Installation et exécution](#installation-et-exécution)
- [Limites et pistes d'amélioration](#limites-et-pistes-damélioration)

---

## Contexte et problématique

La transition énergétique rend critique la compréhension du lien entre météo et production renouvelable : le rayonnement solaire, la température, l'humidité et les précipitations conditionnent directement la disponibilité et le rendement des installations solaires et éoliennes.

**Objectif** — analyser la relation entre conditions météorologiques et production d'énergie renouvelable, structurée en quatre questions :

| | Question |
|---|---|
| **Q1** | Dans quelle mesure les précipitations (pluie, neige) affectent-elles la production ? Comment les systèmes solaires et éoliens répondent-ils à leurs variations ? |
| **Q2** | Comment les variations de température influent-elles sur l'efficacité de production ? |
| **Q3** | Comment la production varie-t-elle au fil de l'année et des heures de la journée ? Existe-t-il des saisons ou des moments particulièrement favorables ? |
| **Q4** | Comment les différents types de météo (ensoleillé, pluvieux, venteux, orageux, nuageux) influent-ils sur la production ? |

---

## Structure du dépôt

| Fichier | Rôle |
|---|---|
| `renewable_energy_weather_analysis.ipynb` | Notebook complet : description des données, analyse univariée, visualisations, focus 2018, ACP, conclusion (79 cellules, 37 exécutées) |

> **Important :** le jeu de données `solar_weather.csv` n'est **pas versionné** dans ce dépôt. Le notebook est fourni avec ses sorties déjà calculées (tableaux et graphiques visibles sans réexécution), mais le relancer nécessite de se procurer le CSV et de le placer à la racine.

---

## Jeu de données

`solar_weather.csv` — **196 776 lignes × 17 colonnes**, mesures **horaires** collectées via l'**API OpenWeather**. Aucune valeur manquante.

| Colonne | Description |
|---|---|
| `Time` | Horodatage de la mesure |
| `Energy delta[Wh]` | **Variable cible** — production d'énergie |
| `GHI` | Irradiance horizontale globale (rayonnement solaire reçu par une surface horizontale) |
| `temp`, `pressure`, `humidity` | Température, pression, humidité |
| `wind_speed` | Vitesse du vent |
| `rain_1h`, `snow_1h` | Précipitations (pluie, neige) sur 1 h |
| `clouds_all` | Couverture nuageuse |
| `isSun` | Présence de soleil (0/1) |
| `sunlightTime`, `dayLength` | Durée d'ensoleillement, longueur du jour |
| `SunlightTime/daylength` | Ratio ensoleillement / durée du jour |
| `weather_type` | Type de météo (voir codage ci-dessous) |
| `hour`, `month` | Heure et mois extraits de l'horodatage |

**Codage de `weather_type`** et distribution observée :

| Code | Météo | Effectif | Part |
|---:|---|---:|---:|
| 4 | stormy (orageux) | 73 004 | 37,1 % |
| 2 | rainy (pluvieux) | 35 428 | 18,0 % |
| 3 | windy (venteux) | 31 660 | 16,1 % |
| 5 | cloudy (nuageux) | 28 708 | 14,6 % |
| 1 | sunny (ensoleillé) | 27 976 | 14,2 % |

**Autres distributions clés :**

- `isSun` : 52,0 % des heures avec soleil, 48,0 % sans — cohérent avec l'alternance jour/nuit.
- `Energy delta[Wh]` : **51,3 % des lignes sont à une production nulle ou négative** (100 850 lignes à `≤ 0`), ce qui correspond aux heures nocturnes. Seules 0,13 % des heures dépassent 4 392 Wh.
- `GHI` : 53,3 % des mesures dans l'intervalle `(-1, 5]`, soit un rayonnement quasi nul. 1,1 % seulement au-delà de 200.

> Ces trois distributions traduisent le même fait : **la moitié du jeu de données correspond à la nuit**. C'est une caractéristique structurante à garder en tête pour lire toutes les moyennes présentées ensuite.

---

## Démarche d'analyse

```
solar_weather.csv  (196 776 × 17)
      │
      ▼
[1] Description          df.info() · valeurs manquantes · doublons
      │                  suppression de la colonne sunlightTime
      │                  séparation quantitatif / qualitatif
      ▼
[2] Corrélations         matrice de corrélation (variables quantitatives)
      │                  information mutuelle (isSun, weather_type) via
      │                  mutual_info_regression + LabelEncoder
      ▼
[3] Analyse univariée    tableaux statistiques (effectifs, fréquences,
      │                  cumuls) sur weather_type, isSun, GHI, Energy delta
      ▼
[4] Visualisations       production quotidienne, mensuelle, annuelle
      │                  production vs humidité / température
      │                  croisements énergie × vent × précipitations
      ▼
[5] Focus année 2018     année de production maximale
      │                  énergie par type de météo, par mois
      ▼
[6] ACP                  StandardScaler + PCA sur les 11 variables
      │                  quantitatives · critère du coude · projection 3D
      ▼
[7] Conclusion
```

**Stack technique :** `pandas`, `numpy`, `matplotlib`, `seaborn`, `scikit-learn` (`mutual_info_regression`, `LabelEncoder`, `StandardScaler`, `PCA`).

---

## Principaux résultats

### Saisonnalité et cycle temporel (Q3)

- La production présente une **forte saisonnalité** : élevée en été (juin–août), faible en hiver (décembre–février), ce qui reflète la disponibilité du rayonnement solaire.
- La courbe de production quotidienne reproduit **presque la même forme chaque année**.
- **2018 est l'année de production maximale**, ce qui a motivé une section d'analyse dédiée.

### Précipitations et vent (Q1)

- Production d'énergie et taux de précipitations évoluent **en sens opposé** : la baisse des précipitations en 2018 a accompagné le pic de production solaire.
- La vitesse moyenne du vent évolue également à l'inverse de la production solaire. Le notebook propose l'hypothèse d'un **report vers l'éolien** lorsque le solaire baisse — hypothèse non testée, voir les limites.

### Température et humidité (Q2)

- Relation **non monotone** entre production et température : il existe une **plage de températures optimale**, la production diminuant aux températures plus basses comme plus élevées. C'est le résultat physiquement attendu (le rendement des panneaux photovoltaïques chute au-delà d'une température de jonction optimale).
- La production moyenne **augmente avec l'humidité**, résultat contre-intuitif signalé comme tel dans le notebook. Voir les limites.

### Type de météo (Q4)

- Les **jours ensoleillés** présentent la production moyenne la plus élevée, sur le périmètre 2018.

### ACP

ACP sur les **11 variables quantitatives** (après centrage-réduction). Variance expliquée par composante :

| CP | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| % | 32,3 | 15,1 | 11,4 | 8,8 | 7,9 | 7,2 | 6,5 | 5,0 | 2,9 | 2,3 | 0,7 |

Les **3 premières composantes sont retenues** par le critère du coude et cumulent **58,8 %** de la variance totale. La projection 3D colorée par `Energy delta[Wh]` permet de visualiser la distribution de la production dans l'espace factoriel.

> **Écart à signaler :** le texte d'interprétation du notebook annonce 29,68 % / 13,84 % / 10,49 % pour les trois premières composantes, alors que la sortie effectivement stockée dans le notebook donne **32,33 % / 15,07 % / 11,44 %**. Le commentaire date d'une exécution antérieure et n'a pas été mis à jour. Les chiffres du tableau ci-dessus sont ceux des sorties réelles.

---

## Installation et exécution

Python 3.9+ recommandé.

```bash
git clone https://github.com/imenmhiri/renewable-energy-weather-analysis.git
cd renewable-energy-weather-analysis

python -m venv .venv
# Windows
.venv\Scripts\activate
# Linux / macOS
source .venv/bin/activate

pip install pandas numpy matplotlib seaborn scikit-learn jupyter
```

**Consultation seule** — le notebook contient déjà toutes ses sorties, aucune donnée n'est requise :

```bash
jupyter notebook renewable_energy_weather_analysis.ipynb
```

**Réexécution** — placer `solar_weather.csv` à la racine du dépôt, puis exécuter les cellules dans l'ordre. L'ordre importe : plusieurs cellules modifient `df` en place (suppression de colonne, `set_index("Time")`), une exécution partielle ou désordonnée produit des erreurs.

---

## Limites et pistes d'amélioration

### Limites méthodologiques

**1. Les moyennes sont diluées par les heures nocturnes.** 51,3 % des lignes ont une production `≤ 0`. Toutes les moyennes calculées sur l'ensemble des heures (`groupby("humidity").mean()`, production mensuelle, etc.) mélangent jour et nuit. Comme la durée du jour varie avec la saison, **la part de zéros varie elle aussi** — une partie de la « saisonnalité » observée mesure donc la longueur du jour autant que le rendement réel. Filtrer sur `isSun == 1` donnerait des moyennes interprétables.

**2. La relation humidité → production est très probablement un artefact.** Physiquement, humidité et nébulosité réduisent le rayonnement incident. Une corrélation positive s'explique mieux par une **variable confondante** (saison, température, durée du jour) que par un effet causal. Le notebook signale la complexité de la relation mais ne la décompose pas.

**3. La variable cible est incluse dans l'ACP.** `df_quantitatif` contient `Energy delta[Wh]` aux côtés des 10 variables explicatives. L'ACP mélange donc prédicteurs et cible, ce qui rend les composantes difficiles à interpréter comme structure des *facteurs* météorologiques. Colorer ensuite la projection par cette même variable est partiellement circulaire.

**4. L'hypothèse du report solaire → éolien n'est pas testée.** `Energy delta[Wh]` est une mesure unique et agrégée : rien dans le jeu de données ne distingue production solaire et production éolienne. L'interprétation d'un transfert entre filières n'est donc pas vérifiable avec ces données.

**5. La section « 3.3 ACP » est rattachée à l'étude 2018 mais porte sur toutes les années.** L'ACP est calculée sur `df_quantitatif` (196 776 lignes, toutes années), pas sur `df_filtered_2018`. Le titre induit en erreur.

**6. Corrélation et information mutuelle ne sont pas commentées.** La matrice de corrélation et le graphique d'information mutuelle sont produits, mais aucune cellule n'en tire de conclusion explicite — alors que ce sont les deux outils les plus directement liés à la problématique.

### Défauts de code

**7. Bug d'étiquetage du camembert (cellule 36-37).** `effectifs` provient de `value_counts()`, qui **trie par effectif décroissant**, et non par ordre d'intervalle. La liste `etiquettes` est en revanche écrite dans l'ordre croissant des bornes. Les deux ordres divergent : l'intervalle `(3137.5, 3765]` (6 824 occurrences) précède `(2510, 3137.5]` (6 352 occurrences) dans les données, de sorte que **ces deux parts du graphique portent des étiquettes inversées**. Correctif : trier avec `.sort_index()` avant de tracer, ou construire les étiquettes depuis `effectifs['Modalité']`.

**8. `warnings.filterwarnings("ignore")` masque des avertissements réels.** En particulier le `SettingWithCopyWarning` de la cellule 65 : `df_filtered_2018` est issu d'un `.loc`, et y écrire une colonne est une opération ambiguë. Utiliser `.copy()` explicitement lors du filtrage.

**9. La vérification des doublons est incomplète.** `df.duplicated()` renvoie la série booléenne complète sans agrégation ; `df.duplicated().sum()` est nécessaire pour conclure.

**10. Deux graphiques sans `plt.show()`** (cellules 43 et 65) : l'objet `Axes` de matplotlib est affiché en sortie brute avant l'image.

**11. `etiquettes` est défini puis inutilisé en cellule 34**, avec 6 éléments pour 9 intervalles.

### Pistes d'amélioration

- **Restreindre les analyses de rendement aux heures de jour** (`isSun == 1`) et présenter séparément la disponibilité (durée du jour) et le rendement horaire.
- **Isoler les effets confondants** : régression multiple ou corrélations partielles pour séparer l'effet propre de l'humidité de celui de la saison et de la température.
- **Refaire l'ACP sur les seules variables explicatives**, puis projeter la cible en variable supplémentaire — la lecture des axes devient interprétable.
- **Exploiter la dimension temporelle** : décomposition saisonnière (`statsmodels.tsa.seasonal_decompose`), profils horaires moyens par mois, autocorrélation.
- **Passer à la modélisation** : la problématique est bien posée pour une régression (prédire `Energy delta[Wh]` à partir des variables météo) avec validation temporelle (`TimeSeriesSplit`) plutôt qu'un découpage aléatoire.
- **Industrialiser** : ajouter un `requirements.txt` avec versions épinglées, un script de téléchargement du CSV, et extraire les fonctions `createStatTable1` / `createStatTable2` dans un module réutilisable.

---

## Conclusion du notebook

L'étude a porté sur l'impact de divers facteurs — ensoleillement, durée du jour, température, humidité, précipitations, vent — sur la production d'énergie solaire. La production la plus élevée a été observée en **2018**, ce qui a justifié une analyse approfondie de cette année, complétée par une ACP visant à identifier les variables les plus représentatives des données.

---

## Licence

Aucune licence déclarée. Projet à visée académique.
