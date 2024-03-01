# REQUETES ET REPONSES LAPLACE IMMO

## 1- nombre total d'appartements vendus au 1er semestre 2020 ?

```
SELECT COUNT(*) AS nombre_appartements_vendus
FROM VENTES v
INNER JOIN BIENS b
ON v.biens_id = b.id_biens
WHERE YEAR(v.date_vente) = 2020
AND v.date_vente BETWEEN '2020-01-01' AND '2020-06-30'
AND b.Type_local = 'Appartement';

```

_/ réponse : 31 378 appartements /_

## 2- nombre de ventes d'appartements par région pour le 1er semestre 2020

```
SELECT r.reg_nom,
       COUNT(*) AS nombre_ventes_appartements
FROM VENTES v
JOIN BIENS b ON v.biens_id = b.id_biens
JOIN COMMUNES c ON b.code_departement = c.code_departement
                                  AND b.code_commune = c.code_commune
JOIN REGIONS r ON c.code_region = r.reg_code
WHERE YEAR(v.date_vente) = 2020
AND MONTH(v.date_vente) BETWEEN 1 AND 6
AND b.Type_local = 'Appartement'
GROUP BY r.reg_nom;
```

| reg_nom                    | nombre_ventes_appartements |
| -------------------------- | -------------------------- |
| Ile-de-France              | 13995                      |
| Provence-Alpes-Côte d'Azur | 3649                       |
| Auvergne-Rhône-Alpes       | 3253                       |
| Nouvelle-Aquitaine         | 1932                       |
| Occitanie                  | 1640                       |
| Pays de la Loire           | 1357                       |
| Hauts-de-France            | 1254                       |
| Grand Est                  | 984                        |
| Bretagne                   | 983                        |
| Normandie                  | 862                        |
| Centre-Val de Loire        | 696                        |
| Bourgogne-Franche-Comté    | 376                        |
| Corse                      | 223                        |
| Martinique                 | 94                         |
| La Réunion                 | 44                         |
| Guyane                     | 34                         |
| Guadeloupe                 | 2                          |

## 3- proportion des ventes d'appartements par le nombre de pièces

```
SELECT b.Nombre_pieces,
       COUNT(*) AS nombre_ventes_appartements,
       CONCAT(
           FORMAT(
               (COUNT(*) / (SELECT COUNT(*) FROM VENTES v
                            INNER JOIN BIENS b ON v.biens_id = b.id_biens
                            WHERE b.Type_local = 'Appartement'
                              AND YEAR(v.date_vente) = 2020
                              AND MONTH(v.date_vente) <= 6)) * 100, -- Calcul du ratio
               2), -- Nombre de décimales après la virgule
           '%') AS ratio_ventes_appartements -- Ajout du symbole "%"
FROM VENTES v
INNER JOIN BIENS b ON v.biens_id = b.id_biens
WHERE b.Type_local = 'Appartement'
  AND YEAR(v.date_vente) = 2020
  AND MONTH(v.date_vente) <= 6
GROUP BY b.Nombre_pieces;

```

réponse :
| Nombre_pieces | nombre_ventes_appartements | ratio_ventes_appartements |
| ------------- | -------------------------- | ------------------------- |
| 3 | 8966 | 28.57% |
| 1 | 6739 | 21.48% |
| 6 | 204 | 0.65% |
| 2 | 9783 | 31.18% |
| 4 | 4460 | 14.21% |
| 5 | 1114 | 3.55% |
| 8 | 17 | 0.05% |
| 7 | 54 | 0.17% |
| 0 | 30 | 0.10% |
| 9 | 8 | 0.03% |
| 10 | 2 | 0.01% |
| 11 | 1 | 0.00% |

## 4- Liste des 10 départements où le prix du mètre carré est le plus élevé

```
SELECT c.code_departement,
       ROUND(AVG(v.prix / b.Surface_Carrez), 2) AS prix_m2_moyen
FROM VENTES v
INNER JOIN BIENS b ON v.biens_id = b.id_biens
INNER JOIN COMMUNES c ON b.code_departement = c.code_departement
AND b.code_commune = c.code_commune
WHERE b.Type_local = 'Appartement'
  AND YEAR(v.date_vente) = 2020
  AND MONTH(v.date_vente) <= 6
GROUP BY c.code_departement
ORDER BY prix_m2_moyen DESC
LIMIT 10;

```

