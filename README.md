## Travail réalisée par AITLBIZ Kaoutar 
## ENS Marrakech



## TP – Entrepôt de données pour la gestion des inscriptions des étudiants

**1. Contexte général**

**L’université dispose d’un SI opérationnel depuis plus de dix ans pour gérer :**

- les étudiants (informations personnelles et statut administratif),

- les filières et cursus (Licence, Master, Doctorat),

- les inscriptions (annuelles/semestrielles, statut, frais).

- Ces données sont réparties sur plusieurs bases relationnelles et quelques fichiers Excel, ce qui provoque :

- redondances et incohérences,

- difficulté à produire des rapports consolidés,

- absence d’outil d’analyse multidimensionnelle.

L’objectif du TP est de mettre en place un entrepôt de données en étoile alimenté par un flux ETL Talend.

**2. Modèle décisionnel cible**

**AFFICHAGE**


<img width="315" height="607" alt="image" src="https://github.com/user-attachments/assets/913899d0-2880-421c-948f-4b7ee864ea73" />


<img width="307" height="612" alt="image" src="https://github.com/user-attachments/assets/d9fd1391-dcef-4be9-9e19-17c9dd8dc7cb" />


2.1. Axes d’analyse (dimensions)

Les principaux axes d’analyse retenus sont :

- Étudiant


```text
Nom, prénom, date de naissance, sexe, adresse, ville, région…



```


- Filière


```text
Code filière, nom, niveau (Licence/Master/Doctorat),

Département / faculté.
```


- Temps


```text
Date d’inscription, année universitaire, semestre,

Mois, trimestre, jour.
```


2.2. Tables de dimensions



- DIM_ETUDIANT


```text
id_etudiant (clé technique, surrogate key)

num_etudiant_source

nom, prenom, date_naissance, sexe

adresse, ville, region

statut_administratif

Attributs de suivi (date de début/fin de validité, etc., si besoin)
```



- DIM_FILIERE



```text
id_filiere

code_filiere_source

nom_filiere

niveau (Licence/Master/Doctorat)

departement
```



- DIM_TEMPS


```text
id_temps

date_calendrier

annee

mois

trimestre

semestre

jour_semaine
```

**AFFICHAGE**


<img width="362" height="538" alt="image" src="https://github.com/user-attachments/assets/4e04df98-25cb-4eac-a472-af28e2f16b6c" />



2.3. Table de faits


```text
F_FACT_INSCRIPTIONS (dans le TP la table fact est nommée tp1)

id_fact (optionnel – clé technique)

id_etudiant (FK → DIM_ETUDIANT)

id_filiere (FK → DIM_FILIERE)

id_temps (FK → DIM_TEMPS)

Attributs de l’inscription :

statut_inscription (nouvel inscrit, redoublant…)

type_inscription / année universitaire

montant_frais (mesure principale)
```



<img width="1187" height="544" alt="image" src="https://github.com/user-attachments/assets/60ebfaa1-fdc3-4b26-86de-1d7847eecda1" />





3. Préparation des sources dans Talend
3.1. Métadonnées Excel



Dans le Repository → Fichier Excel :

Créer les connexions pour chaque fichier source :

- Etudiants 0.1

- filieres 0.1

- inscriptions 0.1

- Définir pour chaque fichier :

- chemin du fichier,

- feuille utilisée,

- ligne d’en-tête,

- type de chaque colonne (String, Date, Integer, etc.).

3.2. Connexions aux bases de données

- Dans le Repository → Connexions aux bases de données :

- Créer les connexions MySQL suivantes (d’après les captures) :


```text
etudiant 0.1 → base cible pour DIM_ETUDIANT,

filieres 0.1 → base cible pour DIM_FILIERE,

dim_temps_etudiant 0.1 → base pour DIM_TEMPS,

inscriptions 0.1 → table de staging ou base transactionnelle,

tp1 0.1 → base du Data Warehouse (table de faits tp1).
```

- Renseigner pour chacune : URL, base, utilisateur, mot de passe, puis tester la connexion.

4. Chaîne ETL dans Talend (Job Etudiants 0.1)
4.1. Vue d’ensemble

**Le job Etudiants 0.1 se compose de plusieurs flux :**

- Chargement de la dimension Étudiant.

- Chargement de la dimension Filière.

- Chargement de la dimension Temps.

- Chargement de la table Inscriptions (staging).

- Construction de la table de faits tp1 via des jointures (tMap avec lookups).

**Les composants principaux utilisés sont :**

- tFileInputExcel (icône XLS) pour lire les fichiers,

- tMap pour les transformations et jointures,

- tMysqlOutput pour charger les tables MySQL.



**AFFICHAGE**


<img width="1495" height="679" alt="image" src="https://github.com/user-attachments/assets/69685901-a9ee-4aae-a87a-fca816bb931b" />


4.2. Étape 1 – Chargement de DIM_ETUDIANT

- Flux : Etudiants (XLS) → tMap_2 → tMysqlOutput_2

- tFileInputExcel / Etudiants

- Utilise la métadonnée Etudiants 0.1.

- Lit toutes les lignes d’étudiants.

- tMap_2 – Transformation

**Sélection et renommage des colonnes :**

- Num_Etu → num_etudiant_source,

- Nom, Prenom, etc.

