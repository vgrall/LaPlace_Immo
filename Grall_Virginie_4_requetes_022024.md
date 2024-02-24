# REQUETES ET REPONSES LAPLACE IMMO

## nombre total d'appartements vendus au 1er semestre 2020 ?

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

## nombre de ventes d'appartements par région pour le 1er semestre 2020

```
SELECT r.reg_nom,
       COUNT(*) AS nombre_ventes_appartements
FROM VENTES v
JOIN BIENS b ON v.biens_id = b.id_biens
JOIN COMMUNES c ON b.code_departement = c.code_departement
                                  AND b.code_commune = c.code_commune
JOIN pREGIONS r ON c.code_region = r.reg_code
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

## proportion des ventes d'appartements par le nombre de pièces

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

## Liste des 10 appartements où le prix du mètre carré est le plus élevé

```
SELECT b.id_biens,
       b.Surface_reelle,
       v.prix,
       v.prix / b.Surface_reelle AS prix_m2
FROM VENTES v
INNER JOIN BIENS b ON v.biens_id = b.id_biens
WHERE b.Type_local = 'Appartement'
  AND YEAR(v.date_vente) = 2020
  AND MONTH(v.date_vente) <= 6
ORDER BY prix_m2 DESC
LIMIT 10;
```

Réponse :
| id_biens | Surface_reelle | prix | prix_m2 |
| -------- | -------------- | -------------- | ------------ |
| 30603 | 10 | 9 000 000,00 € | 900 000,00 € |
| 28105 | 5 | 1 290 000,00 € | 258 000,00 € |
| 28556 | 12 | 2 703 385,00 € | 225 282,08 € |
| 5552 | 13 | 2 634 000,00 € | 202 615,38 € |
| 7556 | 42 | 7 620 000,00 € | 181 428,57 € |
| 30403 | 1 | 175 000,00 € | 175 000,00 € |
| 3066 | 8 | 1 298 500,00 € | 162 312,50 € |
| 5212 | 62 | 8 600 000,00 € | 138 709,68 € |
| 33529 | 19 | 2 200 000,00 € | 115 789,47 € |
| 30287 | 17 | 1 900 000,00 € | 111 764,71 € |

## Prix moyen du mètre carré d'une maison en Île-de-France

```
SELECT AVG(v.prix / b.Surface_reelle) AS prix_m2_maison
FROM VENTES v
INNER JOIN BIENS b ON v.biens_id = b.id_biens
INNER JOIN COMMUNES c ON b.code_departement = c.code_departement
AND b.code_commune = c.code_commune
WHERE b.Type_local = 'Maison'
  AND c.code_region = '11'
  AND YEAR(v.date_vente) = 2020
  AND MONTH(v.date_vente) <= 6;
```

réponse : 3 997,71 € du m2 en Île de France

## Liste des 10 appartements les plus chers avec la région et le nombre de mètres carrés

```
SELECT b.id_biens,
       c.code_region,
       b.Surface_reelle,
       v.prix
FROM VENTES v
INNER JOIN BIENS b ON v.biens_id = b.id_biens
INNER JOIN COMMUNES c ON b.code_departement = c.code_departement
AND b.code_commune = c.code_commune
WHERE b.Type_local = 'Appartement'
  AND YEAR(v.date_vente) = 2020
  AND MONTH(v.date_vente) <= 6
ORDER BY v.prix DESC
LIMIT 10;
```

Réponse :
| id_biens | code_region | Surface_reelle | prix |
| -------- | ----------- | -------------- | ------------ |
| 7053 | 11 | 98 | 999 000,00 € |
| 3224 | 11 | 22 | 99 938,00 € |
| 473 | 84 | 57 | 99 900,00 € |
| 19038 | 84 | 40 | 99 900,00 € |
| 4596 | 28 | 53 | 99 900,00 € |
| 10323 | 11 | 53 | 99 900,00 € |
| 23893 | 52 | 74 | 99 900,00 € |
| 3349 | 93 | 44 | 99 900,00 € |
| 12423 | 11 | 28 | 99 900,00 € |
| 31245 | 28 | 58 | 99 900,00 € |

## taux d'évolution du nombre de ventes entre le premier et le second trimestre de 2020

```
SELECT  -- Sélection des colonnes
        nb_ventes_trimestre_1,
        nb_ventes_trimestre_2,
        ((nb_ventes_trimestre_2 - nb_ventes_trimestre_1) / nb_ventes_trimestre_1) * 100 AS taux_evol_nb_ventes -- Calcul du taux d'évolution
