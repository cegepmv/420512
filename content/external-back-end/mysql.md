+++
date = '2026-06-26T10:18:19-04:00'
draft = true
title = 'Mysql'
+++

## Téléchargement de MySQL

1. [Lien de téléchargement pour Windows](https://dev.mysql.com/downloads/installer/){target="_blank"}
2. Cliquez sur le premier bouton **Download**
3. Cliquez sur *No thanks, just start my download*
4. Exécutez le fichier que vous venez de télécharger
5. Choisissez l'option d'installation *Custom*
6. Dans la section *Products to be installed*, il doit y avoir :
    * La version la plus récente de MySQL Servers trouvée dans : `MySQL Servers` -> `MySQL Servers` -> `MySQL Server 8.0` -> *Sélectionnez le premier*
    * La version la plus récente de MySQL Workbench trouvée dans : `Applications` -> `MySQL Workbench` -> `MySQL Workbench 8.0` -> *Sélectionnez le premier*
    * La version la plus récente de MySQL Shell trouvée dans : `Applications` -> `MySQL Shell` -> `MySQL Shell 8.0` -> *Sélectionnez le premier*
7. Faites *Next* ou *Execute* jusqu'à ce que vous croisiez **Account and Roles**. Dans cette fenêtre, choisissez un mot de passe pour votre base de données. Vous devez vous en rappeler. Pour mon cours, je vous conseille de mettre `abc-123`.
8. Faites *Next* ou *Execute* jusqu'à ce qu'il vous propose *Finish*.
9. Ouvrir l'application **MySQL Workbench**.
10. Vous devriez voir qu'il existe déjà une connexion *Local instance MySQL*. Cliquez dessus.

> **Félicitations !** Vous êtes prêt à créer votre première table !

---

## MySQL depuis la ligne de commande

```bash
su -
wget [https://dev.mysql.com/get/mysql-apt-config_0.8.32-1_all.deb](https://dev.mysql.com/get/mysql-apt-config_0.8.32-1_all.deb)
sudo dpkg -i mysql-apt-config_0.8.32-1_all.deb
apt update
apt upgrade

apt install mysql-server
rm mysql-apt-config_0.8.32-1_all.deb

```

Pour vous connecter à MySQL, faites : `mysql -u root -p`. Entrez le mot de passe que vous avez choisi à la création de votre base de données.

Pour vérifier la connexion, vous pouvez faire : `show databases;`.

Directement dans le terminal, vous pouvez maintenant entrer du code SQL qui agira sur votre base de données.

---

## SQL - Rapidement

### Créer une base de données

```sql
CREATE DATABASE bd;

```

* `CREATE DATABASE` : Instruction SQL qui indique au système de gestion de base de données que vous souhaitez créer une nouvelle base de données.
* `bd` : Le nom de la base de données.
* `;` : **Ne jamais oublier le point-virgule.**

### Utiliser une base de données

```sql
USE bd;

```

Le SGBD va changer le contexte actif de la session SQL pour la base de données appelée `bd`.

### Supprimer une base de données

```sql
DROP DATABASE bd;

```

Utilisée pour supprimer complètement une base de données d'un serveur SQL.

### Créer une table

```sql
CREATE TABLE etudiant (
    matricule INT PRIMARY KEY,
    nom_complet VARCHAR(100),
    groupe INT
);

```

La commande pour créer une table est `CREATE TABLE` suivi par le nom de la table `etudiant`. Ensuite, entre parenthèses, chaque ligne représente une colonne de la table nouvellement créée.

Exemple : `matricule INT PRIMARY KEY`

* `matricule` : Le nom de la colonne.
* `INT` : Le type des données de la colonne.
* `PRIMARY KEY` : Indique que cette colonne est une clé primaire.

### Sélectionner des données

```sql
SELECT * FROM etudiant;

```

Récupère toutes les colonnes de la table "etudiant".

* `SELECT` : Indique à la base de données que vous souhaitez sélectionner des données.
* `*` : Opérateur qui signifie "toutes les colonnes".
* `FROM etudiant` : Spécifie la table cible.

### Altérer et manipuler les tables

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

### Insérer des données

```sql
INSERT INTO etudiants 
VALUES  (123456, "Stefan Salvatore", 1),
        (112347, "Damon Salvatore", 2);

```

Insère deux enregistrements. Chaque enregistrement est défini par une série de valeurs correspondant aux colonnes dans l'ordre de la structure de la table.

```sql
INSERT INTO etudiants (matricule, nom_complet)
VALUES  (123456, "Elena Gilbert");

```

Si vous ne voulez pas ajouter toutes les colonnes, spécifiez les colonnes cibles entre parenthèses.

### Mettre à jour des données

```sql
UPDATE etudiant
SET nom_complet = "Klaus"
WHERE matricule = 112347;                        

```

---

## Exercice

Insérez d'abord ces étudiants :

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

1. Faites une requête pour afficher les étudiants du groupe 1.
2. Faites une requête pour mettre Diana Lee dans le groupe 1.
3. Faites une requête pour afficher les noms des étudiants dont le matricule est entre 3 et 8 exclusivement.

{{% expand title="Afficher la Solution" %}}

1. Afficher les étudiants du groupe 1 :
```sql
SELECT * FROM etudiant
WHERE groupe=1;

```


2. Mettre Diana Lee dans le groupe 1 :
```sql
UPDATE etudiant
SET groupe = 1
WHERE matricule = 4;                        

```


3. Étudiants avec matricule entre 3 et 8 exclusivement :
```sql
SELECT nom_complet FROM etudiant
WHERE matricule BETWEEN 4 and 7;

```



{{% /expand %}}
