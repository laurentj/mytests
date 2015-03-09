comportement de composer avec des changements dans le dépôt local de vendor/monmodule/. 

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
