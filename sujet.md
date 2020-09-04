GENIAL - Hackathon septembre 2020
====

# Contexte et But

La licence d'informatique possède en L1 une matière d'initiation aux
systèmes d'exploitation (IS1). Le but de cette matière est de faire en
sorte que les étudiants soient à l'aise avec un système de type
unix/linux, et en particulier avec le shell. L'évaluation de cette
matière se fait sur machine, grâce à un programme qui a commencé à
être écrit il y a une petite dizaine d'années et a été modifié et complété
régulièrement depuis en fonction des besoins. Ce programme remplit son
rôle, mais est extrêment lourd à manipuler et, alors qu'ajouter ou
supprimer une question dans l'énoncé de l'examen est une opération de
base, elle est de fait extrêmement longue à mettre en place.

Le but de ce hackathon est de réécrire cet outil. Cette réécriture est
un vrai besoin et si le résultat obtenu est satisfaisant, votre
programme sera sûrement utilisé pendant plusieurs années (ce n'est donc
pas un programme-jouet, mais une vraie commande). Pour que ce
soit possible, il faut bien entendu que votre solution réponde à
toutes les exigences (dont certaines ne sont pas discutables car elles
dépendent du fait que le programme doit s'exécuter sur les machines du
script, mais d'autres peuvent certainement être assouplies.)

Une autre conséquence de cette potentielle utilisation future est que
**votre code doit être documenté** car il sera maintenu et mis à niveau
régulièrement.

# Plusieurs types d'utilisateurs

Il existe plusieurs rôles (ou types d'utilisateurs) : étudiant, enseignant
surveillant, enseignant créateur, enseignant correcteur. Dans la
suite on décrit le point de vue de chacun d'entre eux. Pour comprendre
l'application dans sa globalité, il faut avoir en permanence tous les
rôles en tête. Si on devait mettre en avant les deux points les plus
importants à respecter, ce serait :

* le programme de contrôle doit être facile d'utilisation pour les
  étudiants (car il s'agit de contrôler leurs connaissances, pas leur
  aptitude à utiliser le programme de contrôle) ;
* il faut sauvegarder un maximum de données au fur et à mesure du
  déroulement du contrôle afin d'être capable _a
  posteriori_ de comprendre ce qui s'est passé pendant le contrôle.

Pour chaque type d'utilisateurs, nous avons réalisé un petit film
explicatif. Attention : 

* les films ne sont pas exhaustifs dans les demandes que nous avons,
il faut donc bien lire ce qui suit; le but des films est de mieux
comprendre le sujet ;
* les films représentent ce qui se passe dans les programmes actuels,
  pas ce qui doit se passer dans ce que vous ferez : il ne faut pas que
  ça vous limite !

## Le contrôle vu par un Étudiant
Le programme central est celui qui exécute le contrôle : chaque
étudiant le lance dans un terminal, et utilise un ou plusieurs autres
terminaux pour exécuter les commandes nécessaires pour répondre aux
questions.

Le programme de contrôle est vu comme une boucle par l'étudiant, dont
l'itération est la suivante :

* affichage de l'énoncé d'une question ;
* action de l'étudiant et/ou saisie de la réponse, puis validation ; 
* vérification de la réponse / de l'action, puis :
  * si bonne réponse : passage à la question suivante, 
  * sinon : retour à la question (incidence sur le calcul du score).

À la fin du contrôle (soit parce que l'étudiant a terminé ou abandonné,
soit parce que le temps imparti est écoulé), l'étudiant voit s'afficher
un résumé de son contrôle (nombre de tentatives, temps mis et nombre
d'indices regardés pour chaque question) et une évaluation de son score.
Plus aucun fichier ou processus créé par ou pour le contrôle ne doit
exister. Pour simplifier ce point, tous les fichiers créés par et pour le
contrôle se situent dans une même arborescence de racine `REP_CONTROLE`
(voir la section Paramètres).

### Questions
Les questions sont de deux types :

* question attendant une réponse (par exemple : "Quel répertoire
  contient  un fichier de lien `toto`?" ou "Quel est le numéro d'i-nœud du
  fichier `[1]`?")
* question attendant une action (par exemple: "Supprimer toute
  l'arborescence de racine `[0]`")

(La notation `[n]` sera expliquée plus bas dans la section [Identifiants](#Identifiants).)

Chaque question peut être complétée par un ou plusieurs indices. Ces
indices n'apparaissent qu'à la demande explicite de l'étudiant
(avec une incidence sur le calcul du score).

Chaque question s'affiche avec :

* son numéro ;
* son énoncé ;
* son caractère obligatoire ou non (une question non obligatoire est
  incluse dans le barême, mais l'étudiant peut renoncer à y répondre
  et choisir de passer à la question suivante de l'exercice) ;
* le temps restant ;
* le nombre d'indices disponibles pour la question.

### Menu intermédiaire
À tout moment, un étudiant doit pouvoir faire apparaître un menu qui
lui permet de faire une des actions suivantes :

* abandonner le contrôle de façon définitive ;
* abandonner la question courante si elle est facultative ;
* obtenir l'indice suivant (s'il y en a et qu'il en reste) (incidence sur le calcul du score).

Dans la version actuelle l'étudiant utilise la combinaison de touches
`Ctrl-c` pour accéder à ce menu (mais ce n'est pas une obligation).

Pour éviter tout malentendu, le programme doit mettre en garde
clairement l'étudiant et plusieurs fois avant qu'il ne puisse
effectivement abandonner le contrôle ou abandonner une question (de
façon à éviter toute contestation de type "je n'ai pas fait
exprès"). Un contrôle abandonné ne peut pas être repris.

Dans la version actuelle, une question abandonnée ne peut pas être
reprise, mais c'est un des points que l'équipe enseignante d'IS1
souhaite voir améliorer : nous aimerions qu'un étudiant puisse décider
de laisser de côté un exercice, mais qu'il puisse y revenir ensuite
(la notion d'exercice est décrite plus bas). Cela suppose que l'état
du SGF et l'existence d'éventuels processus mis en place par le
programme de contrôle soit remis dans un état équivalent au moment du
retour à l'exercice.


### Contrôle arrêté de façon intempestive 

Si l'étudiant quitte le contrôle par le menu, il est clair qu'il ne doit
pas pouvoir le reprendre. Malheureusement il arrive que le programme
exécutant le contrôle s'arrête pour d'autres raisons (la plupart du temps
suite à une mauvaise manipulation de l'étudiant, mais il peut arriver que
le problème vienne d'une erreur de programmation de l'enseignant créateur
ou d'un problème de réseau au script). Dans ce cas, l'étudiant doit
pouvoir relancer le programme du contrôle et que **le contrôle reprenne
là où il s'était arrêté** (avec le même passé que si l'interruption
n'avait pas eu lieu). Cela signifie en particulier que l'historique des
réponses de l'étudiant doit perdurer et que l'état du SGF et l'existence
d'éventuels processus mis en place par le programme de contrôle sont
remis dans un état équivalent au moment où le contrôle est relancé,
mais aussi que le temps restant pour faire le contrôle doit
correspondre au temps écoulé depuis le début effectif du contrôle
(c'est à l'enseignant surveillant de juger s'il ajoute du temps à un
étudiant ou pas).

### <a id="Identifiants">Identifiants</a>
Le programme communique avec les étudiants en utilisant des noms pour les
liens, mais également en utilisant des identifiants pour les fichiers. On
ne peut pas utiliser les numéros d'i-nœuds comme identifiants pour les
fichiers car ça n'aurait pas de sens d'écrire "Créez un fichier de lien
`toto` dans le répertoire `REP`, avec pour numéro d'i-nœud 12345", on est
donc obligé d'écrire "Créez un fichier de lien `toto` dans le répertoire
`REP`, son identifiant est `[3]`". Les identifiants utilisés pour les
fichiers sont des nombres entre crochets. On souhaite qu'ils apparaissent
dans l'ordre croissant des entiers strictement positifs tout le long de
l'énoncé.

De la même façon, on ne peut pas utiliser un pid pour identifier un
processus qu'on demande à l'utilisateur de lancer. On identifie les
processus par des lettres minuscules entre crochets (`[a]`), en
utilisant ces lettres dans l'ordre alphabétique.

## Personnalisation et paramétrisation : ensemble des étudiants
Il est important que les étudiants ne puissent pas tricher et que
l'épreuve soit de même niveau de difficulté pour chacun. La technique
mise en œuvre par le programme actuel est de personnaliser certains
paramètres numériques ainsi que les noms des liens dans le SGF et des
exécutables : pour ces noms, on utilise des tableaux contenant des noms
et on tire au hasard dedans pour chaque étudiant. Les étudiants ne
connaissant pas l'arborescence de leur voisin, il leur est difficile de
"traduire" les commandes de ce voisin pour les appliquer à leur propre
situation.

Cela implique bien entendu que le programme doit garder en mémoire les
noms qui sont susceptibles de revenir dans des questions ultérieures ou
d'intervenir dans des réponses, et éventuellement d'autres indices
comme l'identifiant associé à un lien. Bien entendu, toutes ces
valeurs peuvent être sauvegardées dans des fichiers. Pour permettre de
"rejouer" le contrôle, il est aussi possible de sauvegarder les
graines associées aux tirages alétoires, afin de repartir de ces mêmes
graines.

Un étudiant est identifié par son numéro d'étudiant qui est également
son uid sur les machines du script.

film de présentation du rôle étudiant : [https://youtu.be/7kRFPAtJK2Q]( https://youtu.be/7kRFPAtJK2Q )

## Enseignant surveillant
L'enseignant surveillant est l'enseignant dans la salle pendant que le
contrôle se déroule. Il doit avoir accès à plusieurs utilitaires, car
il a besoin de pouvoir :

* regarder à quelle question en sont les étudiants qui passent le
  contrôle, en particulier les étudiants de sa salle (la première
  lettre du nom d'une machine au script désigne la salle dans laquelle
  se trouve cette machine) ;
* regarder l'ensemble des réponses données par un étudiant ;
* regarder les paramètres associés à la question en cours d'un
  étudiant (y compris la réponse que l'étudiant doit donner)
* ajouter ou supprimer du temps à un étudiant (par exemple tiers-temps
  ou étudiant en retard) ; l'étudiant peut être en train d'exécuter le
  programme de contrôle au moment de cette modification ;
* faire apparaître un menu pour les enseignants dans le déroulé du
  contrôle de l'étudiant (menu protégé par mot de passe) pour pouvoir
  sauter une question qui est en principe obligatoire.
  
Bien entendu, la plupart de ces actions doivent être faites sur une
autre machine physique que celle sur laquelle l'étudiant est en train
de faire son contrôle (sauf la dernière, et éventuellement
l'avant-dernière qu'il faudra alors protéger par un mot de passe).

film de présentation du rôle enseignant surveillant : [https://youtu.be/GiDCuZSgpw8](https://youtu.be/GiDCuZSgpw8)

## Enseignant correcteur
L'enseignant correcteur est celui qui s'occupe de mettre les notes
définitives aux étudiants. Ces notes tiennent compte de plusieurs
facteurs :

* le poids de la question (précisé par l'enseignant créateur) ;
* le nombre de tentatives de l'étudiant pour trouver la bonne réponse
  (ou effectuer la bonne action) (déterminé pendant l'exécution du
  programme) ;
* un coefficient de perte de points propre à chaque question pour
  chaque tentative erronée (précisé par l'enseignant créateur).
* le nombre d'indices consultés pour chaque question (déterminé pendant
  l'exécution du programme) ;
* un coefficient de perte de points propre à chaque question pour
  avoir le droit de regarder un indice (précisé par l'enseignant créateur).

La formule de calcul du score d'une question est la suivante :

![](score.png)


L'enseignant correcteur doit avoir accès pour chaque étudiant ayant
passé le contrôle et chaque question du contrôle aux renseignements
suivants :

* nombre de tentatives pour trouver la bonne réponse ;
* nombre d'indices consultés ;
* un indicateur pour savoir si la question a été abandonnée ou pas.

On veut que tous ces renseignements se trouvent dans un unique fichier
de type `csv` (une ligne par étudiant, identifié par son numéro
d'étudiant, un ensemble de colonnes par questions). On veut pouvoir
modifier le poids d'une question et le coefficient de perte associé et
que ce fichier soit régénéré.

Pour des raisons de vérification, l'enseignant correcteur doit
également avoir accès à la trace du contrôle pour chaque étudiant,
sous la forme d'un fichier par étudiant qui regroupe pour chaque
question la bonne réponse et les diverses réponses données par
l'étudiant.

film de présentation du rôle enseignant correcteur : [https://youtu.be/1elyldy6svc](https://youtu.be/1elyldy6svc)

## Enseignant créateur
L'enseignant créateur est celui qui est chargé de mettre en place
le contrôle.

De son point de vue, la boucle du programme est un peu plus compliquée
que celle de l'étudiant :

* mise en place du matériel nécessaire à une question ;
* affichage de l'énoncé de la question ;
* lecture éventuelle de la réponse par l'étudiant ;
* vérification de la réponse / de l'action, puis :
  * si bonne réponse : ménage et passage à la question suivante, 
  * sinon : ménage et retour à la mise en place de la question.

De plus, si l'étudiant décide de passer une question facultative, il
faut parfois mettre plus de choses en place pour la question
suivante : par exemple si l'étudiant saute une question dans laquelle
on lui demandait de créer un fichier, il faudra que le programme crée
le fichier à sa place s'il est nécessaire à la suite de l'épreuve, et
prévienne l'étudiant de cette création. Il est même possible qu'une question sautée ne
donne pas accès à la question immédiatement suivante, mais saute
plusieurs autres questions qui seraient trop dépendantes de la question
que l'étudiant a décidé de sauter.

Du point de vue de l'enseignant créateur, le contrôle est découpé en
plusieurs exercices et chaque exercice est découpé en plusieurs
questions. Le minimum demandé est que le programme fonctionne dans un
cas où les exercices sont indépendants, mais dans l'idéal les
exercices devraient pouvoir se partager entre eux quelques fichiers,
voire une arborescence.

L'enseignant créateur doit pouvoir spécifier les points suivants pour
chaque question d'un exercice :

* la mise en place à faire (c'est du code : par exemple on veut créer
  un fichier de plus de 150 octets dans un endroit précis de
  l'arborescence, avec un certain nom) ;
* l'énoncé de la question (c'est du texte, dans l'idéal il faut
  pouvoir modifier ce texte et que le programme en tienne compte sans
  avoir besoin de recompilation) ; attention : ce texte n'est pas
  indépendant du code car il faut pouvoir utiliser des liens qui
  seront générés par le programme (voir la section Cohérence avec
  l'existant) ;
* les indices liés à la question (texte) ;
* les divers coefficients liés au calcul de la question (texte) ;
* le caractère facultatif ou non de la question (texte) ;
* le nombre de questions à sauter si l'étudiant saute la question
  (texte) ;
* le nettoyage et la mise en place à faire pour la question suivante si l'étudiant
  décide de sauter la question (code) ;
* le test de la réponse de l'étudiant (code ou référence ou valeur) ;
* le nettoyage à faire et la nouvelle mise en place si l'étudiant a
  mal répondu à la question (code) ;
  
Nous souhaiterions que le découpage en exercices soit assez naturel,
par exemple un répertoire par exercice, avec un fichier texte
permettant de spécifier l'ordre des exercices. À l'intérieur de chacun
de ces répertoires, doivent être spécifiées les questions avec leurs
indices, de sorte que l'enseignant créateur n'ait jamais besoin de
spécifier un numéro de question dans le code (mais uniquement un ordre
de question dans un fichier texte), et qu'il soit facile de supprimer
ou d'ajouter une question.

Attention au point suivant déjà souligné : les énoncés des questions doivent
pourvoir comporter des noms de fichiers propres à chaque étudiant
(voir la partie sur la personnalisation) et des références (voir la
partie sur les identifiants) ; pour rappel, ces noms ne
sont pas connus au moment où l'on écrit le programme, mais générés à
la volée (voir la section Cohérence avec l'existant).


film de présentation du rôle enseignant créateur : [https://youtu.be/E4hPE0kruHw](https://youtu.be/E4hPE0kruHw)

# Contraintes techniques et Paramètres
## Paramètres
Le programme de contrôle doit être facilement paramétrable, de façon
transparente pour les étudiants (de préférence donc au niveau de la
compilation). Les paramètres qui doivent être présents sont :

* `MOTDEPASSE` : mot de passe permettant de faire apparaître le menu
  enseignant ;
* `DUREE_EPREUVE` : durée de l'épreuve exprimée en minutes ;
* `REP_CONTROLE` : tous les fichiers (au sens large) créés
  pour le contrôle (par le programme ou par l'étudiant) doivent être
  dans une arborescence dont la racine est `$HOME/REP_CONTROLE`, où
  `$HOME` est le répertoire de login de l'étudiant et `REP_CONTROLE`
  un paramètre (le même pour tous les étudiants, mais qui permet de
  changer ce nom en fonction du contrôle par exemple). Ce répertoire
  doit être créé par le programme de contrôle ;
* `EXE_CONTROLE` : nom de l'exécutable (qui sera placé à la racine du
  compte enseignant) ;
* `PREFIXE` : prefixe des noms de fichier de sauvegarde ;
* `RACINE` : répertoire où le code est installé.

Par ailleurs il est souhaité qu'on puisse spécifier de façon simple
(c'est-à-dire dans un ou plusieurs fichiers texte, pas dans le code) :

* les énoncés des questions (idéalement le programme lit au moment
  opportun l'énoncé dans un fichier, ce qui permet de modifier cet
  énoncé sans recompiler) ;
* les indices associés à chaque question (idem) ;
* le caractère obligatoire ou non de chaque question ;
* le barême de chaque question ;
* le coefficient de perte pour chaque tentative de réponse à une
  question (ce coefficient doit être propre à chaque question) ;
* le nombre maximal de tentatives de réponses autorisées pour une
  question (utilisé essentiellement pour les questions oui/non).

## Contraintes techniques

* Les étudiants ne doivent pas avoir accès  aux sources, ce qui
  implique un langage compilé.
* Certaines opérations à faire pour mettre en place un exercice sont
  assez bas niveau, les enseignants utilisent donc `C` pour programmer
  les exercices. Il faut un langage compatible (nous préconisons
  fortement le `C++`, si vous faites un autre choix il faudra le
  justifier, le défendre et avoir notre accord pour ça car le
  programme sera maintenu par l'équipe enseignante d'IS1).
* Les étudiants ont le droit d'être propriétaires d'un unique fichier
  sur le compte des enseignants au script. Ce fichier devra donc
  contenir toutes les informations concernant le déroulé du contrôle
  de l'étudiant et tous les calculs de notes devront pouvoir être
  extraits de ce fichier.
* Les set-uid et set-gid bits ne sont pas autorisés.

## Cohérence avec l'existant

Dans la version actuelle il y a déjà une banque de questions de taille
non négligeable, que l'équipe enseignante ne souhaite pas perdre. Dans
la mesure du possible il faut donc que les formats soient les mêmes:

* chaque question est écrite dans un fichier
* on met les coefficients liés à une question dans un fichier csv, une
  ligne par question

Voici un exemple de texte de question existant:

```
Dans le répertoire [[6]], créer une __copie__ [[7]] du fichier [[5]], sous le nom $$15.1$$.
```

Vous pouvez remarquer que le texte est formaté : l'image suivante
vous montre comment ce texte est vu par un étudiant.

![](question.png)
