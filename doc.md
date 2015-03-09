#### Travailler sur les sources d'un paquet dans vendor/


Dans un projet géré avec Composer, il peut arriver que l'on veuille modifier un des paquets que Composer a installé, afin par exemple de finir de mettre au point et tester ce paquet directement dans le projet, sans avoir à faire de multiples commits dans le dépot d'origine du paquets. Voici comment faire.

Tout d'abord, dans le composer.json du projet,  pour la version du paquet il faut indiquer le nom d'une branche (ex: dev-master) ou une version instable : Composer fera alors un clone du depot du paquet dans vendor/myvendor/mypackage/.

Cependant il faut faire attention à ce que l'on fait dans le répertoire vendor/myvendor/mypackage/, car un "composer update" peut faire lui aussi des modifications. À savoir donc avant de commiter quoi que ce soit :

* dans vendor/myvendor/mypackage/, deux "remote repository" sont créés par Composer. Le premier est le "origin" classique, le deuxième est "composer". Ce dernier par défaut ne permet pas de faire des push, car l'url des push pointe vers le depot "en lecture". De plus lors d'un update, ce sont les branches distantes du remote "composer" qui sont mis à jour localement.
* faire un "composer update" du projet alors qu'on travaille dans vendor/myvendor/mypackage/ ne semble pas avoir de conséquence **si il n'y a pas de changements** dans le dépôt d'origine.
* Par contre si il y a des changements dans le dépôt d'origine, un "composer update" **peut effacer vos commits locaux** si ils n'ont pas été "pusher". En effet, Composer met à jour composer/master et la branche master (qui track normalement origin/master). 


Pour éviter tout problème avec un "composer update" (accidentel ou non) :

* faire un checkout de la branche qui pointe vers origin/labranche (celle qui correspond en fait à la version "dev-labranche" que vous avez indiqué dans composer.json
* créer une branche locale à partir de cette branche : ```git checkout -b mychanges```
* faire ses commits dedans. 
* Quand vous ferez un "composer update", Composer ne touchera pas à votre branche "mychanges", mais fera un checkout de "labranche". Il faudra alors refaire un checkout de "mychanges" pour continuer à travailler.
* quand vos commits sont fait, si vous avez fait des "composer update" en cours de route, il faudra faire un rebase, fusionner et pusher

```
git rebase composer/labranche
git checkout labranche
git merge mychanges
git push origin/mychanges
```


##### historique des tests effectués

Ici les tests ont été fait alors que Composer a fait un checkout de origin/master.

* changer de remote (sans modification) + composer update. 
   * exemple: on est sur master -> origin/master et on fait un  git checkout -b cp-master --track composer/master
   * Résultat : RAS
* changement dans la branche tracké via "composer"
   * modif puis composer update/install sans nouvelle version. Résultat : RAS
   * modif puis commiter puis  composer update/install sans nouvelle version. Résultat : RAS
   * modif puis commiter puis pusher puis composer update/install sans nouvelle version. Résultat : on ne peut pas pousser car l'url du remote est celle en lecture seule. Il faut passer sur la branche de origin correspondante, fusionner, et pousser (ou alors modifier l'url push du remote "composer"). Sinon RAS.
* changement dans la branche tracké via "origin"
   * modif puis composer update/install sans nouvelle version. Résultat : RAS
   * modif puis commiter puis  composer update/install sans nouvelle version. Résultat : RAS
   * modif puis commiter puis pusher puis composer update/install sans nouvelle version. Résultat : la branche master du remote "composer" est mis à jour.
   * modif puis composer update/install avec nouvelle version dispo. Résultat : Composer demande si on veut écraser nos modifs, avec possibilité d'annuler la mise à jour.
   * modif puis commiter puis  composer update/install avec nouvelle version dispo. Résultat : Composer écrase nos commits locaux !!
   * modif puis commiter puis pusher puis composer update/install avec nouvelle version dispo. Résultat : le fait de pousser oblige à merger avec l'upstream, donc RAS au final.
* passage à une version stable dans le composer.json, sans/avec modification des sources du module. Résultat : Composer met à jour le composer/master et fait un checkout du tag sur cette branch distante (= detached head). Les commits locaux sont gardés.
* passage d'une version stable à instable. Résultat : il remplace les sources du paquet par un clone du depot du paquet
* changement distant + composer update alors que l'on est sur origin master ou composer master. Résultat : mise à jour du remote composer mais pas du remote origin. Par contre mise à jour de la branche locale origin master mais pas la branche locale composer master.


