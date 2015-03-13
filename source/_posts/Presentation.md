title: Présentation d'ezVIS
date: 2015-03-13 18:02:00
tags:
- présentation
- ezvis
- ezmaster
- vsst
---
# Introduction

ezVIS est le résultat d’une réflexion menée au sein de l’Inist-CNRS sur le besoin d’un outil de mise à disposition et d’exploration de corpus en remplacement d’outil développé précédemment comme SERVIST. Dans le cadre du projet MADEC, le choix a été fait d’aborder l’exploration d’un corpus par sa description à travers un tableau de bord.  Cela a conduit au développement d’un outil de réalisation de tableau de bord décrivant le corpus et y donnant accès.

# ezVIS

L’Inist-CNRS propose à travers son service *Appui au pilotage* des études bibliométriques réalisées à partir de données structurées fournies par les usagers ou issues de bases de données pour assurer le suivi de la production scientifique, mettre en évidence les collaborations, etc. Ces études livrées sous forme de rapport PDF ne répondent pas à toutes les attentes des usagers (réutilisation des graphiques, accès aux données, etc.). Un outil comme [ezVIS](https://github.com/madec-project/ezvis) est la solution retenue pour satisfaire les attentes d’interactivité et de dynamisme du résultat fourni à travers un tableau de bord convivial point d’entrée du rapport en ligne.

## Configuration des rapports

Chaque rapport mis en ligne correspond à une instance configurée de façon relativement simple. Il est possible de créer autant d’instances que nécessaire en reproduisant la même configuration ou en la personnalisant. Cet aspect devrait permettre de multiplier le nombre de rapports en capitalisant et mutualisant les configurations. Ce gain de temps devrait permettre de pousser le travail de personnalisation.

Par ailleurs, les instances peuvent être gérées grâce à une interface à la prise en main aisée.  
La figure 1 présente [l’interface d’administration](https://github.com/madec-project/ezmaster) qui permet créer et gérer (modification, suppression) les instances et de les configurer.

![Figure 1 : interface d’administration et outil de paramétrage d’un rapport](/ezmaster-edit-instance.png)

À partir de données structurées en UTF8 et mises à disposition dans des fichiers de différents formats (csv, tsv ou XML) la configuration consiste à :
- sélectionner les champs à afficher ou à utiliser pour les calculs,
- réaliser les calculs (somme, pourcentage, etc.),
- choisir le type de graphique  (histogramme, camembert, barres horizontales) et les paramétrer (couleurs, seuil, légende).
- définir les facettes associées à chaque graphique.
- déterminer l’affichage des notices.

## Fonctionnalités d’exploration
À ce stade du développement, l’outil offre un rapport web constitué d’une page d’accueil présentant le tableau de bord et un index sous forme de menu à partir desquels il est possible de naviguer vers des informations plus détaillées et les notices correspondantes. Les facettes complètent les graphiques en  proposant  des filtres complémentaires  pour mettre en évidence d’autres résultats. La figure 2 présente un des graphiques du tableau de bord avec les facettes associées et le corpus sous forme de tableau.

![Figure 2 : Détail du tableau de bord](/ezvis-dashboard.png)

La suite du développement prévoit d’autres types de représentations comme des cartes, des réseaux ainsi que des fonctionnalités comme l’export ou la sécurisation de l’interface.

# Exemples d’usages
Le principal besoin auquel répond ezVIS est la création de tableaux de bord mettant en évidence des informations de type bibliométrique. La facilité de création et de configuration d’une instance est l’un des avantages évidents de cet outil qui autorise la multiplication des tableaux de bord.  Toujours dans le domaine des corpus de notices bibliographiques, un tel outil permet également de vérifier le contenu et la qualité des données. 
Enfin, le fait qu’il s’agisse d’un logiciel libre autorise son appropriation au-delà de la production scientifique comme par exemple pour l’analyse de fichiers de « logs » dans le cadre du projet ezPAARSE3.


## Connaissance de la production scientifique liée à une thématique 
À partir d’un corpus constitué thématiquement, il est possible de mettre en évidence des éléments concernant la production scientifique, son évolution ainsi que la répartition en sous-thématiques, par exemple. La figure 3 illustre l’utilisation d’un graphique de type camembert qui représente la répartition thématique du corpus. Il est possible en cliquant sur une partie du graphique d’avoir accès à la liste des résultats correspondants. Il est également possible d’utiliser les facettes pour mettre à jour le graphique de manière dynamique le graphique.

![Figure 3 : Mise en évidence de l’utilisation des facettes pour filtrer les résultats](/ezvis-facets.png)

## Exploration du contenu d’un corpus
Il peut être utile avant la mise en ligne de notices bibliographiques de vérifier la qualité des données ou leur homogénéité lorsque les origines et les formats sont différents. La figure 4 illustre l’exploration du corpus mis à disposition par un éditeur dans le cadre d’un projet.

![Figure 4 : Exploration d’un corpus de notice bibliographique](/ezvis-biblio-corpus.png)

## Analyse des consultations de ressources en ligne
ezVIS peut être utilisé pour réaliser des comptages d’autres types d’informations structurées et le choix d’un logiciel libre favorise fortement l’élargissement de l’usage. La figure 5 illustre l’utilisation d’ezVIS pour la mise en évidence du détail des consultations.

![Figure 5 : Détail  des consultations de ressources numériques](/ezvis-logs.png)
