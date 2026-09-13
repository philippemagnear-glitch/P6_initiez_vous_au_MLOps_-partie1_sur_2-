# Synthèse du notebook 03 — Jointures et feature engineering

## 1. Objectif général

Le notebook 03 enrichit les données clients avec l'historique de leurs demandes de crédit précédentes.

Le principe essentiel est de **ne jamais joindre directement une table détaillée contenant plusieurs lignes par crédit**. Cette opération multiplierait les lignes de `previous_application` et fausserait les calculs.

Les données sont donc progressivement ramenées à la bonne granularité :

```text
Historique mensuel ou paiements : plusieurs lignes par ancien crédit
                              │
                              │ agrégation par SK_ID_PREV
                              ▼
                     une ligne par ancien crédit
                              │
                              │ jointure avec previous_application
                              ▼
          previous_application enrichie : une ligne par ancien crédit
                              │
                              │ agrégation par SK_ID_CURR
                              ▼
                         une ligne par client
                              │
                              │ jointure avec application_train/test
                              ▼
                    dataset final : une ligne par client
```

## 2. Rôle des clés

| Clé | Signification | Utilisation |
|---|---|---|
| `SK_ID_PREV` | identifiant d'une ancienne demande de crédit | relier les historiques détaillés à `previous_application` |
| `SK_ID_CURR` | identifiant du client dans la demande actuelle | regrouper les anciens crédits par client, puis les joindre à `application_train` et `application_test` |

Il existe donc deux changements de granularité :

1. historique détaillé → ancien crédit, avec `SK_ID_PREV` ;
2. anciens crédits → client, avec `SK_ID_CURR`.

## 3. Enrichissement de chaque ancien crédit

### `POS_CASH_balance`

Cette table contient un état mensuel des crédits POS et cash : **10 001 358 lignes** au départ.

Elle est agrégée par `SK_ID_PREV` pour produire **936 325 lignes**, soit au maximum une ligne par ancien crédit.

Principales features créées :

| Feature | Agrégation | Interprétation |
|---|---|---|
| `POS_MONTH_COUNT` | nombre de lignes | profondeur de l'historique mensuel |
| `POS_EVER_DPD` | maximum d'un indicateur de retard | au moins un retard observé |
| `POS_EVER_SEVERE_DPD` | maximum de `SK_DPD_DEF >= 61` | au moins un retard sévère |
| `POS_REMAINING_INSTALLMENTS_LATEST` | valeur du mois le plus récent | échéances restant à payer récemment |

Le mois le plus récent est celui dont `MONTHS_BALANCE` est le plus élevé : par exemple, `-1` est plus récent que `-12`.

### `installments_payments`

Cette table contient **13 605 401 lignes de paiement**. Une même échéance pouvant être réglée en plusieurs fois, deux agrégations sont nécessaires.

Première étape : les paiements sont consolidés par échéance avec :

- `SK_ID_PREV` ;
- `NUM_INSTALMENT_NUMBER` ;
- `NUM_INSTALMENT_VERSION`.

On obtient **12 951 918 échéances consolidées**. Pour chacune, le notebook compare notamment la date et le montant réellement payés avec la date et le montant attendus.

Deuxième étape : les échéances sont agrégées par `SK_ID_PREV`, ce qui produit **997 752 anciens crédits**.

| Feature | Interprétation |
|---|---|
| `INSTAL_INSTALLMENT_COUNT` | nombre d'échéances connues |
| `INSTAL_LATE_PAYMENT_RATIO` | proportion d'échéances payées en retard |
| `INSTAL_MAX_DAYS_LATE` | retard maximal observé |
| `INSTAL_UNDERPAYMENT_RATIO` | proportion d'échéances insuffisamment payées |
| `INSTAL_PAYMENT_RATIO` | montant total payé / montant total attendu |

Cette double agrégation évite de considérer deux paiements partiels comme deux échéances différentes.

### `credit_card_balance`

Cette table contient **3 840 312 états mensuels de cartes de crédit**. Elle est agrégée par `SK_ID_PREV` pour produire **104 307 anciens crédits**.

