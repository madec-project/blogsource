title: "Sprint Review #15W26: consolidation"
date: 2015-07-23 14:00:00
tags:
- ezvis
- bugs
- ezref
- mise à jour
- documentFields
- corpusFields
- readthedocs
- documentation
- usage
- jbj
- castor-clean
---
Ce *sprint* n°13 a été consacré à la consolidation de l'outil ezVIS.
Nous avons principalement modifié la documentation et corrigé des bugs.


# Tâches

- 20 tâches prévues
- 18 tâches effectuées (dont 15 avaient été prévues)
- 24 tâches au total
- plus de 24 points de complexité prévus
- 37,5 points de complexité effectués

Le nombre de points de complexité prévus était très peu précis, car nous avions plusieurs bugs.
Les bugs sont par définition difficile à estimer.
N'ont pas été comptabilisés ici les travaux de déclaration des bugs par les documentalistes/utilisateurs.


# Production

## Basculements des URL publics

Les URL publics des rapports du service Appui au Pilotage de l'Inist pointent maintenant sur la machine de production dont la mise au point a été finalisée (via puppet).
Au passage, les instances ont été copiées de la machine d'intégration vers la machine de production (et vérifiées par les gestionnaires de ces instances).

## Augmentation de l'espace disque

À cause de la manière dont ezVIS gère le cache des requêtes qu'il utilise, la place utilisée par une de ses instances dans la base de données augmente à mesure que ses utilisateurs en font des usages variés.

