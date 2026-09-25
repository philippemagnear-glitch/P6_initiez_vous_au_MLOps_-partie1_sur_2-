# Credit Scoring — Pipeline MLOps

Projet de scoring crédit réalisé dans le cadre d'un TP OpenClassrooms. L'objectif est de prédire le risque de défaut d'un client à partir des données **Home Credit Default Risk**, tout en tenant compte du coût métier des erreurs de prédiction.

Le projet couvre la préparation des données, les jointures entre plusieurs historiques de crédit, la comparaison de modèles avec MLflow, l'optimisation de LightGBM et l'enregistrement du pipeline final dans le Model Registry.

## Objectif métier

La classe positive (`TARGET = 1`) représente un client en défaut de paiement.

Les deux erreurs n'ont pas le même impact :

- **FN** : mauvais client prédit bon, donc crédit accordé à tort ;
- **FP** : bon client prédit mauvais, donc crédit refusé à tort.

Un faux négatif étant considéré comme dix fois plus coûteux, le critère principal est :

```text
coût métier = 10 × FN + FP
```

La ROC AUC, le recall de la classe 1 et le F1-score complètent l'interprétation, mais ne remplacent pas l'objectif métier lors de l'optimisation finale.

## Pipeline du projet

```text
Données brutes
    ↓
Exploration, nettoyage et encodage
    ↓
Agrégation de bureau_balance puis jointure avec bureau
    ↓
Agrégation des historiques POS, paiements et cartes de crédit
    ↓
Enrichissement de previous_application puis agrégation par client
    ↓
Contrôles avant modélisation
    ↓
Comparaison de quatre familles de modèles avec MLflow
    ↓
Sélection du dataset, optimisation de LightGBM et du seuil métier
    ↓
Évaluation finale, feature importance et Model Registry
```

Les tables historiques ne sont jamais jointes directement lorsqu'elles contiennent plusieurs lignes par crédit. Elles sont d'abord agrégées par `SK_ID_PREV`, puis les anciennes demandes sont agrégées par `SK_ID_CURR`. Cette méthode évite la multiplication des lignes et produit une seule ligne de modélisation par client.

## Notebooks

Les notebooks doivent être exécutés dans l'ordre.

| Notebook | Rôle |
|---|---|
| `01_exploration_nettoyage_application_train_test.ipynb` | exploration, traitement des anomalies, encodage et préparation de `application_train` et `application_test` |
| `02_jointure_bureau_balance_bureau.ipynb` | agrégation de l'historique mensuel du bureau, enrichissement de `bureau`, puis jointure avec les applications |
| `03_enrichissement_previous_application.ipynb` | agrégation de `POS_CASH_balance`, `installments_payments` et `credit_card_balance`, puis enrichissement des clients |
| `04_controles_avant_modelisation.ipynb` | contrôles structurels, valeurs manquantes, constantes, déséquilibre de la cible et risque de fuite de données |
| `05_modelisation_mlflow.ipynb` | comparaison de Logistic Regression, Random Forest, XGBoost et LightGBM, avec et sans pondération des classes |
| `06_optimisation_modele.ipynb` | sélection de la version du dataset, optimisation métier de LightGBM, évaluation finale et enregistrement du modèle |

Une synthèse détaillée des jointures du notebook 03 est disponible dans [`docs/03_synthese_jointures_feature_engineering.md`](docs/03_synthese_jointures_feature_engineering.md).

## Modélisation et résultats

La comparaison initiale montre que les modèles non pondérés détectent mal la classe minoritaire. LightGBM pondéré est retenu car il offre un compromis cohérent entre recall, faux négatifs, faux positifs, F1-score, ROC AUC et stabilité en validation croisée.

Dans le notebook 06 :

- le split est stratifié : 80 % pour l'entraînement et la validation croisée, 20 % conservés pour l'évaluation finale ;
- cinq versions du dataset sont comparées avec un LightGBM de référence identique ;
- la version sans `BUREAU_EVER_SEVERE_DPD` et `BUREAU_EVER_RECENT_DPD_12M` est retenue avec 310 variables ;
- 16 configurations LightGBM et 17 seuils fixes sont évalués sur 5 folds stratifiés ;
- seul le coût métier moyen sert à choisir le couple hyperparamètres-seuil.

