title: Configuration d'ezVIS / minimum
date: 2015-03-19 23:38:00
tags:
- ezvis
- configuration
---
# Introduction

Pour créer un rapport web avec ezVIS, il faut configurer l'application. Il y a 3 volets:
1. connexion avec la base de données
2. intégration des données
3. ajout de graphiques

Ils sont tous gérables dans le fichier de configuration du rapport, qui est au format [JSON](http://fr.wikipedia.org/wiki/JavaScript_Object_Notation) (JavaScript Object Notation). Son extension est `.json`et son préfixe est obligatoirement le même que le nom du répertoire dans lequel se trouvent les fichiers de données.
Nous prendrons l'exemple d'un fichier `data.json` placé au même niveau que le répertoire `data`, qui contient un fichier au format CSV dont nous reparlerons plus tard.

# Connexion avec la base de données

Le système de gestion de base de données utilisé par ezVIS est [mongoDB](http://mongodb.org), et une connexion par défaut est utilisée, qui permet de retrouver les données liées au rapport dans la *database* appelée `castor`.
Si vous voulez changer cet emplacement, modifiez la clé `connexionURI`du fichier `data.json` (valeur par défaut: `mongodb://localhost:27017/castor/`).

À l'intérieur de `castor`, mongo range ses données dans des collections. Sans indication supplémentaire, ezVIS nomme la collection avec une clé de hachage calculée à partir du chemin du répertoire `data`. Cela donne un nom illisible pour un humain, mais quasi-unique en fonction du chemin.
Si vous avez l'intention de manipuler les données dans mongoDB (par exemple, simplement pour réinitialiser les données, avec [`castor-clean`](https://github.com/castorjs/castor-clean)), il est prudent de renseigner la clé `collectionName`, dans notre cas avec la valeur `"data"`par exemple:

```javascript
  "collectionName": "data",
```

Tant que vous y êtes, il est prudent de renseigner aussi les clés `title`et `description`, pour vous y retrouver quand vous relirez ce fichier dans quelques mois...

```javascript
  "title": "Rapport d'exemple de configuration",
  "description": "D'après l'article \"Configuration d'ezVIS / minimum\"",
  "collectionName": "data",
```
