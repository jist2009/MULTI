# TP2 : Déploiement de code sur processeur programmable 

## A objectifs 

## C. Modélisation de l'architecture matérielle 

### Q C1 : Quelles valeurs doivent prendre les paramètres icache_words, icache_sets

--> voir le cache pibus_mips32_xcache.cpp
icache_words = 4
dcache_words = 4
icache_sets = 8
dcache_sets = 8  
icache_ways = 1
dcache_ways = 1  
wbuf_deph = 8

### Q C2 : pourquoi le segment seg_reset n'est il pas assigné au meme composant matériel que les 6 autres segments mémoire ? 

seg_reset => boot , rom , 0xBFC0 0000 , 4ko
seg_kcode => OS , ram, 0x8000 0000, 16 ko
seg_kunc => OS, ram , 0x8200 0000, 64 ko
seg_kdata => 0x0040 0000 16 ko
seg_code => 0x0100 0000 , 16 ko
seg_stack => 0x0200 0000 16 ko

hypothèse 1 : pour décharger l'ordinateur, il faut désactiver les autres composants, donc le seg_reset doit etre dans le rom  

réponse : 

1. La persistance (Non-volatilité)

La RAM est une mémoire volatile : elle perd tout son contenu dès que l'alimentation électrique est coupée. Si le code de boot (seg_reset) était stocké en RAM, l'ordinateur se retrouverait "amnésique" à chaque démarrage, sans aucune instruction pour savoir comment initialiser le matériel ou charger le système d'exploitation. La ROM, en revanche, conserve ses données de manière permanente.
2. Le point d'entrée du processeur

Au moment où vous allumez le processeur (le "Reset"), le compteur de programme (Program Counter) pointe par construction matérielle vers une adresse fixe (ici 0xBFC00000).

    Il faut qu'un code valide soit présent à cette adresse dès la première microseconde.

    Comme la RAM est vide au démarrage, seul un composant de type ROM peut fournir ces instructions initiales.



### Q C 3: pourquoi le segment seg_tty doit etre non cachable

non cachable : le transfert depuis ou vers ce segment ne doit pas traversé par un cache. 

1e impression : je pense qu'il peut etre cachable, il y a zero pb. 

réponse : 
    LECTURE 
    - si le segment était cachable : 
        le processeur lirait la première touche, stock dans son cache, lors lectures suivantes, renverrait toujours la meme valeur stoker dans le cache. Car le cache verra que la donnée est toujours la meme adresse. Il considère que meme adresse = meme variable. il ne verra pas que c'est une nouvelle valeur. la 1e val sera un masque.

    - non cachable : Le processeur est forcé d'aller lire directement sur le bus système à chaque fois, garantissant qqu'il voit la donnée la plus récente. 

    ECRITURE 
    - avec cache : processus doit écrire dans le cache avant dans la RAM, décalage entre écran et ton input.
    
    - sans cache : directement dans la RAM. instantanée.


### Q C 4: Parmi les 8 segments utilisés dans cette architecture, quels sont les segments protégés. Comment est réalisée cette protection ?

seg_reset, seg_kcode, seg_kunc, seg_kdata. 

    A. La segmentation de l'espace d'adressage (mécanisme matériel)
        -> user [0x00 à 0x7F] , kernel [0x80 à 0xFF] 

    B. Le processeur possède un indicateur interne STATUS REGISTER

RAM et ROM sont le meme, 
prototypage ? mot oublié
avoir une RAM est compliqué car besoin d'autres composant tq disque (périphérique lent)

chargement de code binaire -> utiliser loader = object c++, argument au constructeur du composant RAM. 

Le constructeur du loader prend lui-même pour argument le chemin définissant le fichier contenant le code binaire. Si le code binaire est contenu dans plusieurs fichiers séparés, il faut donner autant de noms de fichiers que nécessaire. 

sys.bin et app.bin dans soft 

## D Système d'exploitation: GIET 

les fichiers kernel 

### D1 : Quelles informations un programme utilisateur doit il fournir au OS lors syscall? Quelle technique de GIET transmettre ces informations ? 

    syscall a besoin numéro du syscall et 4 arguments si il en a besoin 
    JSP à répondre plus tard 

### D2 : Que Contient les deux tableaux _cause_vector[16] et _syscall_vector[32] ? 

    cause_vector correspond a un tableau dont les valeurs representent les causes des exceptions
    syscall_vector est un tableau de pointeur qui chaque case renvoie vers une fonction du kernel. 

    cause_vector   est initialisé dans le fichier exc_handler.c 
    syscall_vector est initialisé dans le fichier sys_handler.c

### Par quoi ils sont indexés ? 



### D3 : donnez la suite d'appels de fonctions déclenchée par l'appel système proctime()

    chemin de l'appel proctime():
    Dans le main -> t'appel la fonction proctime en mode user
    proctime() -> exe un syscall = interruption SYSCALL_PROCTIME 
    syscall proctime() -> 
    giet -> il regarde appeler handler interruption ou handler système 
    handler systeme -> regarde le numéro de l'int et numéro syscall 
    _proctime()

### D4 : estimation coût du syscall entre le brachement à proctime() et retour fonction 

entre 40 et 50 cycles 


## E. Génération du code binaire 

### E1 Donnez trois raisons qui justifient que le code boot s'exécute nécessairement en mode superviseur 

    1. L'accès direct aux registres de configuration matérielle 
    -> configurer matérielle accessible que en mode superviseur 
    2. La configuration de la protection mémoire (MMU)
    -> il crée les barrières de sécurité , il doit être au dessus de ces barrières. 
    3. mise en place du vecteur d'exceptions et d'interruptions 
    -> 

### E2 Convention (non standard) permettant au code de boot du GIET de récupérer l'adresse de la première instruction de la fonction main() ?


main -> boucle "Hello World" 



# G1 : 1000000 cycles / 7.5 secondes = 133333 cycles par secondes
# G2 : Combien y-a-t-il de cycles d'attente dans les états de l'automate du composant maître
    2 cycles d'attente dans l'état maitre

# G 3 : 
    2 cycles d'attente en READ_WAIT

# G4 : Combien faut-il de cycles à l'automate du composant maître pour afficher un caractère sur le composant PIBUS_MULTI_TTY ? 
    3 cycles : STS_REQ, STS_AD, STS_DT

#  Question G5 : Dessinez le chronogramme correspondant aux 20 premiers cycles d'exécution, en représentant : les états internes des 4 automates des 4 composants, ainsi que les signaux du Pibus : REQ, GNT, SEL_RAM, SEL_TTY, A, LOCK, READ, D, ACK. 


# Question de moi

1. quand je tape le clavier, la donnée qui apparait, qui créait son adresse ? 
-> l'architecte matériel décide de l'adresse. 
-> le clavier dépose le code de la lettre dans un petit tiroir électronique (un registre) situé à l'intérieur du composant TTY. Ce registre est fixé physiquement à l'adresse définie. 

2. Une adresse pour toute la phrase ou par mot ? 
-> en majorité, le matériel ne traite qu'un seul caractère à la fois par adresse. 
pour ce tp, une adresse pour la donnée : 0x9000 0000 lecture de caractère
une adresse pour l'état : 0x9000 000 si une nouvelle touche a été pressée. 


3. Fonctionnement d'une requete vers le tty , par qui , quel chemin , 
