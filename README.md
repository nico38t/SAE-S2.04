# SAE-S2.04
Exploitation d'une base de donnée.



___ Sujet Partie 1 ___
https://www-info.iut2.univ-grenoble-alpes.fr/intranet/enseignements/S2.04/BD.xhtml
______________________

2.1) Exploration

Requette temoin : openfoodfacts=> SELECT url FROM openfoodfacts WHERE product_name = 'jsp';


2.2) Extraction et nettoyage 

Enlever les erreur de data :  where code no in (select code where data_quality_errors_tags like '%%')
