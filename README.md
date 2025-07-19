# 🚀 Projet DBT – Démo et Initialisation

## 🔍 Qu'est-ce que DBT ?

**DBT** (Data Build Tool) est un outil open-source conçu pour **simplifier et structurer la transformation des données** dans les entrepôts de données modernes.

Il permet aux **data analysts** et **data engineers** d’écrire des transformations en SQL tout en appliquant les **bonnes pratiques du développement logiciel** :

- Modularité
- Réutilisabilité
- Gestion des dépendances
- Versionnement

DBT permet de transformer des données **brutes** en données **prêtes à l’analyse**, via des modèles SQL exécutés dans le **data warehouse**.

Il existe deux versions :
- **DBT Cloud** (version hébergée)
- **DBT Core** (version gratuite en local – utilisée ici)

---

## ⚙️ Installation de DBT Core

DBT Core est un **package Python**. Pour l'utiliser, vous devez :

1. Avoir **Python** installé
2. Installer DBT et un adaptateur pour votre base de données (ici : MySQL)

---

## 📥 Création de la base de données (avec SQLAlchemy + Pandas)

```python
import pandas as pd
from sqlalchemy import create_engine
from sqlalchemy_utils import database_exists, create_database

# Connexion à la base MySQL
username = "root"
password = "password"
host = "localhost"
port = 3306
database = "my_dbt_db"

DATABASE_URI = f'mysql+pymysql://{username}:{password}@{host}:{port}/{database}'
engine = create_engine(DATABASE_URI)

if not database_exists(engine.url):
    create_database(engine.url)

# Chargement des fichiers CSV
liste_tables = ["customers", "items", "orders", "products", "stores", "supplies"]

for table in liste_tables:
    csv_url = f"https://raw.githubusercontent.com/dsteddy/jaffle_shop_data/main/raw_{table}.csv"
    df = pd.read_csv(csv_url)
    df.to_sql(f"raw_{table}", engine, if_exists="replace", index=False)
```

> 💡 Remplace `root` et `password` par les identifiants de **votre installation MySQL locale**.

🔐 Si votre mot de passe contient un caractère spécial (`@`, `/`, etc.), pensez à **l’encoder** (par exemple `@` devient `%40`).

---

## 🛠️ Configuration du fichier `profiles.yml` pour DBT

Une fois la base de données créée, configurez DBT en modifiant `profiles.yml`.

### 📁 Emplacement :

- **Windows** : `C:\Users\VotreNom\.dbt\profiles.yml`
- **Mac/Linux** : `~/.dbt/profiles.yml`

### 💡 Exemple :

```yaml
dbt_quest:
  target: dev
  outputs:
    dev:
      type: mysql
      server: localhost
      port: 3306
      database: my_dbt_db
      schema: my_dbt_db
      username: root
      password: password
      driver: MySQL ODBC 8.0 ANSI Driver
```

---

## 🧱 Structure typique d’un projet DBT

| Dossier/Fichier         | Description |
|-------------------------|-------------|
| `dbt_project.yml`       | Fichier de configuration du projet |
| `models/`               | Modèles SQL transformant les données |
| `macros/`               | Fonctions personnalisées en Jinja |
| `tests/`                | Tests de validation des données |
| `analyses/`             | Analyses exploratoires en SQL |
| `snapshots/`            | Suivi de l’évolution d’une table |
| `seeds/`                | Données CSV intégrées |
| `logs/`                 | Fichiers de logs DBT |

> ✅ Dans ce projet :  
> `DBT_DEMO/dbt_projet/analyses/.gitkeep` permet de garder le dossier `analyses` versionné même s’il est vide.

---

## 🧼 Fichier `.gitignore` recommandé

Pour éviter d’ajouter des fichiers sensibles ou inutiles dans votre dépôt :

```
# Environnement virtuel
my-dbt-env/

# Logs DBT
logs/

# Cache
dbt_modules/
dbt_packages/
__pycache__/

# Fichiers temporaires
*.pyc
*.ipynb_checkpoints/

# Fichiers de configuration sensibles
~/.dbt/
```

---

## ✅ Tests dans DBT

### 🔧 Tests intégrés

DBT propose plusieurs tests intégrés à écrire dans les fichiers `.yml` :

- `unique` : Vérifie l’unicité des valeurs
- `not_null` : Vérifie qu’il n’y a pas de valeurs nulles
- `accepted_values` : Vérifie que les valeurs appartiennent à un ensemble autorisé
- `relationships` : Vérifie l’intégrité référentielle

#### Exemple dans `stg_customers.yml` :

```yaml
version: 2

models:
  - name: stg_customers
    columns:
      - name: customer_id
        tests:
          - not_null
          - unique
      - name: customer_name
        tests:
          - not_null
```

> 🧠 Remarque : cette quête utilise une **ancienne version de DBT Core**, où les tests sont directement listés dans les modèles. Les versions récentes utilisent `data_tests`.

---

### 🧪 Tests personnalisés

Les tests personnalisés sont des requêtes SQL stockées dans le dossier `tests`.

#### Exemple :

Vérifier que `ordered_at` ne contient pas de dates futures :

📄 `tests/order_dates_in_the_past.sql` :

```sql
with invalid_orders as (
    select *
    from {{ ref('stg_orders') }}
    where ordered_at > current_date
)
select count(*)
from invalid_orders
having count(*) > 0
```

> ✅ Si aucun résultat n’est retourné, le test est **réussi**.

---

### ▶️ Exécuter les tests

```bash
dbt test
```

Pour exécuter un seul test :

```bash
dbt test --select order_dates_in_the_past
```

💡 En cas d’erreur liée à protobuf, entrez :

```bash
pip install protobuf==4.25.5
```

---

## 📚 Documentation dans DBT

### 🏗️ Génération :

```bash
dbt docs generate
dbt docs serve
```

### 📝 Ajouter des descriptions :

#### Exemple dans `stg_customers.yml` :

```yaml
version: 2

models:
  - name: stg_customers
    description: Customer data with basic cleaning and transformation applied, one row per customer.
    columns:
      - name: customer_id
        description: The unique key for each customer.
      - name: customer_name
        description: The full name of the customer.
```

#### Exemple dans `sources.yml` :

```yaml
version: 2

sources:
  - name: jaffle_shop
    schema: my_dbt_db
    description: E-commerce data for the Jaffle Shop
    tables:
      - name: raw_customers
        description: One record per person who has purchased one or more items
```

#### Utiliser un fichier Markdown externe :

📄 `store_id_description.md` :

```jinja
{% docs store_id_description %}

This is the ID for the store.

It references the ID column from the stores table.

{% enddocs %}
```

📄 `stg_orders.yml` :

```yaml
      - name: store_id
        description: "{{ doc('store_id_description') }}"
        tests:
          - not_null
```

---

## 🌱 Seeds dans DBT

Les **seeds** sont des fichiers `.csv` stockés dans le dossier `seeds/` et chargés dans le data warehouse.

### Exemple : `customer_status.csv`

| status | min_orders |
|--------|------------|
| bronze | 1          |
| silver | 5          |
| gold   | 10         |

### 📥 Charger les seeds :

```bash
dbt seed
```

Cela crée une table `customer_status`.

### Utilisation dans un modèle :

```sql
with status_data as (
    select *
    from {{ ref('customer_status') }}
)
select *
from status_data
```

---


📁 Projet : https://github.com/Sanlexis95/DBT  
🧑‍💻 Auteur : [Sanlexis95](https://github.com/Sanlexis95)