FROM (
        SELECT  -- Sélection des colonnes
                (SELECT COUNT(*) -- Calcul du nombre de ventes au cours du premier trimestre
                 FROM VENTES v1
                 WHERE YEAR(v1.date_vente) = 2020
                     AND MONTH(v1.date_vente) <= 3) AS nb_ventes_trimestre_1,
                (SELECT COUNT(*) -- Calcul du nombre de ventes au cours du deuxième trimestre
                 FROM VENTES v2
                 WHERE YEAR(v2.date_vente) = 2020
                     AND MONTH(v2.date_vente) > 3
                     AND MONTH(v2.date_vente) <= 6) AS nb_ventes_trimestre_2
) AS ventes_trimestre; -- Nom de la sous-requête


```

Réponse
| nb_ventes_trimestre_1 | nb_ventes_trimestre_2 | taux_evol_nb_ventes |
| --------------------- | --------------------- | ------------------- |
| 16776 | 17393 | 3.677 % |

## classement des régions par rapport au prix au mètre carré des appartements de plus de 4 pièces

```
SELECT c.code_region,
       AVG(v.prix / b.Surface_reelle) AS prix_m2_appartement_4_pieces
FROM VENTES v
INNER JOIN BIENS b ON v.biens_id = b.id_biens
INNER JOIN COMMUNES c ON b.Code_departement = c.code_departement
AND b.Code_commune = c.code_commune
WHERE b.Type_local = 'Appartement'
  AND b.Nombre_pieces > 4
  AND YEAR(v.date_vente) = 2020
  AND MONTH(v.date_vente) <= 6
GROUP BY c.code_region
ORDER BY prix_m2_appartement_4_pieces DESC;
```

Réponse
| code_region | prix_m2_appartement_4_pieces |
| ----------- | ---------------------------- |
| 11 | 8 003,39 € |
| 4 | 3 659,83 € |
| 94 | 3 046,47 € |
| 93 | 3 005,24 € |
| 84 | 2 768,87 € |
| 75 | 2 510,18 € |
| 53 | 2 271,86 € |
| 32 | 2 203,61 € |
| 52 | 2 186,72 € |
| 76 | 2 096,42 € |
| 28 | 1 994,25 € |
| 24 | 1 428,51 € |
| 44 | 1 313,26 € |
| 27 | 1 068,93 € |
| 2 | 564,22 € |

## liste des communes ayant eu au moins 50 ventes au 1er trimestre

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

## différence en pourcentage du prix au mètre carré entre un appartement de 2 pièces et un appartement de 3 pièces

```
SELECT
    ((prix_m2_3pieces - prix_m2_2pieces) / prix_m2_2pieces) * 100 AS difference_pourcentage
FROM (
    SELECT
        AVG(v.prix / b.Surface_reelle) AS prix_m2_2pieces
    FROM VENTES v
    INNER JOIN BIENS b ON v.biens_id = b.id_biens
    WHERE b.Type_local = 'Appartement'
      AND b.Nombre_pieces = 2
) AS prix_m2_2pieces,
(
    SELECT
        AVG(v.prix / b.Surface_reelle) AS prix_m2_3pieces
    FROM VENTES v
    INNER JOIN BIENS b ON v.biens_id = b.id_biens
    WHERE b.Type_local = 'Appartement'
      AND b.Nombre_pieces = 3
) AS prix_m2_3pieces;
```

Reponse :
| difference_pourcentage |
| ---------------------- |
| \-13,04 % |

## Les moyennes de valeurs foncieres pour le top 3 des communes des départements 6,14,33,59,69

```
SELECT c.nom_commune,
       AVG(v.prix) AS moyenne_valeur_fonciere
FROM VENTES v
INNER JOIN BIENS b ON v.biens_id = b.id_biens
INNER JOIN COMMUNES c ON b.code_departement = c.code_departement
AND b.code_commune = c.code_commune
WHERE c.code_departement IN ('06', '14', '33', '59', '69')
  AND YEAR(v.date_vente) = 2020
  AND MONTH(v.date_vente) <= 6
GROUP BY c.nom_commune
ORDER BY moyenne_valeur_fonciere DESC
LIMIT 3;
```

Réponse :
| nom_commune | moyenne_valeur_fonciere |
| --------------------- | ----------------------- |
| Saint-Jean-Cap-Ferrat | 968 750,00 € |
| Eze | 655 000,00 € |
| Lège-Cap-Ferret | 549 500,64 € |

## Les 20 communes avec le plus de transactions pour 1000 habitants pour les communes qui dépassent les 10000 habitants

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
