- [Livrables](#livrables)

- [Échéance](#%c3%89ch%c3%a9ance)

- [Quelques éléments à considérer](#quelques-éléments-à-considérer-pour-les-parties-2-et-3-)

- [Travail à réaliser](#travail-%c3%a0-r%c3%a9aliser)

# Sécurité des réseaux sans fil

## Laboratoire 802.11 Sécurité WPA Entreprise

__A faire en équipes de deux personnes__

### Objectif :

1.	Analyser les étapes d’une connexion WPA Entreprise avec une capture Wireshark
2.	Implémenter une attaque WPE (Wireless Pwnage Edition) contre un réseau WPA Entreprise
1.  Implémenter une attaque GTC Dowgrade contre un réseau WPA Entreprise


## Quelques éléments à considérer pour les parties 2 et 3 :

Les parties 2 et 3 nécessitent du matériel particulier. Si vous avez travaillé jusqu'ici avec l'interface WiFi interne de votre laptop, il y a des fortes probabilités qu'elle puisse aussi être utilisée pour les attaques Entreprise. Cela dépendra de la capacité de votra interface d'être configurée en mode AP. Ces attaques ne fonctionnent pas avec toutes les interfaces Alfa. Il faudra utiliser le bon modèle.

En principe, il devrait être possible de démarrer vos machines en Kali natif (à partir d'une clé USB, avec une distro live par exemple) ou d'employer une autre version de Linux si vous voulez utiliser votre propre interface 

## Voici quelques informations qui peuvent vous aider :

- Solution à l’erreur éventuelle « ```Could not configure driver mode``` » :

```
nmcli radio wifi off
rfkill unblock wlan
```
-	Pour pouvoir capturer une authentification complète, il faut se déconnecter d’un réseau et attendre 1 minute (timeout pour que l’AP « oublie » le client) 
-	Les échanges d’authentification entreprise peuvent être facilement trouvés utilisant le filtre d’affichage « ```eap``` » dans Wireshark
-   Il est __impératif__ de bien fixer le cannal lors de vos captures


## Travail à réaliser

### 1. Analyse d’une authentification WPA Entreprise

Dans cette première partie (la moins fun du labo...), vous allez capturer une connexion WPA Entreprise au réseau de l’école avec Wireshark et fournir des captures d’écran indiquant dans chaque capture les données demandées.

A tittre d'exemple, voici [une connexion WPA Entreprise](files/auth.pcap) qui contient tous les éléments demandés. Vous pouvez utiliser cette capture comme guide de ce que la votre doit contenir. Vous pouvez vous en servir pour votre analyse __comme dernière ressource__ si vos captures ne donnent pas le résultat désiré ou s'il vous manquent des éléments importants dans vos tentatives de capture.

Pour réussir votre capture, vous pouvez procéder de la manière suivante :

- Identifier l'AP le plus proche, en identifiant le canal utilisé par l’AP dont la puissance est la plus élevée (et dont le SSID est HEIG-VD...). Vous pouvez faire ceci avec ```airodump-ng```, par exemple

- Lancer une capture avec Wireshark

- Etablir une connexion depuis un poste de travail (PC), un smartphone ou n'importe quel autre client WiFi. __Attention__, il est important que la connexion se fasse à 2.4 GHz pour pouvoir sniffer avec les interfaces Alfa

- Comparer votre capture au processus d’authentification donné en théorie (n’oubliez pas les captures d'écran pour illustrer vos comparaisons !). En particulier, identifier les étapes suivantes :
	- Requête et réponse d’authentification système ouvert
	
  - Requête et réponse d’association (ou reassociation)
  - Négociation de la méthode d’authentification entreprise (TLS?, TTLS?, PEAP?, LEAP?, autre?)
  - Phase d’initiation
  - Phase hello :
  	- Version TLS
  	- Suites cryptographiques et méthodes de compression proposées par le client et acceptées par l’AP
  	- Nonces
  	- Session ID
  - Phase de transmission de certificats
   	- Echanges des certificats
  	- Change cipher spec
  - Authentification interne et transmission de la clé WPA (échange chiffré, vu par Wireshark comme « Application data »)
  - 4-way handshake



#### Requête et réponse d’authentification système ouvert

Requête d'authentification du client :

![image-20220512154532177](figures/image-20220512154532177.png)

Réponse d'authentification de l'AP :

![image-20220512154437407](figures/image-20220512154437407.png)



#### Requête et réponse d’association (ou reassociation)

Requête d'association du client :

![image-20220512154936430](figures/image-20220512154936430.png)

Réponse d'association de l'AP :

![image-20220512155057802](figures/image-20220512155057802.png)



#### Négociation de la méthode d’authentification entreprise

Nous n'avons pas obtenu de paquets concernant la négociation de la méthode d’authentification entreprise donc voici celle du fichier de [capture](files/auth.pcap) fourni. 

Méthode d'authentification proposée par le serveur : 

![image-20220512160537560](figures/image-20220512160537560.png)

Refus du client et proposition d'une autre méthode d'authentification (EAP-PEAP) :

![image-20220512160655300](figures/image-20220512160655300.png)



#### Phase d’initiation

Pour la phase d'initiation, nous avons obtenu la requête venant du serveur mais pas la réponse du client, elle vient donc du fichier de [capture](files/auth.pcap) fourni. 

Requête envoyée par le serveur d'authentification :

![image-20220512161056064](figures/image-20220512161056064.png)

Réponse envoyée par le client où l'on peut voir son identité en clair :

![image-20220512161326207](figures/image-20220512161326207.png)



#### Phase hello

Pour la phase hello, nous avons obtenu le message envoyé par le serveur mais pas celui du client. Nous avons donc utilisé le fichier de [capture](files/auth.pcap) fourni. Bien que nous ayons reçu le message envoyé par le serveur, nous avons décidé de montrer le paquet venant du fichier de capture fourni afin de rester cohérent au niveau des échanges.

Version TLS utilisée par le client :

![image-20220512161909289](figures/image-20220512161909289.png)

Version TLS utilisée par le serveur :

![image-20220512162510312](figures/image-20220512162510312.png)



Méthodes de chiffrement et de compression (ici aucune) proposées par le client :

![image-20220512162032158](figures/image-20220512162032158.png)

Méthode de chiffrement et de compression acceptées par le serveur :

![image-20220512162640048](figures/image-20220512162640048.png)



Nonce du client :

![image-20220512162235928](figures/image-20220512162235928.png)

Nonce du serveur : 

![image-20220512162741080](figures/image-20220512162741080.png)



Session ID du client :

![image-20220512162344568](figures/image-20220512162344568.png)

Session ID du serveur : 

![image-20220512162810222](figures/image-20220512162810222.png)



#### Phase de transmission de certificats

Pour la phase de transmission de certificat, nous avons obtenu ce message envoyé par le serveur :

![image-20220512163542225](figures/image-20220512163542225.png)

Comme EAP-PEAP est utilisé, seul le serveur envoie des certificats au client.



#### Echanges des certificats

Voici le message envoyé par le serveur concernant l'échange de certificats :

![image-20220512163818448](figures/image-20220512163818448.png)



#### Authentification interne et transmission de la clé WPA

On observe bien que Wireshark voit cet échange chiffré comme des paquets «Application data» :

![image-20220512164022332](figures/image-20220512164022332.png)



#### 4-way handshake

Comme nous n'avons pas pu obtenir tous les messages du 4-way handshake WPA lors de notre capture, nous avons utilisé ceux venant du fichier de [capture](files/auth.pcap) fourni :

![image-20220512164125085](figures/image-20220512164125085.png)



### Répondez aux questions suivantes :

> **_Question :_** Quelle ou quelles méthode(s) d’authentification est/sont proposé(s) au client ?
> 
> **_Réponse :_** EAP-TLS

---

> **_Question:_** Quelle méthode d’authentification est finalement utilisée ?
> 
> **_Réponse:_** Comme le client a refusé d'utiliser la méthode EAP-TLS proposée par le serveur, il a donc proposé d'utiliser EAP-PEAP et le serveur a accepté.

---

> **_Question:_** Arrivez-vous à voir l’identité du client dans la phase d'initiation ? Oui ? Non ? Pourquoi ?
> 
> **_Réponse:_** Oui car le client a envoyé son identité en clair. Cependant il n'est pas obligatoire de l'envoyer, le client peut rester anonyme si le serveur d'authentification est configuré pour l'accepter.

---

> **_Question:_** Lors de l’échange de certificats entre le serveur d’authentification et le client :
> 
> - a. Le serveur envoie-t-il un certificat au client ? Pourquoi oui ou non ?
> 
> **_Réponse:_** Oui car on est en EAP-PEAP et avec cette méthode, le serveur doit s'authentifier auprès du client.
> 
> - b. Le client envoie-t-il un certificat au serveur ? Pourquoi oui ou non ?
> 
> **_Réponse:_** Non car on est en EAP-PEAP et seul le serveur doit s'authentifier auprès du client. Par contre, si EAP-TLS avait été utilisée, le client aurait aussi dû présenter un certificat.

---

__ATTENTION__ : pour l'utilisation des deux outils suivants, vous __ne devez pas__ configurer votre interface en mode monitor. Elle sera configurée automatiquement par l'outil en mode AP.



### 2. Attaque WPA Entreprise (hostapd)

Les réseaux utilisant une authentification WPA Entreprise sont considérés aujourd’hui comme étant très surs. En effet, puisque la Master Key utilisée pour la dérivation des clés WPA est générée de manière aléatoire dans le processus d’authentification, les attaques par dictionnaire ou brute-force utilisés sur WPA Personnel ne sont plus applicables. 

Il existe pourtant d’autres moyens pour attaquer les réseaux Entreprise, se basant sur une mauvaise configuration d’un client WiFi. En effet, on peut proposer un « evil twin » à la victime pour l’attirer à se connecter à un faux réseau qui nous permette de capturer le processus d’authentification interne. Une attaque par dictionnaire ou même par brute-force peut être faite sur cette capture, beaucoup plus vulnérable d’être craquée qu’une clé WPA à 256 bits, car elle est effectuée sur le compte d’un utilisateur.

Pour faire fonctionner cette attaque, __il est impératif que la victime soit configurée pour ignorer les problèmes de certificats__ ou que l’utilisateur accepte un nouveau certificat lors d’une connexion. Si votre connexion ne vous propose pas d'accepter le nouveau certificat, faites une recherche pour configurer votre client pour ignorer les certificats lors de l'authentification.

Pour implémenter l’attaque :

- Installer [```hostapd-wpe```](https://www.kali.org/tools/hostapd-wpe/) (il existe des versions modifiées qui peuvent peut-être faciliter la tâche... je ne les connais pas mais si vous en trouvez une qui vous rend les choses plus faciles, vous pouvez l'utiliser et nous apprendre quelque chose ! Dans le doute, utiliser la version originale...). Lire la documentation [du site de l’outil](https://github.com/OpenSecurityResearch/hostapd-wpe), celle de Kali ou d’autres ressources sur Internet pour comprendre son utilisation
- Modifier la configuration de ```hostapd-wpe``` pour proposer un réseau semblable (mais pas le même !!!) au réseau de l’école ou le réseau de votre préférence, sachant que dans le cas d'une attaque réelle, il faudrait utiliser le vrai SSID du réseau de la cible
- Lancer une capture Wireshark
- Tenter une connexion au réseau (ne pas utiliser vos identifiants réels)
- Utiliser un outil de brute-force (```john```, ```hashcat``` ou ```asleap```, par exemple) pour attaquer le hash capturé (utiliser un mot de passe assez simple pour minimiser le temps)

### Répondez aux questions suivantes :

> **_Question :_** Quelles modifications sont nécessaires dans la configuration de hostapd-wpe pour cette attaque ?
>
> **_Réponse :_**  Il faut modifier le fichier hostapd-wpe.conf.
>
> Dans ce fichier, nous avons modifié le SSID en le remplaçant par `HIEG-VD` pour proposer un nom semblable au réseau HEIG-VD :
>
> ![image-20220519135701239](figures/image-20220519135701239.png)
>
> Nous avons également modifié le nom de l'interface par `wlan0` :
>
> ![image-20220526141944758](figures/image-20220526141944758.png)
>
> Puis nous avons modifié le paramètre `eap_fast_a_id_info` pour qu'il corresponde à notre SSID :
>
> ![image-20220519144915722](figures/image-20220519144915722.png)
>
> Nous obtenons finalement un résultat :
>
> ![image-20220519144619572](figures/hostapd-wpe.png)

---

> **_Question:_** Quel type de hash doit-on indiquer à john ou l'outil que vous avez employé pour craquer le handshake ?
>
> **_Réponse:_** On doit lui indiquer le type `NETNTLM` :
>
> ![image-20220519144728243](figures/john.png)
>
> Le mot de passe est donc : 123456

---

> **_Question:_** Quelles méthodes d’authentification sont supportées par hostapd-wpe ?
>
> **_Réponse:_** Selon le GitHub de [OpenSecurityResearch](https://github.com/OpenSecurityResearch/hostapd-wpe), voici les méthodes supportées :
>
> ```
>     1. EAP-FAST/MSCHAPv2 (Phase 0)
>     2. PEAP/MSCHAPv2
>     3. EAP-TTLS/MSCHAPv2
>     4. EAP-TTLS/MSCHAP
>     5. EAP-TTLS/CHAP
>     6. EAP-TTLS/PAP
> ```




### 3. GTC Downgrade Attack avec [EAPHammer](https://github.com/s0lst1c3/eaphammer) 

[EAPHammer](https://github.com/s0lst1c3/eaphammer) est un outil de nouvelle génération pour les attaques WPA Entreprise. Il peut en particulier faire une attaque de downgrade GTC, pour tenter de capturer les identifiants du client __en clair__, ce qui évite le besoin de l'attaque par dictionnaire.

- Installer ```EAPHammer```. Lire la documentation du site de l’outil ou d’autres ressources sur Internet pour comprendre son utilisation
- Modifier la configuration de ```EAPHammer``` pour proposer un réseau semblable au réseau de l’école ou le réseau de votre préférence. Le but est de réaliser une GTC Downgrade Attack.
- Lancer une capture Wireshark
- Tenter une connexion au réseau


### Répondez aux questions suivantes :

> **_Question :_** Expliquez en quelques mots l'attaque GTC Downgrade
>
> **_Réponse :_** Le but de l'attaque GTC downgrade est de forcer le client à utiliser la méthode EAP-GTC demandant au client un one-time password. Le problème dans cette méthode est que la réponse du client est envoyée en clair. De plus, de nombreux appareils clients ne parviennent pas à indiquer le type de mot de passe qu'ils demandent lorsqu'ils demandent aux utilisateurs des credentials. C'est pourquoi ceux-ci vont présenter un formulaire de login générique à l'utilisateur, qui va penser qu'il doit entrer ses credentials de connexion au réseau. Ils passeront donc en clair sur le réseau et pourront être interceptés par l'attaquant.
>
> Voilà par exemple ce que nous avons pu récupérer avec EAPHammer:
>
> ![image-20220519195113252](figures/image-20220519195113252.png)

---

> **_Question:_** Quelles sont vos conclusions et réflexions par rapport à la méthode hostapd-wpe ?
> 
> **_Réponse:_** Hostapd était bien plus facile à installer et à faire fonctionner que EAPHammer. Par contre, EAPHammer était plus facile à utiliser. En effet, tout peut se faire en ligne de commande alors qu'avec Hostapd il est nécessaire de modifier un fichier de configuration. De plus, il permet de nous fournir directement le password en clair. Avec Hostapd, nous sommes obligés d'utiliser John pour l'obtenir.




### 4. En option, vous pouvez explorer d'autres outils comme [eapeak](https://github.com/rsmusllp/eapeak) ou [crEAP](https://github.com/W9HAX/crEAP/blob/master/crEAP.py) pour les garder dans votre arsenal de pentester.

(Il n'y a pas de rendu pour cette partie...)

## Livrables

Un fork du repo original . Puis, un Pull Request contenant :

-	Captures d’écran + commentaires
-	Réponses aux questions

## Échéance

Le 2 juin 2022 à 23h59
