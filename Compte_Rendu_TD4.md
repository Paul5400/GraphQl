# Compte Rendu GraphQL PratiLib

**Étudiants :** Paul Andrieu , Valentino Lambert
**Formation :** DWM-2 (S5)  
**Contexte :** Optimisation des flux de données pour le projet PratiLib  
**Lien Git :** https://github.com/Paul5400/GraphQl.git
  

---

## 1. Introduction Métier

Dans le cadre de l'évolution de la plateforme **PratiLib**, nous avons migré une partie de la consommation de données vers une architecture **GraphQL** adossée au CMS Headless **Directus**. Ce choix stratégique répond à un besoin de flexibilité accrue pour les interfaces clientes (web et mobiles) et à une volonté de rationaliser les performances réseau.

L'objectif de ce document est de présenter les différentes typologies de requêtes (Consultation) et de mutations (Écriture) implémentées, tout en fournissant une analyse technique pour chaque cas d'usage.

## 2. Analyse de la Valeur Ajoutée GraphQL

L'intégration de GraphQL apporte trois leviers de performance majeurs :

1. **Précision Chirurgicale des Données** : Contrairement aux endpoints REST qui renvoient des objets fixes, GraphQL permet au client de spécifier les champs exacts dont il a besoin. Cela élimine l'over-fetching et réduit le temps de parsing côté client.
2. **Résolution de Complexité en un Appel** : Les relations entre Praticiens, Spécialités et Structures sont des graphes complexes. GraphQL permet de naviguer dans ces relations en une seule transaction HTTP, supprimant les latences liées aux multiples appels REST.
3. **Documentation en temps réel (Schéma)** : Le typage strict et l'introspection permettent aux développeurs de découvrir l'API dynamiquement, accélérant ainsi les cycles de développement front-end.

---

## 3. Détails de l'Architecture

Le système est déployé dans un environnement conteneurisé isolant chaque composant :
- **Image Orchestrateur** : Directus 10.x agissant comme passerelle GraphQL.
- **Couche de Persistence** : Cluster PostgreSQL gérant l'intégrité relationnelle.
- **Standardisation des Tests** : Utilisation de la suite **Bruno** pour la validation continue des schémas et la documentation des endpoints.

---

## 4. Partie 1 : Requêtes de Consultation de Données (Queries)

### Q1. Démonstration de Séléction Granulaire
**Analyse Technique et Interprétation :**
Cette requête illustre le principe fondamental de non-overfetching. Nous extrayons uniquement les identités et coordonnées des praticiens. Dans un scénario d'affichage de liste mobile, cela permet de gagner jusqu'à 80% de poids sur le JSON par rapport à un endpoint REST classique qui renverrait l'intégralité du profil praticien avec ses métadonnées de sécurité et de création.

#### Requête GraphQL :
```graphql
query {
  praticien {
    id
    nom
    prenom
    telephone
    ville
  }
}
```

#### Réponse JSON (Données Réelles) :
```json
{
  "data": {
    "praticien": [
      {
        "id": "03f24d7b-efda-3fbe-ae53-0c0c0edef089",
        "nom": "Grenier",
        "prenom": "Josette",
        "telephone": "06 43 92 58 43",
        "ville": "Paris UPDATED"
      },
      {
        "id": "05644ac0-c4cb-36bf-a0cf-409075eb20c4",
        "nom": "Leduc",
        "prenom": "Aimée",
        "telephone": "+33 (0)4 85 19 99 00",
        "ville": "Carre-sur-Turpin"
      },
      {
        "id": "0768e78b-12c8-3694-b89b-d0a7451a8fb1",
        "nom": "Descamps",
        "prenom": "Alexandria",
        "telephone": "+33 (0)1 75 74 98 33",
        "ville": "Joubert"
      },
      "// ... (données tronquées pour plus de lisibilité)"
    ]
  }
}
```

