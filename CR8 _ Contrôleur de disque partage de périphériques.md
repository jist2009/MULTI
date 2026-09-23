---
title: 'TP8 : Contrôleur de disque & partage de périphériques'

---

# TP8 : Contrôleur de disque & partage de périphériques


## A. Objectifs

    Le premier objectif de ce TP est d'analyser le fonctionnement d'un nouveau contrôleur de périphérique : le composant IOC (Input Output Controller), qui peut être utilisé pour effectuer des transferts de données entre la mémoire et un périphérique de stockage externe (disque magnétique, clé USB, etc...).

    Le second objectif est d'analyser les problèmes     posés par le partage des périphériques quand plusieurs programmes s'exécutent en parallèle sur plusieurs processeurs, et utilisent le même périphérique. L'architecture matérielle est donc l'architecture multi-processeurs générique, déjà utilisée dans les TP5, TP6, et TP7.
    
    
## B. Contrôleur de disque

#### Question B1 : Quelle est la signification des arguments block_size et latency du constructeur du composant matériel PibusBlockDevice ?
    
    block_size : taille d'un bloc disque en octets.

    latency : latence d'accès au disque exprimée en cycles. Elle permet de simuler le temps mécanique d'accès à un vrai disque .


#### Question B2 : Combien de blocs une image occupe-t-elle sur le disque, pour des blocs de 512 octets ?

    Une image fait 128 × 128 = 16384 octets. Avec des blocs de 512 octets :
    
    16384 / 512 = 32 blocs
    
#### Question B3 : Quels sont les registres adressables du contrôleur de disque, et quel effet a une écriture ou une lecture dans chacun de ces registres ?

