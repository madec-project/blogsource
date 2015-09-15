title: "Sprint Review #15W30: gestion des erreurs"
date: 2015-09-14 14:00:00
tags: 
- csv-string
- bugs
- erreurs
- validation
- chargement
- jbj
- log
---
Ce quatorzième et avant-dernier sprint avait pour thème *la gestion des erreurs*.
Nous avons apporté quelques modifications concernant la gestion des erreurs et leur signalement, ajouté une visualisation du chargement des données à `ezvis` et quelques moyens de valider ce chargement.
Pendant la chasse au bug que comporte tout sprint, nous avons écrit une commande permettant de mieux situer d'où viennent certaines erreurs: `csv-string`.
Au passage, nous avons ajouté des fonctionnalités à JBJ (dans l'optique de faciliter l'utilisation de ressources externes).


## Tâches

- 9 tâches prévues
- 6 tâches terminées
- plus de 21,5 points de complexité prévus
- 32 points de complexité effectués

Comme [la dernière fois](/2015/07/23/Sprint-Review-15W26-consolidation/#Tâches), nous étions en présence d'un bug, par essence d'estimation difficile tant qu'on n'a pas entamé son analyse.

## Gestion des erreurs

Les erreurs de chargement sont maintenant sauvegardées dans un fichier `instance_errors.log` (où `instance` est le nom de l'instance, c'est-à-dire le nom du répertoire où se trouvent les données).

Lorsque la variable d'environnement `NODE_ENV` ne vaut pas `production`, ces erreurs sont aussi affichées dans la sortie standard d'erreur.
En clair, cela signifie que si on utilise ezvis via un terminal on peut voir ces erreurs (sauf si la variable en question a été créée/modifiée), mais que pour l'instant, l'utilisation exclusive d'ezmaster ne le permet pas.

[#66](https://github.com/madec-project/ezvis/issues/66)
[#67](https://github.com/madec-project/ezvis/issues/67)

## Visualisation du chargement

Lors du chargement des données (par exemple au démarrage d'ezvis), au lieu de laisser l'administrateur d'une instance dans l'expectative, à ne pas savoir où en est le chargement, on affiche maintenant la progression du chargement en direct:

<script type="text/javascript" src="https://asciinema.org/a/8xu5zzhclccty0sahnstip58z.js" id="asciicast-8xu5zzhclccty0sahnstip58z" async></script>
[#63](https://github.com/madec-project/ezvis/issues/63)

## Validation du chargement

À la fin d'un chargement, un fichier `instance_load.log` existe avec des informations sur les opérations de synchronisation qui ont eu lieu:

```
$ cat data15_load.log
/Lorraine_WOS_Moselle_publis2007-2012_SansLorraineSansMoselle_UTF8.csv : 3449
Total    : 3449 documents
Fri Sep 11 2015 17:47:55 GMT+0200 (CEST)
---------------------------------
```

[#68](https://github.com/madec-project/ezvis/issues/68)


## Amélioration des messages d'erreurs de JBJ

Jusqu'à présent, les erreurs de JBJ (le langage utilisé pour la configuration d'ezvis) étaient traitées de manière hétérogène.
Dorénavant, elles sont toutes traitées de la même manière, et affichées lors du chargement.
On ajoute, derrière le message d'erreur qui peut encore être abscons (c'est un message d'erreur javascript), le nom de l'action qui a provoqué cette erreur (mais pas le nom de ses alias).

[#64](https://github.com/madec-project/ezvis/issues/64)

## csv-string

Lors de la mise au point d'une instance, on peut rencontrer une erreur JBJ concernant une action `parseCSV` ou `parseCVSFile`.
Sachant que ces actions JBJ utilisent une bibliothèque appelée [csv-string](https://www.npmjs.com/package/csv-string), il est pratique de pouvoir reproduire (puis éliminer) ces erreurs en dehors du processus ezvis (qui peut être long, si le nombre de documents est élevé).
C'est pourquoi j'ai écrit une [commande `csv-string`](https://github.com/parmentf/csv-string-command) qui applique l'analyse d'un CSV avec les options par défaut en utilisant la même bibliothèque qu'ezvis: csv-string.

Son installation est similaire à celle d'ezvis:

```bash
$ npm install -g csv-string-command
```

Cette commande lit l'entrée standard et écrit sur la sortie standard (et éventuellement la sortie standard d'erreur).

## Chargement de ressources externes en CSV

Lors d'un test d'utilisation de [ressources externes](/2015/05/04/Sprint-Review-15W14-ressources-externes/#Accès_aux_ressources_externes), nous nous sommes rendus compte qu'il était bien plus facile d'accéder à des ressources au format JSON, qu'à des ressources au format CSV.

1. l'action [`parseCSV`](https://github.com/castorjs/node-jbj/blob/master/README.md#parsecsv-separator) est faite pour analyser une chaîne de caractère représentant un *champ* (une colonne), et non un fichier CSV complet. Elle renvoie uniquement la première ligne d'un fichier CSV.
2. l'action [`parseCSVFile`](https://github.com/castorjs/node-jbj/blob/master/README.md#parsecsvfile-separator) que nous avons ajoutée pallie le problème précédent, mais ne permet pas d'obtenir un objet directement utilisable (équivalent à un tableau associatif), mais un tableau de tableaux:
    ```javascript
    [
      [
        "Afghanistan",
        "AFG"
      ],
      [
        "Aland Islands",
        "ALA"
      ]
    ]
    ```
    nous avons donc crée une nouvelle action, [`arrays2objects`](https://github.com/castorjs/node-jbj/blob/master/README.md#arrays2objects-key-value), qui permet de modifier ce tableau (transformer les tableaux internes en objets) qui pourront ensuite être utilisés par l'action [`array2object`](https://github.com/castorjs/node-jbj/blob/master/README.md#array2object-key-value) pré-existante (que l'on utilisait déjà avec le fichier JSON externe).

Avant:

```javascript
    "$TCMonde": {
      "$?": "http://localhost:35000/ESI_AllFields_20150407.json",
      "parseJSON": true,
      "array2object": true
    }
```

Maintenant:

```javascript
    "$TCMonde": {
      "$?": "http://localhost:35000/ESI_AllFields_20150407.tsv",
      "parseCSVFile": "\t",
      "arrays2objects": true,
      "array2object": true
    }
```

[#65](https://github.com/madec-project/ezvis/issues/65)
