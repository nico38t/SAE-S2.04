/* where code not in (select code where data_quality_errors_tags like '%%') */


select countries FROM openfoodfacts where code not in (select code where data_quality_errors_tags like '%%') AND food_groups = 'en:one-dish-meals';

/*
dico

United States
États-Unis
Etats-Unis
united states
us
US
usa
USA
*/

-- Nous allons verifier que les etats unis ne sont pas decouper en differents etats 
-- avec des requetes de ce type :

SELECT count(*) FROM openfoodfacts WHERE countries_en LIKE '%colorado%';
SELECT count(*) FROM openfoodfacts WHERE countries_en LIKE '%texas%';
SELECT count(*) FROM openfoodfacts WHERE countries_en LIKE '%alabama%';
-- nous pouvons constater que les etats ne sont pas nommé separement.


--request tout les nom USA: 
select distinct(countries) from openfoodfacts where countries_en ='United States';




-- nous creons maintenant une vue appelée openfoodclean qui ne comporte pas les erreures
-- nous pouvons donc constater une difference de rasultat 

openfoodfacts=> SELECT count(*) FROM openfoodfacts WHERE countries_en ='us';
 count 
-------
 14865
(1 row)

openfoodfacts=> SELECT count(*) FROM openfoodclean WHERE countries_en ='us';
 count 
-------
 13588
(1 row)

--creer la base temporaire que nous allons modifier.
CREATE TEMP TABLE openfoodclean AS 
SELECT 
    code, 
    url, 
    product_name, 
    brands_tags, 
    stores, 
    owner, 
    food_groups, 
    labels_tags, 
    countries, 
    countries_tags, 
    quantity, 
    fat_100g, 
    saturated_fat_100g, 
    sugars_100g, 
    proteins_100g, 
    carbohydrates_100g, 
    energy_100g, 
    salt_100g, 
    sodium_100g, 
    nutriscore_score, 
    nutriscore_grade 
FROM 
    openfoodfacts 
WHERE 
    code NOT IN (SELECT code FROM openfoodfacts WHERE data_quality_errors_tags LIKE '%%') 
    AND code is not null
    AND char_length(code)=13
    AND nutriscore_grade is not null
    AND countries IN (SELECT DISTINCT(countries) FROM openfoodfacts WHERE countries_en = 'United States') 
    AND food_groups = 'en:one-dish-meals';

--

--source du calcule pour computed_energy = chat gpt
-- creation d'une table ne contenant que les information qui nous interesse
CREATE TEMP TABLE openfoodclean AS 
SELECT 
    code, 
    url, 
    product_name, 
    brands_tags, 
    stores, 
    owner, 
    food_groups, 
    labels_tags, 
    countries, 
    countries_tags, 
    quantity, 
    fat_100g, 
    saturated_fat_100g, 
    sugars_100g, 
    proteins_100g, 
    carbohydrates_100g, 
    energy_100g, 
    salt_100g, 
    sodium_100g, 
    nutriscore_score, 
    nutriscore_grade, 
    (proteins_100g*4)+(carbohydrates_100g*4)+(fat_100g*9) AS computed_energy_100g,
    CASE 
       WHEN labels_tags LIKE '%organic%' THEN TRUE
       WHEN labels_tags = '' THEN NULL
       ELSE FALSE
    END AS organic,
    CASE 
        WHEN ingredients_analysis_tags LIKE '%en:vegan%' THEN TRUE
        WHEN ingredients_analysis_tags = '' THEN NULL
        ELSE FALSE
    END AS vegan,  
    CASE
        WHEN ingredients_analysis_tags LIKE '%en:vegetarian%' THEN TRUE
        WHEN ingredients_analysis_tags = '' THEN NULL
    END AS vegetarian,
    CASE
        WHEN ingredients_analysis_tags LIKE '%en:palm_oil%' THEN TRUE
        WHEN ingredients_analysis_tags = '' THEN NULL
    END AS palm_oil,
    CASE
        WHEN nutrient_levels_tags ~* 'en:fat-in-high-quantity' THEN 'h'
        WHEN nutrient_levels_tags ~* 'en:fat-moderate-quantity' THEN 'm'
        WHEN nutrient_levels_tags ~* 'en:fat-low-quantity' THEN 'l'
        ELSE NULL
    END AS fat_levels,
    CASE
        WHEN nutrient_levels_tags ~* 'en:sugars-in-high-quantity' THEN 'h'
        WHEN nutrient_levels_tags ~* 'en:sugars-moderate-quantity' THEN 'm'
        WHEN nutrient_levels_tags ~* 'en:sugars-low-quantity' THEN 'l'
        ELSE NULL
    END AS sugars_levels,
    CASE
        WHEN nutrient_levels_tags ~* 'en:salt-in-high-quantity' THEN 'h'
        WHEN nutrient_levels_tags ~* 'en:salt-moderate-quantity' THEN 'm'
        WHEN nutrient_levels_tags ~* 'en:salt-low-quantity' THEN 'l'
        ELSE NULL
    END AS salt_levels

FROM 
    openfoodfacts
WHERE 
    code NOT IN (SELECT code FROM openfoodfacts WHERE data_quality_errors_tags LIKE '%%') 
    AND code is not null
    AND char_length(code)=13
    AND nutriscore_grade is not null
    AND fat_100g > 0 And fat_100g < 100
    AND saturated_fat_100g > 0 AND saturated_fat_100g < 100
    AND sugars_100g > 0 AND sugars_100g < 100
    AND proteins_100g > 0 AND proteins_100g < 100
    AND carbohydrates_100g > 0 AND carbohydrates_100g < 100
    AND salt_100g > 0 AND salt_100g < 100
    AND sodium_100g > 0 AND sodium_100g < 100
    AND countries IN (SELECT DISTINCT(countries) FROM openfoodfacts WHERE countries_en = 'United States') 
    AND food_groups = 'en:one-dish-meals';



-- pour trouver les aliments BIO
SELECT product_name,
       CASE 
        WHEN ingredients_analysis_tags LIKE '%en:vegan%' THEN TRUE
        WHEN ingredients_analysis_tags = '' THEN NULL
        ELSE FALSE
    END AS vegan
FROM openfoodclean;


/*
CREATE TEMP TABLE openfoodclean AS SELECT code, url, product_name, brands_tags, stores, owner, food_groups, labels_tags, countries, countries_tags, quantity, fat_100g, saturated_fat_100g, sugars_100g, proteins_100g, carbohydrates_100g, energy_100g, salt_100g, sodium_100g, nutriscore_score, nutriscore_grade FROM openfoodfacts WHERE code NOT IN (SELECT code FROM openfoodfacts WHERE data_quality_errors_tags LIKE '%%') and countries in 
(select distinct(countries) from openfoodfacts where countries_en ='United States') and food_groups = 'en:one-dish-meals';
*/
--