Réponse :
| code_departement | prix_m2_moyen |
| ---------------- | ------------- |
| 75               | 12 083,32 €   |
| 92               | 7 277,08 €    |
| 94               | 5 515,70 €    |
| 74               | 4 772,08 €    |
| 6                | 4 743,69 €    |
| 93               | 4 409,90 €    |
| 78               | 4 314,01 €    |
| 69               | 4 125,31 €    |
| 2A               | 3 935,64 €    |
| 33               | 3 833,45 €    |

## 5- Prix moyen du mètre carré d'une maison en Île-de-France

```
SELECT AVG(v.prix / b.Surface_Carrez) AS prix_m2_maison
FROM VENTES v
INNER JOIN BIENS b ON v.biens_id = b.id_biens
INNER JOIN COMMUNES c ON b.code_departement = c.code_departement
AND b.code_commune = c.code_commune
WHERE b.Type_local = 'Maison'
  AND c.code_region = '11'
  AND YEAR(v.date_vente) = 2020
  AND MONTH(v.date_vente) <= 6;
```

réponse : 3 764,84 € du m2 en Île de France

## 6- Liste des 10 appartements les plus chers avec la région et le nombre de mètres carrés

```
SELECT b.id_biens,
       c.code_region,
       v.prix,
       r.reg_nom AS region,
       b.Surface_Carrez AS surface_carrez
FROM VENTES v
INNER JOIN BIENS b ON v.biens_id = b.id_biens
INNER JOIN COMMUNES c ON b.code_departement = c.code_departement
AND b.code_commune = c.code_commune
INNER JOIN REGIONS r ON c.code_region = r.reg_code
WHERE b.Type_local = 'Appartement'
  AND YEAR(v.date_vente) = 2020
  AND MONTH(v.date_vente) <= 6
ORDER BY v.prix DESC
LIMIT 10;


```

Réponse :
| id_biens | code_region | prix      | region                     | surface_carrez |
| -------- | ----------- | --------- | -------------------------- | -------------- |
| 7053     | 11          | 999 000 € | Ile-de-France              | 99,7           |
| 3224     | 11          | 99 938 €  | Ile-de-France              | 22,63          |
| 12423    | 11          | 99 900 €  | Ile-de-France              | 27,84          |
| 10323    | 11          | 99 900 €  | Ile-de-France              | 54,04          |
| 473      | 84          | 99 900 €  | Auvergne-Rhône-Alpes       | 56,02          |
| 15495    | 11          | 99 900 €  | Ile-de-France              | 19,4           |
| 3349     | 93          | 99 900 €  | Provence-Alpes-Côte d'Azur | 44,8           |
| 4596     | 28          | 99 900 €  | Normandie                  | 61,21          |
| 19038    | 84          | 99 900 €  | Auvergne-Rhône-Alpes       | 40,62          |
| 31245    | 28          | 99 900 €  | Normandie                  | 57,79          |

## 7- taux d'évolution du nombre de ventes entre le premier et le second trimestre de 2020

```
WITH
vente1 AS (
 SELECT round(count(id_vente),2) AS nbventes1
 FROM VENTES
 WHERE date_vente BETWEEN "2020/01/01" AND "2020/03/31"),
vente2 AS (
 SELECT round(count(id_vente),2) AS nbventes2
 FROM VENTES
 WHERE date_vente BETWEEN "2020/04/01" AND "2020/06/30")
SELECT round(((nbventes2 - nbventes1) / nbventes1 * 100), 2) AS "Taux d'évolution"
FROM vente1, vente2;

```

Réponse
3,68%

## 8- classement des régions par rapport au prix au mètre carré des appartements de plus de 4 pièces

