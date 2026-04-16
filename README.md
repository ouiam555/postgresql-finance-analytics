# 🏦 Pipeline de Données Bancaires

Un pipeline ETL (Extraction, Transformation, Chargement) complet pour ingérer, nettoyer et analyser des données de transactions bancaires dans une base de données PostgreSQL.

---

## 📋 Description

Ce projet lit un fichier CSV de transactions bancaires (`bank.csv`), nettoie et normalise les données, les charge dans une base de données PostgreSQL structurée en plusieurs tables relationnelles, puis exécute des requêtes analytiques pour produire des rapports métier.

---

## 🗂️ Structure de la Base de Données

Le schéma relationnel se compose de **5 tables** et **1 vue** :

| Table | Description |
|---|---|
| `client` | Informations client : score de crédit, segment |
| `agence` | Agences bancaires avec leur nom et ville |
| `produit` | Produits bancaires (catégorie, nom) |
| `compte` | Relation entre clients et produits |
| `transactions` | Transactions financières (montant, statut, risque, anomalie) |
| `vw_client_transactions` | Vue analytique croisant clients, transactions, agences et produits |

### Colonnes attendues dans `bank.csv`

```
transaction_id, client_id, date_transaction, montant, devise,
taux_change_eur, categorie, produit, agence, type_operation,
statut, score_credit_client, segment_client, is_anomaly, categorie_risque
```

---

## ⚙️ Prérequis

- Python 3.8+
- PostgreSQL (en cours d'exécution localement ou accessible à distance)
- Un fichier `bank.csv` à la racine du projet

### Dépendances Python

```bash
pip install pandas psycopg2-binary sqlalchemy python-dotenv
```

---

## 🔧 Configuration

Créez un fichier `.env` à la racine du projet avec les variables suivantes :

```env
DB_USER=votre_utilisateur
DB_PASSWORD=votre_mot_de_passe
DB_HOST=localhost
DB_PORT=5432
DB_NAME=nom_de_la_base
```

---

## 🚀 Utilisation

Lancez le notebook Jupyter ou exécutez le script Python :

```bash
jupyter notebook bank.ipynb
```

Le pipeline effectue automatiquement les étapes suivantes :

1. **Création de la base de données** si elle n'existe pas encore
2. **Lecture et nettoyage** du fichier `bank.csv`
3. **Suppression et recréation** du schéma (tables et vue)
4. **Insertion des données** dans les tables (`client`, `agence`, `produit`, `compte`, `transactions`)
5. **Création des index** pour optimiser les performances
6. **Validation** du nombre d'enregistrements par table
7. **Exécution de requêtes analytiques** et export des résultats en CSV

---

## 📊 Analyses Produites

Le pipeline génère 4 rapports CSV :

| Fichier | Contenu |
|---|---|
| `total_transactions_by_agence_month.csv` | Nombre et montant total des transactions par agence et par mois |
| `clients_below_average_balance.csv` | Clients dont le solde total est inférieur à la moyenne nationale |
| `default_rate_by_risk_segment.csv` | Taux de défaut (transactions annulées) par segment client et catégorie de risque |
| `top_10_agencies.csv` | Les 10 agences avec le plus fort volume de transactions complétées |

---

## 🧹 Règles de Nettoyage des Données

- Les valeurs textuelles vides ou `NaN` sont remplacées par `pd.NA`
- Le `score_credit_client` manquant est remplacé par la **médiane** (valeur clippée entre 0 et 1000)
- Le `segment_client` manquant est remplacé par `"Inconnu"`
- Le statut des transactions est normalisé vers : `Completed`, `Pending`, `Cancelled`
- La catégorie de risque est normalisée vers : `Low`, `Medium`, `High`
- Les transactions avec un montant nul ou manquant sont **exclues**

---

## 🗃️ Index Créés

Pour optimiser les performances des requêtes analytiques :

```sql
idx_transactions_client   -- Sur transactions(client_id)
idx_transactions_date     -- Sur transactions(date_transaction)
idx_transactions_agence   -- Sur transactions(agence_id)
idx_client_segment        -- Sur client(segment_client)
```

---

## 📁 Structure du Projet

```
.
├── bank.ipynb                          # Notebook principal (pipeline ETL)
├── bank.csv                            # Fichier de données source (requis)
├── .env                                # Variables d'environnement (à créer)
├── total_transactions_by_agence_month.csv   # Rapport généré
├── clients_below_average_balance.csv        # Rapport généré
├── default_rate_by_risk_segment.csv         # Rapport généré
└── top_10_agencies.csv                      # Rapport généré
```

---

## ⚠️ Notes Importantes

- Le schéma est **entièrement recréé** à chaque exécution (les données existantes sont supprimées).
- Le fichier `bank.csv` doit être présent à la racine avant de lancer le pipeline.
- Toutes les variables d'environnement dans `.env` sont **obligatoires**.