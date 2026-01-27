# Collection Bruno - TD4 GraphQL

Cette collection Bruno contient l'ensemble des requêtes et mutations GraphQL du TD4.

## Structure

```
bruno/
├── queries/          # 10 requêtes GraphQL (Q1-Q10)
├── mutations/        # 8 mutations GraphQL (M1-M8)
├── environments/     # Configuration de l'environnement local
└── bruno.json        # Métadonnées de la collection
```

## Configuration

### Prérequis
- Bruno CLI ou Bruno Desktop installé
- Directus en cours d'exécution sur `http://localhost:8082`

### Variables d'environnement

Créez un fichier `.env` à la racine du projet GraphQL :

```bash
DIRECTUS_TOKEN=votre_token_admin
```

Pour obtenir un token :
```bash
curl -X POST "http://localhost:8082/auth/login" \
  -H "Content-Type: application/json" \
  -d '{"email": "admin@example.com", "password": "password"}' \
  | jq -r .data.access_token
```

## Utilisation

### Avec Bruno Desktop
1. Ouvrir Bruno
2. Importer la collection depuis le dossier `bruno/`
3. Sélectionner l'environnement "Local"
4. Exécuter les requêtes

### Avec Bruno CLI
```bash
# Installer Bruno CLI
npm install -g @usebruno/cli

# Exécuter une requête spécifique
bru run bruno/queries/Q1.bru --env Local

# Exécuter toutes les queries
bru run bruno/queries --env Local

# Exécuter toutes les mutations
bru run bruno/mutations --env Local
```

## Contenu

### Queries (10)
- **Q1** : Liste des praticiens (champs de base)
- **Q2** : Praticiens avec label de spécialité
- **Q3** : Filtrage par ville (Paris)
- **Q4** : Praticiens avec structure et spécialité
- **Q5** : Recherche par motif d'email (.fr)
- **Q6** : Filtrage sur relation (structure à Paris)
- **Q7** : Utilisation des alias
- **Q8** : Utilisation des fragments
- **Q9** : Requête paramétrée avec variables
- **Q10** : Relation inverse (structures avec praticiens)

### Mutations (8)
- **M1** : Création d'une spécialité
- **M2** : Création d'un praticien
- **M3** : Mise à jour d'un praticien
- **M4** : Création avec liaison directe
- **M5** : Création imbriquée (upsert)
- **M6** : Ajout à une spécialité existante
- **M7** : Liaison à une structure
- **M8** : Suppression groupée

## Notes

- Les mutations utilisent des identifiants dynamiques (UUID/timestamp) pour éviter les conflits
- Toutes les requêtes nécessitent une authentification Bearer
- Le port par défaut est 8082 (configurable dans `environments/Local.bru`)
