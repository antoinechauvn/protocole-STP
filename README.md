# spanning-tree-protocol

### Qu'est-ce le Spanning-Tree-Protocol?
```
Le Spanning Tree Protocol (aussi appelé STP) est un protocole réseau de niveau 2 permettant de déterminer 
une topologie réseau sans boucle dans les LAN avec ponts. Il est défini dans la norme IEEE 802.1D et est
basé sur un algorithme décrit par Radia Perlman en 1985
```

# Fonctionnement du Spanning-Tree
L'algorithme du spanning tree procède en plusieurs phases.

* D’abord il va élire le commutateur racine
* Ensuite il va déterminer les ports racines de chaque commutateur.
* Après, il va sélectionner les ports désignés sur chaque segment.
* Et pour finir, il va bloquer les autres ports, pour éviter les boucles.

```
Tout ce processus se fait à l’aide des messages BPDU que s’échangent les commutateurs
Les BPDU’s sont des trames qui sont échangées régulièrement, à peu près environ toutes les 2 secondes, et
permettent aux switches de garder une trace des changements sur le réseau afin d’activer ou de désactiver
les ports des équipements

Il existe trois types de BPDU :

* la configuration BPDU(CBPDU), utilisé pour le calcul du spanning tree
* la notification de changement de topologie (TCN) BPDU, utilisé pour annoncer les changements
topologiques
* l'acquittement de changement de notification de la Topologie (TCA).
```


