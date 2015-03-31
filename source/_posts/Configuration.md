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

`title` sera utilisé pour le titre du rapport (la fenêtre du navigateur) et `description` sera beaucoup plus discrètement placée dans les métadonnées HTML du rapport (visible surtout des moteurs de recherche).

À condition d'avoir déjà installé `ezvis`, vous pouvez d'ores et déjà le lancer avec:

```bash
ezvis data
```

# Intégration des données
Après avoir lancé ezVIS, un rapport vide devrait être consultable à l'URL http://localhost:3000/ (3000 étant le port par défaut, on peut le changer via la clé `port`).

Le menu gauche doit donner accès au *dashboard* et aux *documents*.

Seul un message signalant qu'aucune configuration n'existe encore apparaît sur ces deux pages.

Nous allons commencer par configurer simplement la liste des documents. Elle se présente sous la forme d'une table affichant, dans l'ordre où ils sont déclarés dans le fichier `data.json`, tous les champs déclarés *visibles*.

Prenons un exemple simple, un fichier `data.csv` qui sera placé dans le répertoire `data`:

```csv
year,person
1912,Alan Turing
1927,Marvin Minsky
```

C'est un fichier CSV (Comma Separated Values) qui aurait pu être exporté d'un tableur, comme Excel ou LibreOffice.
Pour que ces données soient affichées dans la page http://localhost:3000/documents.html, il faut que nous déclarions les champs `year`et `person` (qui sont automatiquement placés dans une clé `content.json` de chaque document dans la base, au démarrage d'ezVIS).
Le premier document sera placé dans la base sous forme d'un document JSON, et la partie qui nous intéresse aura cette forme:

```json
{
  "content": {
    "json": {
      "year": "1912",
      "person": "Marvin Minsky"
    }
  }
}
```

Pour déclarer les champs, nous utiliserons la clé `documentFields` (qui au passage créera une copie du contenu des champs mais directement à la racine du document):

```json
{
  "documentFields": {
    "$year": {
	  "get": "content.json.year",
	  "visible": true,
	  "label": "Année"
    },
    "$person": {
      "get": "content.json.person",
      "visible": true,
      "label": "Nom"
    }
  }
}
```

Cette déclaration modifiera le document précédent pour donner ceci:

```json
{
  "content": {
    "json": {
      "year": "1912",
      "person": "Marvin Minsky"
    }
  },
  "year": "1912",
  "person": "Marvin Minsky"
}
```

Cela peut paraître inutile pour l'instant, mais l'intérêt de cette redondance deviendra évident avec la manipulation de champs multivalués, par exemple.

Le fait d'avoir déclaré une clé `visible` avec une valeur `true` implique que le champ sera visible dans la table des documents.

La partie `label` de la déclaration donne le nom la colonne correspondante dans la table des documents.


# Ajout de graphique

Pour ajouter un graphique, il suffit de sélectionner un type parmi:

1. histogramme `histogram`
2. barres horizontales `horizontalbars`
3. camembert `pie`
4. carte géographique `map`
5. réseau `network`

et d'y associer un champ (calculé ou non).

Le plus simple est un histogramme.

Tous les graphiques sont inclus dans le `dashboard` (tableau de bord), c'est-à-dire la page d'accueil d'ezVIS.
