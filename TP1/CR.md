# TP1: "Prototypage Virtuel avec SoCLib"
* Taha Sebai 21103800
* Dylan Morais Ramos 21112920

## 2.3 Composant fifo_gcd_master

**Compte-tenu de l'algorithme de calcul du PGCD implémenté par le composant fifo_gcd_coprocessor, que se passe-til si un des deux opérandes transmis au coprocesseur a la valeur 0? Comment peut-on modifier le composant fifo_gcd_master pour que ceci ne se produise jamais?**

Le PGCD ne peut pas être égale à 0 ! EN effet, si une des deux opérandes est nulle, le la condition de sortie de la boucle ne sera jamais atteinte. On ne va riebn soustraire à l'autre opérande. Elle va garder sa valeur initiale donc **opa** sera toujours différent de **opb**. Pour éviter ce problème, on ajoute un comparateur pour comparer opa et opb à 0. Si les opérandes sont non nulles, on peut calculer le pgcd. Sinon, si une des deux opérandes est nulle, on retourne par exemple 0 pour indiquer qu'il y a une erreur au composant FIFO_GCD_MASTER

## 3.1 Ecriture du modèle CABA du coprocesseur

Pour les registres du coprocesseur, on a besoin uniquement de r_opa et r_opb. En effet, pour l'état COMPARE, on a pas besoin de mémoriser le résultat de la comparaison de la boucle précédente. Pour le résultat, on peut simplement retourner la valeur contenue dans l'un des deux registres cités précédemment. Ajouter un registre pour stocker le résultat serait redondant.

Les ports sont les même que pour le maître, à savoir : un reset pour réinitialiser l'automate, une horloge pour synchroniser l'automate, p_in pour la lecture des opérandes, et p_out pour le résultat du calcul.

Les fonctions membre sont également les mêmes que celles du maître. Il n'y a pas de fonction genMealy() car les valeurs de sortie de l'automate ne dépendent pas des valeurs d'entrée, seulement pour la première itération (pour placer les valeurs dans les registres r_opa et r_opb).


Implémentation des fonctions membre.
**transition**:
si la valeur lu par le pipe de restn est 0, alors on reset le composant à l'état initial READ_OPA via le registre r_fsm (reset actif à 0).
Ensuite selon les différents états du composant lu dans r_fsm, on applique le traitement adéquat.
READ_OPA (RESP. READ_OPB): si ROK, FIFO non vide. lecture de la valeur contenu dans le port entrée du pipe puis passage à l'état read_opb (RESP. COMPARE), sinon on ne fait rien donc blocage sur l'état actuel r_fsm.
COMPARE:
selon le resultat de la comparaison entre r_opa et r_opb on transite vers l'état DECR_A ou DECR_B ou vers write_res si r_opa == r_opb.
WRITES_RES:
a la fin de l'ecriture vers le port sortie du pipe on transite vers READ_OPA l'état initial.

**genMoore**:
On lit l'état de l'automate. A l'atat READ_A (resp. READ_B), l'attribut read de p_in doit être à 1 car c'est dans cet état qu'on effectue une lecture. Cependant, l'attribut write de p_out doit être à 0.
Dans les états COMPARE, DECR_A et DECR_B, les attributs read et write des ports doivent tout les deux être à 0 car aucun transfert n'est effectué (c'est là qu'on fait la comparaison et la soustraction, donc pas besoin de lire ou d'écrire).
Par conséquent, pour tout état ou il n'y a pas de transfert vers l'autre automate, p_out.data doit être à 0.
Finalment, pour l'état WRITE_RES, on effectue le traitement inverse que pour READ_OPB. Au lieu de ne rien écrire dans p_out.data, on écrit le résultat contenu dans r_opa (mais ça pourrait être r_opb !).


## 3.2 Ecriture du modèle CABA de la top-cell
```c
FifoSignals<uint32_t> 			signal_fifo_c2m("signal_c2m");
```
Il faut déclarer la FIFO pour l'écriture du résultat du PGCD du coproc au maître.

```c
    FifoGcdMaster 				master("master", seed);
	FifoGcdCoprocessor			coproc("coprocessor");
```
Création des automates. On attribut un nom pour chaque composant et la seed pour le maître (pour générer les valeurs aléatoire).