![image](https://user-images.githubusercontent.com/83721477/163803182-89b02ada-d7f2-47f6-979a-4feb68bfb33b.png)
![image](https://user-images.githubusercontent.com/83721477/163815751-0b5ce7e1-4ee8-40aa-8aba-4940bda135c4.png)

### 1. Élection du commutateur racine
```
Dans un réseau commuté, le root bridge (commutateur racine) est élu. Chaque commutateur a une adresse MAC
et un numéro de priorité paramétrable (0x8000 par défaut), ces deux nombres constituant l'identifiant de
pont (bridge identifier, BID). Le commutateur avec la priorité la plus basse l'emporte, et en cas
d'égalité, c'est l'adresse MAC la plus basse qui l'emporte.
```

![image](https://user-images.githubusercontent.com/83721477/163805286-382d5e6f-7214-4098-890c-bc6e9489e929.png)
![1](https://user-images.githubusercontent.com/83721477/163814466-d9da6934-6add-46bb-9d95-f5196c21c7d9.png)


*Note: Dès que le root-bridge est élu, tous les liens vont se mettre soit en "forwarding", soit en "blocking"* <br>
*Note bis: Tous les ports du root-bridge seront obligatoirement en "forwarding"* <br>

#### Pour chaque port il existe 3 rôles:
* ROOT-PORT
* DESIGNATED PORT
* BLOCKING PORT

```
En général, l'administrateur du réseau influence le résultat de l'élection pour que le commutateur racine
choisi soit le plus près possible du cœur de réseau. Pour cela, il configure la priorité du commutateur
racine le plus opportun en fonction de la topologie du réseau, ainsi que la priorité d'un autre
commutateur qui deviendra commutateur racine en cas de défaillance du commutateur racine principal.
```

### 2. Détermination des ports racine (ROOT-PORT)
```
Lorsqu'un switch reconnait qu'il n’est pas le ROOT, il marque le port sur lequel il reçoit ces BPDU comme
son port racine. L'élection d'un root port est effectuée d'après les champs path cost et port ID
d'un paquet BPDU. C'est le chemin le plus rapide vers le ROOT-BRIDGE.
```

#### Algorithme de définition d'un ROOT-PORT:
* Le port avec la bande passante la plus élevée <br>
Si égalité
* La valeur du Bridge ID la plus faible des switchs connectés <br>
Si égalité
* Le numéro de port (port ID) le plus faible <br>
  le port ID est constitué de deux élements:
  * La priorité du port codée sur 8 bits (0-255)
  * L’identifiant du port codé sur 8 bits, dépendant du matériel donc non modifiable.

### 3. Détermination des ports désignés (DESIGNATED PORT)
```
Tout les segments contenant un ROOT-PORT seront automatiquement mis en designated port.
Les designated port seront en état "forwarding". Les autres lisaisons posséderont un port en
"designated port" et un port en "blocking port"
```

![2](https://user-images.githubusercontent.com/83721477/163815330-513e6bf0-01c1-4c12-9517-c90b2be13569.png)

#### Algorithme de définition d'un DESIGNATED-PORT:
* La liaison avec le coût le plus faible pour attendre le "root-bridge" (root path cost) <br>
Si égalité
* La priorité la plus faible (BRIDGE ID) <br>
Si égalité
* L'adresse MAC la plus faible <br>

### Blocage des autres ports (BLOCKING-PORT)
```
Les ports qui ne sont ni racine, ni désignés sont bloqués. Un port bloqué peut recevoir des paquets
BPDU mais ne peut pas en émettre.
```

![3](https://user-images.githubusercontent.com/83721477/163815561-56c0a7e4-1e6f-4eb5-bffc-49d8f9e8ac75.png)

### Changements de topologie
```
En cas de changement de topologie, lorsqu'un lien est coupé ou qu'un commutateur tombe en panne ou est
éteint, l'algorithme est exécuté à nouveau et un nouvel arbre recouvrant est mis en place. Les boucles
assurent donc de la redondance.
```

Vous avez dû remarquer qu’à chaque fois qu’on branche un câble sur un switch, une LED au-dessus du port, clignote en orange avant de passer au vert. <br>

Quand la LED clignote orange, c’est que le spanning tree est en train de déterminé l’état de l’interface <br>

#### Le port est en mode "écoute" pendant 15 secondes
```
Dans cette phase, il recevra et enverra des BPDU uniquement. Aucune adresse MAC ne sera associée au port
et aucune transmission de donnée n’est possible.
```

#### Ensuite le port passe en mode 'apprentissage' pendant 15 secondes aussi.
```
Dans ce mode, il continue d'envoyer et de recevoir des BPDU, mais cette fois-ci, le switch est capable
d’apprendre les adresses MAC. Les transmissions de données ne sont toujours pas possibles.
```
#### À la fin de ces 15s, Le switch rentre en état de « transfert »
```
C’est dans ce mode qu’il est capable de transférer des données, c’est-à-dire de faire son boulot de
commutation !
```
<br>
L’ensemble de ces étapes ont donc pris 30 secondes pour passer du mode écoute au mode de transfert.

# VARIANTES STP
![STP-Intro-Comparatif](https://user-images.githubusercontent.com/83721477/163820241-23910f35-2929-4443-b9fc-4c5485f0c05e.png)
![image](https://user-images.githubusercontent.com/83721477/166204643-dfae5dc3-3d90-4bc6-a43e-1c408224ba56.png)


Au fur et à mesure que les réseaux commençaient à se développer et à devenir plus complexes, les VLAN ont été introduits, permettant la création de plusieurs réseaux logiques et physiques. Il était alors nécessaire d'exécuter plusieurs instances de STP afin de s'adapter à chaque réseau - VLAN. Ces instances multiples sont appelées Multiple Spanning Tree (MST), Per-VLAN Spanning Tree (PVST) et Per-VLAN Spanning Tree Plus (PVST+).

Afin de prendre en charge les informations VLAN supplémentaires, le champ ID système étendu a été introduit, empruntant 12 bits à la priorité de pont d'origine :

<br>
La valeur Bridge Priority et l' extension Extended System ID constituent ensemble une valeur de 16 bits (2 octets). La priorité de pont constituant les bits les plus à gauche est une valeur comprise entre 0 et 61440 . L' ID système étendu est une valeur de 1 à 4095 correspondant au VLAN respectif participant au STP. La priorité de pont s'incrémente par blocs de 4096 pour permettre à l' extension d'ID système de se faufiler entre chaque incrément. Ceci est clairement montré dans l'analyse ci-dessous :

## PortFast

PortFast est une amélioration de Cisco qui permet à un switch de commencer la communication beaucoup plus rapidement. Il se configure uniquement sur les ports du switch qui sont en mode accès.
Généralement on l’utilise en combinaison avec la protection BPDU, comme cela, dès que le port reçoit une BPDU, il shutdown le port immédiatement.
* PortFast n’est actif qu’à condition de l’interface ne soit pas un trunk
* PortFast désactive le comportement normal de STP et expose donc le switch aux dangers de boucles. Il faut donc s’en prémunir et se servir de fonctionnalités complémentaires

## BPDU GUARD

BPDU = Bridge Protocol Data Unit

Le BPDUGuard permet d’ignorer toutes les trames BPDU reçu sur un port utilisant le PortFast
![image](https://user-images.githubusercontent.com/83721477/166995622-eb1c3966-ea70-4f46-a8c5-ef738c02d71e.png)

BPDUGuard a pour but de désactiver une interface qui recevrait un BPDU alors qu’elle n’aurait pas dû.
Une interface pour laquelle BPDUGuard est activé, et qui reçoit un BPDU, sera placée en état errDisabled (down/down, même état qu’en cas de violation de Port-Security).

Pour être à nouveau fonctionnelle, l’interface devra être manuellement remise à zéro (shutdown / no shutdown) ou alors il faut utiliser les fonctionnalités « errdisable recovery » (ce sera pour un autre article).

Configuration de BPDUGuard:
```
Switch(config)#interface FastEthernet 0/1
Switch(config-if)#spanning-tree bpduguard enable
```
*Note: Ne confondez pas BPDUGuard et BPDUFilter, ce dernier a pour but d’empêcher une interface d’émettre des BPDUs mais aussi de les ignorer quand ils entrent, mais ne résout en rien le problème de base, bien au contraire.*