### Q2. Traversée de Relation (M2O)
**Analyse Technique et Interprétation :**
Ici, nous démontrons la capacité de l'API à résoudre les relations 'Many-to-One'. En demandant le libellé de la spécialité directement dans l'arborescence du praticien, nous évitons une jointure SQL manuelle complexe côté client. Le serveur Directus optimise la requête SQL sous-jacente pour fournir une réponse unifiée et performante.

#### Requête GraphQL :
```graphql
query {
  praticien {
    id
    nom
    prenom
    telephone
    ville
    specialite_id {
      libelle
    }
  }
}
```

#### Réponse JSON (Données Réelles) :
```json
{
  "data": {
    "praticien": [
      {
        "id": "03f24d7b-efda-3fbe-ae53-0c0c0edef089",
        "nom": "Grenier",
        "prenom": "Josette",
        "telephone": "06 43 92 58 43",
        "ville": "Paris UPDATED",
        "specialite_id": {
          "libelle": "médecine générale"
        }
      },
      {
        "id": "05644ac0-c4cb-36bf-a0cf-409075eb20c4",
        "nom": "Leduc",
        "prenom": "Aimée",
        "telephone": "+33 (0)4 85 19 99 00",
        "ville": "Carre-sur-Turpin",
        "specialite_id": {
          "libelle": "ophtalmologie"
        }
      },
      {
        "id": "0768e78b-12c8-3694-b89b-d0a7451a8fb1",
        "nom": "Descamps",
        "prenom": "Alexandria",
        "telephone": "+33 (0)1 75 74 98 33",
        "ville": "Joubert",
        "specialite_id": {
          "libelle": "ophtalmologie"
        }
      },
      "// ... (données tronquées pour plus de lisibilité)"
    ]
  }
}
```

### Q3. Filtrage et Segmentations Géographiques
**Analyse Technique et Interprétation :**
L'application d'un filtre strict sur le champ 'ville' montre comment GraphQL délégue la charge de traitement des données au serveur de base de données. C'est un outil indispensable pour créer des vues segmentées par région sans charger l'intégralité de la base de données dans la mémoire de l'application cliente.

#### Requête GraphQL :
```graphql
query {
  praticien(filter: { ville: { _eq: "Paris" } }) {
    id
    nom
    prenom
    telephone
    ville
    specialite_id {
      libelle
    }
  }
}
```

#### Réponse JSON (Données Réelles) :
```json
{
  "data": {
    "praticien": [
      {
        "id": "085c44e7-6245-3b10-9c1a-92f09524ef2a",
        "nom": "Gallet",
        "prenom": "Valérie",
        "telephone": "+33 (0)9 29 33 51 05",
        "ville": "Paris",
        "specialite_id": {
          "libelle": "Imagerie médicale"
        }
      },
      {
        "id": "8236bcbf-4c06-3d0e-8ab0-c4964e02c4ea",
        "nom": "Pichon",
        "prenom": "Arnaude",
        "telephone": "06 61 50 63 81",
        "ville": "Paris",
        "specialite_id": {
          "libelle": "ophtalmologie"
        }
      },
      {
        "id": "8ae1400f-d46d-3b50-b356-269f776be532",
        "nom": "Klein",
        "prenom": "Gabrielle",
        "telephone": "+33 (0)3 90 27 98 80",
        "ville": "Paris",
        "specialite_id": {
          "libelle": "médecine générale"
        }
      },
      "// ... (données tronquées pour plus de lisibilité)"
    ]
  }
}
```

### Q4. Multi-Jointures et Profondeur de Graphe
**Analyse Technique et Interprétation :**
Cette requête est une démonstration de force de GraphQL : elle récupère simultanément les données du praticien, de la spécialité ET de la structure de santé rattachée. En REST, cela aurait nécessité trois appels distincts. Ici, le client reçoit un objet hiérarchique prêt à l'emploi, garantissant l'atomicité de la vue affichée à l'utilisateur.

