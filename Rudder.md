# ID
| m.tetu                   | 51.15.255.79    | 51.15.193.19     | Formation-S44

Change paswd Blueleo37

clé rudder
pub  4096R/474A19E8 2011-12-15 Rudder Project (release key) <security@rudder-project.org>
      Key fingerprint = 7C16 9817 7904 212D D58C  B4D1 9322 C330 474A 19E8
# Installation de Ruder
```
sudo wget --quiet -O /etc/apt/trusted.gpg.d/rudder_apt_key.gpg "https://repository.rudder.io/apt/rudder_apt_key.gpg"
```

Passer en root 

```
sudo su
```
## Ajout du repo
```
echo "deb http://repository.rudder.io/apt/8.0/ $(lsb_release -cs) main" > /etc/apt/sources.list.d/rudder.list
```

```
apt-get update
```
## Installation rudder

```
apt-get install rudder-server
```
## Création utilisateur 
```
rudder server create-user -u USERNAME
```

Blueleo37

![[Pasted image 20231031165550.png]]
# Install Agent
```
wget --quiet -O /etc/apt/trusted.gpg.d/rudder_apt_key.gpg "https://repository.rudder.io/apt/rudder_apt_key.gpg"
```
pub  4096R/474A19E8 2011-12-15 Rudder Project (release key) <security@rudder-project.org>
      Key fingerprint = 7C16 9817 7904 212D D58C  B4D1 9322 C330 474A 19E8
```
echo "deb http://repository.rudder.io/apt/8.0/ $(lsb_release -cs) main" > /etc/apt/sources.list.d/rudder.list
```
```
apt-get update
```
```
apt-get install rudder-agent
```
## Conf agent 

Configurer l'adresse ip publique du serveur

```
rudder agent policy-server <rudder server ip or hostname>
```

Puis dans l'interface de rudder
autoriser les réseaux 

![[Pasted image 20231102093705.png]]

## Ajouter le noeud
```
rudder agent inventory
```

Accepter l'agent dans l'IHM, la machine où l'agent est installé sera visible ici puis il faudra la sélectionner et "accepter" pour l'autoriser à communiquer
![[Pasted image 20231102093758.png]]

## MAJ de l'agent 
```
rudder agent update
```

## Démarrer l'agent
```
rudder agent run
```

# Lister des users

```
cat /etc/passwd
```

# Ajouter user

Créer les user dans le répertoire cible, ici ce sera dans home. 
sur le serveur 
```
mkdir /home/user01
```
```
useradd -d /home/username -p password username
```

Rudder 
	User01
	![[Pasted image 20231103110703.png]]
	user02
	![[Pasted image 20231103110723.png]]
	![[Pasted image 20231103110806.png]]
## Change psswd
```
passwd username
```
# Modif user à un shell bash

```
vim /etc/passwd
```

Modifier la ligne 

appuyer sur la touche "inser" pour entrer en mode insertion.
Modfier la ligne du user en ajoutant bash à la fin du path

![[Pasted image 20231102101600.png]]

une fois la modif effectuée, appuyer sur "échap" pour quitter le mode insertion puis ":x" pour enregistrer et quitter.
":q!" pour quitter sans enregistrer
":q" quitter

Test user01 
![[Pasted image 20231102103617.png]]

# Modif user no login
Modifier la fin de la ligne du user 
```
/sbin/nologin
```
![[Pasted image 20231102102254.png]]

Test connexion user02
![[Pasted image 20231102103157.png]]
![[Pasted image 20231102103639.png]]
Le compte est bien désactivé
## Définition no login via rudder 
Je créer une directive pour demande à rudder de vérifier que le user est bien dans dans mode.
![[Pasted image 20231103113807.png]]
![[Pasted image 20231103113823.png]]
# Configurer accès sudoers

Ouvrir le fichier /etc/sudoers

```
visudo
```

Ajouter la ligne à la fin du fichier
```
username ALL=(ALL) /usr/bin/nano /var/log/syslog
```
![[Pasted image 20231102111640.png]]

# Modifier port SSH

```
vim /etc/ssh/sshd_config
```
Changer aussi l'autorisation de se connecter en root via ssh, décommenter "PermitRootLongin" et tapez "no"
![[Pasted image 20231102114931.png]]

Redémarrer le service ssh 

```
systemctl restart sshd
```