```
SELECT c.code_region, r.reg_nom,
       ROUND(AVG(v.prix / b.Surface_Carrez), 2) AS prix_m2_appartement_4_pieces_arrondi
FROM VENTES v
INNER JOIN BIENS b ON v.biens_id = b.id_biens
INNER JOIN COMMUNES c ON b.Code_departement = c.code_departement
AND b.Code_commune = c.code_commune
INNER JOIN REGIONS r ON c.code_region = r.reg_code
WHERE b.Type_local = 'Appartement'
  AND b.Nombre_pieces > 4
  AND YEAR(v.date_vente) = 2020
  AND MONTH(v.date_vente) <= 6
GROUP BY c.code_region, r.reg_nom
ORDER BY prix_m2_appartement_4_pieces_arrondi DESC;

```

Réponse
| code_region | reg_nom                    | prix_m2_appartement_4_pieces_arrondi |
| ----------- | -------------------------- | ------------------------------------ |
| 11          | Ile-de-France              | 8 806,67 €                           |
| 4           | La Réunion                 | 3 659,83 €                           |
| 93          | Provence-Alpes-Côte d'Azur | 3 616,71 €                           |
| 94          | Corse                      | 3 117,88 €                           |
| 84          | Auvergne-Rhône-Alpes       | 2 903,86 €                           |
| 75          | Nouvelle-Aquitaine         | 2 476,50 €                           |
| 53          | Bretagne                   | 2 427,14 €                           |
| 52          | Pays de la Loire           | 2 329,21 €                           |
| 32          | Hauts-de-France            | 2 199,92 €                           |
| 76          | Occitanie                  | 2 107,24 €                           |
| 28          | Normandie                  | 2 026,31 €                           |
| 44          | Grand Est                  | 1 560,91 €                           |
| 24          | Centre-Val de Loire        | 1 459,98 €                           |
| 27          | Bourgogne-Franche-Comté    | 1 260,73 €                           |
| 2           | Martinique                 | 574,77 €                             |

## 9- liste des communes ayant eu au moins 50 ventes au 1er trimestre

```
SELECT c.nom_commune,
       COUNT(*) AS nb_ventes
FROM VENTES v
INNER JOIN BIENS b ON v.biens_id = b.id_biens
INNER JOIN COMMUNES c ON b.code_departement = c.code_departement
AND b.code_commune = c.code_commune
WHERE YEAR(v.date_vente) = 2020
  AND MONTH(v.date_vente) <= 3
GROUP BY c.nom_commune
HAVING nb_ventes >= 50;
```

Réponse :
| nom_commune | nb_ventes |
| --------------------------- | --------- |
| Paris 17e Arrondissement | 228 |
| Nice | 173 |
| Bordeaux | 157 |
| Paris 20e Arrondissement | 127 |
| Nantes | 119 |
| Paris 12e Arrondissement | 110 |
| Boulogne-Billancourt | 99 |
| Paris 7e Arrondissement | 87 |
| Asnières-sur-Seine | 81 |
| Courbevoie | 80 |
| Toulouse | 78 |
| Antibes | 77 |
| Marseille 4e Arrondissement | 72 |
| Montreuil | 65 |
| Angers | 64 |
| Nîmes | 63 |
| Sète | 62 |
| Paris 8e Arrondissement | 62 |
| La Ciotat | 62 |
| Rennes | 61 |
| Paris 4e Arrondissement | 60 |
| Saint-Maur-des-Fossés | 56 |
| Ajaccio | 54 |
| Puteaux | 53 |
| Issy-les-Moulineaux | 50 |

## 10- différence en pourcentage du prix au mètre carré entre un appartement de 2 pièces et un appartement de 3 pièces

```
WITH prix_m2_2pieces AS (
    SELECT AVG(v.prix / b.Surface_Carrez) AS prix_m2
    FROM VENTES v
    INNER JOIN BIENS b ON v.biens_id = b.id_biens
    WHERE b.Type_local = 'Appartement'
      AND b.Nombre_pieces = 2
),
prix_m2_3pieces AS (
    SELECT AVG(v.prix / b.Surface_Carrez) AS prix_m2
    FROM VENTES v
    INNER JOIN BIENS b ON v.biens_id = b.id_biens
    WHERE b.Type_local = 'Appartement'
      AND b.Nombre_pieces = 3
)
SELECT ((pm3.prix_m2 - pm2.prix_m2) / pm2.prix_m2) * 100 AS difference_pourcentage
FROM prix_m2_2pieces pm2, prix_m2_3pieces pm3;

```

Reponse :
| difference_pourcentage |
| ---------------------- |
| \-12,68 % |