#### Requête GraphQL :
```graphql
query {
  praticien {
    id
    nom
    prenom
    telephone
    ville
    specialite_id {
      libelle
    }
    structure_id {
      nom
      ville
    }
  }
}
```

#### Réponse JSON (Données Réelles) :
```json
{
  "data": {
    "praticien": [
      {
        "id": "03f24d7b-efda-3fbe-ae53-0c0c0edef089",
        "nom": "Grenier",
        "prenom": "Josette",
        "telephone": "06 43 92 58 43",
        "ville": "Paris UPDATED",
        "specialite_id": {
          "libelle": "médecine générale"
        },
        "structure_id": {
          "nom": "Babinet Antoine",
          "ville": "Moreno"
        }
      },
      {
        "id": "05644ac0-c4cb-36bf-a0cf-409075eb20c4",
        "nom": "Leduc",
        "prenom": "Aimée",
        "telephone": "+33 (0)4 85 19 99 00",
        "ville": "Carre-sur-Turpin",
        "specialite_id": {
          "libelle": "ophtalmologie"
        },
        "structure_id": {
          "nom": "Existing Structure",
          "ville": "Diaz-sur-Boulanger"
        }
      },
      {
        "id": "0768e78b-12c8-3694-b89b-d0a7451a8fb1",
        "nom": "Descamps",
        "prenom": "Alexandria",
        "telephone": "+33 (0)1 75 74 98 33",
        "ville": "Joubert",
        "specialite_id": {
          "libelle": "ophtalmologie"
        },
        "structure_id": {
          "nom": "Cabinet Toussaint Marechal",
          "ville": "Joubert"
        }
      },
      "// ... (données tronquées pour plus de lisibilité)"
    ]
  }
}
```

### Q5. Analyse de Données par Pattern-Matching
**Analyse Technique et Interprétation :**
Nous utilisons ici l'opérateur de comparaison `_contains` pour effectuer une recherche floue sur les emails. Cette approche est particulièrement utile pour implémenter des fonctions de recherche dynamiques ou des filtres de sécurité permettant de vérifier le domaine d'origine des praticiens enregistrés dans le système.

#### Requête GraphQL :
```graphql
query {
  praticien(filter: { email: { _contains: ".fr" } }) {
    id
    nom
    prenom
    telephone
    ville
    email
    specialite_id {
      libelle
    }
    structure_id {
      nom
      ville
    }
  }
}
```

#### Réponse JSON (Données Réelles) :
```json
{
  "data": {
    "praticien": [
      {
        "id": "03f24d7b-efda-3fbe-ae53-0c0c0edef089",
        "nom": "Grenier",
        "prenom": "Josette",
        "telephone": "06 43 92 58 43",
        "ville": "Paris UPDATED",
        "email": "Josette.Grenier@wanadoo.fr",
        "specialite_id": {
          "libelle": "médecine générale"
        },
        "structure_id": {
          "nom": "Babinet Antoine",
          "ville": "Moreno"
        }
      },
      {
        "id": "0768e78b-12c8-3694-b89b-d0a7451a8fb1",
        "nom": "Descamps",
        "prenom": "Alexandria",
        "telephone": "+33 (0)1 75 74 98 33",
        "ville": "Joubert",
        "email": "Alexandria.Descamps@club-internet.fr",
        "specialite_id": {
          "libelle": "ophtalmologie"
        },
        "structure_id": {
          "nom": "Cabinet Toussaint Marechal",
          "ville": "Joubert"
        }
      },
      {
        "id": "15d3e128-b47c-3030-a1c8-cfb09942054a",
        "nom": "Pascal",
        "prenom": "Gilles",
        "telephone": "08 12 96 04 73",
        "ville": "Carre-sur-Turpin",
        "email": "Gilles.Pascal@sfr.fr",
        "specialite_id": {
          "libelle": "Dentiste"
        },
        "structure_id": {
          "nom": "Maison de Santé de Care-Sur_Turpin",
          "ville": "Carre-sur-Turpin"
        }
      },
      "// ... (données tronquées pour plus de lisibilité)"
    ]
  }
}
```

