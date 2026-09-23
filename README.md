# Architecture des Systèmes Multi-Processeurs 

Ce dépôt regroupe l'ensemble des travaux pratiques réalisés dans le cadre de l'UE "Architecture des Systèmes Multi-Processeurs". Le projet explore la modélisation matérielle, le prototypage virtuel "au cycle près" et l'interaction entre le matériel et le système d'exploitation embarqué (GIET) sur une architecture basée sur un processeur MIPS32.

## Contenu des Travaux Pratiques
- **TP1: Protocole Pibus & prototypage virtuel:** Modélisation matérielle en System et analyse détaillé du protocole de bus PIBUS avec des automates maîtres et cibles.
- **TP2: Déploiement de code sur processeur programmable:** Utilisation d'un compilateur croisé GCC pour compiler et déployer du code C sur une architecture MIPS32.
- **TP3: Architecture interne du contrôleur de caches L1:** Étude et implémentation des automates de contrôle matériels (ICACHE_FSM, PIBUS_FSM= pour la gestion des défauts de cache.
- **TP4: Caractérisation et dimensionnement des caches:** évaluation des performances (taux de MISS, CPI) en fonction de la capacité, de la largeur et de la profondeur du tampon d'écritures postées.
- **TP5: Partage du bus dans les architectures multi-processeurs:** Utilisation d'un composant Frame Buffer et analyse du gain de performance (speedup) et de la saturation du bus lors du passage à l'échelle (de 1 à 8 processeurs).
- **TP6: Entrées/sorties et Interruptions vectorisées:** Mécanismes de communication avec les périphériques (Timer, TTY) via le routage logiciel et matériel du contrôleur d'interruptions (ICU).
- **TP7: Périphériques à capacité DMA:** Paramétrage et synchronisation d'un contrôleur Direct Memory Access (automates MASTER et TARGET) pour décharger le processeur lors des transferts vidéos.
- **TP8: COntrôleur de disque et partage des périphériques:** Gestion d'un composant IOC (Input Output Controller) multi-blocs, exclusion mutuelle via spinlocks (LL/SC) et mécanisme de cohérence de cache (Snoop).
- **TP9: Applications multi-tâches coopératives:** Synchronisation de tâches (modèle producteur/consommateur) et résolution des problèmes de "deadlock" liés au réordonnancement des intructions et aux caches.
- **TP10: Partage du processeur/ Communication des tâches:**  Multiplexage temporel(round-robin), sauvegarde et restauration des contextes de tâches via les routines d'interruption horloge.

## Prérequis
Languages: C, C++, Assembleur MIPS32\
Modélisation: SystemC\
Plateforme: SoCLib (modèles de simulation de composants matériels)\
Système d'exploitation: GIET (Interruptions, Exceptions)\
Compilation : GCC Cross-Compiler (mipsel-unknown-elf)
