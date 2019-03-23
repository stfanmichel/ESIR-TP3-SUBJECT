# ESIR-TP3 - Express

Nous avons utilisé dans les deux premiers TP le module http standard pour réaliser notre serveur web (serveur web est un grand mot...).
L'API http est très complète mais d'un peu trop "bas niveau".
Il existe plusieurs modules node de plus haut niveau pour réaliser des serveur web, nous allons ici utiliser l'un des plus populaire : Express.

## Objectifs :

- Préparer un environnement pour express
- Comprendre la décomposition en fichier / répertoires
- Comprendre les principes de Express
- Utiliser et comprendre la notion de router
- Réaliser des services REST

## Sujets abordés :

- Express
- REST / CRUD
- Pattern d'injection

## Lien utiles :

- Outillage (npm, node, git, curl, postman etc.) : https://slides.com/stephmichel/deck-4#/
- Express : Le cours sur Express de Benoît.

## Modules node utilisés

- express : https://www.npmjs.com/package/express
- body-parser : https://www.npmjs.com/package/body-parser
- uuid : https://www.npmjs.com/package/uuid

Pour le bon déroulement du TP et pour vous familiariser avec GIT, lorsque vous liser une ligne du genre (Tag: BLA-BLA-BLA),
c'est qu'il est temps de commiter vos modifications afin de pouvoir revenir à ce niveau de code plus tard si besoin.
Ceci vous permettra également de vous y retrouver lorsque le correctif vous sera fourni.

# Initialisation d'un projet

Il existe des modules comme par exemple, express-generator pour vous aider à construire une arborescence de projet Express from sratch.
C'est un bon outil pour débuter : https://slides.com/stephmichel/deck-4#/4/1
Cependant, afin de bien comprendre l'organisation en fichier nous l'allons pas utilise cet outils (dans un premier temp).

Comme les TP précédents, vous aller initier un nouveau projet GIT/npm (je ne rappelle pas les étapes ici, vous devez commencer à les connaitre).

# STEP 1 : Premier service REST avec express

Vous devez ici :

- créer un fichier principal (app.js).
- installer le module express
- Réaliser un permier serveur web basé sur express qui répond à l'URL http://localhost:3000/users en retournant au format JSON une liste d'utilisateurs.

(tag: **TP3-ESIR-STEP1**)

# STEP 2 : Routes express et CRUD

Vous devez coder l'ensemble des opérations CRUD sur les users (avec en plus la possibilité de listeer tous les utilisateurs) en se basant sur la sémantique http et en respectant les bonnes pratiques de nommage (voir cours ici https://slides.com/stephmichel/http#/7).

Votre user devra posséder les propriétés suivantes (au minimum) : id, login, age et name.

Exemple :

    {id = '45745c60-7b1a-11e8-9c9c-2d42b21b1a3e', login:'pedro', name:'Pedro Ramirez', age:35)

Vous utiliserez les _express.Router_ afin d'organiser proprement votre code (voir la fin de la page https://expressjs.com/en/guide/routing.html).
Pour cela vous créerez un répertoire "routes" qui contiendra un fichier dans lequel vous coderez l'ensemble des opérations demandées (Création, lecture, mise à jour, suppression, etc.).

Vous penserez à tester les cas limites (paramètres incorrects, objets non trouvé, etc.) et remonter le code HTTP adéquat (200, 201, 400, 404, etc.). Voir ici pour un rappel : https://slides.com/stephmichel/http#/7/4

Vous utiliserez le module body-parser (https://www.npmjs.com/package/body-parser) afin de pour facilement accéder aux body des requêtes HTTP de type POST ou PATCH.

Vous penserez (c'est une bonne habitude à prendre) à inclure un numéro de version de votre api dasn l'URL de celle-ci (http://localhost:3000/v1/users).

Vous utiliserez le module uuid (https://www.npmjs.com/package/uuid) afin de générer un identifiant unique aux objets "user" que vous créerez.

    const uuidv1 = require('uuid/v1');
    uuidv1(); // ⇨ '45745c60-7b1a-11e8-9c9c-2d42b21b1a3e'

Il n'y a pas besoin pour le moment de gérer la persistance fichier ou BDD des utilisateurs créés, une simple persistance mémoire dans un tableau suffira.

## Rappels Javascript

Cette étape vous emmenera à manipuler des tableaux en Javascript. Voici ici quelques rappels sur des méthodes qui pourront vous être utiles :

Filtrer un tableau : https://developer.mozilla.org/fr/docs/Web/JavaScript/Reference/Objets_globaux/Array/filter

    const filteredArray = monTableau.filter((item)=> ... )

Retrouver la position d'un élément dans un tableau : https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/findIndex

    const index = monTableau.findIndex((item)=> item.id === ... )

Supprimer un élément dans un tableau à partir de son index : https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/splice

    monTableau.splice(index,1)

Copier les propriétés d'un objet dans un autre objet (sans passer par une variable intermédiaire) : https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign

    Object.assign(oldObject, newObject)

## Vérification des services REST

A la fin de cette étape vous disposerez des services suivants :
GET /v1/users
GET /v1/users/id
POST /v1/users
PATCH /v1/users/id
DELETE /v1/users/id

Vérifiez leur fonctionnement avec l'outil de votre choix (curl, postman).

Exemples :

    curl -i http://127.0.0.1:3000/v1/users

    curl -i http://127.0.0.1:3000/v1/users/45745c60-7b1a-11e8-9c9c-2d42b21b1a3e

    curl -i -X POST -H 'Content-Type: application/json' -d '{"name":"Robert", "login":"roro", "age":24}' http://localhost:3000/v1/users

    curl -i -X PATCH -H 'Content-Type: application/json' -d '{"name":"Roberto", "age":24}' http://localhost:3000/v1/users/45745c60-7b1a-11e8-9c9c-2d42b21b1a3e

    curl -i -X DELETE http://localhost:3000/v1/users/45745c60-7b1a-11e8-9c9c-2d42b21b1a3e

(tag: **TP3-ESIR-STEP2**)

# STEP 3 : Externaliser l'accès aux données

Pour le moment les données (la variable qui contient le listers de users) sont stockées au niveau de la route ce qui n'est évidemment pas une bonne pratique.
Nous allons ici utiliser un pattern d'injection pour fournir à notre route un "module" qui lui permettra d'accéder au modèle de données sous-jacent en lui masquant l'implémentation (la route n'a pas à la connaître).

Vous allez donc créer un module qui répond à l'interface suivante :

- _getAll()_ : Retourne la liste de tous les utilisateurs sous forme de tableau.
- _get(id)_ : Retoune l'utilisateur qui a pour id le paramètre id.
- _update(id)_ : Met à jour l'utilisateur qui a pour id le paramètre id.
- _remove(id)_ : Supprime l'utilisateur qui a pour id le paramètre id.
- _add(user)_ : Ajoute l'utilisateur en paramètre à la liste des utilisateurs. Assigne un id généré (uuid) à cet utilisateur et le retourne.

Lors du chargement de la route dans app.js, vous lui fournirez votre implémentation de l'interface ci-dessus (on utilisera une closure pour cela).
Modifier ensuite le code de votre "users route" pour utiliser l'interface.

La "users route" n'est maintenant plus dépendante directement du model. On verra dans un prochain TP qu'il s'agit d'une bonne pratique de programmation et que cela facile la réalisation des tests unitaires.

Vérifiez à nouveau son fonctionnement à l'aide de curl comme à l'étape 2.

(tag: **TP3-ESIR-STEP3**)