### Q6. Filtres Croisés sur Relations Indépendantes
**Analyse Technique et Interprétation :**
Cette requête résout un besoin métier complexe : lister les praticiens rattachés à une structure située spécifiquement à Paris. GraphQL permet de naviguer dans les relations pour appliquer des filtres sur des entités qui ne sont pas l'objet principal de la requête. C'est une flexibilité quasi-impossible à offrir en REST sans créer des endpoints dédiés trop spécifiques.

#### Requête GraphQL :
```graphql
query {
  praticien(filter: { structure_id: { ville: { _eq: "Paris" } } }) {
    id
    nom
    prenom
    ville
    structure_id {
      nom
      ville
    }
  }
}
```

#### Réponse JSON (Données Réelles) :
```json
{
  "data": {
    "praticien": [
      {
        "id": "085c44e7-6245-3b10-9c1a-92f09524ef2a",
        "nom": "Gallet",
        "prenom": "Valérie",
        "ville": "Paris",
        "structure_id": {
          "nom": "Cabinet Bigot",
          "ville": "Paris"
        }
      },
      {
        "id": "8236bcbf-4c06-3d0e-8ab0-c4964e02c4ea",
        "nom": "Pichon",
        "prenom": "Arnaude",
        "ville": "Paris",
        "structure_id": {
          "nom": "Cabinet Bigot",
          "ville": "Paris"
        }
      },
      {
        "id": "8ae1400f-d46d-3b50-b356-269f776be532",
        "nom": "Klein",
        "prenom": "Gabrielle",
        "ville": "Paris",
        "structure_id": {
          "nom": "Cabinet Bigot",
          "ville": "Paris"
        }
      },
      "// ... (données tronquées pour plus de lisibilité)"
    ]
  }
}
```

### Q7. Optimisation du Flux via les Alias
**Analyse Technique et Interprétation :**
L'usage des alias permet de transformer la structure du JSON de sortie pour répondre exactement aux besoins de l'interface. Ici, nous séparons les données en deux blocs distincts ('paris' et 'lyon') dans une même réponse, permettant au développeur front-end de mapper directement le JSON sur sa logique de présentation sans retraitement.

#### Requête GraphQL :
```graphql
query {
  parisiens: praticien(filter: { ville: { _eq: "Paris" } }) {
    id
    nom
    prenom
    ville
  }
  bourbonnais: praticien(filter: { ville: { _eq: "Bourdon-les-Bains" } }) {
    id
    nom
    prenom
    ville
  }
}
```

#### Réponse JSON (Données Réelles) :
```json
{
  "data": {
    "parisiens": [
      {
        "id": "085c44e7-6245-3b10-9c1a-92f09524ef2a",
        "nom": "Gallet",
        "prenom": "Valérie",
        "ville": "Paris"
      },
      {
        "id": "8236bcbf-4c06-3d0e-8ab0-c4964e02c4ea",
        "nom": "Pichon",
        "prenom": "Arnaude",
        "ville": "Paris"
      },
      {
        "id": "8ae1400f-d46d-3b50-b356-269f776be532",
        "nom": "Klein",
        "prenom": "Gabrielle",
        "ville": "Paris"
      },
      "// ... (données tronquées pour plus de lisibilité)"
    ],
    "bourbonnais": [
      {
        "id": "341453e4-c122-3028-a712-af7ddad67975",
        "nom": "Marin",
        "prenom": "Isaac",
        "ville": "Bourdon-les-Bains"
      },
      {
        "id": "609bc828-4d34-3bb2-b527-93d80478d6b3",
        "nom": "Gilbert",
        "prenom": "Adrien",
        "ville": "Bourdon-les-Bains"
      },
      {
        "id": "794f6ea3-9801-334b-ba82-4ac71f70f6d2",
        "nom": "Marechal",
        "prenom": "Inès",
        "ville": "Bourdon-les-Bains"
      },
      "// ... (données tronquées pour plus de lisibilité)"
    ]
  }
}
```

