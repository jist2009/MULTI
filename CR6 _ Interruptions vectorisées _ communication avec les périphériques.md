---
title: 'TP6 : Interruptions vectorisées / communication avec les périphériques'

---

# TP6 : Interruptions vectorisées / communication avec les périphériques

## A. Objectifs

    Le but de ce TP est d'analyser les mécanismes de communication par interruptions entre les périphériques et le système d'exploitation. Dans une première partie de ce TP, on illustre sur une architecture bi-processeurs le mécanisme des interruptions vectorisées en utilisant un Timer programmable, capable de générer des interrutions périodiques. Dans une seconde partie, on analyse en détail le mécanisme permettant à un programme de lire des catactères à partir d'un terminal TTY. 

## B. Architecture matérielle


![tp6_topcell_icu](https://hackmd.io/_uploads/ByyMTSt6Wx.png)


## C. Composants périphériques

### Question C1 : Pourquoi le composant PibusMultiTimer est-t-il une cible, et pas un maître sur le bus ? Quelle est la signification de l'argument ntimer du constructeur ? Quels sont les registres adressables de ce composant, quelles sont leurs adresses, et quelle est la fonctionnalité de chacun d'entre eux ?

        Le composant PibusMultiTimer est une cible car chaque processeur va programmer son timer selon la durée de l'interruption horloge voulue.

        ntimer : nombre de timers programmables 

        voici les registres adressables de ce composant :

            
            r_value : s'incremente de 1 a chaque cycle -> adresse = 0 
            r_period : nombre de cycles entre 2 irq -> adresse = 8
            r_running : configure le mode de fonctionnement   -> adresse = 4
            r_irq : permet de reconnaitre l'existence de l'IRQ-> adresse = 12




### Question C2 : Pourquoi le composant PibusIcu est-il une cible sur le bus ? Quelle est la signification de l'argument nirq du constructeur ? Quelle est la signification de l'argument nproc du constructeur ? Dans une architecture multi-processeurs, comment le logiciel peut-il aiguiller la ligne d'interruption connectée à l'entrée IRQ_IN[i] du composant ICU vers le processeur connecté à la sortie IRQ_OUT[j] du composant ICU ? Pour chaque port de sortie IRQ_OUT[i], le composant ICU contient plusieurs registres adressables. Quels sont ces registres ? Quelles sont leurs adresses ? Quelle est la fonctionnalité de chacun d'entre eux ?

        le composant PibusIcu est une cible sur le bus car il est charger de recevoir les interruptions des differents péripheriques.

        nproc : nombre de  ligne d'interruption en sortie( correspond au nombre de processeur sur le Pibus)

        Pour aiguiller la ligne d'interruption au sein de l'ICU, il existe des registre offrant differentes actions pouvant etre applique a la ligne et des portes logique.

        


        - ICU_INT            (0x00)  (Lecture seule)   retourne les 32 IRQ d’entrée.
        - ICU_MASK           (0x04)  (Lecture seule)   retourne la valeur actuelle du masque.
        - ICU_MASK_SET       (0x08)  (Écriture seule)  masque <= masque | wdata.
        - ICU_MASK_RESET     (0x0C)  (Écriture seule)  masque <= masque & ~wdata.
        - ICU_IT_VECTOR      (0x10)  (Lecture seule)   index de l’IRQ active la plus petite(la plus prioritaire).
 





###  Question C3 : Pourquoi l'adresse de base du segment associé au composant PibusIcu doit-elle être alignée sur un multiple de 32*8 octets ? Quel serait le coût matériel de relâcher cette contrainte ?

        L’alignement sur 32*8 octets permet un décodage simplifié.
        Le supprimer implique un surcoût matériel (additionneur + logique) et une baisse des performances.


### Question C4 : En analysant le contenu du fichier tp5_top.cpp, précisez comment ces 4 lignes d'interruption sont connectées sur les ports IRQ_IN[i] du contrôleur ICU. 

        - IRQ_IN[2+2i] : TIMER[i]
        - IRQ_IN[3+2i] : TTY[i]

## D. Lancement des tâches


### Question D2
    1000000:	004012dc 	0x4012dc

    1000004:	004013f0 	tge	v0,zero,0x4f

### Question D3: Comment force-t-on GCC à construire cette table de sauts au début du segment seg_data ? 

    __attribute__ ((constructor)) : permet de metre l'adresse de la premiere ins dans seg_data

### Question D4 : Comment expliquez-vous que le programme de calcul du PGCD reste bloqué sur la saisie de l'opérande X ?    

    les interruptions tty ne sont pas initialisées dans le boot


## E. Activation du Timer

On veut maintenant activer les interruptions provenant du TIMER

### Question E1: Rappelez comment un processeur se branche à la routine ISR pertinente lorsqu'il reçoit une requête d'interruption. Analysez le code contenu dans les fichier giet.s et irq_handler.c, et décrivez la séquence d'appels de fonction entre le branchement à l'adresse 0x80000180 (point d'entrée dans le GIET) et le branchement à la routine _isr_timer.

    Pour chaque IRQ gérée par le processeur
    ○ Initialiser les entrées du vecteur d’interruptions avec la bonne ISR.

    Une ISR ne reçoit aucun argument concernant l’instance du périphérique , mais comme le GIET est statique, l’ISR peut déterminer quelle instance de périphérique utiliser en se basant sur le numéro de cœur.

        – Elle doit lire ou écrire une donnée ou un statut de fin de commande
        – Elle doit toujours acquitter l’IRQ, avec une méthode dépendant du composant.


    -Séquence d'appels de fonction:    _sys_handler -> _int_handler



### Question E2: Que fait la routine d'interruption _isr_timer ? 

    récupère le timer correspondant au processeur:

        proc_id = _procid();
        timer_address = (unsigned int*)&seg_timer_base + (proc_id * TIMER_SPAN);

    Acquitte l’interruption :
    timer_address[TIMER_RESETIRQ] = 0; /* reset IRQ */


    Affiche a quel cycle a eu lieu l'interruption sur le tty:
        _putk("\n\n!!! Interrupt timer received at cycle: ");
        char *buf = "          ";
        int date = (int)_proctime();
        _itoa_dec(date, buf);

        _putk(buf);

### Question E3: 

    - IRQ_IN[2+2i] : TIMER[i]
    - IRQ_IN[3+2i] : TTY[i]
    

