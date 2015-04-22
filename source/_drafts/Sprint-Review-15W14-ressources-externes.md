title: "Sprint Review #15W14: ressources externes"
date: 2015-04-28 10:00:00
tags:
- blog
- twitter
---
Voici venue la revue de sprint numéro 10, qui a pour thème principal *les ressources externes*.

# madec-project.github.io

J'ai créé un page pour l'organisation `madec-project` sur [GitHub](https://github.com), celle qui contient tous les programmes liés au projet MADEC. GitHub offrant l'hébergement d'un site statique, nous avions déjà opté pour [Hexo](http://hexo.io) pour ce blog.

Pour obtenir une URL relativement courte (en tout cas plus courte que http://madec-project.github.io/blogfr), il a fallu nommer le dépôt contenant ce site `madec-project.github.io`, ce qui nous autorise une URL sans le nom du dépôt: http://madec-project.github.io/

# Twitter

Un compte Twitter [ezvis_project](http://twitter.com/ezvis_project) a aussi été créé (en ces temps de communication autour d'ezVIS, ça peut servir).

# ezVIS

## login/password

On peut maintenant limiter l'accès à un rapport ezVIS en utilisant la clé `access`, contenant un `login` et un mot de passe soit `plain`, soit `sha1`:

```javascript
  "access": {
    "login": "user",
    "sha1" : "37fa265330ad83eaa879efb1e2db6380896cf639"
  }
```

Le mot de passe sous forme `plain` est simplement le mot de passe en clair. Mais comme ce n'est pas une bonne pratique, on peut remplacer son usage par celui de `sha1` qui remplace un mot de passe par son [empreinte SHA-1](http://fr.wikipedia.org/wiki/SHA-1).

Ainsi, on connaît l'empreinte du mot de passe, mais pas le mot de passe lui-même.

Pour obtenir l'empreinte SHA-1 d'un mot de passe, on peut utiliser des commandes comme `shasum` ou `sha1sum` (en n'incluant pas de passage à la ligne dans le mot de passe), ou bien des sites de génération comme [SHA1 online](http://www.sha1-online.com/).