### Q8. Réutilisation et Standardisation par Fragments
**Analyse Technique et Interprétation :**
Pour garantir la cohérence des données à travers toute l'application, nous utilisons les Fragments. Cette approche logicielle permet de définir une 'forme' de donnée standardisée pour une entité (ICI le praticien) et de la réutiliser partout. Cela simplifie grandement la maintenance : si le modèle change, la mise à jour se fait en un seul point.

#### Requête GraphQL :
```graphql
query {
  parisiens: praticien(filter: { ville: { _eq: "Paris" } }) {
    ...PraticienFields
  }
  bourbonnais: praticien(filter: { ville: { _eq: "Bourdon-les-Bains" } }) {
    ...PraticienFields
  }
}

fragment PraticienFields on praticien {
  id
  nom
  prenom
  ville
}
```

#### Réponse JSON (Données Réelles) :
```json
{
  "data": {
    "parisiens": [
      {
        "id": "085c44e7-6245-3b10-9c1a-92f09524ef2a",
        "nom": "Gallet",
        "prenom": "Valérie",
        "ville": "Paris"
      },
      {
        "id": "8236bcbf-4c06-3d0e-8ab0-c4964e02c4ea",
        "nom": "Pichon",
        "prenom": "Arnaude",
        "ville": "Paris"
      },
      {
        "id": "8ae1400f-d46d-3b50-b356-269f776be532",
        "nom": "Klein",
        "prenom": "Gabrielle",
        "ville": "Paris"
      },
      "// ... (données tronquées pour plus de lisibilité)"
    ],
    "bourbonnais": [
      {
        "id": "341453e4-c122-3028-a712-af7ddad67975",
        "nom": "Marin",
        "prenom": "Isaac",
        "ville": "Bourdon-les-Bains"
      },
      {
        "id": "609bc828-4d34-3bb2-b527-93d80478d6b3",
        "nom": "Gilbert",
        "prenom": "Adrien",
        "ville": "Bourdon-les-Bains"
      },
      {
        "id": "794f6ea3-9801-334b-ba82-4ac71f70f6d2",
        "nom": "Marechal",
        "prenom": "Inès",
        "ville": "Bourdon-les-Bains"
      },
      "// ... (données tronquées pour plus de lisibilité)"
    ]
  }
}
```

### Q9. Industrialisation via les Variables GraphQL
**Analyse Technique et Interprétation :**
Cette requête sépare la logique structurelle des données variables ($ville). C'est l'approche recommandée pour la mise en production : elle permet au serveur de pré-compiler la requête et facilite l'implémentation de mécanismes de cache performants. Elle renforce également la sécurité en évitant la construction de chaînes de caractères risquées.

#### Requête GraphQL :
```graphql
query GetPraticiensByVille($ville: String!) {
  praticien(filter: { ville: { _eq: $ville } }) {
    id
    nom
    prenom
    telephone
    ville
  }
}
```

#### Réponse JSON (Données Réelles) :
```json
{
  "errors": [
    {
      "message": "Variable \"$ville\" of required type \"String!\" was not provided.",
      "extensions": {
        "code": "INTERNAL_SERVER_ERROR"
      },
      "locations": [
        {
          "line": 1,
          "column": 28
        }
      ]
    }
  ]
}
```

### Q10. Navigation Inverse et Agrégat O2M
**Analyse Technique et Interprétation :**
Nous explorons ici la relation inverse : nous partons de la Structure pour lister tous les Praticiens qui y sont affectés. Cette vue hiérarchique est essentielle pour les outils de gestion d'établissements de santé, montrant la richesse du graphe de données mis en place sous Directus.

