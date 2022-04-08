# DVF Navigator

Visualisation des Demandes de Valeur Foncière (https://app.dvf.etalab.gouv.fr/) sous une forme dynamique.

[La visualisation est disponible ici.](https://FBranca.github.io/dvf-navigator/carte.html)

L'objectif est de présenter l'évolutions des ventes de maisons au fil du temps afin de suivre leur évolution et ainsi percevoir l'évolution des prix sur une zone géographique choisie. La visualisation se fait sur une carte et également de manière graphique (à venir). La carte permet de situer les parcelles vendues et voir si certaines zones sont plus dynamiques que d'autres. La vue graphique doit permettre de voir la correlation entre la superficie du bien vendu et son prix de vente.

Actuellement (avril 2022) seule la vue cartographique est implémentée et la visualisation est limité à la seule ville de Thouaré sur Loire.

Si vous êtes interessé, je vous invite à suivre le projet car je vais effectuer plusieurs évolutions dans les jours et semaines à venir.

## Genèse et motivation

Ce projet m'est venu à l'esprit après avoir déménagé en régions Nantaise ou nous avons commencé la recherche de notre nouvelle maison. Nous avons alors été confronté à un marché immobilier difficile à comprendre, les prix nous semblaient incohérents. Chose étrange, les prix des maisons dans Nantes intra-muros semblent avoir un prix similaire à celles des communes aux alentours, ce qui est plutôt déstabilisant lorsque l'on vient de la région parisienne où les différences de prix sont nettements marquées au fur et à mesure que l'on s'éloigne de Pairs. Pour nous aider, nous avons beaucoup utilisé les données DVF (https://fr.wikipedia.org/wiki/Demande_de_valeurs_fonci%C3%A8res) qui donnent accès à l'ensemble des transaction immobilières des dernières années. Ces données sont accessibles librement sous différentes formes et l'état propose un portail permettant de visualiser ceci sur la carte de france : https://app.dvf.etalab.gouv.fr/. Ce portail est une vraie mine d'or pour mieux cerner le marché immobilier d'une zone géographique. Il impose cependant d'examiner chaque parcelle individuellement et il est difficile de se faire une idée  générale rapidement. Les données DVF existent également dans des formes brutes qu'il est possible de réutiliser. Certains organismes fournissent ces données sous une forme simplifiée, c'est notament le cas du CEREMA qui produit le "DVF plus" (https://datafoncier.cerema.fr/donnees/autres-donnees-foncieres/dvfplus-open-data).

Il y a certainement matière à visualiser ce jeu de données sous de nouveaux angles. Cela tombe bien, j'ai un peu de temps libre en ce moment :) c'est une bonne occasion de parfaire mes connaissances sur les SIG et me remettre dans les sujets de visualisation de données, notament via l'utilisation de la librairie d3js qui excelle dans le rendu de graphiques dynamiques.


## Points structurants et limitations

Le projet est concu pour fonctionner entièrement dans un navigateur, en interrogeant les sources de données déjà existantes. Cela présentes plusieurs avantages :
* Pas de mise en place d'un serveur, ce qui permet de se concentrer sur le rendu et la visualisation des données.
* Permet l'hébergement du projet entièrement sur Github.
La contrepartie se situe au niveau des performances : sans back-end il n'y a pas de possibilité de retravailler et filter les données avant de les transmettre au navigateur pour les présenter. Il faut donc composer avec les sources de données publiques existantes en l'état.

Pour l'instant seule la ville de Thouaré sur Loire est couverte. C'est un corrolaire de l'absence de back-end : le jeu de données est obligatoirement statique, il faut le générer manuellement et pour le moment j'ai concentré mes efforts sur cette ville. Il est assez simple d'en ajouter de nouvelles : une simple extraction dans les fichiers CSV du "DVF plus" suffit. L'idée à terme est de trouver un moyen de récupérer automatiquement ces données afin d'élargir le projet à d'autres communes. 

Enfin le projet n'utilise pas, pour le moment, de module bundler (webpack ou autre), l'ensemble des ressources (styles, html et javascript) sont regroupés dans un fichier HTML. 

## Trys and errors

### Récupération des parcelles cadastrales

Un des sujets ayant donné du fil a retordre est la récupération des parcelles cadastrales. C'est en grande partie dû au fait que je n'avais qu'une connaissance limité des possibilités et protocoles proposés par les serveurs de cartographie.

Ces données doivent être récupérées sous forme vectorielle et assorties d'un identifiant sur chaque parcelle cadastrale permettant de faire le lien avec le DVF. Les flux de type raster (images) ont été exclus d'emblé.

J'ai commencé par tenter d'utiliser le protocole WFS (https://fr.wikipedia.org/wiki/Web_Feature_Service) qui me semblait correspondre parfaitement au besoin. Comme son nom l'indique il permet d'accéder aux données sous forme de "features". Sur le papier c'est exactement ce qu'il fallait : une API capable de fournir la liste des parcelles avec leurs identifiants (pour faire le lien avec le DVF) et leur geométrie (pour affichage sur la cartographie). Après une rapide recherche, il se trouve que les données des parcelles cadastrales sont accessible sur un serveur de l'IGN : https://wxs.ign.fr/parcellaire/geoportail/wfs?service=WFS.
Le souci de cette méthode est que le serveur doit effectuer une extraction de données qui n'est pas triviale et prend un temps important (plusieurs secondes). Les données ne sont donc pas disponibles immédiatement. Pire : le nombre de résultats renvoyés est limité à 1000 sur les serveur de l'IGN, probablement pour éviter une charge de travail trop élevée. Le volume de données retourné est également important : 800ko pour 1000 parcelles en utilisant le format JSON.
Bref, cette solution n'est clairement pas exploitable car lente et ne permettant de couvrir qu'une très faible zone géographique (pour rester dans la limite des 1000 parcelles).

Le second essai a été de récupérer les données des parcelles qui sont disponibles sous plusieurs formats et à plusieurs granularités sur le site https://cadastre.data.gouv.fr/data/etalab-cadastre. Pour commencer j'ai donc récupéré les données concernant les parcelles de Thouaré-sur-Loire en optant pour le format geojson car il est aisement compris par les différentes librairies de rendu cartographique.
L'affichage des parcelles en surimpression du fond cartographique fourni par Openstreetmap n'a posé aucun problème. Il a également été assez simple de retrouver une parcelle à partir de son identifiant, et donc pouvoir faire le lien avec les données DVF.
C'est donc un succès. Il est cependant à modérer par deux points : 
* Un fois décompressé, le fichier JSON contenant les parcelles pèse 3,5 Mo pour la seule commune de Thouaré-sur-Loire. Pour un travail en local sur un PC de développement ce n'est pas un souci, mais une fois en production c'est un volume de données non négligeable que le navigateur doit télécharger.
* Cette méthode ne fonctionne que sur une ville dont on a préalablement récupéré les données cadastrales. Pour que cela fonctionne il m'a fallu d'abord décompresser les données récupérées sur le site web du cadastre car la librairie openlayers n'accepte pas des données compressées. J'aurais aimé pouvoir référencer directement les données sur le site du cadastre, cela aurait permis d'accéder directement aux données des différentes villes, mais en l'état ce n'est pas simple à réaliser. Je n'ai trouvé que très peu de librairies permettant la décompression de données en javascript et n'ai pas investigué plus en avant.

La dernière voie m'a été inspiré en regardant le fonctionnement du site https://cadastre.data.gouv.fr/map dont les parcelles cadastrales peuvent être selectionnées en cliquant et pour lesquelles on peut consulter leurs données. Ce site utilise des tuiles vectorielles. Contrairement au protocole WFS, le serveur n'a ici aucun traitement à faire : il se contente de transmettre les tuiles qui correspondent à la région et au niveau de zoom actuellement affiché. Les performances sont donc très bonnes, d'autant plus que le protocole utilisé pour transferer les données est ici binaire et très compact. Il s'agit du format PBF, basé sur protocol buffers, qui permet d'obtenir des volumes de données bien plus faibles qu'un format textuel de type JSON ou XML.
Cette solution permet de lever les freins de l'essais précédent. Les performances sont excellentes, et le flux est accessible pour l'ensemble de la france directement depuis les serveurs en place.