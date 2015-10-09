title: "Sprint Review #15W38: peaufinage"
date: 2015-10-12 16:00:00
tags:
- ezvis
- ezmaster
- bugs
- configuration
- login
- sccm
---
Pour ce quinzième et dernier sprint, nous devions terminer, polir, affiner, régler, en un mot *peaufiner* l'application ezVIS.

# Tâches

- 19 tâches prévues
- 17 tâches terminées
- plus de 43 points de complexité prévus
- 30 points de complexité effectués

D'une manière générale, nous nous sommes concentrés sur la stabilité de l'application, et donc sur la réduction de la dette technique.

Nous avons aussi rencontré des problèmes sur ezMaster, qui ne pouvaient apparaître qu'après suffisamment d'utilisation, et résolu un bug dont la présence était aléatoire.


# Dette technique

La [dette technique](https://fr.wikipedia.org/wiki/Dette_technique) est la distance à parcourir, en termes de développement, pour parvenir au programme le plus cohérent et le plus à facile maintenir.

Les actions suivantes ont réduit cette dette.

### Correction de connexionURI en connectionURI

Le programme et ses options étant intégralement en anglais, il nous semblait incohérent de laisser une option avec une orthographe française: `connexionURI`.

GitHub&nbsp;: [#79](https://github.com/madec-project/ezvis/issues/79)

## Préparer ezvis à une évolution de castor-core

Le cœur d'ezVIS est un module nommé `castor-core` dont nous savons qu'il va évoluer (notamment les URL utilisés par ezVIS). Les routes (ou URL) fournies par `castor-core` version 2 seront encore disponibles dans sa version 3, mais préfixées par `/-/v2`.
Nous avons donc changé tous les appels à ces URL dans ezVIS.

GitHub&nbsp;: [#81](https://github.com/madec-project/ezvis/issues/81)

## Figer les dépendances ou pas?

Pour éviter des surprises lors des futures installations d'ezVIS, au cas où un des modules dont il dépend ne respecterait pas le [*semantic versioning*](http://semver.org/), nous avons pensé qu'il serait utile de figer les numéros de version de ces dépendances.

Il existe justement une commande du gestionnaire de modules de node qui le permet: [`npm shrinkwrap`](https://docs.npmjs.com/cli/shrinkwrap).
Malheureusement, celle-ci ne distingue [pas encore](https://github.com/npm/npm/issues/2679#issuecomment-137893324) les modules optionnels des modules obligatoires, et il se trouve qu'un module optionnel n'est pas utile ailleurs que sur Mac, mais que de plus il ne s'y installe pas, cassant ainsi l'installation d'`ezvis` dès qu'on utilise *shrinkwrap* (le rendant ainsi obligatoire).
La [feuille de route de npm](https://github.com/npm/npm/wiki/Roadmap-area-of-focus%3A-bundling) laisse à penser que d'ici un an, ce problème n'existera plus. D'ici là, nous compterons sur la gestion  sémantique de version des modules. S'ils la pratiquaient tous, moins de problèmes seraient à craindre.

GitHub&nbsp;: [#82](https://github.com/madec-project/ezvis/issues/82) [#85](https://github.com/madec-project/ezvis/issues/85)

# Interface

## Icones enlevées

Plusieurs icones étaient présentes dans l'entête d'ezVIS: celle des alertes (qui avertissait quand une synchronisation avait eu lieu, mais nous nous sommes aperçus que personne ne s'en servait), et celle de l'utilisateur (qui n'a jamais été fonctionnelle).

{% asset_img ezvis-top-right-icons.png [Icones enlevées] %}

Nous avons donc supprimé ces icones de la page.

GitHub&nbsp;: [#69](https://github.com/madec-project/ezvis/issues/69)

## Couleur des graphes superposés

Jusqu'à présent, le [graphe superposé](/2015/06/23/Sprint-Review-15W23-calculs-complexes/#Graphes_superposés) avait une couleur fixe&nbsp;: le jaune.

![Le graphe superposé jaune](/2015/06/23/Sprint-Review-15W23-calculs-complexes/ezvis-overlay-6.7.2.png)

Si cette couleur convient la plupart du temps, nous avons souhaité donner le choix au gestionnaire en ajoutant l'option `color` à la partie [`overlay`](https://ezvis.readthedocs.org/en/latest/Histogram/#overlay) de la configuration&nbsp;:

```javascript
  "overlay": {
    "label": "Taux de citation normalisé",
    "flying": ["normalizeCitationRatio"],
    "color": "red"
  },
```

{% asset_img ezvis-overlay-red.png [overlay red] %}

GitHub&nbsp;: [#73](https://github.com/madec-project/ezvis/issues/73)

# Corriger l'authentification derrière un reverse proxy

ezVIS peut être configuré pour n'[autoriser l'accès](/2015/05/04/Sprint-Review-15W14-ressources-externes/#login/password) qu'à un utilisateur particulier.
Dans notre établissement, les instances d'ezVIS sont derrière un reverse proxy (ou [proxy inverse](https://fr.wikipedia.org/wiki/Proxy_inverse)) dont le comportement n'a pas été cohérent: l'adresse IP du visiteur était soit l'adresse de ce proxy (comportement attendu), soit une adresse locale (127.0.0.1), autorisant alors l'accès à l'instance.
Nous avons donc corrigé ezVIS pour qu'il tienne compte de l'entête HTTP `x-forwarded-for` qui, elle, contient bien l'adresse IP du visiteur (pas celle du proxy).

GitHub&nbsp;: [cd94d07](https://github.com/madec-project/ezvis/commit/cd94d07423e5d7cc460ee949d024193b2cda63e8)

# Installation automatique avec SCCM sous Windows

Nous voulions pouvoir installer automatiquement, via le logiciel [SCCM](https://fr.wikipedia.org/wiki/System_Center_Configuration_Manager), ezVIS sur plusieurs postes Windows à la fois, dans les services de notre établissement.
Malheureusement, SCCM prenant l'identité de l'administrateur de la machine pour installer, il n'a pas de répertoire utilisateur. Ce répertoire utilisateur étant indispensable à l'installeur Windows de node pour fonctionner, nous avons dû renoncer à ce projet.

Malgré tout, l'installation manuelle de node est très simple, nous avons donc opté pour un compromis en automatisant uniquement l'installation de MongoDB, ce qui simplifie tout de même la procédure d'installation à l'INIST.


# ezMaster

## Remplacement de SlickGrid par un tableau HTML

{% asset_img ezmaster-11-instances.png [ezMaster avec 11 instances, ascenseur présent] %}

En dépassant 13 instances dans ezMaster, nous avons rencontré une limite: l'ascenseur disparait au-delà de 13 instances, empêchant toute action sur les dernières (configuration, ajout de données, suppression, ...)&nbsp;:

{% asset_img ezmaster-14-instances.png [ezMaster avec 14 instances, ascenseur absent] %}

La technologie utilisée, [SlickGrid](https://github.com/mleibman/SlickGrid/wiki), est complexe et inutile pour le nombre d'instances que nous gérons: nous l'avons remplacée par un simple tableau HTML sans pagination, ni filtre, ni tri.

{% asset_img ezmaster-html.png [ezMaster avec tableau HTML pur] %}

GitHub&nbsp;: [ezmaster#2](https://github.com/madec-project/ezmaster/issues/2)

## Remplacer le _ par un - dans l'URL publique

{% asset_img ezmaster-url.png [À la création d'une instance, le nom technique contient des tirets, et plus des soulignés] %}

Jusqu'à présent, le nom technique d'une instance est composé du nom du projet, de l'étude, et optionnellement d'une version, le tout séparé par des soulignés.
Dorénavant, et pour mieux satisfaire les normes sur les URL, ces séparateurs seront des tirets.

GitHub&nbsp;: [#80](https://github.com/madec-project/ezvis/issues/80)

# Profitez!

Pour profiter des améliorations présentées:

```bash
$ npm install --production -g ezvis
```
