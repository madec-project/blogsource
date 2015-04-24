title: "Sprint Review #15W14: ressources externes"
date: 2015-04-28 10:00:00
tags:
- blog
- twitter
- jbj
- ezvis
- ezref
- login
- flyingFields
---
Voici venue la revue de sprint numéro 10, qui a pour thème principal *les ressources externes*.

# madec-project.github.io

J'ai créé un page pour l'organisation `madec-project` sur [GitHub](https://github.com), celle qui contient tous les programmes liés au projet MADEC. GitHub offrant l'hébergement d'un site statique, nous avions déjà opté pour [Hexo](http://hexo.io) pour ce blog.

Pour obtenir une URL relativement courte (en tout cas plus courte que http://madec-project.github.io/blogfr), il a fallu nommer le dépôt contenant ce site `madec-project.github.io`, ce qui nous autorise une URL sans le nom du dépôt: http://madec-project.github.io/

# Twitter

Un compte Twitter [ezvis_project](http://twitter.com/ezvis_project) a aussi été créé (en ces temps de communication autour d'ezVIS, ça peut servir).

# vimadec / vpmadec / puppet

Le travail avec Patrice commence à porter ses fruits:

- ajout d'un script d'installation d'une app ezmaster à partir de l'URL de son `.tar.gz`,
- mise sous surveillance du processus `ezmaster` afin de le relancer automatiquement quand il s'arrête (via [forever](https://www.npmjs.com/package/forever)).
- demande de création d'une machine virtuelle de production `vpmadec`.

# ezVIS

## login/password

On peut maintenant limiter l'accès à un rapport ezVIS en utilisant la clé `access`, contenant un `login` et un mot de passe soit `plain`, soit `sha1`:

```javascript
  "access": {
    "login": "user",
    "sha1" : "37fa265330ad83eaa879efb1e2db6380896cf639"
  }
```

Le mot de passe sous forme `plain` est simplement le mot de passe en clair.
Mais comme ce n'est pas une bonne pratique, on peut remplacer son usage par
celui de `sha1` qui remplace un mot de passe par son [empreinte
SHA-1](http://fr.wikipedia.org/wiki/SHA-1).

Ainsi, on connaît l'empreinte du mot de passe, mais pas le mot de passe lui-même.

Pour obtenir l'empreinte SHA-1 d'un mot de passe, on peut utiliser des
commandes comme `shasum` ou `sha1sum` (en n'incluant pas de passage à la ligne
dans le mot de passe), ou bien des sites de génération comme [SHA1
online](http://www.sha1-online.com/).

## Accès aux ressources externes

### ezref

On peut avoir besoin de ressources externes à ezvis (parce que de simples fichiers), mais avec la possibilité de remplacer ces fichiers de référence (tables de correspondance, listes, ...).

Utiliser `ezref`, l'application pour ezMaster qui met à disposition via le protocole http des fichiers qu'ezMaster vous permet de déposer est la solution.

Pour pouvoir stocker les ressources: ezref (serveur web statique).

### flyingFields

Les `flyingFields` sont les cousins des `documentFields` et des
`corpusFields`. Ils sont un croisement, dans le sens où ils permettent
l'interopérabilité des uns et des autres.

### JBJ

Pour pouvoir appliquer une table de correspondance présente dans les
`corpusFields`, on a ajouté une action `mappingVar` à JBJ, qui fonctionne
comme `mapping`, mais dont les arguments sont différents.

### Exemple: externaliser la table de correspondance d'une carte géographique

Pour projeter des données sur la carte du monde, jusqu'ici, on était obligé de
traduire les noms des pays en codes ISO (c'est ainsi que sont identifiés les
pays sur la carte):

```javascript
    "$fields.country" : {
      "label": "Pays d'affiliation",
      "path" : "content.json.Pays",
      "parseCSV": ";",
      "foreach": {
        "mapping": {
          "Albania" : "ALB",
          "Algeria" : "DZA",

          "Zaire" : "COD",
          "Zambia" : "ZMB"
        }
      }
    }
```

Nous avons amélioré l'opérateur `mapping` de JBJ pour qu'il puisse traiter directement tout un tableau:

```javascript
    "$fields.country" : {
      "label": "Pays d'affiliation",
      "path" : "content.json.Pays",
      "parseCSV": ";",
      "mapping": {
        "Albania" : "ALB",
        "Algeria" : "DZA",

        "Zaire" : "COD",
        "Zambia" : "ZMB"
      }
    }
```

Puis, nous avons créé un opérateur similaire à `mapping` qui, au lieu de
prendre en entrée l'objet courant et en paramètre la table de correspondance,
permet de mettre en paramètre deux noms de variable: l'entrée et la table.
Il s'appelle `mappingVar` (ou `combine`).

```json
{
  "set": {
    "arg": {
      "a": "Aha!",
      "b": "Baby"
    },
    "input": "a"
  },
  "mappingVar": [
    "input",
    "arg"
  ]
}
```

Ceci renvoie `"Aha!"`.

Donc, grâce à `ezref`, on peut mettre la table de correspondance sur un serveur externe, et le charger dans un `corpusFields`, accessible comme une variable dans les `flyingFields`:

```javascript
  "documentFields": {
    "$fields.country" : {
      "label": "Pays d'affiliation",
      "path" : "content.json.Pays",
      "parseCSV": ";",
      "foreach": {
        "trim": true
      }
    }
  },
  "corpusFields": {
    "$country2iso": {
      "$?" : "http://localhost:35000/country2iso3.json",
      "parseJSON": true
    }
  },
  "flyingFields": {
    "$country2": {
      "$_id": {
        "combine" : ["_id", "country2iso"]
      },
      "mask": "_id,value"
    }
  }
```

Quand on veut avoir le nombre de valeurs distinctes de `fields.country`, on peut utiliser l'URL http://localhost:3000/compute.json?operator=distinct&field=fields.country :

```json
{

  data: [
    {
      _id: "Albania",
      value: 2
    },
    {
      _id: "Algeria",
      value: 15
    }
  ]
}
```

Mais en utilisant `&flying=country2` en plus, on applique le JBJ correspondant à ce champ: 
http://localhost:3000/compute.json?operator=distinct&field=fields.country&flying=country2

```json
{

  data: [
    {
      _id: "ALB",
      value: 2
    },
    {
      _id: "DZA",
      value: 15
    }
  ]
}
```