```c
	coproc.p_clk(signal_clk); 
	coproc.p_resetn(signal_resetn);
	coproc.p_in(signal_fifo_m2c);
	coproc.p_out(signal_fifo_c2m);
```
On relit les ports aux signaux correspondants. C'est pareil que pour master à l'exception des ports d'entrés et sorties qui doivent être reliés dans l'autre sens (la sortie du maître est l'entré du coproc et vice versa).


## 3.3 génération et lancement du simulateur

```
************************ iteration 1
  cycle = 215
  opa   = 1965102536
  opb   = 1639725855
  pgcd  = 1
************************ iteration 2
  cycle = 339
  opa   = 706684578
  opb   = 1926601937
  pgcd  = 1
************************ iteration 3
  cycle = 479
  opa   = 71238646
  opb   = 1147998030
  pgcd  = 2
************************ iteration 4
  cycle = 597
  opa   = 1038816544
  opb   = 940714160
  pgcd  = 16
************************ iteration 5
  cycle = 829
  opa   = 789063065
  opb   = 464968134
  pgcd  = 1
************************ iteration 6
  cycle = 1077
  opa   = 887950355
  opb   = 46124838
  pgcd  = 1
************************ iteration 7
  cycle = 1219
  opa   = 576618719
  opb   = 1428715137
  pgcd  = 1
************************ iteration 8
  cycle = 1519
  opa   = 747317929
  opb   = 80357754
  pgcd  = 1
************************ iteration 9
  cycle = 1721
  opa   = 1344159057
  opb   = 636297752
  pgcd  = 1
************************ iteration 10
  cycle = 1969
  opa   = 1332307227
  opb   = 90589962
  pgcd  = 3
************************ iteration 11
  cycle = 2339
  opa   = 1731248480
  opb   = 1148562729
  pgcd  = 1
************************ iteration 12
  cycle = 2553
  opa   = 1944426409
  opb   = 278880157
  pgcd  = 1
************************ iteration 13
  cycle = 2695
  opa   = 1730141091
  opb   = 1050303144
  pgcd  = 3
************************ iteration 14
  cycle = 3985
  opa   = 1496116049
  opb   = 1851439417
  pgcd  = 1
************************ iteration 15
  cycle = 4131
  opa   = 322615642
  opb   = 1466708419
  pgcd  = 1
************************ iteration 16
  cycle = 4321
  opa   = 2094128415
  opb   = 140234531
  pgcd  = 1
************************ iteration 17
  cycle = 4833
  opa   = 958950626
  opb   = 653329346
  pgcd  = 2
************************ iteration 18
  cycle = 5249
  opa   = 2066836468
  opb   = 1030189272
  pgcd  = 4
************************ iteration 19
  cycle = 5461
  opa   = 1801327376
  opb   = 958169364
  pgcd  = 4
************************ iteration 20
  cycle = 5795
  opa   = 1970903432
  opb   = 442906793
  pgcd  = 1
************************ iteration 21
  cycle = 9459
  opa   = 1423137499
  opb   = 711370139
  pgcd  = 1
************************ iteration 22
  cycle = 9605
  opa   = 489031631
  opb   = 1999756218
  pgcd  = 1
************************ iteration 23
  cycle = 9755
  opa   = 2140085276
  opb   = 1236349560
  pgcd  = 4
************************ iteration 24
  cycle = 9919
  opa   = 2080113972
  opb   = 1336760686
  pgcd  = 2
```
**Quelle est la duréee moyenne d'une itération?**
Nombre de cycles total : `9919`

Nombre d'itération : `24`

Moyenne : $9919\div24 = 413$


**temps d'exécution du programme**
Pour calculer le temps d'exécution, on utilise la commande `time` :
```
time ./simulator.x 10000
```
Résultat :
```
real    0m0,013s
user    0m0,008s
sys     0m0,000s
```

**CASS**
```
************************ iteration 1
  cycle = 215
  opa   = 1965102536
  opb   = 1639725855
  pgcd  = 1
************************ iteration 2
  cycle = 339
  opa   = 706684578
  opb   = 1926601937
  pgcd  = 1
************************ iteration 3
  cycle = 479
  opa   = 71238646
  opb   = 1147998030
  pgcd  = 2
************************ iteration 4
  cycle = 597
  opa   = 1038816544
  opb   = 940714160
  pgcd  = 16
************************ iteration 59919
  cycle = 829
  opa   = 789063065
  opb   = 464968134
  pgcd  = 1
************************ iteration 6
  cycle = 1077
  opa   = 887950355
  opb   = 46124838
  pgcd  = 1
************************ iteration 7
  cycle = 1219
  opa   = 576618719
  opb   = 1428715137
  pgcd  = 1
************************ iteration 8
  cycle = 1519
  opa   = 747317929
  opb   = 80357754
  pgcd  = 1
************************ iteration 9
  cycle = 1721
  opa   = 1344159057
  opb   = 636297752
  pgcd  = 1
************************ iteration 10
  cycle = 1969
  opa   = 1332307227
  opb   = 90589962
  pgcd  = 3
************************ iteration 11
  cycle = 2339
  opa   = 1731248480
  opb   = 1148562729
  pgcd  = 1
************************ iteration 12
  cycle = 2553
  opa   = 1944426409
  opb   = 278880157
  pgcd  = 1
************************ iteration 13
  cycle = 2695
  opa   = 1730141091
  opb   = 1050303144
  pgcd  = 3
************************ iteration 14
  cycle = 3985
  opa   = 1496116049
  opb   = 1851439417
  pgcd  = 1
************************ iteration 15
  cycle = 4131
  opa   = 322615642
  opb   = 1466708419
  pgcd  = 1
************************ iteration 16
  cycle = 4321
  opa   = 2094128415
  opb   = 140234531
  pgcd  = 1
************************ iteration 17
  cycle = 4833
  opa   = 958950626
  opb   = 653329346
  pgcd  = 2
************************ iteration 18
  cycle = 5249
  opa   = 2066836468
  opb   = 1030189272
  pgcd  = 4
************************ iteration 19
  cycle = 5461
  opa   = 1801327376
  opb   = 958169364
  pgcd  = 4
************************ iteration 20
  cycle = 5795
  opa   = 1970903432
  opb   = 442906793
  pgcd  = 1
************************ iteration 21
  cycle = 9459
  opa   = 1423137499
  opb   = 711370139
  pgcd  = 1
************************ iteration 22
  cycle = 9605
  opa   = 489031631
  opb   = 1999756218
  pgcd  = 1
************************ iteration 23
  cycle = 9755
  opa   = 2140085276
  opb   = 1236349560
  pgcd  = 4
************************ iteration 24
  cycle = 9919
  opa   = 2080113972
  opb   = 1336760686
  pgcd  = 2

```

*Remarque : le nombre de cycles moyen d'une itération est le même.*

Temps d'exécution pour fast_simulator :
```
real    0m0,010s
user    0m0,004s
sys     0m0,005s
```
