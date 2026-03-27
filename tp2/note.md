# TP2 : Déploiement de code sur processeur programmable 

# A. Objectifs 
arbitre bus, proc, ROM, RAM, TTY
ligne de cache : 4 mots
- miss instruction : proc gelé durant MISS, controleur de cache exe transaction en rafale = lire une ligne de cache
- miss data : proc gelé, controleur transaction rafale = lire ligne de cache
- read uncached : lect donnée non cachable 
### pourquoi une donnée ne doit pas être cachable ? 
--> inutile 
--> pour le tty, il génère des données différents mais sur la meme adresse, si existance cache, donnée dans cache jamais changé car le cache voit que la nouvel donnée a la meme adresse que celle dans le cache, donc pourquoi changez. ### Donc le cache ne vérifie pas qu'ils n'ont pas la meme valeur --> je pense qu'on a pas inventé ce système car coute cher et le problème n'existe pas 

- write : proc non tjs gelé, tampon écriture . écriture un mot sur le pibus.


# C Modélisation de l'architecture matérielle

### Question C1 : Quelles valeurs doivent prendre les paramètres icache_words, icache_sets, icache_ways, dcache_words, dcache_sets, dcache_ways , wbuf_depth pour donner aux caches les caractéristiques demandées ci-dessus?

    size_t icache_ways = 1; // instruction cache number of ways size_t icache_sets = 8; // instruction cache number of sets size_t icache_words = 4; // instruction cache number of words per line size_t dcache_ways = 1; // data cache number of ways size_t dcache_sets = 8; // data cache number of sets size_t dcache_words = 4; // data cache number of words per line

### Question C2 : Pourquoi le segment seg_reset n'est il pas assigné au même composant matériel que les 6 autres segments mémoire seg_kcode, seg_kdata, seg_kunc, seg_stack, seg_code, seg_data

    Le segment seg_reset n'est pas assigné au même composant matériel car c'est un segment mémoire qui ne doit pas disparaître hors-tension puisqu'il est nécessaire au boot de l'architecture.

###  Question C3 Expliquer pourquoi le segment seg_tty doit être non cachable.

   pour le tty, il génère des données différents mais sur la meme adresse, si existance cache, donnée dans cache jamais changé car le cache voit que la nouvel donnée a la meme adresse que celle dans le cache, donc pourquoi changez. ### Donc le cache ne vérifie pas qu'ils n'ont pas la meme valeur --> je pense qu'on a pas inventé ce système car coûte cher et le problème n'existe pas 

###  Question C4 Parmi les 8 segments utilisés dans cette architecture, quels sont les segments protégés (c'est à dire accessibles seulement quand le processeur est en mode superviseur). Comment est réalisée cette protection ? 

  adresse > 0x7FFF FFFF = protégé, sinon utilisateur
  segments protégés : seg_kcode, seg_kunc, seg_kdata, seg_tty, seg_reset
  dans le processus, on peut changer de mode
  
---------------------------------------------
Il faut chargé en mémoire le code binaire exécutable par le loader(un programme) pris dans le disk. 

-----------------------------------
il faut créer le loader 

### executé le code 



# D. Système d'exploitation: GIET

###  Question D1 Quelles informations un programme utilisateur doit-il fournir au système d'exploitation lorsqu'il exécute un appel système ? Quelle est la technique utilisée par le GIET pour transmettre ces informations? 

Le programme utilisateur lors d'un appel système fournit au système d'exploitation:

    le numéro de l'appel système qui permettra de l'identifier
    la liste des arguments de l'appel

Pour cela le GIET effectue un syscall en assembleur en spécifiant les bonnes valeurs aux registres.

GIET = Gestionnaire d'Interruptions, Exceptions et Trappes (= appels systèmes)
--> non réaliste car manque mémoire virtuelle

### Question D2 Ouvrez le fichier giet.s. Que contiennent les deux tableaux _cause_vector[16] et _syscall_vector[32]? Par quoi sont-ils indexés ? Dans quels fichiers ces tableaux sont-ils initialisés ? 

    _cause_vector contient 16 fonctions qui chacune une exception, il est indexés par les numéros d'exception. Ces fonctions sont appelée lors de leur exception respective. Il est initialisé dans exc_handler.c.

    _syscall_vector contient les 32 appels systèmes que l'on peut faire en tant qu'utilisateur. Il est indexé par leur numéro respectif. Ces fonctions sont appelée lors de leur appel système respectif. Il est initialisé dans sys_handler.c.

###  Question D3 En analysant successivement le contenu des fichiers stdio.c, giet.s, sys_handler.c, drivers.c, donnez précisément la suite d'appels de fonctions déclenchée par l'appel système proctime(). 

    proctime() -> syscall -> _sys_handler -> _syscall_vector[0] -> _proctime -> mfc0

###  Question D4 Donnez une estimation du coût (en nombre de cycles) de cet appel système, entre le branchement à la fonction proctime(), et le retour de la fonction. 

    On aura: 1 cycle pour proctime, 24 cycles pour _sys_handler mfc0 -> 1 cycles

    On aura donc environ 26 cycles lors de cet appel système.


# E. Génération du code binaire 

###  Question E1 Donnez trois raisons qui justifient que le code boot s'exécute nécessairement en mode superviseur. 

Le code boot doit s'exécuter en modes superviseur car:

    On a besoin d'initialiser les périphériques
    D'accéder à la mémoire
    De faire des appels systèmes qui ne sont pas disponibles en utilisateur
    Un utilisateur ne doit pas pouvoir modifier le boot de la machine car celà pourrait l'endommager

###  Question E2 Quelle est la convention (non standard) permettant au code de boot du GIET de récupérer l'adresse de la première instruction de la fonction main() ? 

    Pour récupérer l'adresse de la fonction main on prend l'adresse de base des données.

###  Question E3 Que se passe-t-il si les adresses définies dans ces deux fichiers ne sont pas égales entre elles ? Que se passe-t-il si l'adresse construite par le processeur ne correspond à aucun segment défini dans l'architecture ? 

    Si les adresses ne sont pas égales entre elles, le logiciel et le matériel ne pourront pas interagir entre eux. Si le processeur passe une adresse en dehors du code système exécutable, elle sera considérée comme invalide par l'esclave.

###  Question E4 En analysant le contenu du fichier sys.ld, déterminez quels sont les objets logiciels placés dans chacun des 2 segments qui contiennent du code système exécutable : seg_reset, seg_kcode. 

    seg_reset possède le code du boot. seg_kcode possède le code du logiciel et le GIET.

### Question 5 : En analysant le contenu du fichier sys.bin.txt, déterminez la taille effective du code dans les deux segments seg_reset et seg_kcode.

    



### 
# F. Execution 
