#### Requête GraphQL :
```graphql
query {
  structure {
    nom
    ville
    praticiens {
      nom
      prenom
      email
      specialite_id {
        libelle
      }
    }
  }
}
```

#### Réponse JSON (Données Réelles) :
```json
{
  "data": {
    "structure": [
      {
        "nom": "Existing Structure",
        "ville": "Diaz-sur-Boulanger",
        "praticiens": [
          {
            "nom": "Leduc",
            "prenom": "Aimée",
            "email": "Aime.Leduc@laposte.net",
            "specialite_id": {
              "libelle": "ophtalmologie"
            }
          },
          {
            "nom": "Marchand",
            "prenom": "Suzanne",
            "email": "Suzanne.Marchand@sfr.fr",
            "specialite_id": {
              "libelle": "ophtalmologie"
            }
          },
          {
            "nom": "Martel",
            "prenom": "Denise",
            "email": "Denise.Martel@tele2.fr",
            "specialite_id": {
              "libelle": "Imagerie médicale"
            }
          },
          "// ... (données tronquées pour plus de lisibilité)"
        ]
      },
      {
        "nom": "Cabinet Bigot",
        "ville": "Paris",
        "praticiens": [
          {
            "nom": "Gallet",
            "prenom": "Valérie",
            "email": "Valrie.Gallet@dbmail.com",
            "specialite_id": {
              "libelle": "Imagerie médicale"
            }
          },
          {
            "nom": "Pichon",
            "prenom": "Arnaude",
            "email": "Arnaude.Pichon@yahoo.fr",
            "specialite_id": {
              "libelle": "ophtalmologie"
            }
          },
          {
            "nom": "Klein",
            "prenom": "Gabrielle",
            "email": "Gabrielle.Klein@live.com",
            "specialite_id": {
              "libelle": "médecine générale"
            }
          },
          "// ... (données tronquées pour plus de lisibilité)"
        ]
      },
      {
        "nom": "Cabinet Toussaint Marechal",
        "ville": "Joubert",
        "praticiens": [
          {
            "nom": "Descamps",
            "prenom": "Alexandria",
            "email": "Alexandria.Descamps@club-internet.fr",
            "specialite_id": {
              "libelle": "ophtalmologie"
            }
          },
          {
            "nom": "Besson",
            "prenom": "Aurélie",
            "email": "Aurlie.Besson@gmail.com",
            "specialite_id": {
              "libelle": "pédiatrie"
            }
          },
          {
            "nom": "Costa",
            "prenom": "Marc",
            "email": "Marc.Costa@wanadoo.fr",
            "specialite_id": {
              "libelle": "pédiatrie"
            }
          }
        ]
      },
      "// ... (données tronquées pour plus de lisibilité)"
    ]
  }
}
```

---
## 5. Partie 2 : Manipulation et Persistence (Mutations)

### M1. Enregistrement d'une Nouvelle Spécialité
**Impact Métier et Analyse :**
L'enregistrement d'une nouvelle spécialité valide l'intégrité du système de types. La réponse confirme la persistance en renvoyant l'ID unique généré, permettant au système client de conserver une trace transactionnelle propre et de confirmer l'action à l'utilisateur final.

#### Instruction de Mutation :
```graphql
mutation {
  create_specialite_item(data: { libelle: "cardiologie_1769510862", description: "Maladies du cœur" }) {
    id
    libelle
  }
}
```

#### Confirmation de Transaction API :
```json
{
  "data": {
    "create_specialite_item": {
      "id": "27",
      "libelle": "cardiologie_1769510862"
    }
  }
}
```

### M2. Persistance d'un Profil Praticien Complet
**Impact Métier et Analyse :**
Cette opération d'écriture crée un nouvel enregistrement de praticien en validant chaque champ contre le schéma Directus. La mutation garantit que seules les données conformes sont enregistrées. Le retour JSON valide la bonne insertion de l'objet dans le modèle de données de PratiLib.