Configuration retenue :

| Paramètre | Valeur |
|---|---:|
| `n_estimators` | 300 |
| `learning_rate` | 0.05 |
| `num_leaves` | 31 |
| `min_child_samples` | 20 |
| seuil de décision | 0.50 |

Résultats sur les 61 502 clients du jeu final jamais utilisé pour la sélection :

| Métrique | Résultat |
|---|---:|
| coût métier | 30 807 |
| faux négatifs | 1 568 |
| faux positifs | 15 127 |
| recall classe 1 | 0.684 |
| F1-score | 0.289 |
| ROC AUC | 0.778 |

Le coût normalisé passe d'environ 421,6 à 500,9 pour 1 000 clients entre l'entraînement et le jeu final. Avec la baisse du recall et de la ROC AUC, cet écart indique un surapprentissage modéré.

L'importance globale par gain met principalement en avant `EXT_SOURCE_2`, `EXT_SOURCE_3`, `EXT_SOURCE_1` et plusieurs variables issues des historiques de paiement. Elle décrit l'utilisation des variables par LightGBM, sans établir de relation causale.

## MLflow

MLflow utilise une base SQLite locale pour suivre :

- les paramètres des modèles ;
- les métriques de validation et d'évaluation ;
- les tags décrivant les étapes expérimentales ;
- le graphique d'importance des variables ;
- le pipeline final avec son preprocessing.

Le modèle final est enregistré dans le Model Registry sous le nom :

```text
credit_scoring_lightgbm
```

Pour ouvrir l'interface depuis la racine du projet :

```bash
mlflow ui --backend-store-uri sqlite:///mlflow.db
```

L'interface est ensuite accessible sur <http://127.0.0.1:5000>.

La base `mlflow.db` et les artefacts locaux sont volontairement ignorés par Git. Ils sont recréés lors de l'exécution des notebooks de modélisation.

## Structure du dépôt

```text
.
├── data/
│   ├── raw/                 # données sources non versionnées
│   └── processed/           # datasets intermédiaires non versionnés
├── docs/                    # documentation des tables et jointures
├── notebooks/               # pipeline exécutable de 01 à 06
├── requirements.txt         # dépendances Python
├── .gitignore
└── README.md
```

## Installation

Pré-requis : Python et un environnement virtuel.

```bash
python -m venv .venv
```

Activation sous Windows :

```powershell
.venv\Scripts\Activate.ps1
```

Installation des dépendances :

```bash
python -m pip install -r requirements.txt
```

Principales bibliothèques : pandas, scikit-learn, LightGBM, XGBoost, MLflow, Matplotlib et Jupyter.

## Données

Les données proviennent de la compétition Kaggle [Home Credit Default Risk](https://www.kaggle.com/competitions/home-credit-default-risk/data). Elles ne sont pas incluses dans ce dépôt.

Placer les fichiers suivants dans `data/raw/` :

```text
application_train.csv
application_test.csv
bureau.csv
bureau_balance.csv
previous_application.csv
POS_CASH_balance.csv
installments_payments.csv
credit_card_balance.csv
```

## Exécution

Les chemins des notebooks sont définis relativement au dossier `notebooks`. Pour conserver cette organisation :

```bash
cd notebooks
jupyter notebook
```

Exécuter ensuite les notebooks de `01` à `06`. Les trois premiers génèrent progressivement les fichiers nécessaires dans `data/processed/`.

Le notebook 06 utilise une séparation finale jamais consultée pendant la sélection du dataset, des hyperparamètres ou du seuil. Cette séparation doit être conservée pour éviter toute fuite d'information vers l'évaluation finale.

## Limites

- Le coût `10 × FN + FP` traduit une hypothèse métier pédagogique et devrait être calibré avec des coûts financiers réels avant une mise en production.
- Le modèle présente un surapprentissage modéré entre l'entraînement et les données finales.
- L'importance LightGBM est globale et ne fournit ni explication locale complète ni interprétation causale.
- Le projet couvre l'expérimentation et le suivi du modèle, mais pas encore son déploiement en production ni son monitoring après déploiement.