**Nettoyage :**

- StringHandling.TRIM() sur les champs texte,


 **Préparation de la clé technique :**

- générant un identifiant dans Talend (tRowGenerator ou routine).

- tMysqlOutput_2 – Insertion dans MySQL

- Connexion : etudiant 0.1.

- Table cible : DIM_ETUDIANT.

- Action sur la table : Drop and create ou Insert selon le scénario.

- Gestion des doublons éventuels (clé unique sur num_etudiant_source).

4.3. Étape 2 – Chargement de DIM_FILIERE

- Flux : filieres (XLS) → tMap_3 → tMysqlOutput_3

- tFileInputExcel / filieres

- Utilise la métadonnée filieres 0.1.

- tMap_3 – Transformation

**Mapping des colonnes :**

- Code_Fil → code_filiere_source,

- Nom_Filiere, Niveau, Departement.

- Normalisation des niveaux (ex. “L” → “Licence”).

- tMysqlOutput_3 – Insertion dans DIM_FILIERE

- Connexion : filieres 0.1.

- Table : DIM_FILIERE.

- Option de clé technique id_filiere gérée côté base (auto-increment).

4.4. Étape 3 – Chargement de DIM_TEMPS

- Flux : source (date) → tMap_5 → tMysqlOutput_5

**Source de dates**

- générées à partir des dates d’inscription,


- tMap_5 – Dérivation des attributs temporels

**Pour chaque date_calendrier :**

- calcul de annee (TalendDate.getPartOfDate("YEAR", date)),

- mois (MONTH),



- semestre (1 ou 2),

- jour_semaine.

- tMysqlOutput_5 – Insertion dans DIM_TEMPS

- Connexion : dim_temps_etudiant 0.1.

- Table : DIM_TEMPS.



4.5. Étape 4 – Chargement de la table INSCRIPTIONS (staging)

- Flux : inscriptions (XLS) → tMap_1 → tMysqlOutput_1

- tFileInputExcel / inscriptions

- Utilise la métadonnée inscriptions 0.1.

- tMap_1 – Nettoyage et préparation

- Conversion des dates d’inscription (String → Date).

- Normalisation des valeurs de statut_inscription (N, R, D → valeurs lisibles).

- Contrôle des montants (BigDecimal ou Double, valeurs négatives mises à 0).

- tMysqlOutput_1 – Table de staging

- Connexion : inscriptions 0.1.

- Table : INSCRIPTIONS_STAGE (ou équivalent).

- Cette table servira de base principale pour construire la table de faits.

4.6. Étape 5 – Construction de la table de faits TP1


- Les flux issus de tMysqlOutput_2 (étudiants) et tMysqlOutput_3 (filières) alimentent tMap_4 en tant que lookups.

**Clés de jointure :**

**affichage**


<img width="1919" height="1017" alt="image" src="https://github.com/user-attachments/assets/a5fb996f-8e56-4625-a1d9-8e17a60a2ccf" />


------------------------------------



<img width="1158" height="665" alt="image" src="https://github.com/user-attachments/assets/7927aaba-c7e6-49b1-bb57-1f038cce09bb" />



- INSCRIPTIONS_STAGE.num_etudiant_source = DIM_ETUDIANT.num_etudiant_source,

- INSCRIPTIONS_STAGE.code_filiere_source = DIM_FILIERE.code_filiere_source.

- tMap_4 – Assemblage des clés et mesures





**Détermination de id_temps :**

On joignant avec DIM_TEMPS via la date,


- Connexion : tp1 0.1.


- Clefs étrangères (id_etudiant, id_filiere, id_temps) pointent vers les dimensions.



**AFFICHAGE**

<img width="1919" height="424" alt="image" src="https://github.com/user-attachments/assets/600619f0-05ed-4fb1-8ace-f9aa7a6f6ed9" />


**6. Exécution et validation**

- Lancer le job Etudiants 0.1.

- Vérifier dans la console Talend l’absence d’erreurs.





**AFFICHAGE**




<img width="1478" height="826" alt="image" src="https://github.com/user-attachments/assets/6f2d4bd0-1d05-404b-ade7-6dba99cf0db8" />



------------------




<img width="591" height="339" alt="image" src="https://github.com/user-attachments/assets/52955885-0173-42f6-b1db-4a9bdad137df" />




-------------------------





<img width="749" height="671" alt="image" src="https://github.com/user-attachments/assets/d8cda241-86f1-4ec8-a1d6-61fefe146310" />




---------------------------








**Contrôler dans MySQL :**

- nombre de lignes dans DIM_ETUDIANT, DIM_FILIERE, DIM_TEMPS,

- nombre de lignes dans la table tp1.

**Faire quelques requêtes de test, par exemple :**


```text
-- nombre d’inscriptions par filière et année
SELECT f.nom_filiere, t.annee, COUNT(*) nb_inscriptions, SUM(montant_frais) total_frais
FROM tp1 fact
JOIN DIM_FILIERE f  ON fact.id_filiere = f.id_filiere
JOIN DIM_TEMPS   t  ON fact.id_temps   = t.id_temps
GROUP BY f.nom_filiere, t.annee;
```

**7. Exploitation pour le reporting**

**L’entrepôt permet désormais :**

- des analyses par filière / niveau / département,


 
 