Pour se connecter à nouveau, il faudra utiliser la commande suivante

ssh user@fqdn or ip adress -p port

```
ssh m.tetu@51.15.255.79 -p 2022
```
![[Pasted image 20231102143901.png]]

# Connexion SSH Keys

## Vérifier si une clé est déjà présente
```
ls -l ~/.ssh/id*
```

Si vous obtenez ceci, c'est qu'aucune clé n'est présente 
![[Pasted image 20231102144652.png]]
## Création de clé 
### Depuis le client Windows 
Ouvrir un terminal/Powershell
```
ssh-keygen
```
Par défaut la clé sera créée dans le fichier C:\Users\micka_o90zgqr/.ssh/id_rsa

Définir une passphrase, celle-ci sécurisera la clé pour afficher le contenu.

### Transférer la clé sur le serveur
Transférer la clé publique sur le serveur linux en copiant le contenu id_rsa dans le .shh dans windows et coller le contenu dans /home/m.tetu/.ssh/authorized_keys

![[Pasted image 20231103000911.png]]
### Connexion ssh key auth depuis windows
Powershell 
```
ssh -i "C:\Users\micka_o90zgqr\.ssh\id_rsa" -p 2022 m.tetu@51.15.255.79
```

![[Pasted image 20231103000832.png]]

Modifier le paramètre "PasswordAuthentification" en no dans sshd-config
```
vim /etc/ssh/sshd_config
```
![[Pasted image 20231103001414.png]]

Copie via rudder
![[Pasted image 20231103113352.png]]
# Config firewall 

Refuser toutes les connexions entrantes
```
ufw default deny incoming
```
Autoriser les sorties
```
ufw default allow outgoing
```

Autoriser ssh 

```
ufw allow ssh
```
Je vais modifier par la suite le port du ssh pour mieux sécuriser l'accès, le n° de port que j'utiliserai sera 2022.
```
ufw allow 2022/tcp
```
Ajouter les ports souhaités à configurer.

Activer le pare-feu. Attention si vous êtes en SSH à bien avoir ouvert le port sinon votre connexion sera coupée.

## Activer le pare-feu
```
ufw enable
```

# MAJ

```
apt update && apt upgrade
```

# Client (nftables rudder)

Ajouter les règles automatiquement sur le client via rudder . La commande bash pour ajouter un port 
```
nft add rule ip filter INPUT tcp dport 81 accept
```
![[Pasted image 20231103110929.png]]
![[Pasted image 20231103110953.png]]
![[Pasted image 20231103111020.png]]
# installer fail2ban
Il permet de bannir des tentatives (ip) de connexions suspectes avec un labs de temps et un nombres max de tentatives
```
apt install fail2ban
```
Voir la config 

```
vim /etc/fail2ban/jail.conf
```
![[Pasted image 20231102121422.png]]

# Config apache 
Apache étant déjà installé sur le serveur, je vais modifier la config du port pour écouter le 81 car sur le 80 j'ai déjà l'IHM de rudder

Via Rudder

## Modifier le port sur apache 
```
vim /etc/apache2/ports.conf
```

Modifier le port d'écoute, j'ai choisi le 81

![[Pasted image 20231102135913.png]]

Modifier le virtual host
```
vim /etc/apache2/sites-available/000-default.conf
```
![[Pasted image 20231102140009.png]]

Vérifier que l'accès fonctionne sur un navigateur 
![[Pasted image 20231102140138.png]]

## Modifier la page apache

Renommer le fichier index.html en .old puis créer un nouveau fichier index.html

```
rm /var/www/html/index.html /var/www/html/index.html.old 
```
```
vim /var/www/html/index.html
```
J'ai configuré mon fichier de cette manière
![[Pasted image 20231102141459.png]]
Puis tester la page
![[Pasted image 20231102141558.png]]

# Audit et conformité 

## Créer une règle

## Droits fichier syslog
Pour mettre en place une règle, il faut dans un premier temps créer une directive.
Dans la liste, sélectionner la rubrique puis cliquer sur "create".
![[Pasted image 20231103005339.png]]
Donner un nom puis mettre le réglage sur "audit"
![[Pasted image 20231103005458.png]]
Ensuite configurer les paramètres 
![[Pasted image 20231103005527.png]]


![[Pasted image 20231103005231.png]]

## SSH
![[Pasted image 20231103002425.png]]
![[Pasted image 20231103002357.png]]

