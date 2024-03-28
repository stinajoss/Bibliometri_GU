# Bibliometri_GU
Kod för att ta ut data (från bibmet) och göra bibliometriska analyser av GU:s publikationsdata

# SQL-exempel query för att ta ut underlag till bibmetdata_to_bibtex.ipynb
--för att skapa publikationslista baserat på pe.id--
SELECT subquery.id,
       STRING_AGG(subquery.author_name, '; ') as author,
       subquery.pubyear, subquery.title, subquery.sourcetitle, subquery.sourcevolume, subquery.sourceissue, subquery.sourcepages, subquery.identifier_value AS doi
FROM (
    SELECT p.id, 
           pe.last_name || ', ' || SUBSTRING(pe.first_name FROM 1 FOR 1) as author_name,
           pv.pubyear, pv.title, pv.sourcetitle, pv.sourcevolume, pv.sourceissue, pv.sourcepages, pi.identifier_value
    FROM publications p
    JOIN publication_versions pv                ON pv.id=p.current_version_id
    JOIN people2publications p2p             ON p2p.publication_version_id=pv.id
    JOIN publication_types pt ON pt.id=pv.publication_type_id
    JOIN people pe ON pe.id=p2p.person_id
    LEFT OUTER JOIN publication_identifiers pi 		ON pi.publication_version_id = pv.id
    WHERE p.deleted_at IS NULL
    AND p.published_at IS NOT NULL
    AND (p.process_state NOT IN ('DRAFT', 'PREDRAFT') OR p.process_state IS NULL)
    AND pv.pubyear BETWEEN 2023 AND 2023
    AND EXISTS (
        SELECT 1 FROM people2publications p2p2
        WHERE p2p2.publication_version_id = pv.id
        AND p2p2.person_id IN (30, 61853, 61854, 78047, 78058, 80431, 80431, 80677, 80887, 81328, 82032, 82314, 83569, 83710, 83814, 83919, 84323, 84871, 86483, 100222, 100707, 101823, 112097, 112098, 113272, 118033, 123151, 140728, 154859, 175318, 181226, 184682, 211250, 244864, 273693, 397537, 421673, 671927, 685219, 721932, 758976, 785038, 787362, 806028, 816170, 838859, 865711, 874345, 875360, 877278, 886539, 886540, 900284, 929542, 930747, 938870, 950856, 951944, 957583, 957583, 962718, 992487, 113311, 1139791, 1142744, 1167427, 190176, 1173894, 61856, 98606, 842049, 217254)
    )
    AND pi.identifier_code='doi'
) subquery
WHERE subquery.author_name IS NOT NULL
GROUP BY subquery.id, subquery.pubyear, subquery.title, subquery.sourcetitle, subquery.sourcevolume, subquery.sourceissue, subquery.sourcepages, subquery.identifier_value
