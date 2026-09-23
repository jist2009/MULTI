---
title: 'TP7 : Contrôleur DMA'

---

# TP7 : Contrôleur DMA

## A. Objectifs

Le but de ce TP est d'analyser le fonctionnement d'un périphérique plus complexe que ceux analysés dans le TP6. Un périphérique possédant une capacité DMA (Direct Memory Access) se comporte à la fois comme un maître capable de lire ou d'écrire directement en mémoire, et comme une cible capable - comme n'importe quel périphérique - de recevoir des commandes provenant du système d'exploitation. 


## B. Contrôleur DMA

![tp7_topcell_dma](https://hackmd.io/_uploads/HyhvNuSaWg.png)


#### Question B1 : Quels sont les registres adressables du contrôleur DMA, et quel est l'effet d'une lecture ou d'une écriture dans chacun de ces registres ? Pourquoi l'adresse de base du segment associé au contrôleur DMA en tant que cible doit-elle être alignée sur une frontière de bloc de 32 octets ?


    ![Example Image](Multi_7_DMA_English-2p-1.pdf.png)

    Les registres adressable du controleur DMA sont :

    - SOURCE (0x00) Read/Write Source buffer base address
    - DEST (0x04) Read/Write Destination buffer base address
    - LENGTH/STATUS (0x08) Read/Write Transfer length (bytes) / Status
    - RESET (0x0C) Write Only Software reset & IRQ acknowledge
    - NOIRQ (0x10) Read/Write IRQ disabled when non zeo


    Écrire dans le registre LENGTH initie le transfert
    Écrire dans le registre SOURCE charge l'adresse de debut de la zone memoire à lire
    Écrire dans le registre DEST charge l'adresse de debut de la zone memoire à écrire 
    Écrire dans le registre RESET reinitialise le controleur


#### Question B2 : Quelle est la signification de l'argument burst du constructeur du composant PibusDma ?

    L'argument burst du constructeur du composant PibusDma permet de prciser la longueur de la rafale


#### Question B3 : Pourquoi faut-il deux automates (MASTER_FSM et TARGET_FSM) pour contrôler le coprocessseur DMA ?

![tp7_target_fsm](https://hackmd.io/_uploads/S1S2EOS6bl.png)

![tp7_master_fsm](https://hackmd.io/_uploads/BJVpE_HTbl.svg)



    Le DMA agit à la fois en tant que maître et esclave. Pour chacun de ces comportements, le DMA a besoin d'un automate.
    TARGET_FSM: Réponds au commande du MIPS et gère la configuration des requetes et les lectures des statuts 
    MASTER_FSM: Éxecute le transfert mémoire et s'occupe des requetes sur le BUS.

#### Question B4 : Ce composant matériel contient évidemment d'autres registres que les 5 registres adressables. En analysant le modèle SystemC contenu dans le fichier pibus_dma.cpp , décrivez précisément la fonction de la bascule r_stop.

    La bascule r_stop trouve sont utilité lors de la synchronisation des deux FSMs. Elle reveille notamment le MASTER_FSM lorsqu'elle est mise à 0 par TARGET_FSM lors de l'écriture dans LENGTH et MASTER_FSM la désactive en la mettant à 1 lorsque le transfert est terminé.



#### Question B5 : Complétez le graphe ci-dessous représentant la fonction de transition de l'automate MASTER_FSM du composant PibusDma.
(finir le graphe)



## C. Architecture matérielle

#### Question C1: Quelle est la longueur par défaut d'une rafale en nombre de mots de 32 bits ? Quel est l'avantage d'utiliser des grosses rafales ? Quelle est la conséquence sur le matériel d'une augmentation de la longueur de la rafale ?

    La longueur par défault d'une rafale en nombre de mots de 32 bits est 16.
    -> DMA_BURST 16

    L'avantage d'utiliser des grosses rafales réside dans l'accès au bus. En effet lorsqu'on fait une rafale, on reduit le surcoût lié à l'arbitrage du bus car on y accède qu'une seule fois pour plusieurs transfert.

    Augmenter la longueur maximale de la rafale augmente la taille des registres qui stockent cette longueur.


#### Question C2: Quelle est l'adresse de base du segment associé au périphérique DMA ? quel est son numéro de cible pour le composant BCU ? Le périphérique DMA étant aussi un maître sur le bus, il est connecté au composant BCU par les signaux REQ_DMA et GNT_DMA. Quel est son numéro de maître pour le BCU ? Sur quel port d'entrée du composant ICU est connecté la ligne d'interruption IRQ contrôlée par le DMA ?

    L'adresse de base du segment lié au périphérique DMA est 0x93000000
        -> SEG_DMA_BASE 0x93000000

    Son numéro de cible pour le composant BCU est le 6.
        -> DMA_INDEX 6

    Son numéro de maître pour le BCU est le nombre de processeur soit NPROCS, donc 1.
        -> bcu.p_req[nprocs] (signal_req_dma);
        -> bcu.p_gnt[nprocs] (signal_gnt_dma);

    La ligne d'interruption IRQ controlée par le DMA est connecté au port 0 du composant ICU 
        -> icu.p_irq_in[0](signal_irq_dma);


## D. Application logicielle

#### Question D1 : L'appel système fb_sync_write() n'utilise pas le coprocesseur DMA. Quel composant matériel effectue-t-il le transfert des pixels de l'image entre le tampon mémoire dans l'espace utilisateur et la mémoire video (frame buffer) ? Expliquez pour quoi cet appel système est bloquant. La réponse se trouve dans les fichier stdio.c et drivers.c. 

    Le transfert des pixels de l'image entre le tampon mémoire dans l'espace utilisateur et la mémoire video est effectué par le preocesseur.

    L'appel systeme fb_sync_write est bloquant car lors de son éxécution il effectue une copie de la memoire qui se trouve en espace user. Il doit donc acceder au bus, ce qui lui fait dépenser des cycles 



#### Question D2 : Compilez et exécutez sur le prototype virtuel cette première application logicielle n'utilisant pas le contrôleur DMA. Quelle est la durée de construction d'une image (temps de remplissage du buffer) ? Quelle est la durée d'affichage ? 

    La durée de construction d'une image est de 2 432 456 cycles environ
    La durée de l'affichage est de 406 500 cycles -> ici, il suffit de soustraire le début du build OK au display OK
    Le processeur est monopoliser pensdant toute la durée du transfert sans dma.
 

####  Question D3 : Quelle est la différence entre l'appel système fb_sync_write() et l'appel système fb_write() ? Quelle est l'utilité de l'appel système fb_completed() ? 

    _fb_sync_write()

    * Transfer data from an memory buffer to the frame_buffer device with a
    * memcpy. The source memory buffer must be in user address space.


    _fb_write()
    
    * Transfer data from an memory buffer to the frame_buffer device using a DMA.
    * The source memory buffer must be in user address space.

    La différence entre ces deux appels système reside dans l'utilisation du dma pour effectuer la copie de la memoire en espace user.

     _fb_completed()

    l'appel système fb_completed() permet verrifier que le transfert memoire est terminer. 

####  Question D4 : Quelle est la durée d'affichage d'une image avec le DMA ? 

    #initializes the ICU MASK[0] register
    la    $26,    seg_icu_base
    addiu $26,    $26,    0         # ICU[0]
    li    $27,    0b00000001        # IRQ_DMA[0] sans IRQ_TTY[0] (pour le rajouter -> 0b00001001) 
    sw    $27,    8($26)
```cpp
        fb_write(0, BUF, NLINE * NPIXEL);

        if(fb_completed() != 0){
            tty_printf("\n!!! error in fb_completed syscall !!!\n" );
            exit();
        }
```
        Avec le DMA, la durée d'affichage d'une image est de 42899 cycles environ.

####  Question D5 : Quel défaut observez-vous sur le bord gauche de l'image affichée ? Expliquez précisément la cause de ce dysfonctionnement. 

    Sur le bord gauche de l'image affichée, on observe que les images se chevauchent.
    Ce dysfonctionement est causé par le fait que les deux composants utilisent le meme buffer.
    Dans cette configuration, le DMA est lent car on a réduit la longeur de la rafale à 1 mot de 32 bits ce qui va causer l'écrasement des pixels par les pixels écrit par le processeur. 

####  Question D6 : Comment cette variable est-elle utilisée par les deux appels sytème fb_write() et fb_completed() ? Dans quelle fonction trouve-t-on le code de mise à 1 de la variable _dma_busy ? Dans quelle fonction trouve-t-on le code de mise à 0 ? Dans quel segment est stockée cette variable ? 
    

    Dans fb_write(),la variable _dma_busy est utilisée pour attendre la disponibilité du DMA et pour prendre le verrou et pouvoir écrire dans les registres pour configurer l'écriture.
    
```cpp
        while (_dma_busy[proc_id] != 0)
        {
            delay = (_proctime() & 0xF) << 4;
            for (i = 0; i < delay; i++)
                asm volatile("nop");
        }
```  


    Dans fb_completed(),dans fb_write(),la variable _dma_busy est utilisée afin de detecter la fin du transfert en comparant l'index correspodant au porcesseur avec 0.

```cpp
        while (_dma_busy[proc_id] != 0)
            asm volatile("nop");
```


    Le code de mise à 1 de la variable _dma_busy se trouve dans la fonction fb_write()

    Le code de mise à 0 de la variable _dma_busy se trouve dans la fonction de routine de l'interruption levée par le DMA _isr_dma().

    
    La variable _dma_busy est stockée dans le segment SEG_KUNC_BASE car c'est elle est non cacheable.


## E. Pipeline logiciel

| Composant | Période 1     | Période 2     | Période 3     | Période 4     | Période 5     | Période 6     |
|-----------|---------------|---------------|---------------|---------------|---------------|---------------|
| **PROC**  | Construit [1] | Construit [2] | Construit [3] | Construit [4] | Construit [5] |               |
| **DMA**   |               | Affiche [1]   | Affiche [2]   | Affiche [3]   | Affiche [4]   | Affiche [5]   |


#### Question E1: Il faut synchroniser le pipeline. Quelle condition doit être testées par le logiciel pour passer de la période (n) à la période (n + 1) ?

    Pour passer de la période (n) à la période (n + 1), le logiciel doit tester que l'affichage de l'image (n) terminé.

#### Question E2: Quel est le gain (en nombre de cycles) apporté par le parallélisme pipeline, par rapport a une execution séquentielle ? Comment interprétez-vous ce résultat ?

    Avec le parallélisme pipeline, le nombre de cycles necessaire à l'affichage est d'environ 2899 cycles, on gagne donc 40 000 cycles par rapport à l'execution séquentiel


## F. Traitement des erreurs


#### Question F1 : Pourquoi le système d'exploitation interdit-il que l'adresse du tampon source (dans le cas de l'appel système fb_write()) ou l'adresse du tampon destination (dans le cas de l'appel système fb_read()) appartienne à la zone protégée de l'espace adressable ? Pourquoi ce type d'erreur doit-il absolument être détecté avant que le contrôleur DMA commence à effectuer le transfert ?

    Le système d'exploitation interdit que l'adresse du tampon source ou destination appartienne à la zone protégée car un programme utilisateur ne doit jamais pouvoir accéder à la mémoire réservée au noyau.

    Ce type d'erreur doit être détecter avant que le contrôleur DMA commence à effectuer le transfert car l'operation de transfert est irreversible et qu'il y aura deja une faille de securité sinon.  


#### Question F2 : Quel est le mécanisme qui permet au contrôleur DMA de signaler ce type erreur au programme utilisateur ?

    C'est le mécanisme d'interruption qui permet au contrôleur DMA de signaler ce type erreur au programme utilisateur.

    le DMA détecte une erreur 
        -> le DMA lève une irq 
            -> l'irq est envoyé à l'ICU 
                -> l'ICU le transmet au processeur 
                    -> le processeur traite l'irq via isr_dma() 
                        -> isr_dma() met dma_busy à 0 
                            -> fb_completed() retourne une valeur differente de 0.

    

## G. Amélioration du parallélisme 


| Composant  | Période 1     | Période 2     | Période 3     | Période 4     | Période 5     | Période 6   |
|------------|---------------|---------------|---------------|---------------|---------------|-------------|
| **PROC_0** | Construit [1] | Construit [2] | Construit [3] | Construit [4] | Construit [5] |             |
| **PROC_1** | Construit [1] | Construit [2] | Construit [3] | Construit [4] | Construit [5] |             |
| **PROC_2** | Construit [1] | Construit [2] | Construit [3] | Construit [4] | Construit [5] |             |
| **PROC_3** | Construit [1] | Construit [2] | Construit [3] | Construit [4] | Construit [5] |             |
| **DMA**    |               | Affiche [1]   | Affiche [2]   | Affiche [3]   | Affiche [4]   | Affiche [5] |
