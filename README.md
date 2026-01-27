# PratiLib GraphQL - TD4

Ce projet constitue le rendu du **TD 4 : GraphQL avec Directus** pour le module de PHP Avancé (S5). Il explore l'utilisation des requêtes (Queries) et des modifications (Mutations) via l'API GraphQL d'un CMS Headless.

## 🚀 Objectifs du TP

- Configurer et interroger une API GraphQL avec **Directus**.
- Maîtriser l'extraction de données complexes (relations, filtrage, alias, fragments).
- Comprendre et implémenter les mutations pour la persistance des données.
- Gérer l'authentification via tokens statiques et JWT.

## 📁 Structure du Projet

```text
GraphQL/
├── queries/          # 10 Requêtes GraphQL (.graphql)
├── mutations/        # 8 Mutations GraphQL (.graphql)
├── bruno/           # Collection Bruno pour tests automatisés
├── Compte_Rendu_TD4.md  # Rapport complet avec résultats JSON
└── README.md         # Présentation du projet
```

## 🛠 Configuration et Utilisation

### Prérequis
Le serveur **Directus** doit être opérationnel sur le port `8082`.
- **Endpoint GraphQL** : `http://localhost:8082/graphql`

### Exécution des requêtes
Vous pouvez tester les requêtes via la collection **Bruno** incluse dans le dossier `bruno/`.

## 📄 Rapport

Le rapport final détaillé (Interprétations techniques, code et résultats JSON) est disponible ici : 
👉 **[Compte_Rendu_TD4.md](./Compte_Rendu_TD4.md)**

---
**Auteur** : Paul Andrieu (DWM-2)
**Dépôt Git** : [Paul5400/GraphQL](https://github.com/Paul5400/GraphQL)
