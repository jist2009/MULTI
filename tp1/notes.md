# Prototypage virtuel & Protocole PiBus

## A. Objectifs
    - technique de modélisation de matériel en langage SystemC
    - analyse protocol du PiBus, pas de proc prog

plateforme de prototypage SocLib -> modélise et simule archi matérielles intégrées sur puce.

Comprendre le protocole de PiBus :
    - maitre qui est un automate cablé qu'une tache : boucle infinie de 4 taches:
        1. lecture 4*32bits
        2. afficher 16 transactions 
        3. attente écriture clavier 
        4. lecture caractère tapé 

le PiBus transport que 1 octet à la fois.

struct TTY: 4 registres de 32 bits. DISPLAY, STATUS, KEYBUF, CONFIG

point de vue maitre : transaction simple (32 bits) nécessite au minimum trois cycles:
    - cycle allocation 
    - cycle commande 
    - cycle réponse 

transaction en rafale = pipeline -> transfert de l'adresse (i+1) , transfert de donnée (i)

l'adresse comporte : bits fort -> cible particulière , bits faible -> octet particulier dans la cible 

PibusSegBcu -> bits fort -> désigne cible 
table des segments -> découpage en segment de l'espace adressage physique

## B Pour démarrer 

    prendre les fichiers
    TP sur ordi de la fac car : /users/outil/soc/soclib-lip6/pibus

## C Automate du composant PibusSimpleRam

4 Ports d'entrée , 1 port de sortie , 1 port de bi-directionnel DT

### Automate RAM 

input : SEL, READ, GO, DELAY, ADR_OK
SEL : cible choisi par le maitre 
READ : sens de l'échange 
Adresse_OK : adresse
OPC : non utilisé 
Delay : 



## F Modélisation de l'architecture matérielle

1. Declaration des signaux permettant d'interconnecter les composants
2. definition de la table des segments 
3. definition et paramétrisation des composants 
4. définition de la net-list
5. lancement de la simulation 

### Q F1 : instancier et connecter 2 composants matériels : PibusSimpleMaster, PibusSimpleRam 

besoin : ram.h master.h,
instanciation faites 

table de segments => table associative avec 5 caractéristiques : 
- nom de segment 
- adresse de base
- taille (en octet)
- index de la cible à laquelle il est associé 
- Booleen si segment cachable 

### Q F2 : définir segment associé au TTY 

c fait 

### Q F3 : init "Hello world!" 

////////////////////////////////////////////////////////////////////////
void write_seg(uint32_t* tab, size_t index, uint32_t data, uint32_t opc)
{
    // The memory organisation is litle endian
    switch (opc) {
    case PIBUS_OPC_BY0 :  // write byte 0 
	tab[index] = (tab[index] & 0xFFFFFF00) | (data & 0x000000FF);
        break;
    case PIBUS_OPC_BY1 :  // write byte 1 
	tab[index] = (tab[index] & 0xFFFF00FF) | (data & 0x0000FF00);
        break;
    case PIBUS_OPC_BY2 :  // write byte 2 
	tab[index] = (tab[index] & 0xFF00FFFF) | (data & 0x00FF0000);
        break;
    case PIBUS_OPC_BY3 :  // write byte 3 
	tab[index] = (tab[index] & 0x00FFFFFF) | (data & 0xFF000000);
        break;
    case PIBUS_OPC_HW0 :  // write lower half  
	tab[index] = (tab[index] & 0xFFFF0000) | (data & 0x0000FFFF);
        break;
    case PIBUS_OPC_HW1 :  // write upper half 
	tab[index] = (tab[index] & 0x0000FFFF) | (data & 0xFFFF0000);
        break;
    case PIBUS_OPC_WDU :  // write word
	tab[index] = data;
        break;
    case PIBUS_OPC_NOP :  // no write  
        break;
    default :
	printf("ERROR in PibusSimpleRam\n");
	printf("illegal value of the PIBUS OPC field for a WRITE : %0x\n", opc);
	printf("the supported values are : BY0/BY1/BY2/BY3\n");
	printf("                           HW0/HW1/WDU/NOP\n");
	exit(1);
	break;
    } // end switch 


Initialisation de la chaine de caractère : 
La RAM envoie vers le TTY la data , "Hello World!" il contient 12 caractères = 3 mots.
Or le bus transfert 1 mot à chaque fois, donc il y a 3 transfert.
La vérification se fait dans 





















# G1 : 1000000 cycles / 7.5 secondes = 133333 cycles par secondes
# G2 : Combien y-a-t-il de cycles d'attente dans les états de l'automate du composant maître
    2 cycles d'attente dans l'état maitre

# G 3 : 
    2 cycles d'attente en READ_WAIT

# G4 : Combien faut-il de cycles à l'automate du composant maître pour afficher un caractère sur le composant PIBUS_MULTI_TTY ? 
    3 cycles : STS_REQ, STS_AD, STS_DT

#  Question G5 : Dessinez le chronogramme correspondant aux 20 premiers cycles d'exécution, en représentant : les états internes des 4 automates des 4 composants, ainsi que les signaux du Pibus : REQ, GNT, SEL_RAM, SEL_TTY, A, LOCK, READ, D, ACK. 