C'est pourquoi nous avons fait augmenter par le service Ingénierie de Production l'espace disque disponible sur la machine de production (c'était 10 Go au total sur la machine d'intégration, c'est 200 Go sur la machine de production).


# Bugs

## Déclaration des bugs par les gestionnaires

Ce sprint ayant été dédié à la consolidation d'ezVIS, nous avons demandé aux gestionnaires des instances déjà existantes de faire une déclaration formelle des bugs qu'ils ont rencontré.
De plus, ils ont dû faire en faire qu'on puisse reproduire ces bugs.
Nous avons donc utilisé l'application ezREF présente sur la machine d'intégration, et fait utiliser l'interface de dépôt de fichiers sur ce serveur pour y mettre:
- la description du dysfonctionnement (dans un fichier `*README.txt`)
- la configuration de l'instance (dans un fichier `*.json`)
- le(s) corpus dans un ou plusieurs fichier(s) dont préfixe était commun à tous les fichiers concernant ce bug.

Tout le monde a parfaitement joué le jeu et nous avons obtenu la description de 5 bugs:

```
(-rw-rw-r--)    3.6M    Bibliotep_XML_14_08_corpus_final_NCT.xml
(-rw-rw-r--)    4.2M    Bibliotep_XML_14_10_corpus_final_NCT.xml
(-rw-rw-r--)    438B    Bibliotep_XML_bibliotep.README.txt
(-rw-rw-r--)    25.4k   Bibliotep_XML_bibliotep.json
(-rw-rw-r--)    1.1k    bibliotep_entier_pertes.README.txt
(-rw-rw-r--)    6.0k    bibliotep_entier_pertes.json
(-rw-rw-r--)    1.7M    bibliotep_entier_pertes_2005-2006.csv
(-rw-rw-r--)    2.5M    bibliotep_entier_pertes_2006-2007.csv
(-rw-rw-r--)    2.7M    bibliotep_entier_pertes_2007-2008.csv
(-rw-rw-r--)    2.7M    bibliotep_entier_pertes_2008-2009.csv
(-rw-rw-r--)    2.8M    bibliotep_entier_pertes_2009_2010.csv
(-rw-rw-r--)    2.6M    bibliotep_entier_pertes_2010-2011.csv
(-rw-rw-r--)    2.7M    bibliotep_entier_pertes_2011-2012.csv
(-rw-rw-r--)    2.7M    bibliotep_entier_pertes_2012-2013.csv
(-rw-rw-r--)    2.2M    bibliotep_entier_pertes_2013-2014.csv
(-rw-rw-r--)    2.7M    bibliotep_entier_pertes_2014-2015.csv
(-rw-rw-r--)    6.3k    jsoncorpus_istex.json
(-rw-rw-r--)    498B    jsoncorpus_istex_README.txt
(-rw-rw-r--)    30.7M   jsoncorpus_istex_data.json
(-rw-rw-r--)    25.2k   maj_config-corpus_ezpaarse_CONFIG-ancienneconfig.json
(-rw-rw-r--)    25.9k   maj_config-corpus_ezpaarse_CONFIG-nouvelleconfig.json
(-rw-rw-r--)    5.2M    maj_config-corpus_ezpaarse_CORPUS-initial.csv
(-rw-rw-r--)    5.2M    maj_config-corpus_ezpaarse_CORPUS-modifie.csv
(-rw-rw-r--)    995B    maj_config-corpus_ezpaarse_README.txt
(-rw-rw-r--)    5.7k    wos_france_lux_csv.json
(-rw-rw-r--)    2.9M    wos_france_lux_csv_CORPUS.csv
(-rw-rw-r--)    1.1k    wos_france_lux_tsv-csv_README.txt
(-rw-rw-r--)    5.8k    wos_france_lux_tsv.json
(-rw-rw-r--)    1.5M    wos_france_lux_tsv_CORPUS-SansGuillemet_1-500Ref.txt
(-rw-rw-r--)    1.4M    wos_france_lux_tsv_CORPUS-SansGuillemet_501-999Ref.txt
(-rw-rw-r--)    1.5M    wos_france_lux_tsv_CORPUS_1-500Ref.txt
(-rw-rw-r--)    1.4M    wos_france_lux_tsv_CORPUS_501-999Ref.txt
```


## Correction du chargement incomplet

Dans plusieurs des bugs déclarés, lors du chargement des données au premier lancement, ezVIS ne rendait pas la main: la déclaration `Files and Database are synchronised.` n'arrivait pas (même quand tous les documents avaient été chargés), et donc encore moins le calcul des `corpusFields` (les métriques sur le corpus).
Souvent d'ailleurs, on pouvait contourner ce problème en relançant simplement l'instance (ezVIS détectait alors que le(s) fichier(s) n'avaient pas changés, et passait directement à l'étape suivante: le calcul des métriques).

Il s'est avéré que ce cas arrivait quand une erreur survenait lors du chargement (soit un problème de *parsing* du fichier, soit un problème JBJ lors du calcul des `documentFields`).
L'analyse a révélé que lors du chargement, les erreurs étaient complètement ignorées, et qu'ezVIS essayait quand même de traiter les données, sans même afficher l'erreur.

Dorénavant, l'erreur est affichée, accompagnée du nom du fichier et du numéro du document, dans ce fichier, pour lequel l'erreur s'est produite.

Exemple:

```bash
$ ezvis bibliotep_entier_pertes
Core version : 2.5.0
Configuration : /home/parmentf/dev/castorjs/bugs/bibliotep_entier_pertes.json
Theme : /home/parmentf/dev/castorjs/ezvis
App    : ezvis 6.7.3
Source : /home/parmentf/dev/castorjs/bugs/bibliotep_entier_pertes
Server is listening on port 3000: http://localhost:3000
Index field : annee/annee_1
Index field : titre/titre_1
Index field : wid/wid_1
Index field : neoplasms/neoplasms_1
Index field : techniques/techniques_1
Index field : pays/pays_1
Index field : auteurs/auteurs_1
Index field : vpmid/vpmid_1
Index field : elements/elements_1
Index field : pmid/pmid_1
Index field : anatomical/anatomical_1
Index field : isotopes/isotopes_1
Index field : source/source_1
Index field : text/text_text
 error [TypeError: Cannot call method 'slice' of undefined] in file /home/parmentf/dev/castorjs/bugs/bibliotep_entier_pertes/bibliotep_entier_pertes_2014-2015.csv document # 3431
 error [TypeError: Cannot call method 'slice' of undefined] in file /home/parmentf/dev/castorjs/bugs/bibliotep_entier_pertes/bibliotep_entier_pertes_2014-2015.csv document # 3643
 error [TypeError: Cannot call method 'slice' of undefined] in file /home/parmentf/dev/castorjs/bugs/bibliotep_entier_pertes/bibliotep_entier_pertes_2014-2015.csv document # 3645
 error [TypeError: Cannot call method 'slice' of undefined] in file /home/parmentf/dev/castorjs/bugs/bibliotep_entier_pertes/bibliotep_entier_pertes_2014-2015.csv document # 3738
 error [TypeError: Cannot call method 'slice' of undefined] in file /home/parmentf/dev/castorjs/bugs/bibliotep_entier_pertes/bibliotep_entier_pertes_2014-2015.csv document # 3855
 error [TypeError: Cannot call method 'slice' of undefined] in file /home/parmentf/dev/castorjs/bugs/bibliotep_entier_pertes/bibliotep_entier_pertes_2014-2015.csv document # 3895
Files and Database are synchronised.
127.0.0.1 - - [20/Jul/2015:15:41:03 +0000] "GET /compute.json?operator=count&field=wid HTTP/1.1" 200 - "-" "-"
127.0.0.1 - - [20/Jul/2015:15:41:03 +0000] "GET /compute.json?operator=distinct&field=annee HTTP/1.1" 200 - "-" "-"
127.0.0.1 - - [20/Jul/2015:15:41:03 +0000] "GET /compute.json?operator=distinct&field=isotopes HTTP/1.1" 200 - "-" "-"
Corpus fields computed.
```

Cette correction a modifier le comportement d'ezVIS sur plusieurs des bugs déclarés, révélant alors que c'était plutôt les fichiers d'origine qui ne respectaient pas le format demandé (CVS et l'échappement des guillemets, par exemple).

