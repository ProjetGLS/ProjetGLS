---
title: "Un Environnement de Calcul Domaine-Spécifique"
header-includes:
 - \usepackage[hmargin=2.5cm,vmargin=3cm]{geometry}
---

<div style="text-align: center;">
<div style="display:flex;flex-direction:column;align-items:center;"><img src="logo.png" data-external="1" style="width:25em">
    <p>Méta-modèle des schémas de tables (Figure 1.1)</p></div>
<h1>Un Environnement de Calcul Domaine-Spécifique</h1>
<p><b>Projet d’Ingénierie Dirigée par les Modèles</b><br>

Groupe L12-03:<br>

Bechu Alex<br>

Bessel Théo<br>

Bontinck Laerian<br>

Demazure Clément<br>

Truong Nell</p>

</div>

<div style="page-break-before: always;">
</div>

## Sommaire
- [Sommaire](#Sommaire)
- [Table des figures](#Table-des-figures)
- [Introduction](#Introduction)
- [Contenu du rendu](#Contenu-du-rendu)
  - [Division du repository en submodules](#Division-du-repository-en-submodules)
- [Intégration continue](#Intégration-continue)
    - [Sous-projet `:Metamodels`](#Sous-projet-metamodels)
- [Schémas de table](#Schémas-de-table)
    - [Metamodélisation](#Metamodélisation)
    - [Validation](#Validation)
- [Scripts](#Scripts)
    - [Metamodélisation](#Metamodélisation)
    - [Exemple de modèle](#Exemple-de-modèle)
    - [Outil de création de scripts](#Outil-de-création-de-scripts)
    - [Transformation Modèle à texte](#Transformation-modèle-à-texte)
    - [Validation](#Validation)
- [Algorithmes](#Algorithmes)
    - [Metamodélisation](#metamodélisation)
    - [Exemple d'utilisation](#exemple-utilisation)
- [Librairie de traitement](#Librairie-de-traitement)
- [Conclusion](#Conclusion)

## Table des figures
- Figure 1.1 : Méta-modèle des tables
- Figure 2.1 : Exemple du calcul d'une moyenne
- Figure 2.2 : Méta-modèle des scripts
- Figure 2.3 : Script d'exemple à obtenir
- Figure 2.4 : Fichier descriptif du script
- Figure 2.5 : Visualisation Graphique dans Sirius
- Figure 2.6 : Outils de création d'éléments
- Figure 3.1 : Méta-modèle des Algorithme
- Figure 5.1 : Retour sur le projet

<div style="page-break-before: always;">
</div>

## Introduction

Avec l'explosion de l'informatique vint la nécessité d'outils de traitement large de données, c'est pourquoi nous allons dans ce projet tenter de développer un outil de traitement de tables de données.

Ce projet s'articule autour de plusieurs grands axes :
- l'intégration continue du projet
- le développement des tables de données
- le développement d'un langage de calcul dédié
- le développement d'un outil de développement d'algorithmes 
- le développement d'une librairie de traitement à partir d'un schémas de table

## Contenu du rendu
L'intégralité des sources du projets sont disponibles [ici sur la page GitHub dédiée.](https://github.com/ProjetGLS/ProjetGLS)


### Division du repository en submodules

Afin de diviser le projet en différentes sous-parties et éviter de tous travailler en même temps sur un repository en commun, nous avons décidé d'opter pour la création d'un git de développement sur GitHub, comprenant lui-même plusieurs submodules (*metamodels*, *transformations*, *dsl*, et *textgen*), tous sous la même organisation GitHub afin de gérer plus facilement l'accès au projet et les permissions des membres.

Le repository principal, `ProjetGLS`, contient directement les quatres repository submodules mentionnés ci-dessus.

```
┐ ProjetGLS
│
├─┐ metamodels (submodule)
│ │
│ ├─┐ models
│ │ ├── table.ecore
│ │ ├── script.ecore
│ │ ├── algorithm.ecore
│ │ ├── library.ecore
│ │ ├── project.genmodel
│ │ ├── script.odesign
│ │ └── (instances dynamiques et exemples)
│ │
│ └─┐ target
│   └── (code généré et code de validation)
│
├─┐ textgen (submodule)
│ ├── (conversion de script en code Python)
│ └── (conversion de tables en .csv)
│
└── (divers fichiers de setup gradle)
```

Cette organisation nous permet de pouvoir faire plusieurs branches par submodule, et ainsi éviter un enfer de branches git sur l'entièreté du projet.

## Intégration continue

Afin d'essayer de faire de l'intégration continue sur le projet nous avons mis en place un environnement de build utilisant Gradle.
Le projet est ainsi architecturé en différents sous projets qui sont stockés comme submodules git sur le dépôt du projet :

### Sous-projet `:Metamodels`
Ce sous-projet contient les métamodèles au format `.ecore` et permet d'automatiser certaines tâches :
- La génération automatique de code `Java` à l'aide de la tâche `:Metamodels:generate`.
- La validation des métamodèles à l'aide des tâches `:Metamodels:validateScript`, `:Metamodels:validateTable`, `:Metamodels:validateLibrary` et `:Metamodels:validateAlgorithm` qui permettent de vérifier que les modèles respectent bien les contraintes définies dans les métamodèles ainsi que les contraintes statiques écrites en Java à l'aide de l'API des streams

### Sous-projet `:TextGen`

Ce sous-projet contient les transformations modèle à texte qui utilisent acceleo.

## Schémas de table

Dans cette partie, il est question de la stucture de table de données à proprement parler. 

### Métamodelisation

Les données sont stockés dans des colonnes qui sont rassemblés sous formes de tables, selon le méta-modèle suivant:
    
<div style="display:flex;flex-direction:column;align-items:center;"><img src="https://sanik.inpt.fr/uploads/76349944-d0eb-4e27-a75d-ec54243c859d.png" data-external="1" style="width:25em">
    <p>Méta-modèle des schémas de tables (Figure 1.1)</p></div>
    
Chacunes de ces colonnes possède un identifiant uniques, doit potentiellement respecter un certains nombre de contraintes et peut dérivé d'un traitement par un algorithme impliquant d'autre colonnes. 
Les contraintes sont représentés par des String car elle n'ont pas été traitées, elles sont donc dans le méta-modèle de manière purement figuratives. 

De plus pour faciliter les références entre différents éléments de diférentes tables, chaques tables contient exactement une colonne d'indentifiants qui servira à repérer les différentes lignes de la table.

### Validation

Pour s'assurer de la bonne utilisation des tables et pour éviter un maximum d'erreurs, nous avons définit en Java un certain nombre de règles de validation pour les schémas de tables:
- L'uid de chaque colonne au sein d'une même table est unique.
- Le nom de la table respecte les conventions de nommage de Java.
- Une colonne ne peut pas faire réference à elle-même.
    
## Scripts

### Metamodélisation
Dans cette partie, nous nous intéressons à la création de scripts de calculs dans un langage dédié.

L'objectif de cette partie est de créer un langage de script de la forme suivante :

<div style="display:flex;flex-direction:column;align-items:center;"><img src="https://sanik.inpt.fr/uploads/da499ec7-9f7f-423c-969e-9099d51713a1.png" style="width:40em">
<p>Exemple du calcul d'une moyenne (Figure 2.1)</p>
</div>

Nous avons commencé par décrire le méta-modèle de ce langage de calcul:
<div style="display:flex;flex-direction:column;align-items:center;"><img src="https://sanik.inpt.fr/uploads/c925d607-d632-4066-8453-fd5d7134f87a.png" style="width:40em">
<p>Méta-modèle des scripts (Figure 2.2)</p>
</div>

La classe principale (Script) décrit ce que doit contenir un script de calcul, à savoir: une ou plusieurs opérations mathématiques, une seule sortie (car une fonction mathématique ne possède qu'une sortie) et un nombre entier optionnel d'entrées.

Les opérations ont un nom, une arité et un booléen représentant si cette opération est ou non un opérateur infixe (préfixe sinon).
Les opérations ont également des entrées et une unique sortie (à nouveau car se sont des opérations mathématiques).
Les entrées des opérations sont des "Variables", qui peuvent être des entrées, des constantes ou des sorties d'autres opérations.

### Exemple de modèle

À partir de ce métamodèle, nous pouvons créer un script d'exemple:
l'objectif est de réaliser le script de `-max(a,b) + c` où a et b sont des constantes et c est une constante.

<div style="display:flex;flex-direction:column;align-items:center;"><img src="https://sanik.inpt.fr/uploads/03d62ecc-4dc4-44e0-88d6-07379786c802.png" style="width:40em">
<p>Script que nous allons essayer de d'obtenir (Figure 2.3)</p>
</div>

<div style="display:flex;flex-direction:column;align-items:center;"><img src="https://sanik.inpt.fr/uploads/914a7b7b-f300-4a6d-8a82-f5586be85a72.png" style="width:20em">
<p>Script sous forme textuelle (Figure 2.4)</p>
</div>

### Outil de création de scripts

La dernière étape consiste à rendre plus ergonomique la création de scripts via un éditeur graphique, chose que nous avons faite avec l'outil Sirius de Eclipse.

<div style="display:flex;flex-direction:column;align-items:center;"><img src="https://sanik.inpt.fr/uploads/2f976d9b-293a-4fdd-8adc-6ace0ce2f3fc.png" style="width:40em">
<p>Visualisation Graphique dans Sirius (Figure 2.5)</p>
</div>

<div style="display:flex;flex-direction:column;align-items:center;"><img src="https://sanik.inpt.fr/uploads/648e78b6-29bf-464f-a784-aa980c396666.png" style="width:10em">
<p>Outils de création d'éléments (Figure 2.6)</p>
</div>

Grace à ces outils, il est possible de créer facilement des script et les enregistrer.
Il faudra par la suite les passer par l'étape de validation avant de pouvoir les exécuter. Ceci sera fait via une task Gradle dédiée pour se passer d'Eclipse.

**Exemple :**
```
❯ gradle :metamodels:validateScript --args="models/instances/Script.xmi"

> Task :metamodels:validateScript
Visiting ScriptImpl
Résultat de validation pour models/instances/Script.xmi:
- Script: OK
- Operation: OK
- Input: OK
- Output: OK
Fini.


❯ gradle :metamodels:validateScript --args="models/instances/Script2.xmi"

> Task :metamodels:validateScript
Visiting ScriptImpl
Résultat de validation pour models/instances/Script2.xmi:
- Script: OK
- Operation: 2 erreurs trouvées
=> Erreur dans max [src.script.impl.OperationImpl@63a12c68 (name: max, arity: 2, infix: false)]: L'operation a comme entree sa sortie
=> Erreur dans - [src.script.impl.OperationImpl@505fc5a4 (name: -, arity: 2, infix: false)]: L'operation - n'a pas le bon nombre d'operandes
- Input: OK
- Output: OK
Fini.
```

### Transformation Modèle à texte

Deux transformatons modèles à textes ont été définies à l'aide de l'outil Acceleo :

- Une première permettant de générer du code Python à partir d'un modèle de `Script`. Le code généré définit des constantes de manière explicite dans le corps des fonctions générées pour plus de lisibilité et les expressions sont entièrement parenthésées.
L'attribut `infix` du métamodèle `Script` a été largement utilisé pour les opérateurs binaires du langage Python.
Le code généré ne gère l'importation d'aucun module, il faudrait donc gérer cela de manière externe en ajoutant un en-tête générique définissant la liste des imports avant exécution.
    
- Une seconde transformation permet de générer des fichiers CSV à partir du métamodèle `Library`.

### Validation

Si le métamodèle a été conçu pour contraindre les scripts au maximum, certaines contraintes de validation ne peuvent être modélisées par celui-ci, d'où la nécéssité d'un certain nombre de règles de validation supplémentaires pour éviter une définition d'un script invalide.
- Le nom du script respecte les conventions de nommage de Java.
- Le nombre d'opérandes d'une opération doit être égal à l'arité de cette opération.
- Une opération ne peut être infixe que si elle a une arité de 2.
- Une opération ne peut pas avoir comme entrée sa sortie.
- La sortie doit nécessairement être reliée.
    
## Algorithmes
### Métamodelisation

Après avoir créer des tables de données et un langage de script pour appliquer des transformations sur les données, On definit des algorithmes qui permettent, à l'instar des fonctions dans un language de programmation traditionel de généraliser un traitement simple pour l'appliquer sur différentes colonnes de données.

<div style="display:flex;flex-direction:column;align-items:center;"><img src="https://sanik.inpt.fr/uploads/87dbf93b-e9f1-4053-8c5c-8fafe5150b6c.png" style="width:15em">
    <p>Méta-modèle des Algorithmes (Figure 3.1)</p></div>

Ici un algorithme peut, toujours dans l'idée de ressemblé au maximum à des fonctions, prendre en entré un nombre quelconque de colonne de données auxquelle il appliquera un script d'entrée associé. Avant d'appliquer un script de sortie pour rassembler et finir le traitement de toutes les données pour enfin restituer une colonne unique contenant le resultat du traitement.

### Exemple d'utilisation
On pourrait par exemple imaginer un algorithme calculant une moyenne entre différentes notes avec différentes echelles de notations, les colonnes d'entrées serait les notes en questions, à chaque colonne serait associé un script pour en normaliser les valeurs en les ramenant à une note sur 20, et un script de sortie calculerait simplement la moyenne à partir des valeurs normalisé

## Librairie de traitement

<div style="display:flex;flex-direction:column;align-items:center;"><img src="https://sanik.inpt.fr/uploads/55e6e055-9956-4e0d-ac7d-325368c98cca.png" style="width:15em">
<p>Méta-modèle des Librairies (Figure 4.1)</p></div>

<!-- TODO -->
_TODO : Partie de Alex_


## Conclusion
**Clément** : Je suis personellement très insatisfait de mon travail, mon principale frein dans ce projet étant le manque d'organisation et de communication entre les différents groupes au sein du projet    
    
**Nell** : Avec Théo, j'ai pu faire l'essentiel de ce que voulais faire et faire l'entièreté du langage des petits calculs. Nous n'avons malheureusement pas pu intégrer celui-ci aux tables car cette partie du projet a moins avancée que la nôtre.<br>
Je retiens cependant que j'ai passé beaucoup plus de temps à me battre avec Eclipse, les dépendances et le setup du projet dans Eclipse plutôt qu'à avancer le projet en lui-même.

On notera par ailleurs le manque de documentation de Sirius, cocace pour un outil développé par une entreprise revendiquant créer des "Logiciels de modélisation ouverts et adaptables".

Les projets open-source : \*possèdent de la doc\*

<div style="display:flex;flex-direction:column;align-items:center;"><img src="https://sanik.inpt.fr/uploads/52993e16-90a8-4015-9b2d-8c0748888135.png" style="width:15em">
<p>Retour sur le projet (Figure 5.1)</p>
<!-- TODO changer le numéro de figure -->
</div>
    
**Laérian** : Je dois admettre que pour ce qui est de la partie validation de plusieurs métamodèles, j'ai passé plus de temps à essayer de faire fonctionner gradle qu'à travailler. Au final, j'ai fait mon code avec la moitié des classes manquantes et une détection syntaxique cassée.

Avec plus d'implication dans le projet, et plus de temps passé à essayer de comprendre comment manipuler les outils avant de les utiliser, j'aurais probablement apporté plus au projet que ce que j'ai pû faire finalement.
   
**Théo** : Je suis globalement très satisfait du travail que nous avons pu faire sur la gestion des scripts avec Nell (qui a beaucoup travaillé avec moi sur le projet) de manière locale ainsi que de ce que j'ai pu mettre en place en terme d'intégration continue mais également de mon travail pour faire en sorte que les métamodèles fait par les différents membres du groupe puissent être cohérents dans leur ensemble et fonctionner conjointement (mise en place des cross-references, unification des notations, etc.).

Cependant j'aurais aimé pouvoir compléter cette dernière avec un support d'XText qui n'a pas été possible car la majorité des dépôts relatifs aux tâches ANT et à Gradle semblaient malheureusement laissés à l'abandon.
    
J'aurais également aimé mettre en place un support de Sirius Web mais le manque de documentation de la part d'OBEO à propos de l'outil Sirius ainsi que de nombreux plantages inexpliqués de Sirius Web lors de mes tests m'ont découragés dans la mise en place du support de ce dernier.

Je reste globalement déçu du travail global de certains membres de l'équipe qui n'ont pas sû faire remonter correctement leurs difficultés ou l'ont fait très tardivement dans la timeline du projet et je garde l'impression d'avoir passé beaucoup du temps dédié au projet à régler des problèmes dans le code poussé sur le dépôt git pour tenter de garantir le bon fonctionnement de ce dernier. Peut-être que la mise en place d'un pipeline de test automatique sur ce dépôt pourrait être une piste à explorer pour de futurs projets.
Je pense que le principale frein à la finalisation d'une version fonctionnelle du projet a été un fort manque de communication dans une partie de l'équipe malgré la mise en place d'un dépôt git partagé ainsi que d'un canal de communication entre nous.

**Alex** : _TODO : Partie de Alex_