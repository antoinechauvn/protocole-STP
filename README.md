# spanning-tree-protocol
Approfondissement du protocole spanning-tree (Norme IEEE 802.1D)

### Qu'est-ce le Spanning-Tree-Protocol?
```
Le Spanning Tree Protocol (aussi appelé STP) est un protocole réseau de niveau 2 permettant de déterminer une topologie réseau sans boucle dans les LAN avec ponts. Il est défini dans la norme IEEE 802.1D et est basé sur un algorithme décrit par Radia Perlman en 1985
```

# Fonctionnement du Spanning-Tree
L'algorithme du spanning tree procède en plusieurs phases.

* D’abord il va élire le commutateur racine
* Ensuite il va déterminer les ports racines de chaque commutateur.
* Après, il va sélectionner les ports désignés sur chaque segment.
* Et pour finir, il va bloquer les autres ports, pour éviter les boucles.

```
Tout ce processus se fait à l’aide des messages BPDU que s’échangent les commutateurs
Les BPDU’s sont des trames qui sont échangées régulièrement, à peu près environ toutes les 2 secondes, et permettent aux switches de garder une trace des changements sur le réseau afin d’activer ou de désactiver les ports des équipements

Il existe trois types de BPDU :

la configuration BPDU(CBPDU), utilisé pour le calcul du spanning tree ;
la notification de changement de topologie (TCN) BPDU, utilisé pour annoncer les changements topologiques ;
l'acquittement de changement de notification de la Topologie (TCA).
```


![image](https://user-images.githubusercontent.com/83721477/163803182-89b02ada-d7f2-47f6-979a-4feb68bfb33b.png)

### 1. Élection du commutateur racine
```
Dans un réseau commuté, le root bridge (commutateur racine) est élu. Chaque commutateur a une adresse MAC et un numéro de priorité paramétrable (0x8000 par défaut), ces deux nombres constituant l'identifiant de pont (bridge identifier, BID). Le commutateur avec la priorité la plus basse l'emporte, et en cas d'égalité, c'est l'adresse MAC la plus basse qui l'emporte.
```

![image](https://user-images.githubusercontent.com/83721477/163805286-382d5e6f-7214-4098-890c-bc6e9489e929.png)

*Note: Dès que le root-bridge est élu, tous les liens vont se mettre soit en "forwarding", soit en "blocking"* <br>
*Note bis: Tous les ports du root-bridge seront obligatoirement en "forwarding"* <br>

#### Pour chaque port il existe 3 rôles:
* ROOT-PORT
* DESIGNATED PORT
* BLOCKING PORT

```
En général, l'administrateur du réseau influence le résultat de l'élection pour que le commutateur racine choisi soit le plus près possible du cœur de réseau. Pour cela, il configure la priorité du commutateur racine le plus opportun en fonction de la topologie du réseau, ainsi que la priorité d'un autre commutateur qui deviendra commutateur racine en cas de défaillance du commutateur racine principal.
```

### 2. Détermination des ports racine (ROOT-PORT)
```
Lorsqu'un switch reconnait qu'il n’est pas le ROOT, il marque le port sur lequel il reçoit ces BPDU comme son port racine.
L'élection d'un root port est effectuée d'après les champs path cost et port ID d'un paquet BPDU. C'est le chemin le plus rapide vers le ROOT-BRIDGE.
```

#### Algorithme de définition d'un ROOT-PORT:
* Le port avec la bande passante la plus élevée <br>
Si égalité
* La valeur du Bridge ID la plus faible <br>
Si égalité
* Le numéro de port (port ID) le plus faible <br>

### 3. Détermination des ports désignés (DESIGNATED PORT)
```
Tout les segments contenant un ROOT-PORT seront automatiquement mis en designated port. Les designated port seront en état "forwarding".
Les autres lisaisons posséderont un port en "designated port" et un port en "blocking port"
```

#### Algorithme de définition d'un DESIGNATED-PORT:
* La liaison avec le coût le plus faible pour attendre le "root-bridge" (root path cost) <br>
Si égalité
* La priorité la plus faible (BRIDGE ID) <br>
Si égalité
* L'adresse MAC la plus faible <br>

### Blocage des autres ports (BLOCKING-PORT)
Les ports qui ne sont ni racine, ni désignés sont bloqués. Un port bloqué peut recevoir des paquets BPDU mais ne peut pas en émettre.

### Changements de topologie
En cas de changement de topologie, lorsqu'un lien est coupé ou qu'un commutateur tombe en panne ou est éteint, l'algorithme est exécuté à nouveau et un nouvel arbre recouvrant est mis en place. Les boucles assurent donc de la redondance.