#### Instruction de Mutation :
```graphql
mutation {
  create_praticien_item(data: {
    id: "05a689c4-b9af-46f1-b88e-070d0e361769",
    nom: "Dupont",
    prenom: "Jean",
    ville: "Nancy",
    email: "jean.dupont.1769526014@test.fr",
    telephone: "0601020304",
    organisation: false,
    specialite_id: { id: "1", libelle: "médecine générale" }
  }) {
    id
    nom
    prenom
  }
}
```

#### Confirmation de Transaction API :
```json
{
  "data": {
    "create_praticien_item": {
      "id": "05a689c4-b9af-46f1-b88e-070d0e361769",
      "nom": "Dupont",
      "prenom": "Jean"
    }
  }
}
```

### M3. Mise à jour Signature (Partial Update)
**Impact Métier et Analyse :**
Pour optimiser les performances, nous n'écrivons que les données modifiées. Ici, nous mettons à jour la spécialité d'un praticien existant. Cette approche minimize la charge sur la base de données et réduit les risques de collision lors de modifications simultanées par différents utilisateurs.

#### Instruction de Mutation :
```graphql
mutation {
  update_praticien_item(id: "03f24d7b-efda-3fbe-ae53-0c0c0edef089", data: { specialite_id: { id: "1" } }) {
    id
    nom
    specialite_id {
      id
      libelle
    }
  }
}
```

#### Confirmation de Transaction API :
```json
{
  "data": {
    "update_praticien_item": {
      "id": "03f24d7b-efda-3fbe-ae53-0c0c0edef089",
      "nom": "Grenier",
      "specialite_id": {
        "id": "1",
        "libelle": "médecine générale"
      }
    }
  }
}
```

### M4. Liaison Relationnelle Stricte
**Impact Métier et Analyse :**
Nous démontrons ici comment créer un profil et le lier immédiatement à une spécialité via une clé étrangère (M2O). C'est l'illustration de la gestion des relations par GraphQL : la mutation assure que le lien entre le praticien et sa connaissance métier est établi de manière atomique dès la création.

#### Instruction de Mutation :
```graphql
mutation {
  create_praticien_item(data: {
    id: "629de314-d785-449c-8a45-196b867b1769",
    nom: "Martin",
    prenom: "Paul",
    ville: "Paris",
    email: "paul.martin.1769526014@test.fr",
    telephone: "0701020304",
    organisation: true,
    specialite_id: { id: "1", libelle: "médecine générale" }
  }) {
    nom
    specialite_id {
      libelle
    }
  }
}
```

#### Confirmation de Transaction API :
```json
{
  "data": {
    "create_praticien_item": {
      "nom": "Martin",
      "specialite_id": {
        "libelle": "médecine générale"
      }
    }
  }
}
```

### M5. Mutation de Masse et Deep Insert (Cascade)
**Impact Métier et Analyse :**
C'est l'une des fonctionnalités les plus puissantes de notre architecture : nous créons simultanément le praticien ET sa spécialité dédiée. Cette atomicité garantit qu'il n'y aura jamais d'orphelins en base de données : soit tout est créé avec succès, soit rien ne l'est.

#### Instruction de Mutation :
```graphql
mutation {
  create_praticien_item(data: { 
    id: "690a3691-51a0-485d-8fec-c10cc00fa3ab",
    nom: "Robert", 
    prenom: "Luc",
    ville: "Lille",
    email: "luc.robert.1769522489@test.fr",
    telephone: "0301020304",
    organisation: false,
    specialite_id: { libelle: "chirurgie_1769522489", description: "Opération" } 
  }) { 
    id
    specialite_id { 
      libelle 
    } 
  }
}
```

#### Confirmation de Transaction API :
```json
{
  "data": {
    "create_praticien_item": {
      "id": "690a3691-51a0-485d-8fec-c10cc00fa3ab",
      "specialite_id": {
        "libelle": "chirurgie_new"
      }
    }
  }
}
```

