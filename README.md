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

## 💡 Pourquoi `.gitignore` est essentiel ?

- 🔐 Protéger vos **informations sensibles** (mots de passe, profils)
- 🚀 Garder un dépôt **propre et léger**
- 🤝 Faciliter la **collaboration** avec d’autres développeurs

---

## 🧪 Challenge

👉 Créez vous-même un projet DBT local  
👉 Initialisez-le avec `dbt init`  
👉 Hébergez-le sur GitHub (comme ici : [Sanlexis95/DBT](https://github.com/Sanlexis95/DBT))  
👉 Vérifiez que vous excluez bien les bons fichiers via `.gitignore`

---

📁 Projet : https://github.com/Sanlexis95/DBT  
🧑‍💻 Auteur : [Sanlexis95](https://github.com/Sanlexis95)
