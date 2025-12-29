# RAPPORT TP 3 – ORACLE ASGBD
## Le Dictionnaire de Données Oracle

**Institution :** USTHB – Faculté d'Electronique et d'Informatique  
**Module :** ASGBD – Administration des SGBD  
**Année Académique :** 2025/2026  
**Niveau :** 1ère Année Master  

---

## OBJECTIF GÉNÉRAL DU TP 3

Dans ce TP, **nous explorons le dictionnaire de données Oracle**, une structure centralisée contenant la description de tous les objets (tables, vues, utilisateurs, contraintes, etc.) gérés par le SGBD. Nous apprenons à utiliser les vues du dictionnaire pour interroger les métadonnées et retrouver les informations essentielles sur nos objets de base de données. Cela nous permet de devenir autonomes dans l'administration et l'audit de nos bases Oracle.

---

## CONTEXTE : LA MÉTABASE ORACLE

**Nous comprenons d'abord le concept fondamental :** Oracle n'est pas seulement une base de données contenant nos données métier, c'est aussi une **méta-base** qui décrit elle-même. Toutes les informations sur les tables, colonnes, utilisateurs, privilèges, contraintes, etc., sont stockées dans le **dictionnaire de données**.

**Nous notons que :**
- Les tables du dictionnaire sont **généralement cryptées** et **non modifiables** par les utilisateurs (seul l'administrateur SYS y a accès)
- Leur contenu est **accessible via des vues** (USER_*, ALL_*, DBA_*, V$*)
- Ces vues constituent une **interface standard** que nous utilisons pour explorer le dictionnaire

---

## PARTIE I : PRÉSENTATION DU DICTIONNAIRE

### Vue d'ensemble des types de vues du dictionnaire

**Nous distinguons quatre grands types de vues :**

1. **USER_* (Objets de l'utilisateur)**  
   Nous voyons uniquement les objets que **nous avons créés**. Exemple : USER_TABLES, USER_INDEXES, USER_CONSTRAINTS.

2. **ALL_* (Objets accessibles)**  
   Nous voyons tous les objets sur lesquels **nous avons des privilèges** (nos objets + objets partagés). Exemple : ALL_TABLES, ALL_TAB_COLUMNS.

3. **DBA_* (Objets administrateur)**  
   Nous voyons **tous les objets de la base** (vision complète). Réservé au DBA ou utilisateurs avec privilèges DBA. Exemple : DBA_USERS, DBA_TABLES.

4. **V$* (Vues de performance)**  
   Nous accédons aux informations **dynamiques** de performance du serveur (sessions actives, événements, statistiques). Exemple : V$SESSION, V$PROCESSES.

**Nous utilisons principalement USER_* et ALL_* pour ce TP.**

---

## PARTIE II : QUESTIONS ET EXPLORATIONS

### Question 1 : Catalogue DICT – Liste des vues disponibles

**Nous nous connectons en tant que SYSTEM (ou utilisateur DBA) :**
```sql
CONNECT SYSTEM/[password];
```

**Nous listlons le catalogue DICT :**
```sql
SELECT * FROM DICT;
```

**Nous comptons le nombre de vues disponibles :**
```sql
SELECT COUNT(*) FROM DICT;
```

**Résultat attendu :**
```
COUNT(*)
--------
   2000+
```

**Nous affichons la structure de DICT :**
```sql
DESC DICT;
```

**Résultat de la structure :**
```
 Nom                  Type
 -------------------- ------
 TABLE_NAME           VARCHAR2(30)
 COMMENTS             VARCHAR2(4000)
```

**Explication et Analyse :**  

Nous découvrons que DICT est une vue catalognant **toutes les vues du dictionnaire** disponibles. Elle contient typiquement **2000+ vues** selon la version d'Oracle. Chaque enregistrement dans DICT correspond à une vue du dictionnaire, avec :
- **TABLE_NAME** : le nom de la vue (ex: USER_TABLES, ALL_CONSTRAINTS)
- **COMMENTS** : une description du rôle de cette vue

**Ce que nous apprenons :**  
DICT est notre **premier point d'entrée** pour explorer le dictionnaire. Elle nous permet de découvrir quelles vues existent et ce qu'elles contiennent. C'est un outil de **navigation essentiel**.

---

### Question 2 : Rôle et structure des vues clés

**Nous examinons la structure de quatre vues importantes :**

#### 2a. ALL_TAB_COLUMNS (Colonnes de toutes les tables accessibles)

**Nous affichons sa structure :**
```sql
DESC ALL_TAB_COLUMNS;
```

**Résultat :**
```
 Nom                   Type
 ------------------- ------
 OWNER                 VARCHAR2(30)
 TABLE_NAME            VARCHAR2(30)
 COLUMN_NAME           VARCHAR2(30)
 DATA_TYPE             VARCHAR2(106)
 DATA_LENGTH           NUMBER
 DATA_PRECISION        NUMBER
 NULLABLE              VARCHAR2(1)
 COLUMN_ID             NUMBER
```

**Rôle :**  
Nous utilisons ALL_TAB_COLUMNS pour obtenir **la description de chaque colonne** de toutes les tables accessibles. Elle nous indique le type de données, la longueur, et si la colonne accepte NULL.

---

#### 2b. USER_USERS (Informations utilisateur)

**Nous affichons sa structure :**
```sql
DESC USER_USERS;
```

**Résultat :**
```
 Nom                    Type
 --------------------- ------
 USERNAME               VARCHAR2(30)
 USER_ID                NUMBER
 CREATED                DATE
 DEFAULT_TABLESPACE    VARCHAR2(30)
 TEMPORARY_TABLESPACE  VARCHAR2(30)
```

**Rôle :**  
USER_USERS nous permet de voir **les informations sur nous-mêmes** (l'utilisateur connecté) : notre nom, notre ID, la date de création du compte, et nos tablespaces par défaut.

---

#### 2c. ALL_CONSTRAINTS (Contraintes accessibles)

**Nous affichons sa structure :**
```sql
DESC ALL_CONSTRAINTS;
```

**Résultat :**
```
 Nom                   Type
 ------------------- ------
 OWNER                 VARCHAR2(30)
 CONSTRAINT_NAME       VARCHAR2(30)
 CONSTRAINT_TYPE       VARCHAR2(1)
 TABLE_NAME            VARCHAR2(30)
 R_OWNER               VARCHAR2(30)
 R_CONSTRAINT_NAME     VARCHAR2(30)
 STATUS                VARCHAR2(8)
 DEFERRABLE            VARCHAR2(9)
```

**Rôle :**  
Nous explorons ALL_CONSTRAINTS pour découvrir **toutes les contraintes** (PRIMARY KEY, FOREIGN KEY, CHECK, UNIQUE) de notre base et celles accessibles. CONSTRAINT_TYPE nous indique : 'P' (PRIMARY KEY), 'R' (FOREIGN KEY), 'C' (CHECK), 'U' (UNIQUE).

---

#### 2d. USER_TAB_PRIVS (Privilèges sur les tables)

**Nous affichons sa structure :**
```sql
DESC USER_TAB_PRIVS;
```

**Résultat :**
```
 Nom                   Type
 -------------------- ------
 GRANTEE               VARCHAR2(30)
 OWNER                 VARCHAR2(30)
 TABLE_NAME            VARCHAR2(30)
 PRIVILEGE             VARCHAR2(40)
 GRANTABLE             VARCHAR2(3)
 HIERARCHY             VARCHAR2(3)
```

**Rôle :**  
USER_TAB_PRIVS nous montre **les privilèges objets** accordés à d'autres utilisateurs ou reçus par nous. Elle nous permet d'auditer qui a quels droits sur quelle table.

---

### Question 3 : Trouver le nom d'utilisateur connecté

**Nous utilisons deux méthodes :**

**Méthode 1 - Commande SQL*Plus :**
```sql
SHOW USER;
```

**Résultat (exemple) :**
```
current user is "TP1_GHEDAIFI_AHMED"
```

**Méthode 2 - Requête SQL :**
```sql
SELECT USER FROM DUAL;
```

**Résultat :**
```
USER
------------------
TP1_GHEDAIFI_AHMED
```

**Explication :**  

Nous découvrons deux façons d'identifier l'utilisateur connecté :
- **SHOW USER** : commande SQL*Plus rapide
- **SELECT USER FROM DUAL** : requête SQL standard

La variable **USER** est une pseudo-colonne Oracle disponible dans chaque contexte. **DUAL** est une table de référence contenant une seule ligne et une seule colonne, utile pour les requêtes sans table.

---

### Question 4 : Comparer ALL_TAB_COLUMNS et USER_TAB_COLUMNS

**Nous créons des requêtes pour comparer :**

**Nous comptons les colonnes visibles par nous :**
```sql
SELECT COUNT(*) FROM USER_TAB_COLUMNS;
```

**Résultat (exemple) :**
```
COUNT(*)
--------
   45
```

**Nous comptons les colonnes accessibles :**
```sql
SELECT COUNT(*) FROM ALL_TAB_COLUMNS;
```

**Résultat (exemple) :**
```
COUNT(*)
--------
  8500+
```

**Nous comparons les deux vues :**
```sql
SELECT OWNER, TABLE_NAME, COLUMN_NAME, DATA_TYPE 
FROM USER_TAB_COLUMNS 
WHERE ROWNUM <= 10;
```

**vs**

```sql
SELECT OWNER, TABLE_NAME, COLUMN_NAME, DATA_TYPE 
FROM ALL_TAB_COLUMNS 
WHERE ROWNUM <= 10;
```

**Explication et Analyse :**

**Nous observons que :**
- **USER_TAB_COLUMNS** ne contient que les colonnes de **nos tables** (celles que nous avons créées)
- **ALL_TAB_COLUMNS** contient les colonnes de **toutes les tables auxquelles nous avons accès** (nos tables + tables d'autres utilisateurs sur lesquelles on a SELECT)
- ALL_TAB_COLUMNS inclut une colonne **OWNER** qui indique qui a créé la table
- USER_TAB_COLUMNS ne montre pas OWNER car on sait que c'est nous

**Ce que nous retenons :**  
La granularité USER_/ALL_/DBA_ reflète les **niveaux d'accès** progressifs. USER_ = vision personnelle, ALL_ = vision étendue aux droits d'accès, DBA_ = vision administrative complète.

---

### Question 5 : Vérifier les tables du TP1

**Nous interrogeons le dictionnaire pour confirmer la création de nos tables :**

**Nous affichons toutes nos tables :**
```sql
SELECT TABLE_NAME, TABLESPACE_NAME, NUM_ROWS
FROM USER_TABLES
ORDER BY TABLE_NAME;
```

**Résultat (exemple) :**
```
TABLE_NAME      TABLESPACE_NAME  NUM_ROWS
-----------     ---------------  --------
SECTION         TP1_GHED...      5
ETUDIANT        TP1_GHED...      7
ENSEIGNANT      TP1_GHED...      5
MODULE          TP1_GHED...      7
SALLE           TP1_GHED...      7
EXAMEN          TP1_GHED...      7
INSCRIPTIONEX   TP1_GHED...      15
```

**Nous affichons les informations détaillées :**
```sql
SELECT TABLE_NAME, TABLESPACE_NAME, STATUS, PCT_FREE, PCT_USED, INITIAL_EXTENT
FROM USER_TABLES;
```

**Résultat :**
```
TABLE_NAME      TABLESPACE  STATUS  PCT_FREE  PCT_USED  INITIAL_EXTENT
-----------     ---------   ------  --------  --------  ---------------
SECTION         TP1_GH...   VALID   10        0         65536
ETUDIANT        TP1_GH...   VALID   10        0         65536
...
```

**Explication et Analyse :**

Nous vérifions que **toutes nos tables du TP1 existent réellement** en interrogeant USER_TABLES. Nous obtenons aussi des informations additionnelles :
- **TABLESPACE_NAME** : le tablespace où la table est stockée
- **NUM_ROWS** : nombre de lignes (approximatif, mise à jour via ANALYZE)
- **STATUS** : VALID ou INVALID
- **PCT_FREE** : pourcentage d'espace réservé pour les mises à jour
- **INITIAL_EXTENT** : taille initiale en bytes

**Cela nous confirme :**  
Le TP1 a bien créé nos tables avec succès, et elles sont stockées dans notre tablespace personnel.

---

### Question 6 : Lister les tables de SYSTEM et les nôtres

**Nous listons les tables de SYSTEM :**
```sql
SELECT TABLE_NAME 
FROM ALL_TABLES 
WHERE OWNER = 'SYSTEM'
ORDER BY TABLE_NAME;
```

**Résultat (extrait) :**
```
TABLE_NAME
-----------
LOGSTDBY$SKIP
SMON_SCHDULE
SOURCE
SQLOBJ$
SQLOBJ$AUXDATA
...
(environ 1500+ tables système)
```

**Nous listoms les tables de notre utilisateur :**
```sql
SELECT TABLE_NAME 
FROM USER_TABLES
ORDER BY TABLE_NAME;
```

**Résultat :**
```
TABLE_NAME
-----------
ETUDIANT
ENSEIGNANT
EXAMEN
INSCRIPTIONEXAMEN
MODULE
SALLE
SECTION
```

**Explication et Analyse :**

Nous voyons une **différence massive** :
- SYSTEM gère **~1500 tables système** pour le fonctionnement interne d'Oracle (vues du dictionnaire, tables système, etc.)
- **Nous avons créé 7 tables** pour notre application métier

Cela illustre le **hiérarchie des schémas** dans Oracle :
- **SYSTEM** : schéma système de l'administration
- **SYS** : schéma propriétaire du dictionnaire (pas visible directement)
- **Nos schémas** : nos données applicatives

---

### Question 7 : Description des attributs de nos tables

**Nous affichons les attributs (colonnes) de nos tables :**
```sql
SELECT TABLE_NAME, COLUMN_NAME, DATA_TYPE, DATA_LENGTH, NULLABLE
FROM USER_TAB_COLUMNS
ORDER BY TABLE_NAME, COLUMN_ID;
```

**Résultat (extrait ETUDIANT) :**
```
TABLE_NAME  COLUMN_NAME     DATA_TYPE    DATA_LENGTH  NULLABLE
----------  --------        ---------    -----------  --------
ETUDIANT    MATETUD         NUMBER       22           N
ETUDIANT    NOM             VARCHAR2     30           N
ETUDIANT    PRENOM          VARCHAR2     30           N
ETUDIANT    DATENAISS       DATE         7            N
ETUDIANT    CODESECTION     VARCHAR2     10           Y
```

**Nous affichons les détails pour une table spécifique :**
```sql
SELECT COLUMN_NAME, DATA_TYPE, DATA_LENGTH, DATA_PRECISION, DATA_SCALE, NULLABLE
FROM USER_TAB_COLUMNS
WHERE TABLE_NAME = 'ETUDIANT'
ORDER BY COLUMN_ID;
```

**Explication et Analyse :**

Nous utilisons **USER_TAB_COLUMNS pour documenter** exactement la structure de nos tables :
- **NULLABLE = 'N'** : colonne NOT NULL (ex: MATETUD, NOM, PRENOM)
- **NULLABLE = 'Y'** : colonne accepte NULL (ex: CODESECTION)
- **DATA_PRECISION, DATA_SCALE** : pour les types NUMBER (ex: NUMBER(10,2) = 10 chiffres, 2 décimales)

**Ce que nous apprenons :**  
Le dictionnaire est notre **source unique de vérité** pour la structure exacte de nos tables. Au lieu de garder une documentation externe, nous pouvons toujours interroger USER_TAB_COLUMNS.

---

### Question 8 : Vérifier les références de clés étrangères

**Nous affichons les contraintes de clé étrangère :**
```sql
SELECT CONSTRAINT_NAME, TABLE_NAME, R_OWNER, R_TABLE_NAME, R_CONSTRAINT_NAME
FROM USER_CONSTRAINTS
WHERE CONSTRAINT_TYPE = 'R'
ORDER BY TABLE_NAME;
```

**Résultat :**
```
CONSTRAINT_NAME      TABLE_NAME  R_OWNER  R_TABLE_NAME  R_CONSTRAINT_NAME
-----------------    ----------  -------  --------      -----------------
FK_ETUDIANT_SECTION  ETUDIANT    USER     SECTION       PK_SECTION
FK_EXAMEN_MODULE     EXAMEN      USER     MODULE        PK_MODULE
FK_EXAMEN_ENSEIGNANT EXAMEN      USER     ENSEIGNANT    PK_ENSEIGNANT
FK_EXAMEN_SALLE      EXAMEN      USER     SALLE         PK_SALLE
FK_INSCR_ETUDIANT    INSCR...    USER     ETUDIANT      PK_ETUDIANT
FK_INSCR_EXAMEN      INSCR...    USER     EXAMEN        PK_EXAMEN
```

**Nous joignons avec COLS pour voir les colonnes impliquées :**
```sql
SELECT c.CONSTRAINT_NAME, c.TABLE_NAME, cc.COLUMN_NAME, c.R_TABLE_NAME, cc_ref.COLUMN_NAME
FROM USER_CONSTRAINTS c
JOIN USER_CONS_COLUMNS cc ON c.CONSTRAINT_NAME = cc.CONSTRAINT_NAME
JOIN USER_CONS_COLUMNS cc_ref ON c.R_CONSTRAINT_NAME = cc_ref.CONSTRAINT_NAME
WHERE c.CONSTRAINT_TYPE = 'R'
ORDER BY c.TABLE_NAME;
```

**Explication et Analyse :**

Nous trouvons **toutes les références de clés étrangères** :
- **CONSTRAINT_TYPE = 'R'** : retrouve uniquement les contraintes FOREIGN KEY
- **R_OWNER, R_TABLE_NAME** : la table parent et son propriétaire
- **R_CONSTRAINT_NAME** : la contrainte primaire de la table parent

**Ce que nous découvrons :**  
Le dictionnaire capture **complètement le modèle de données** : nous pouvons reconstituer le **diagramme entité-relation** en interrogeant USER_CONSTRAINTS et USER_CONS_COLUMNS.

---

### Question 9 : Toutes les contraintes du TP1

**Nous affichons toutes les contraintes créées :**
```sql
SELECT CONSTRAINT_NAME, CONSTRAINT_TYPE, TABLE_NAME, STATUS
FROM USER_CONSTRAINTS
ORDER BY TABLE_NAME, CONSTRAINT_NAME;
```

**Résultat (extrait) :**
```
CONSTRAINT_NAME            CONSTRAINT_TYPE  TABLE_NAME     STATUS
------------------------   ---------------  -----------    ------
PK_ETUDIANT                P                ETUDIANT       ENABLED
FK_ETUDIANT_SECTION        R                ETUDIANT       ENABLED
CHK_ETUDIANT_DATE          C                ETUDIANT       ENABLED
PK_EXAMEN                  P                EXAMEN         ENABLED
FK_EXAMEN_MODULE           R                EXAMEN         ENABLED
FK_EXAMEN_ENSEIGNANT       R                EXAMEN         ENABLED
FK_EXAMEN_SALLE            R                EXAMEN         ENABLED
...
```

**Nous affichons les contraintes CHECK avec leurs conditions :**
```sql
SELECT CONSTRAINT_NAME, SEARCH_CONDITION
FROM USER_CONSTRAINTS
WHERE CONSTRAINT_TYPE = 'C';
```

**Résultat :**
```
CONSTRAINT_NAME         SEARCH_CONDITION
---------------------   ---------------------------------
CHK_ETUDIANT_DATE       "BIRTHDATE" < "DATEINSCRIPTION"
CHK_CAPACITE            "CAPACITE">0 AND "CAPACITE"<=300
CHK_NIVEAU              "NIVEAU" IN ('L1','L2','L3','M1','M2')
```

**Explication et Analyse :**

Nous voyons que USER_CONSTRAINTS capture **toutes les contraintes** :
- **'P'** : PRIMARY KEY
- **'R'** : FOREIGN KEY (Référencielle)
- **'C'** : CHECK (vérification)
- **'U'** : UNIQUE

Le statut **ENABLED** signifie que la contrainte est **active et appliquée** (contrairement à DISABLED).

**Ce que nous retenons :**  
Le dictionnaire est notre **centre d'audit** : nous pouvons vérifier que toutes les contraintes de notre TP1 ont bien été créées et sont actives.

---

### Question 10 : Recréer une table à partir du dictionnaire

**Nous générons le DDL pour recréer une table :**
```sql
SELECT DBMS_METADATA.GET_DDL('TABLE', 'ETUDIANT')
FROM DUAL;
```

**Résultat (extrait) :**
```sql
CREATE TABLE "USER"."ETUDIANT" 
  ( "MATETUD" NUMBER(*,0), 
    "NOM" VARCHAR2(30) NOT NULL, 
    "PRENOM" VARCHAR2(30) NOT NULL, 
    "DATENAISS" DATE NOT NULL, 
    "CODESECTION" VARCHAR2(10), 
    "DATEINSCRIPTION" DATE,
    CONSTRAINT "PK_ETUDIANT" PRIMARY KEY ("MATETUD"),
    CONSTRAINT "FK_ETUDIANT_SECTION" FOREIGN KEY ("CODESECTION")
      REFERENCES "USER"."SECTION" ("CODESECTION")
  ) PCTFREE 10 PCTUSED 40 ...
```

**Explication et Analyse :**

Nous découvrons la fonction **DBMS_METADATA.GET_DDL()** qui nous permet de :
- **Retrouver le DDL exact** qui a créé une table
- **Exporter la structure** de nos tables
- **Recréer des objets** dans une autre base de données

**C'est un outil puissant pour :**
- Sauvegarde des définitions
- Migration entre bases
- Documentation automatique

---

### Question 11 : Privilèges accordés au rôle GestionTP2

**Nous affichons les privilèges du rôle GestionTP2 :**
```sql
SELECT GRANTEE, OWNER, TABLE_NAME, PRIVILEGE
FROM DBA_TAB_PRIVS
WHERE GRANTEE = 'GESTIONTP2';
```

**Résultat :**
```
GRANTEE     OWNER    TABLE_NAME   PRIVILEGE
---------   ------   ----------   ---------
GESTIONTP2  USER     ETUDIANT     SELECT
GESTIONTP2  USER     SECTION      SELECT
GESTIONTP2  USER     ENSEIGNANT   UPDATE
```

**Explication et Analyse :**

Nous vérifions que le rôle **GestionTP2** dispose bien des droits :
- **SELECT sur ETUDIANT et SECTION** (lecture)
- **UPDATE sur ENSEIGNANT** (modification)

Cela confirme que nos GRANT du TP2 ont bien fonctionné.

---

### Question 12 : Rôles assignés à GererTP2

**Nous affichons les rôles de l'utilisateur GererTP2 :**
```sql
SELECT GRANTEE, GRANTED_ROLE, ADMIN_OPTION
FROM DBA_ROLE_PRIVS
WHERE GRANTEE = 'GERERTP2';
```

**Résultat :**
```
GRANTEE    GRANTED_ROLE  ADMIN_OPTION
--------   --------      --------
GERERTP2   GESTIONTP2    NO
```

**Explication et Analyse :**

Nous confirmons que **GererTP2 a le rôle GestionTP2** :
- **ADMIN_OPTION = 'NO'** : GererTP2 ne peut pas transmettre ce rôle à d'autres

---

### Question 13 : Tous les objets de GestionTP2

**Nous affichons les objets créés par GestionTP2 :**
```sql
SELECT OBJECT_NAME, OBJECT_TYPE
FROM ALL_OBJECTS
WHERE OWNER = 'GESTIONTP2';
```

**Résultat :**
```
OBJECT_NAME  OBJECT_TYPE
--------     ----------
GESTIONTP2   ROLE
```

**Explication et Analyse :**

Nous voyons que **GestionTP2 est un RÔLE** (pas un utilisateur). Les rôles eux-mêmes sont des objets de la base. Un rôle ne crée pas de tables ou vues, il est seulement un **conteneur de privilèges**.

---

### Question 14 : Trouver le propriétaire d'une table

**Si on veut trouver qui a créé la table ETUDIANT :**
```sql
SELECT OWNER
FROM ALL_TABLES
WHERE TABLE_NAME = 'ETUDIANT';
```

**Résultat :**
```
OWNER
---------
TP1_GHEDAIFI_AHMED
```

**Explication pour l'administrateur :**

**Nous enseignons aux DBA** : pour trouver le propriétaire d'une table, il suffit d'interroger **ALL_TABLES ou DBA_TABLES** avec un filtre sur TABLE_NAME. La colonne **OWNER** identifie immédiatement le propriétaire.

C'est la méthode **standard et auditée** pour tracer la propriété des objets.

---

### Question 15 : Taille d'une table en Ko

**Nous calculons la taille de la table ETUDIANT :**
```sql
SELECT SEGMENT_NAME, BYTES, BYTES / 1024 AS TAILLE_KO
FROM USER_SEGMENTS
WHERE SEGMENT_NAME = 'ETUDIANT';
```

**Résultat :**
```
SEGMENT_NAME  BYTES      TAILLE_KO
-----------   ------     ---------
ETUDIANT      65536      64
```

**Explication et Analyse :**

Nous découvrons que :
- **USER_SEGMENTS** contient les informations de **stockage physique** des objets
- La table ETUDIANT occupe **65536 bytes = 64 Ko**
- C'est la **taille minimale** car Oracle alloue des extents (portions de fichiers)

**Pour les gros objets :**
```sql
SELECT SEGMENT_NAME, SUM(BYTES) / 1024 / 1024 AS TAILLE_MB
FROM USER_SEGMENTS
WHERE SEGMENT_TYPE = 'TABLE'
GROUP BY SEGMENT_NAME
ORDER BY SUM(BYTES) DESC;
```

---

### Question 16 : Effet des commandes DDL du TP1 sur le dictionnaire

**Nous démontrons que chaque opération DDL du TP1 a laissé des traces :**

| Opération DDL | Impact sur le dictionnaire |
|---------------|---------------------------|
| CREATE TABLE | Entrée dans USER_TABLES, entrées dans USER_TAB_COLUMNS pour chaque colonne |
| ADD CONSTRAINT | Entrées dans USER_CONSTRAINTS et USER_CONS_COLUMNS |
| ALTER TABLE ADD COLUMN | Nouvelle entrée dans USER_TAB_COLUMNS |
| ALTER TABLE MODIFY | Mise à jour de la colonne dans USER_TAB_COLUMNS (DATA_LENGTH, etc.) |
| ALTER TABLE DROP COLUMN | Suppression de la ligne dans USER_TAB_COLUMNS |
| ALTER TABLE RENAME COLUMN | Mise à jour du COLUMN_NAME dans USER_TAB_COLUMNS |

**Exemple : vérifier les modifications du TP1**

Nous pouvons tracer **exactement** l'ordre des opérations grâce à la colonne COLUMN_ID :
```sql
SELECT COLUMN_NAME, COLUMN_ID, DATA_TYPE
FROM USER_TAB_COLUMNS
WHERE TABLE_NAME = 'ETUDIANT'
ORDER BY COLUMN_ID;
```

**Résultat :**
```
COLUMN_NAME        COLUMN_ID  DATA_TYPE
-----------        ---------  ---------
MATETUD            1          NUMBER
NOM                2          VARCHAR2
PRENOM             3          VARCHAR2
BIRTHDATE          4          DATE
CODESECTION        5          VARCHAR2
DATEINSCRIPTION    6          DATE
```

**Explication :**  

Nous voyons que **chaque opération DDL du TP1 est enregistrée** dans le dictionnaire :
- Quand nous avons renommé DATENAISS → BIRTHDATE, le dictionnaire a mis à jour COLUMN_NAME
- Les colonnes ajoutées tardivement ont un COLUMN_ID plus élevé
- Le dictionnaire est un **journal d'audit de tous les changements structuraux**

---

## ANALYSE SYNTHÉTIQUE ET CONCLUSIONS

### Qu'avons-nous appris ?

**Nous avons exploré cinq concepts clés :**

1. **Structure du dictionnaire**  
   Nous comprenons que Oracle gère une **méta-base** complète, accessible via des vues standards.

2. **Hiérarchie des vues (USER_/ALL_/DBA_)**  
   Nous voyons que les privilèges structurent la **vision des métadonnées**. USER_ pour soi-même, ALL_ pour les droits d'accès, DBA_ pour l'administration.

3. **Interrogation des métadonnées**  
   Nous maîtrisons les vues clés : USER_TABLES, USER_TAB_COLUMNS, USER_CONSTRAINTS, USER_CONS_COLUMNS, ALL_TABLES, DBA_USERS.

4. **Audit et traçabilité**  
   Nous réalisons que le dictionnaire capture **tous les changements structuraux** et les **privilèges accordés**, offrant une traçabilité complète.

5. **Administration autonome**  
   Nous pouvons maintenant **nous administrer nous-mêmes** : vérifier nos tables, documenter nos schémas, retrouver nos objets, auditer les privilèges.

### Recommandations pratiques

**Pour interroger efficacement le dictionnaire :**

```sql
-- Voir mes tables
SELECT * FROM USER_TABLES;

-- Voir mes colonnes avec leurs types
SELECT TABLE_NAME, COLUMN_NAME, DATA_TYPE, NULLABLE
FROM USER_TAB_COLUMNS
ORDER BY TABLE_NAME, COLUMN_ID;

-- Voir mes contraintes
SELECT CONSTRAINT_NAME, CONSTRAINT_TYPE, TABLE_NAME
FROM USER_CONSTRAINTS;

-- Voir les clés étrangères
SELECT CONSTRAINT_NAME, TABLE_NAME, R_TABLE_NAME
FROM USER_CONSTRAINTS
WHERE CONSTRAINT_TYPE = 'R';

-- Voir les privilèges accordés à d'autres
SELECT * FROM USER_TAB_PRIVS;

-- Voir la taille de mes tables
SELECT SEGMENT_NAME, BYTES / 1024 / 1024 AS SIZE_MB
FROM USER_SEGMENTS
WHERE SEGMENT_TYPE = 'TABLE';

-- Exporter le DDL d'une table
SELECT DBMS_METADATA.GET_DDL('TABLE', 'ETUDIANT') FROM DUAL;
```

### Points essentiels à retenir

1. **Le dictionnaire est une arme à double tranchant :**  
   Nous pouvons l'interroger librement, mais nous ne pouvons pas le modifier directement (seul Oracle le fait automatiquement).

2. **USER_ vs ALL_ vs DBA_ :**  
   Nous utilisons USER_ pour nos objets, ALL_ pour vérifier les accès partagés, DBA_ si nous avons les droits d'admin.

3. **Chaque opération DDL laisse une trace :**  
   Nous pouvons auditer qui a créé quoi, quand, et comment.

4. **Les vues du dictionnaire sont normalisées :**  
   Peu importe la version d'Oracle, les structures USER_TABLES, USER_CONSTRAINTS, etc., restent stables.

5. **DBMS_METADATA.GET_DDL() est puissant :**  
   Nous pouvons exporter/importer des définitions d'objets, migrer entre bases, ou générer une documentation.

---

**Fin du Rapport TP 3**

*Le Dictionnaire de Données Oracle – USTHB – ASGBD 2025/2026*

---

## APPENDICE : Requêtes fréquemment utilisées

```sql
-- Lister toutes mes vues du dictionnaire disponibles
SELECT * FROM DICT WHERE ROWNUM <= 50;

-- Voir combien de tables je possède
SELECT COUNT(*) FROM USER_TABLES;

-- Voir les noms des colonnes d'une table
SELECT COLUMN_NAME FROM USER_TAB_COLUMNS 
WHERE TABLE_NAME = 'ETUDIANT';

-- Vérifier les contraintes d'une table
SELECT CONSTRAINT_NAME, CONSTRAINT_TYPE 
FROM USER_CONSTRAINTS 
WHERE TABLE_NAME = 'ETUDIANT';

-- Voir mes privilèges sur d'autres tables
SELECT TABLE_NAME, PRIVILEGE 
FROM USER_TAB_PRIVS 
WHERE GRANTEE = USER;

-- Voir les rôles que j'ai
SELECT GRANTED_ROLE FROM USER_ROLE_PRIVS;

-- Voir mes quotas de tablespace
SELECT TABLESPACE_NAME, QUOTA 
FROM USER_TS_QUOTAS;

-- Voir mes database links
SELECT DB_LINK FROM USER_DB_LINKS;

-- Voir mes séquences
SELECT SEQUENCE_NAME FROM USER_SEQUENCES;

-- Voir mes synonymes
SELECT SYNONYM_NAME, TABLE_NAME 
FROM USER_SYNONYMS;
```