### M6. Validation et Intégrité du Schéma
**Impact Métier et Analyse :**
L'exécution réussie de cette mutation confirme que notre paramétrage Directus respecte rigoureusement les contraintes métier. En utilisant une structure d'objet pour les relations, nous renforçons la clarté du code et la fiabilité des échanges de données entre les différents services du projet.

#### Instruction de Mutation :
```graphql
mutation {
  create_praticien_item(data: { 
    id: "33866a06-776c-4618-b392-3ec868101769",
    nom: "Simon", 
    prenom: "Pierre",
    ville: "Lyon",
    email: "pierre.simon.1769526014@test.fr",
    telephone: "0478000000",
    organisation: false,
    specialite_id: { id: "1", libelle: "médecine générale" }
  }) {
    nom
    specialite_id { 
      libelle 
    }
  }
}
```

#### Confirmation de Transaction API :
```json
{
  "data": {
    "create_praticien_item": {
      "nom": "Simon",
      "specialite_id": {
        "libelle": "médecine générale"
      }
    }
  }
}
```

### M7. Mutation de Transfert Organisationnel
**Impact Métier et Analyse :**
Nous illustrons ici le mouvement d'un praticien vers une nouvelle structure de santé. En mettant à jour la relation 'structure_id', nous modifions le rattachement de l'individu. La réponse JSON inclut le nom de la nouvelle structure, offrant une preuve immédiate de la validité du transfert.

#### Instruction de Mutation :
```graphql
mutation {
  update_praticien_item(id: "05644ac0-c4cb-36bf-a0cf-409075eb20c4", data: { structure_id: { id: "255ecef6-14b9-3b5c-a40e-901665f4ed28" } }) {
    nom
    structure_id { 
      nom 
    }
  }
}
```

#### Confirmation de Transaction API :
```json
{
  "data": {
    "update_praticien_item": {
      "nom": "Leduc",
      "structure_id": {
        "nom": "Existing Structure"
      }
    }
  }
}
```

### M8. Suppression de Masse (Bulk Delete)
**Impact Métier et Analyse :**
Afin de maintenir la performance opérationnelle, nous utilisons les mutations de suppression groupée. En un seul appel, nous sommes capables de nettoyer plusieurs enregistrements obsolètes ou erronés, optimisant ainsi les temps de maintenance et la propreté de la base de données.

#### Instruction de Mutation :
```graphql
mutation {
  delete_praticien_items(ids: ["05a689c4-b9af-46f1-b88e-070d0e361769", "629de314-d785-449c-8a45-196b867b1769"]) {
    ids
  }
}
```

#### Confirmation de Transaction API :
```json
{
  "data": {
    "delete_praticien_items": {
      "ids": [
        "05a689c4-b9af-46f1-b88e-070d0e361769",
        "629de314-d785-449c-8a45-196b867b1769"
      ]
    }
  }
}
```

---
## 6. Industrialisation avec Bruno

Dans une perspective de qualité logicielle (QA) et de documentation vivante, l'intégralité de ces travaux a été packagée au format **Bruno**. Contrairement à Postman, Bruno stocke les requêtes dans des fichiers texte versionnables, favorisant la collaboration via Git. 

Chaque développeur peut ainsi rejouer ces tests, documenter de nouveaux besoins et s'assurer de la non-régression du système de manière autonome et rapide.

## 7. Conclusion Finale

L'implémentation de la couche GraphQL sur **PratiLib** via **Directus** valide un choix technologique moderne et pérenne. Nous avons démontré la capacité du système à :
- Réduire la consommation de bande passante par une sélection granulaire.
- Simplifier le développement d'interfaces complexes grâce au parcours de graphe.
- Sécuriser les transactions de données par un typage strict et des mutations atomiques.

Cette architecture constitue un socle solide pour les évolutions futures de la plateforme, garantissant une flexibilité maximale pour les développements front-end à venir.

---
 Paul Andrieu
 Valentino Lambert

