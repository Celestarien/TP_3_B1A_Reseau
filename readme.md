# B1 Réseau 2018 - TP3
## I. Création et utilisation simples d'une VM CentOS

### 4. Configuration réseau d'une machine CentOS

Ca se présentera à peu près pareil pour beaucoup d'OS Linux, les fichiers sont simplement différents parfois.
Cherchez sur internet afin que votre VM :

* Possède une connectivité à Internet
        c'est l'interface NAT qui fait ça
        aucune configuration nécessaire normalement

* Possède une IP privée permettant de joindre l'hôte

    * C'est l'interface host-only qui s'en occupe
    * vous devrez modifier un seul fichier dans ```/etc/sysconfig/network-scripts``` pour ce faire 
    * Vous pouvez ensuite faire ```« ifdown »``` puis ```« ifup »``` sur l'interface pour l'éteinder puis la rallumer 
    * Un petit ip a pour vérifier que le changement a pris effet

* Une fois fait :
    * a) Pour vérifier que l'on a internet depuis la VM il suffit de ping un site aléatoire qui existe bel et bien

    * b) Pour prouver que le PC hôte et la VM peuvent communiquer vous :
        * Ping la VM avec le PC hôte
        * Et ping le PC hôte avec la Vm

    * c) Pour afficher la table de routage, tapez la commande ```« ip route »```
        * 1ère ligne : Montre la route pour aller partout ailleurs que les lignes 2 et 3
        * 2ème ligne : Montre la route prise par l'interface enp0s3
        * 3ème ligne : Montre la route prise par l'interface enp0s8 

###### Par rapport au b) : Si les 2 pings fontionnent c'est que les 2 machines peuvent communiquer ensemble sinon non (si ça ne fonctionne pas essayez de désactiver vos parfeux)

### 5. Faire joujou avec quelques commandes

Ping la VM avec l'hôte :
###### Les actions suivantes sont à réaliser sur le PC hôte

* Ouvrir un terminal
* Tapez la commande ```« ping <adresse IP de la VM> »```

Ping l'hôte avec la VM :
##### L'action suivante est à réaliser sur la VM
* Tapez la commande ```« ping <addresse IP de l'hôte »```

Afficher la table de routage :
* De l'hôte (sous Windows 10) :
    * Ouvrir un terminal
    * Entrez la commande ```« route print »```

* De la VM :
    * Tapez ```« ip route »```

Les lignes qui permettent de voir si les deux machines peuvent se parler sont :
* Sur l'hôte (Windows 10) :
    * La ligne où il a l'adresse IP de votre machine dans la colonne "Destination réseau" et l'adresse IP de la VM dans la colonne "Adr. interface"
* Sur la VM :
    * La 1ère ligne de la table de routage

Téléchargement avec curl :
* Entrez la commande  ```« curl <lien du téléchargement> »```

Connaître l'IP d'un site :
* Utilisez la commande  ```« dig <URL du site> »```

## II. Notion de ports et SSH

### 1. Exploration des ports locaux

Utilisez la commande ss pour :
* Les ports TCP :
    * Tapez la commande ```« dig -t »```
* Les ports en écoute :
    * Tapez la commande ```« dig -lnt »```

### 2. SSH

Connectez l'ordinateur hôte à la VM en SSH avec putty :
* Ouvrir putty
* Entrez l'adresse IP de votre VM
* Choisissez le port sur lequel vous voulez vous connecter
* Cliquez sur ```Ouvrir```
* Un message d'erreur va apparaître, cliquez sur ```Oui```

### 3. Firewall
#### A.
Modifier le fichier ```/etc/ssh/shhd_config``` :
* Entrez la commande ```« sudo nano /etc/ssh/sshd_config »```
* Cherchez la ligne "Port" et modifier le 22 par 2222 par exemple
* Avec votre clavier faite Ctrl + X, puis O pour confirmer le modification

Redémarrez le serveur SSH :
* Utilisez la commande ```« sudo systemctl restart sshd »```

Vérifiez que le port d'écoute est différent de 22 :
* Entrez la commande ```« dig -lnt »```

Connection au serveur à partir du port 2222 :
* Tapez la commande : ```« ssh -p 2222 user@IP »```

Connexion échouée ?  
Il est normal que la connexion est échouée car le port est bloqué. La solution serait de désactiver les firewalls.  

#### B.
Lancer un serveur netcat sur le port 5454 en tcp avec la commande ```« nc -l 5454 »```  
Se connecter au serveur netcat : ```« nc <hote> <port> »```  

### III. Routage statique

#### 1. Préparation des hôtes (vos PCs)

##### Préparation avec câble (RJ45) :

* Branchez votre câble RJ45 à chaque PC

* Allez dans vos paramètres

* Cliquez sur la catégorie ```« Réseau et Internet »```

* Accédez à ```« Modifier les options d'adaptateur »```

* Double-cliquez sur votre interface Ethernet

* Allez dans ```« Protocole internet version 4 »```

* Cochez ```« Utiliser l'adresse IP suivante : »```

* Et remplissez par ```« 192.168.112.X »``` sur les 2 machines (```X``` est un nombre aléatoire qui ne doit pas être le même pour chacun des PCs)

##### Préparation VirtualBox

* Allez dans VirtualBox

* Accédez à la catégorie ```Fichier```

* Cliquez sur ```« Host Network Manager... »```

* Vérifier que les propriétés sont activées

* Cochez sur ```« Configure Adapter Manually »```

* Et remplissez les différentes options

#### 2. Configuration du routage

Bilan des IP de chacune des machines :

* PC 1 : 192.168.101.21
* PC 2 : 192.168.102.5
* VM 1 : 192.168.101.10
* VM 2 : 192.168.102.10

Vous avez 3 réseaux :
* réseau 1, un /24 entre PC1 et VM1 (host-only 1)
* réseau 2, un /24 entre PC2 et VM2 (host-only 2)
* réseau 12, un /30 entre PC1 et PC2 (câble)


Commandes à réaliser :
* PC 1 : ```« route add <IP_2> mask 255.255.255.0 <IP_12_PC2> »```

* PC 2 : ```« ping <adresse du PC 1 dans le réseau 1> »```

* VM 1 :
    * ```« ip route add <IP_12> via <IP_1_PC1> dev <INTERFACE_1_NAME> »```
    * ```« ping <adresse du PC 2 dans le réseau 12> »```
    * ```« ping <adresse du PC 2 dans le réseau 2> »```

* VM 2 : 
    * ```« ping <adresse du PC 1 dans le réseau 12> »```
    * ```« ping <adresse du PC 1 dans le réseau 1> »```

#### 3. Configuration des noms de domaine

Sous Windows :

* Allez dans vos paramètres

* Cliquez sur ```« Système »```

* Accédez à la catégorie ```« Informations système »```

* Et cliquez sur ```« Renommer ce PC »```