| Feature | Agrégation | Interprétation |
|---|---|---|
| `CC_MONTH_COUNT` | nombre de lignes | profondeur de l'historique disponible |
| `CC_UTILIZATION_MEAN` | moyenne solde / limite | utilisation moyenne de la ligne de crédit |
| `CC_DRAWINGS_MONTHLY_MEAN` | moyenne | montant moyen retiré par mois |
| `CC_EVER_SEVERE_DPD` | maximum de `SK_DPD_DEF >= 61` | existence d'un retard sévère |
| `CC_BALANCE_LATEST` | valeur du mois le plus récent | dernier solde connu |

Les divisions sont réalisées uniquement lorsque la limite de crédit est positive afin d'éviter les divisions par zéro.

## 4. Jointure avec `previous_application`

Chaque table enfant agrégée possède désormais au maximum une ligne par `SK_ID_PREV`. Elle peut donc être jointe à `previous_application`, qui contient **1 670 214 anciens crédits**, sans multiplier ses lignes.

Les jointures utilisent :

- la clé `SK_ID_PREV` ;
- une jointure gauche (`left join`) pour conserver tous les anciens crédits ;
- `validate="one_to_one"` pour vérifier l'unicité des clés ;
- un indicateur `*_HAS_HISTORY` pour distinguer l'absence d'historique d'une valeur réellement égale à zéro.

Après chaque jointure, `previous_application` conserve donc ses **1 670 214 lignes**.

## 5. Passage de l'ancien crédit au client

Un client peut avoir plusieurs anciennes demandes. La table enrichie est donc ensuite agrégée par `SK_ID_CURR` afin d'obtenir une seule ligne par client.

Les colonnes natives de `previous_application` permettent notamment de créer :

- le nombre de demandes précédentes ;
- les proportions de demandes approuvées ou refusées ;
- la répartition des types de contrat ;
- le montant total des crédits approuvés ;
- des moyennes sur les annuités, les durées ou les rapports de montants ;
- l'indication que la demande précédente la plus récente a été refusée.

Les informations des tables enfants sont également résumées au niveau client. Par exemple :

- le maximum d'un indicateur binaire signifie **« cela s'est produit au moins une fois »** ;
- la moyenne d'un indicateur binaire devient une **proportion** ;
- la somme représente une **quantité totale** ;
- la moyenne d'un ratio décrit le **comportement moyen** sur les anciens crédits.

Exemples obtenus : `PREV_POS_EVER_SEVERE_DPD`, `PREV_INSTAL_LATE_PAYMENT_RATIO_MEAN`, `PREV_INSTAL_MAX_DAYS_LATE`, `PREV_CC_UTILIZATION_MEAN` et `PREV_CC_BALANCE_LATEST_SUM`.

## 6. Jointure finale avec les applications

La table précédente, désormais unique par `SK_ID_CURR`, est jointe séparément aux datasets déjà enrichis avec les informations du bureau :

- train : **307 510 lignes avant et après la jointure** ;
- test : **48 744 lignes avant et après la jointure**.

La jointure est faite à gauche sur `SK_ID_CURR` avec une validation `one_to_one`. Ainsi :

- aucun client n'est supprimé ;
- aucune ligne n'est dupliquée ;
- train et test restent séparés ;
- seule la table train conserve `TARGET` ;
- les variables explicatives restent alignées entre train et test.

## 7. Mémo sur les agrégations

| Opération | Sens métier habituel |
|---|---|
| `size` ou `count` | combien d'événements ou de crédits ? |
| `sum` | quel total cumulé ? |
| `mean` | quel comportement moyen ou quelle proportion ? |
| `max` sur un indicateur 0/1 | l'événement est-il déjà arrivé ? |
| valeur la plus récente | quelle est la situation actuelle ou la dernière connue ? |

## 8. Formulation courte pour la soutenance

> Les tables enfants contenaient plusieurs lignes mensuelles ou plusieurs paiements pour un même ancien crédit. Je ne pouvais donc pas les joindre directement, car cela aurait multiplié les lignes. J'ai d'abord transformé ces événements en indicateurs métier, puis je les ai agrégés par `SK_ID_PREV` afin d'obtenir une ligne par ancien crédit. J'ai joint ces résultats à `previous_application`, puis j'ai effectué une seconde agrégation par `SK_ID_CURR` pour obtenir une ligne par client. Cette table a enfin été jointe à `application_train` et `application_test`. Les jointures à gauche, les contrôles d'unicité et la vérification des nombres de lignes garantissent qu'aucun client n'est perdu ou dupliqué.
