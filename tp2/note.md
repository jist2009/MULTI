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

### Question C2 : Pourquoi le segment seg_reset n'est il pas assigné au même composant matériel que les 6 autres segments mémoire seg_kcode, seg_kdata, seg_kunc, seg_stack, seg_code, seg_data

###  Question C3 Expliquer pourquoi le segment seg_tty doit être non cachable.

--> pour le tty, il génère des données différents mais sur la meme adresse, si existance cache, donnée dans cache jamais changé car le cache voit que la nouvel donnée a la meme adresse que celle dans le cache, donc pourquoi changez. ### Donc le cache ne vérifie pas qu'ils n'ont pas la meme valeur --> je pense qu'on a pas inventé ce système car coute cher et le problème n'existe pas 

###  Question C4 Parmi les 8 segments utilisés dans cette architecture, quels sont les segments protégés (c'est à dire accessibles seulement quand le processeur est en mode superviseur). Comment est réalisée cette protection ? 
adresse > 0x7FFF FFFF = protégé, sinon utilisateur
segments protégés : seg_kcode, seg_kunc, seg_kdata, seg_tty, seg_reset
dans le processus, on peut changer de mode (je ne retrouve plus la fonction est la variable)
(cours dans L3S6)

---------------------------------------------
Il faut chargé en mémoire le code binaire exécutable par le loader(un programme) pris dans le disk. 

-----------------------------------
il faut créer le loader 

### executé le code 



# D. Système d'exploitation: GIET

GIET = Gestionnaire d'Interruptions, Exceptions et Trappes (= appels systèmes)
--> non réaliste car manque mémoire virtuelle

# je n'ai pas les fichiers, doit prendre depuis l'univ 






# E. Génération du code binaire 

# F. Execution 
