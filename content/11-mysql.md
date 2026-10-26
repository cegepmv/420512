+++
date = '2026-06-25T16:02:29-04:00'
draft = false
weight = 11
pre="11. "
title = 'MySQL'
+++

{{% expand title="À faire à la maison si vous utilisez windows :(" %}}


## 🛠️ Téléchargement de MySQL

1. Rendez-vous sur le [lien de téléchargement pour Windows](https://dev.mysql.com/downloads/installer/).
2. Cliquez sur le premier bouton **Download**.
3. Cliquez sur *No thanks, just start my download*.
4. Exécutez le fichier que vous venez de télécharger.
5. Choisissez l'option d'installation **Custom**.
6. Dans la section *Products to be installed*, assurez-vous de sélectionner :
   * La version la plus récente de **MySQL Servers** (`MySQL Servers` -> `MySQL Server 8.0` -> Sélectionnez le premier).
   * La version la plus récente de **MySQL Workbench** (`Applications` -> `MySQL Workbench` -> `MySQL Workbench 8.0` -> Sélectionnez le premier).
   * La version la plus récente de **MySQL Shell** (`Applications` -> `MySQL Shell` -> `MySQL Shell 8.0` -> Sélectionnez le premier).
7. Faites **Next** ou **Execute** jusqu'à ce que vous croisiez *Account and Roles*. Dans cette fenêtre, choisissez un mot de passe pour votre base de données (pour le cours, il est conseillé de mettre `abc-123`).
8. Faites **Next** ou **Execute** jusqu'à ce qu'il vous propose **Finish**.
9. Ouvrez l'application **MySQL Workbench**.
10. Vous devriez voir qu'il existe déjà une connexion *Local instance MySQL*. Cliquez dessus.

> 🥳 Félicitations ! Vous êtes prêt à créer votre première table !
{{% /expand %}}

---

## 💻 MySQL depuis la ligne de commande

Si vous utilisez un environnement Linux ou un terminal compatible :

```bash
sudo apt update
sudo apt install mysql-server

```

* Pour vous connecter à MySQL, faites : `mysql -u root -p`. Entrez le mot de passe choisi lors de la création de votre base de données.
* Pour vérifier la connexion, vous pouvez faire : `show databases`.
* Directement dans le terminal, vous pouvez maintenant entrer du code SQL qui agira sur votre base de données.

---

## ⚡ SQL - En un clin d'œil

### `CREATE DATABASE`

```sql
CREATE DATABASE bd;

```

* **CREATE DATABASE** : Indique au système de gestion de base de données que vous souhaitez créer une nouvelle base de données.
* **bd** : C'est le nom de la base de données.
* **;** : Ne jamais oublier le point-virgule.

---

### `USE`

```sql
USE bd;

```

* **USE bd** : Le SGBD change le contexte actif de la session SQL pour la base de données appelée `bd`.

---

### `DROP DATABASE`

```sql
DROP DATABASE bd;

```

* **DROP DATABASE bd;** : Utilisée pour supprimer complètement une base de données d'un serveur SQL.

---

### `CREATE TABLE`

```sql
CREATE TABLE etudiant (
    matricule INT PRIMARY KEY,
    nom_complet VARCHAR(100),
    groupe INT
);

```

* La commande pour créer une table est **CREATE TABLE** suivie du nom de la table `etudiant`.
* Entre parenthèses, chaque ligne représente une colonne.
* Par exemple, pour `matricule INT PRIMARY KEY` :
* **matricule** : Le nom de la colonne.
* **INT** : Le type des données de la colonne.
* **PRIMARY KEY** : Indique que cette colonne est une [clé primaire](https://fr.wikipedia.org/wiki/Cl%C3%A9_primaire).


Voici la **seconde partie** :

```markdown
### `SELECT`
```sql
SELECT * FROM etudiant;

```

* **SELECT** : Instruction pour sélectionner des données.
* ***** : Opérateur signifiant "toutes les colonnes".
* **FROM etudiant** : Spécifie la table source des données.

---

### 🔧 Modifications de tables et requêtes courantes

```sql
-- Pour renommer la table etudiant et lui donner le nom etudiants
RENAME TABLE etudiant To etudiants;

-- Pour Supprimer la table etudiants
DROP TABLE etudiants;

-- Ajouter une colonne
ALTER TABLE etudiants
ADD email VARCHAR(100);

-- Renommer une colonne
ALTER TABLE etudiants
RENAME COLUMN email TO adresse_couriel;

-- Changer le type d'une colonne
ALTER TABLE etudiants
MODIFY COLUMN adresse_couriel VARCHAR(50);

-- Modifier l'ordre des colonnes
ALTER TABLE etudiants
MODIFY adresse_couriel VARCHAR(50)
AFTER matricule; -- Si vous voulez placer la colonne en premier, utiliser seulement FIRST

-- Supprimer une colonne
ALTER TABLE etudiants
DROP COLUMN adresse_couriel;

```

---

### `INSERT INTO`

Insertion de plusieurs enregistrements :

```sql
INSERT INTO etudiants 
VALUES  (123456, "Stefan Salvatore", 1),
        (112347, "Damon Salvatore", 2);

```

> Cette requête insère deux enregistrements dans la table `etudiants`. Chaque enregistrement correspond aux colonnes dans l'ordre de définition de la table.

Insertion partielle :

```sql
INSERT INTO etudiants (matricule, nom_complet)
VALUES  (123456, "Elena Gilbert");

```

> Si vous ne voulez pas ajouter toutes les colonnes, mentionnez les colonnes cibles entre parenthèses après le nom de la table.

---

### `UPDATE`

```sql
UPDATE etudiant
SET nom_complet = "Klaus"
WHERE matricule = 112347;                        

```

> Permet de modifier les données d'une colonne existante en filtrant avec une condition `WHERE`.

---

## 🎯 Exercice pratique

1. Ajouter ces étudiants dans votre table :
```sql
INSERT INTO etudiant (matricule, nom_complet, groupe) VALUES
(1, 'Alice Smith', 1),
(2, 'Bob Johnson', 1),
(3, 'Charlie Brown', 2),
(4, 'Diana Lee', 2),
(5, 'Eric Wang', 1),
(6, 'Fiona Chen', 2),
(7, 'George Liu', 1),
(8, 'Hannah Davis', 2),
(9, 'Ian Miller', 1),
(10, 'Jessica Moore', 2);   

```


2. Faites une requête pour afficher les étudiants du groupe 1.
3. Faites une requête pour mettre Diana Lee dans le groupe 1.
4. Faites une requête pour afficher les noms des étudiants dont le matricule est entre 3 et 8 exclusivement.

{{% expand title="Solution" %}}

```sql
-- 1. Afficher les étudiants du groupe 1
SELECT * FROM etudiant
WHERE groupe = 1;

```

```sql
-- 2. Mettre Diana Lee (matricule 4) dans le groupe 1
UPDATE etudiant
SET groupe = 1
WHERE matricule = 4;                        

```

```sql
-- 3. Afficher les noms des étudiants dont le matricule est entre 4 et 7 (strictement entre 3 et 8)
SELECT nom_complet FROM etudiant
WHERE matricule BETWEEN 4 AND 7;

```

{{% /expand %}}