![registre_ioc](https://hackmd.io/_uploads/By4Q4gw6be.png)


    BUFFER: l'adresse de base du tampon en mémoire
        écriture -> Charge l'adresse du buffer source/destination
    
    LBA: le numéro du premier bloc sur le disque
        écriture -> Charge l'index du premier bloc sur le disque
        
    COUNT: le nombre de blocs à transférer
        écriture -> Charge le nombre de blocs à transférer
        
    OP: le sens du transfert
        écriture -> Démarre l'opération 

    IRQ_ENABLE: l'état d'activation des IRQ
        écriture -> active ou déactive les irqs
        
    SIZE: Le nombre total de blocs adressables sur le disque
    
    STATUS: l'état du contrôleur
        
    BLOCK_SIZE: la taille d'un bloc en octets
    
    

#### Question B4 : Quels sont les différentes valeurs de l'état interne du contrôleur de disque qui peuvent être lues par le logiciel, et quelle est la signification de chacun de ces états ?

    BLOCK_DEVICE_IDLE -> Contrôleur prêt à recevoir une commande
    
    BLOCK_DEVICE_BUSY -> Transfert en cours
    
    BLOCK_DEVICE_READ_SUCCESS -> Transfert du disque à la mémoire terminé avec succès
    
    BLOCK_DEVICE_WRITE_SUCCESS -> Transfert de la mémoire au disque terminé avec succès
    
    BLOCK_DEVICE_READ_ERROR -> Erreur lors d'un transfert
    
    BLOCK_DEVICE_WRITE_ERROR -> Erreur lors d'un transfert
    
    BLOCK_DEVICE_ERROR -> Fatal Error
    
    

## C. Architecture matérielle

![tp8_topcell](https://hackmd.io/_uploads/rkIlHFra-e.png)


#### Question C1 : Pourquoi l'utilisation du composant PibusBlockDevice impose-t-elle d'augmenter la valeur du timeout du composant PibusSegBcu ? Quelle valeur faut-il donner au paramètre timeout du constructeur du composant PibusSegBcu ?
    
     l'utilisation du composant PibusBlockDevice impose d'augmenter le timeout car si le timeout du BCU est trop court, il interrompt la transaction avant que le disque ait pu répondre.
     
    Il faut que le timeout soit supérieur à la latence et ici on voir que la latence est égale à 1000:
        #define IOC_LATENCY  1000  
        
#### Question C2 : Quelles sont les valeurs de l'adresse de base et de la longueur du segment associé au contrôleur de disque (IOC). Compte-tenu du nombre variable de processeurs dans cette architecture, quelle sont les longueurs des segments associés aux composants ICU, TTY et TIMER ?
        
    La valeur de l'adresse du segment associé au contrôleur de disque     est 0x92000000 et la longueur est de 0x00000020.
    
    Pour l'ICU, le TTY et le TIMER les longeur sont :
    
```cpp
    #define SEG_TTY_SIZE   16*nprocs 
    #define SEG_TIM_SIZE   16*nprocs 
    #define SEG_ICU_SIZE   32*nprocs 
 ``` 
#### Question C3 : Combien y a-t-il de composants maitres dans cette architecture ? Combien de composants cibles ?
    
    Dans cette architecture, il y'a composants 2(DMA,IOC) maîtres qui sont en même temps cibles, 4 coeurs qui sont maitres et 6 composants cibles(RAM,ROM,TTY,TIMER,FBF,ICU).
    En tout, il y'a 6 maitres et 8 cibles.
    

#### Question C4 : Sachant que l'architecture générique utilisée dans cette architecture permet de faire varier le nombre de coeurs, analysez le fichier tp8_top.cpp pour déterminer combien de lignes d'interruption entrantes reçoit le composant ICU, en provenance des 4 périphériques TTY, TIMER, DMA et IOC ? Combien possède-t-il de lignes d'interruptions sortantes ? Comment les IRQs provenant des périphériques sont-elles connectées sur les ports IRQ_IN[i] du composant ICU ?

    Dans tp8_top.cpp, on retrouve ces lignes :
```cpp
    icu.p_irq_in[0](signal_irq_dma);
    icu.p_irq_in[1](signal_irq_ioc);
    for (size_t i = 0; i < nprocs; i += 1) {
        icu.p_irq_in[2 + 2 * i](signal_irq_tim[i]);
        icu.p_irq_in[3 + 2 * i](signal_irq_tty_get[i]);
        icu.p_irq_out[i](signal_irq_proc[i]);
    }
```

    2 lignes (pour le DMA et l'IOC) 
    +
    4 * 2 lignes (pour le TIMER ET LE TTY )
    
    Donc il y'a 10 lignes d'interruption entrantes et une ligne sortante.
    
    Elles sont connéctés les ports IRQ_IN[i] du composant ICU de la manière suivante:
     - IRQ_IN[0]    : DMA
     - IRQ_IN[1]    : IOC
     - IRQ_IN[2+2i] : TIMER[i]
     - IRQ_IN[3+2i] : TTY[i]
     
## D. Code de boot

#### Question D1 : Rappelez pourquoi l'initialisation du pointeur de pile dépend du numéro de processeur.
    
    L'initialisation du pointeur de pile dépend du numéro de processeur car le sommet de la pile va changer pour executer son programme et éviter les collisions mémoire entre les piles des différents cœurs s'exécutant en parallèle.
    
    
#### Question D2 : Rappelez le mécanisme général qui permet au système d'exploitation de router - par logiciel - les différentes lignes d'interruption entrantes sur le composant ICU vers différents processeurs.

    Chaque cœur possède son propre registre MASK dans l'ICU. Le système d'exploitation configure ce masque afin de décider quelle interruption va réveiller quel cœur.
     On fait un AND entre le masque et le vecteur d'interruption pour obtenir les irq autorisées.

#### Question D3 : Dans le cas d'une architecture à 4 processeurs, quelles sont les valeurs à stocker dans les 4 registres de masque de l'ICU si on veut réaliser le routage suivant: 

    IRQ_TIMER[0], IRQ_TTY[0] IRQ_DMA et IRQ_IOC vers le processeur 0
    IRQ_TIMER[1], IRQ_TTY[1] vers le processeur 1
    IRQ_TIMER[2], IRQ_TTY[2] vers le processeur 2
    IRQ_TIMER[3], IRQ_TTY[3] vers le processeur 3
    
        
    Le masque fait 32 bits mais dans notre architecture, le vecteur d'interruption fait 10 bits :
    bit 0  → DMA
    bit 1  → IOC
    bit 2  → TIMER[0]
    bit 3  → TTY[0]
    bit 4  → TIMER[1]
    bit 5  → TTY[1]
    bit 6  → TIMER[2]
    bit 7  → TTY[2]
    bit 8  → TIMER[3]
    bit 9  → TTY[3]
    
    MASK[0] -> 0x0000000F
    MASK[1] -> 0x00000030
    MASK[2] -> 0x000000C0
    MASK[3] -> 0x00000300
    
## E. Application logicielle de traitement d'image

#### Question E1 : Quels sont les arguments de l'appel système ioc_read() ? Que fait cet appel système ? La réponse se trouve dans les fichiers stdio.c et drivers.c. Cet appel système attend-il que le transfert soit terminé pour rendre la main ? Dans quel cas cet appel système est-il bloquant ?

    Les arguments de l'appel système ioc_read() sont :
        - lba    : l'indice du premier block sur le disque
        - buffer : L'adresse de base du buffer en memoire
        - count  : le nombre de blocs à transférer
         
    l"appel systeme configure le contrôleur IOC pour lancer un transfert disque/mémoire, puis rend la main immédiatement sans attendre la fin du transfert.
    
    L'appel est bloquant lorsque le controller IOC est déjà occupé .
    
#### Question E2 : Que fait l'appel système ioc_completed() ? Quels sont ses arguments ? Cet appel système est-il bloquant ? 
     Il attend la fin du transfert lancé par ioc_read() ou ioc_write() en attendant l'interruption IRQ levée par le contrôleur IOC quand le transfert est terminé. Il lit ensuite le registre STATUS de l'IOC pour récupérer le code de retour.
     
     
#### Question E3 : Quel problème observez-vous lors de l'affichage des images suivantes ? Expliquez précisément quelle est la cause de ce dysfonctionnement. Indication : la cause du problème est liée au fonctionnement du cache de données.

    Le problème est que les images affichées sont incorrectes ou identiques à la première image malgré le chargement de nouvelles images.
     Le contrôleur IOC (comme le DMA) écrit directement en mémoire sans passer par le cache des processeurs. Quand ioc_read() charge une nouvelle image dans buf_in, le cache du processeur contient encore les anciennes données de l'image précédente. Quand le processeur lit buf_in pour le seuillage, il lit les données du cache obsolète au lieu des nouvelles données écrites par l'IOC en RAM.
     
     
#### Question E4 : Quelles conditions font-elles sortir l'automate SNOOP_FSM de l'état IDLE ? La stratégie mise en oeuvre en cas de hit externe est-elle une mise à jour ou une invalidation ?
    
    La SNOOP_FSM sort de l'état IDLE quand le bus est actif ,la transaction est une écriture et l'adresse écrite correspond à une ligne présente dans le cache 

#### Question E5 : Pourquoi la détection de plusieurs hit externes consécutifs pose-t-elle un problème particulier ? Comment ce problème est-il résolu ?

    

## F. Exécution sur architecture multi-processeurs

##### Question F1 : Sur les trois phases de traitement (chargement, filtrage, affichage), lesquelles vont effectivement pouvoir être parallélisées ?

    Parmis les trois phases, il n'y a que le filtrage qui est parallelisable car chaque processeur s'occupe de sa portion de l'image.
    
#### Question F2 : Le contrôleur IOC ne pouvrant effectuer qu'un seul transfert à la fois, décrivez le mécanisme général qui permet de séquentialiser les 4 transferts demandés par les 4 processeurs.

    Le mecanisme permettant de séquentialiser les 4 transferts demandés par les 4 processeurs est le spinlock 

#### Question F3 : Analysez en détail le code de la fonction _ioc_get_lock() que vous trouverez dans le fichier drivers.c, et expliquez ce que fait ce code.

    

    la fonction ioc_get_lock() permet de prendre le lock sur l'ioc en faisant des instructions assembleur (LL/SC) afin de rendre la lecture et l'écriture atomique 

```asm
    _ioc_llsc:
    ll   $2,    0(%0)        #lit la valeur du verrou
    bnez $2,    _ioc_llsc    #teste si le verrou est déja pris
    li   $3,    1         
    sc   $3,    0(%0)        #Store-Conditional  
    beqz $3,   _ioc_llsc     #si sc a échoué → recommencer depuis le début

```

#### Question F4 : Quelle est la fonction système qui libère le verrou protégeant l'accès exclusif au composant IOC. Pourquoi n'est-il pas nécessaire d'utiliser une instruction particulière pour libérer le verrou ?

    C'est la fontion _ioc_completed() qui libère le verrou avec _ioc_lock =0 .
    
    Il n'est pas nécessaire d'utiliser une instruction particulière pour libérer le verrou car un seul proc le détient donc il n'y a pas de compétition possible
    
## G. Réalisation matérielle du LL/SC

#### Question G1 : Pour quelle raison réalise-t-on cet enregistrement du côté du processeur plutôt que du côté de la mémoire ?

    On réalise enregistrement du côté du processeur pour ne pas surcharger la mémoire qui est une ressource partagée par tous. Ainsi, chaque processeur gère son propre enregistrement.
    
### Question G2 : Dans le scénario décrit ci-dessus, comment le processeur P0 est-il informé de l'écriture effectuée par P1 ?

    C'est le snoopy de P0 qui informe l'informe qu'une écriture est effectué, car il surveille les transaction qui ont lieu sur le bus et compare les adresses avec celle de son enregistrement.
 
#### Question G4 : Résumez en une phrase les deux utilisations du mécanisme de snoop qui on été mis en évidence dans ce TP.

    Dans ce tp, le mécanisme de snoop est utilisé pour la 
    cohérence de cache, afin d'invalider une ligne de cache qui aurait été modifier par l'IOC ou le DMA pour éviter aux procs de lire des valeurs anciennes et pour la synchronisation entre processeurs, grâce aux mécanisme de spinlock.
    