Voir sur GitHub: [#46](https://github.com/madec-project/ezvis/issues/46).


## Correction: mettre à jour l'instance quand la configuration est modifiée

Un comportement pratique d'ezVIS a cessé de fonctionner il y a déjà quelques sprints: quand on modifie la configuration d'une instance et qu'on la relance *sans modifier les données*, les documents ne prennent pas en compte les modifications de la configuration.
En particulier, quand un gestionnaire modifie les `documentFields`, ces nouveaux champs ne sont calculés que pour les nouveaux documents (ou pour aucun). C'est très handicapant quand on met au point une configuration car on est alors contraint, quand on passe par ezMaster, de supprimer l'instance et de la recréer (ce qui implique de recharger les données).

Ce comportement a été rétabli: quand on modifie une configuration, si les documents sont plus anciens que le fichier de configuration, ezVIS les mets à jour en prenant en compte la nouvelle configuration.

Voir sur GitHub: [#49](https://github.com/madec-project/ezvis/issues/49).


## Correction: mettre à jour l'instance quand les fichiers sont modifiés

Quand un fichier déjà chargé dans l'instance est remplacé par un fichier du même nom mais contenant des lignes en moins, les lignes ne disparaissaient pas.

Après une enquête approfondie (merci à [Yannick](https://github.com/nojhamster) pour son aide), nous avons trouvé et corrigé le bug.

Voir sur GitHub: [#50](https://github.com/madec-project/ezvis/issues/50).

**ATTENTION :** il est possible que des métriques faisant un comptage des documents ne soient pas mises à jour immédiatement. Dans ce cas, il est nécessaire de redémarrer l'instance (ou de la mettre en pause de la relancer, via ezMaster) pour bénéficier d'un calcul à jour.



# Nouvelle documentation

Il commençait à être difficile de s'y retrouver dans [l'ancienne documentation d'ezVIS](https://github.com/madec-project/ezvis/blob/356d3dc4b49e2e30c3c7684b087212f29537ef51/README.md), qui tenait sur une page, mais était dépourvue de table des matières (et souvent faisait référence à la documentation d'autres projets).

Il a donc été décidé d'utiliser un système dédié à la documentation de projets informatiques, qui se base sur le même format que l'ancienne documentation (Markdown): [ReadTheDocs](https://readthedocs.org/).

La nouvelle documentation, divisée en pages plus courtes, et agrémentée d'illustrations, est donc disponible sur http://ezvis.readthedocs.org/ ou http://ezvis.rtfd.org/.
Elle est mise à jour à chaque mise à jour du dépôt GitHub, et *responsive* (lisible sur un téléphone mobile).
Au besoin, on pourrait même en garder des versions différentes (une pour la version 6.*, et une pour la version suivante, par exemple).

{% asset_img ezvis-readthedocs.png [Nouvelle documentation d'ezVIS, sur ReadTheDocs] %}

Voir sur GitHub: [#51](https://github.com/madec-project/ezvis/issues/51).

# Meilleurs messages d'erreur

## oubli de lancer MongoDB

Quand on a oublié MongoDB avant de lancer ezVIS, il y a maintenant un message d'erreur:

```txt
failed to connect to [localhost:27017]
```

Il est certes sibyllin, mais il est difficile de faire mieux (principalement en raison du fait qu'ezVIS n'établit la connexion à MongoDB que lorsqu'il en a besoin).

## oubli du paramètre

ezVIS a un paramètre obligatoire: le chemin du répertoire où se trouvent les fichiers contenant les données.
Auparavant, le message était très technique, et même les programmeurs avaient besoin de toute leur expérience pour le comprendre.

Maintenant c'est celui ci:

```bash
$ ezvis
Usage:
 ezvis data
  data being a directory path, and data.json the settings file.
  See https://github.com/madec-project/ezvis for more details.
```

Voir sur GitHub: [#52](https://github.com/madec-project/ezvis/issues/52).


## Erreurs JBJ

Afin de pouvoir afficher les erreurs JBJ (dues à la configuration des `documentFields`, `corpusFields` et `flyingFields`), nous avons mené une opération d'homogénéisation du traitement des erreurs dans JBJ.
Il sont maintenant traités comme n'importe quelle erreur dans ezVIS (en particulier lors du chargement des données).


Voir sur GitHub: [#56](https://github.com/madec-project/ezvis/issues/56).


# Camemberts: position des légendes lorsque des labels sont précisés

Dans un `pie`, quand on déclare des `labels`, la position de la légende n'était pas prise en compte (voir image ci-dessous).

Ex:

```json
{
  "field": "Prf",
  "type": "pie",
  "title": "Projets de recherche fédératifs (PRF) - PRF avec labels ds chart",
  "legend": {
    "position": "right"
  },
  "labels": {
    "ESU": "ESU - Environnement sonore urbain",
    "Eval-PDU": "Eval-PDU - Evaluation environnementale des plans de déplacements urbains",
    "GEOCONURB": "GEOCONURB - Géo-connaissances urbaines",
    "MUE": "MUE - Microclimat urbain et énergie",
    "ONEVU": "ONEVU - Observatoire nantais des environnements urbains",
    "PUD": "PUD - Projet urbain durable",
    "SOLURB": "SOLURB - Sols urbains",
    "V&I": "V&I - Ville et image"
  },
  "removeLabels": "true",
}
```

{% asset_img IRSTV_PRF_LabelsDsChart.png [Camembert avec la légende dessous, et non à droite] %}

C'est maintenant fonctionnel.

Voir sur GitHub: [#55](https://github.com/madec-project/ezvis/issues/55).


# JBJ

## Nouveaux alias: `getIndex` & `getIndexVar`

Quand [`getProperty`](http://jbj.rtfd.org/#getproperty) et [`getPropertyVar`](http://jbj.rtfd.org/#getpropertyvar) sont appliqués à un tableau, il est plus naturel d'utiliser `getIndex` et `getIndexVar` (cliquez sur les liens pour voir des exemples dans la documentation de JBJ version ReadTheDocs).

Voir sur GitHub: [#60](https://github.com/madec-project/ezvis/issues/60).


## [JBJ Playground](http://castorjs.github.io/node-jbj/)

Voir sur GitHub: [#61](https://github.com/madec-project/ezvis/issues/61).

### Recherche dans les exemples

Nous avions déjà une recherche dans les actions, mais pas dans les exemples.
C'est fait.

{% asset_img jbj-plgrnd-examples-search.png [Recherche parmi les exemples du JBJ Playground] %}

Voir sur GitHub: [f8cde74](https://github.com/castorjs/node-jbj/commit/f8cde749693fbd6ff59642a5f23f8e7d93cacf1a).

### Agrandissement des éditeurs

La taille des `INPUT`, `STYLESHEET` et `RESULT` était fixe. Elle est maintenant dépendante de la largeur de la fenêtre du navigateur.

{% asset_img jbj-plgrnd-textareas.png [Agrandissment des éditeurs du JBJ Playground] %}

Voir sur GitHub: [PR 12](https://github.com/castorjs/node-jbj/pull/12).

### Supprimer les préfixes des exemples (`Basic: find`)

Les exemples étaient initialement classés par sujet, et partageaient leur `input`.
Nous avons supprimé le premier niveau (sujet).
Nous en avons aussi profité pour supprimer un effet de bord gênant: quand une feuille de style modifiait l'`input`, elle le modifiait aussi pour les autres exemples du même sujet. Chaque exemple est maintenant indépendant.

Voir sur GitHub: [e3915c3](https://github.com/castorjs/node-jbj/commit/e3915c3f9cb280ca4366f13d27eeb64ba881b1d3).

### Ajouter un champ URL

Nous avons ajouté un champ `URL` sous l'`input` pour pouvoir remplir cet éditeur avec la réponse d'une requête renvoyant du JSON.

{% asset_img jbj-plgrnd-url-input.png [Le champ de saisie d'une URL dans le JBJ Playground] %}

{% asset_img jbj-plgrnd-url-json.png [Remplissage automatique de l'input avec le contenu d'une URL dans le JBJ Playground] %}

Voir sur GitHub: [8174aae](https://github.com/castorjs/node-jbj/commit/8174aae0d8fd3a7c8c5c57f0548ff40747363406).

### ezVIS: ajouter des boutons vers le JBJ Playground

Quand on ajoute dans la configuration d'ezVIS une propriété `addlinkstojbj` à `true`, on ajoute un lien vers le JBJ Playground :
- dans la page d'affichage d'une notice (`/display/`) [7f51427](https://github.com/madec-project/ezvis/commit/7f514271584bc032f94eab1f6b1a18a0da5cd900)
- dans la liste des documents (`/documents.html`) [25f64c3](https://github.com/madec-project/ezvis/commit/25f64c3549542dc42a343bd47629c6d0192eea9a)
- dans les graphiques (`/chart.html`) [c0dad22](https://github.com/madec-project/ezvis/commit/c0dad222e63b7473dad342778ffa9437adf12a32)

Voir dans la documentation, [l'annexe sur JBJ](http://ezvis.readthedocs.org/en/latest/JBJ/).

Voir sur GitHub: [#62](https://github.com/madec-project/ezvis/issues/62).

# `castor-clean`: message explicite

Auparavant, [`castor-clean`](https://github.com/castorjs/castor-clean/) était une commande muette: qu'elle ait accompli sa mission ou non, l'utilisateur n'en savait rien.

Maintenant, quand tout s'est bien passé, elle écrit `OK`.
Sinon, le message est:

```
Warning: no collections dropped.
         Either you mistyped the name, or it was already cleaned up.
```

Voir sur GitHub: [8229754](https://github.com/castorjs/castor-clean/commit/8229754b0b6481d5abd9ac62c690bbfe349f89e9).

Pour en profiter: `npm install -g castor-clean`. (version 1.2.0).

# `ezref`: usage

Comme pour [ezVIS](#oubli_du_paramètre), quand on oublie le paramètre obligatoire d'`ezref`, on a maintenant un message indiquant l'usage normal de la commande:

```bash
$ ezref
Usage:
 ezref public
  public being a directory path, and public.json the settings file.
  See https://github.com/madec-project/ezref for more details.
```

# `ezvis`: mise à jour de dépendances

Il existe un site qui recense la fraîcheur des dépendences de projets Node, et qui signale des trous de sécurité potentiels.

[![Badge des dépendances d'ezvis](https://david-dm.org/madec-project/ezvis.svg)](https://david-dm.org/madec-project/ezvis)

J'ai donc procédé à quelques mises à jour (`marked`, `sha1`, et `qs`).

Sans doute à surveiller de près.

Comme d'habitude, pour profiter des ajouts de ce sprint dans ezVIS :

```bash
$ npm install --production -g ezvis
```

À ce jour, c'est la version 6.8.0.
