# SAE-S2.04
Exploitation d'une base de donnée.



___ Sujet Partie 1 ___
https://www-info.iut2.univ-grenoble-alpes.fr/intranet/enseignements/S2.04/BD.xhtml
______________________

2.1) Exploration

Requette temoin : openfoodfacts=> SELECT url FROM openfoodfacts WHERE product_name = 'jsp';


2.2) Extraction et nettoyage 

Enlever les erreur de data :  where code not in (select code where data_quality_errors_tags like '%%')


Créé table temp

create temp table openfoodclean as
select  code, url, product_name,
        brands_tags, stores, owner, food_groups, labels_tags, countries,
        countries_tags, quantity, fat_100g, saturated_fat_100g, 
        sugars_100g, proteins_100g, carbohydrates_100g, energy_100g,
        salt_100g, sodium_100g, nutriscore_score, nutriscore_grade from openfoodfacts
        where code not in (select code where data_quality_errors_tags like '%%');
