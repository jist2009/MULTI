# MULTI

# D'ABORD 

Found no method able to load file soft/app.bin even if tried: 
 elf: failed (Exception: "bad elf file header")
 
## Compilez app.bin
  make dans /soft

## Compilez le fichier tp5_top.cpp pour créer l'exécutable de simulation simul.x 
  soclib-cc -p tp5.desc -t systemcass -o simul.x

./simul.x -NCYCLES 10000000