## Synthèse des directives

Sur toute les directives créées seul la config nftables ne fonctionne pas 
![[Pasted image 20231103151518.png]]
## Dashboard
Le tableau permet en un coup d'œil de détecter le niveau de conformité des machines et de cibler précisément les erreurs.

J'ai créé des règles pour montrer à la fois quand tout est conforme et avec des erreurs pour vérifier si la vérification fonctionne.
![[Pasted image 20231103005937.png]]

## Résolution incidents/problèmes

J'ai créé une règle pour vérifier si mes utilisateurs étaient bien présents dans les sudoers, ce qui n'est pas le cas. user01 a uniquement les droits sudo sur le fichier syslog.
Après avoir ajouté les users dans les sudoers file, la directive est conforme.

![[Pasted image 20231103011124.png]]
Correction des permissions sudoers par rudder
![[Pasted image 20231103022912.png]]

nftables non conforme 
activation de nftables pour la remédiation

![[Pasted image 20231103013025.png]]
J'ai activé le service nftables
![[Pasted image 20231103012647.png]]


## Demande simulée
Suite à une faille CVE XXXXX, sur certaines version un patch doit être déployé. En attendant les daemons apache sur les serveur ciblé (les moins importants) devront être stoppés.
Création d'une directive pour stopper un service ciblé. 
Création d'un groupe et d'une nouvelle catégorie.
![[Pasted image 20231103022517.png]]
Création de la règle 
![[Pasted image 20231103022600.png]]

Contrôle des modifications
![[Pasted image 20231103022648.png]]

# Intégration ITIL

## Gestion des services (itsm)
Rudder permet d'automatiser des tâches mais aussi d'installer un cadre avec des procédures car pour mettre en place un audit ou appliquer une action nous devons dans un premier temps créer une directive parmi une liste prédéfinie puis la liée à une règle. En cas de besoin nous pouvons personnaliser des technique et les ajouer à la liste des celles déjà existantes.
Selon ITIL, un processus est un ensemble d'activités coordonnées combinant des ressources et des aptitudes afin de produire un résultat.
Cette méthode décrit que pour cela les effets devront être mesurables, fournir un résultat spécifique, créer une valeur et répondre à un événement spécifique (appel d'un utilisateur ou demande d'action suite à un ticket reçu à la hotline).

## Gestion des configurations (cmdb)
Rudder permet aussi d'organiser la gestion des configuration car cela permet d'obtenir un modèle logique de l'infrastructure (type d'os, status...). La solution permet de facilité par catégorie la planification, modification et identification des changements au sein du service.

# Recommandations

Cet outil polyvalent permet de personnaliser les audits mais aussi d'agir avec efficacité quand cela est nécessaire. Il faut continuer de développer les directives pour automatiser les tâches au sein de l'organisation pour être plus efficace gagner du temps pour traiter des incidents ou problèmes plus complexes. Cela améliorera aussi la satisfaction utilisateurs/clients. Il nous aidera a anticiper des anomalies et à nous alerter sur des indicateurs définis.
En se basant sur la méthode recommandée par ITIL, l'entreprise devra faire un inventaire de ses serveurs en déployant l'agent sur toutes les machines qui fournira remontera rapidement les données nécessaires pour des actions de remédiations via Rudder (directives) ou manuel sur les machines non conformes. En consolidant la base de connaissances et améliorer la configuration de Rudder, la communication, gestion des incidents/problèmes, gestion des changements et la gestion des configurations seront améliorées tant dans l'efficience que dans la qualité du traitement.
Il faudra aussi tenir compte d'une meilleure maîtrise des coûts.


# Webographie

conf sudoers 
https://www.it-connect.fr/commande-sudo-comment-configurer-sudoers-sous-linux/
nologin 
https://www.tecmint.com/change-a-users-default-shell-in-linux/
gestion users 
https://doc.ubuntu-fr.org/tutoriel/gestion_utilisateurs_et_groupes_en_ligne_de_commande
ufw 
https://www.it-connect.fr/configurer-un-pare-feu-local-sous-debian-11-avec-ufw/#B_Autoriser_une_plage_de_ports
ssh key authentification
https://doc.fedora-fr.org/wiki/SSH_:_Authentification_par_cl%C3%A9
APT config
https://wiki.debian.org/SourcesList#Configuring_Apt_Sources