## 11- Les moyennes de valeurs foncieres pour le top 3 des communes des départements 6,14,33,59,69

```
SELECT nom_commune, code_departement, moyenne_valeur_fonciere
FROM (
    SELECT c.nom_commune,
           c.code_departement,
           AVG(v.prix) AS moyenne_valeur_fonciere,
           ROW_NUMBER() OVER (PARTITION BY c.code_departement ORDER BY AVG(v.prix) DESC) AS row_num
    FROM VENTES v
    INNER JOIN BIENS b ON v.biens_id = b.id_biens
    INNER JOIN COMMUNES c ON b.code_departement = c.code_departement
                           AND b.code_commune = c.code_commune
    WHERE c.code_departement IN ('06', '14', '33', '59', '69')
      AND YEAR(v.date_vente) = 2020
      AND MONTH(v.date_vente) <= 6
    GROUP BY c.code_departement, c.nom_commune
) AS ranked_communes
WHERE row_num <= 3
ORDER BY code_departement, moyenne_valeur_fonciere DESC;
```

Réponse :
| nom_commune            | code_departement | moyenne_valeur_fonciere |
| ---------------------- | ---------------- | ----------------------- |
| Saint-Jean-Cap-Ferrat  | 06               | 968 750,00 €            |
| Eze                    | 06               | 655 000,00 €            |
| Mouans-Sartoux         | 06               | 476 898,00 €            |
| Deauville              | 14               | 251 585,64 €            |
| Benerville-sur-Mer     | 14               | 177 500,00 €            |
| Tourgéville            | 14               | 169 850,00 €            |
| Lège-Cap-Ferret        | 33               | 549 500,64 €            |
| Vayres                 | 33               | 335 000,00 €            |
| Arcachon               | 33               | 307 435,93 €            |
| Bersée                 | 59               | 433 202,00 €            |
| Cysoing                | 59               | 408 550,00 €            |
| Halluin                | 59               | 322 250,00 €            |
| Ville-sur-Jarnioux     | 69               | 485 300,00 €            |
| Lyon 2e Arrondissement | 69               | 455 217,26 €            |
| Lyon 6e Arrondissement | 69               | 426 968,25 €            |

## 12- Les 20 communes avec le plus de transactions pour 1000 habitants pour les communes qui dépassent les 10000 habitants

```
SELECT c.nom_commune,
       COUNT(*) / (MAX(c.population) / 1000) AS transactions_pour_1000_habitants --MAX sert à résoudre l'erreur 'only_full_group_by sans modifier le résultat de la requête
       cette erreur est par défaut dans MySQL et nécessite que chaque colonne dans la clause SELECT qui doit être groupée (GROUP BY) soit une fonction d'agrégation (comme COUNT, SUM, etc)
FROM VENTES v
INNER JOIN BIENS b ON v.biens_id = b.id_biens
INNER JOIN COMMUNES c ON b.code_departement = c.code_departement
AND b.code_commune = c.code_commune
WHERE c.population > 10000
GROUP BY c.nom_commune
ORDER BY transactions_pour_1000_habitants DESC
LIMIT 20;
```

Réponse :
| nom_commune | transactions_pour_1000_habitants |
| ------------------------ | -------------------------------- |
| Paris 2e Arrondissement | 5,84 |
| Paris 1er Arrondissement | 4,92 |
| Paris 3e Arrondissement | 4,69 |
| Arcachon | 4,62 |
| La Baule-Escoublac | 4,58 |
| Paris 4e Arrondissement | 4,08 |
| Roquebrune-Cap-Martin | 3,99 |
| Paris 8e Arrondissement | 3,83 |
| Sanary-sur-Mer | 3,50 |
| Paris 9e Arrondissement | 3,43 |
| La Londe-les-Maures | 3,43 |
| Paris 6e Arrondissement | 3,38 |
| Saint-Cyr-sur-Mer | 3,24 |
| Chantilly | 3,13 |
| Pornichet | 3,06 |
| Saint-Mandé | 3,06 |
| Paris 10e Arrondissement | 3,04 |
| Menton | 2,94 |
| Saint-Hilaire-de-Riez | 2,87 |
| Vincennes | 2,81 |
