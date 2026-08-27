# Instructions Codex — Projet OpenClassrooms TP6

## 1. Périmètre

Travaille uniquement dans le dossier courant de ce projet.

- Ne lis, ne modifies et ne crées aucun fichier en dehors de ce dossier.
- Ne touche jamais aux autres projets ou dossiers de l'ordinateur.
- N'explore pas inutilement tout le projet : inspecte uniquement les fichiers nécessaires à la tâche.
- Ne modifie jamais les données brutes présentes dans `data/raw/`.

## 2. Mode de travail

Avant toute modification :

1. Analyse la demande.
2. Inspecte uniquement les fichiers nécessaires.
3. Présente un plan de maximum 5 points.
4. Liste les fichiers que tu proposes de créer ou modifier.
5. Attends explicitement mon accord.

Ne commence jamais une modification immédiatement.

Fais ensuite uniquement ce qui a été validé.

## 3. Niveau attendu

Ce projet est réalisé dans le cadre d'une formation OpenClassrooms RNCP7 IA Engineer.

Privilégie toujours la solution la plus simple, lisible, robuste et pédagogique qui répond réellement au besoin.

Je dois comprendre le code et être capable de justifier les choix techniques.

Évite :
- le sur-engineering ;
- les architectures inutilement complexes ;
- les abstractions prématurées ;
- les fichiers supplémentaires non nécessaires ;
- les dépendances non indispensables ;
- les optimisations prématurées.

Ne simplifie cependant jamais au détriment d'une bonne pratique fondamentale :
- validation des données ;
- gestion des erreurs ;
- sécurité ;
- gestion correcte des ressources ;
- reproductibilité ;
- lisibilité.

Lorsqu'un choix technique est nécessaire, distingue si utile :
1. l'exigence du cahier des charges ;
2. la bonne pratique fondamentale à conserver ;
3. l'amélioration professionnelle optionnelle qui peut être laissée de côté.

## 4. Code et explications

Commence toujours par l'implémentation la plus simple qui répond réellement au besoin.

N'introduis pas de notion avancée si elle n'est pas nécessaire.

Explique brièvement :
- le rôle du code ajouté ;
- les mécanismes importants ;
- pourquoi ils sont nécessaires ;
- ce qui pourrait poser problème si on les supprimait.

Ne donne pas de longue explication si elle n'est pas demandée.

Dans le code, privilégie des sections lisibles comme :

`# ---------- Chargement des données ----------`
`# ---------- Validation des données ----------`
`# ---------- Entraînement du modèle ----------`

N'ajoute pas de commentaire sur chaque ligne lorsque le code est déjà explicite.

## 5. Modifications de fichiers

Fais de petites modifications ciblées.

- Ne refactorise pas du code sans rapport avec la demande.
- Ne crée pas de fichier non nécessaire.
- Ne supprime jamais un fichier sans autorisation.
- Ne déplace pas ou ne renomme pas de fichier sans autorisation.
- Ne modifie pas plusieurs parties du projet si une modification locale suffit.

Si tu me demandes de modifier manuellement du code, ne donne pas quelques lignes isolées susceptibles de créer des incohérences.

Indique clairement le bloc complet à remplacer et son remplacement complet.

## 6. Python et dépendances

Avant toute installation, vérifie comment le projet gère actuellement ses dépendances.

### Si le projet contient `pyproject.toml`

Utilise :
- `uv add` pour une dépendance ;
- `uv add --dev` pour une dépendance de développement.

### Si le projet contient uniquement `requirements.txt`

- Ne lance jamais `uv init`.
- Ne crée pas de `pyproject.toml`.
- Utilise `uv pip install` si une installation est nécessaire.
- Ajoute ensuite explicitement la dépendance dans `requirements.txt`.

N'utilise jamais `pip install`.

Pour exécuter une commande Python dans l'environnement du projet, utilise `uv run` lorsque cela est compatible avec la configuration existante.

Ne crée, supprime ou recrée jamais :
- `.venv` ;
- un environnement virtuel ;
- un fichier de gestion des dépendances ;

sans mon autorisation explicite.

N'installe aucune nouvelle bibliothèque sans expliquer brièvement pourquoi elle est nécessaire et attendre mon accord.

## 7. Commandes sensibles

Demande mon autorisation avant toute commande pouvant :

- supprimer, déplacer ou écraser des fichiers ;
- installer ou supprimer une dépendance ;
- créer ou modifier un environnement Python ;
- modifier la configuration du projet ;
- modifier Git ;
- lancer une opération ayant des effets importants sur plusieurs fichiers.

## 8. Git et traçabilité

Git sert de point de retour arrière.

Avant une modification importante, vérifie :

`git status`

Après une modification, indique brièvement :

- fichiers créés ;
- fichiers modifiés ;
- commandes exécutées ;
- tests exécutés et résultat.

Lorsque c'est utile, utilise `git diff` pour me permettre de vérifier les changements.

Ne crée jamais automatiquement :
- de commit ;
- de branche ;
- de tag ;
- de push.

Propose seulement un message de commit au format Conventional Commits décrivant réellement la modification.

Le commit sera effectué uniquement après ma validation.

## 9. Réponses et consommation de tokens

Sois concis et évite les informations non nécessaires.

Pour une tâche simple :
- plan : maximum 5 points ;
- une seule solution recommandée par défaut ;
- pas de variantes sauf si elles sont réellement utiles ;
- pas de répétition d'informations déjà données ;
- pas de résumé long après une petite modification ;
- n'affiche pas le contenu complet de fichiers non concernés ;
- ne parcours pas tout le dépôt si quelques fichiers suffisent.

Si une information importante manque et ne peut pas être obtenue en inspectant le projet, pose une question plutôt que de faire une supposition importante.