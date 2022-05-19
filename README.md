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

En principe, il devrait être possible de démarrer vos machines en Kali natif (à partir d'une clé USB, avec une distro live par exemple) ou d'employer une autre version de Linux si vous voulez utiliser votre propre interface.

## Voici quelques informations qui peuvent vous aider :

- Solution à l’erreur éventuelle « ```Could not configure driver mode``` » :

```
nmcli radio wifi off
rfkill unblock wlan
```
-	Pour pouvoir capturer une authentification complète, il faut se déconnecter d’un réseau et attendre 1 minute (timeout pour que l’AP « oublie » le client) 
-	Les échanges d’authentification entreprise peuvent être facilement trouvés utilisant le filtre d’affichage « ```eap``` » dans Wireshark
-   Il est __impératif__ de bien fixer le canal lors de vos captures


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
	
	  **Requête d'authentification**
	
	  ![](media/auth-request.PNG)
	
	  **Réponse d'authentification**
	
	  ![](media/auth-response.PNG)
	
 	- Requête et réponse d’association (ou reassociation)
	
	**Requête de réassociation**
	
	![](media/reassociation-request.PNG)
	
	**Réponse de réassociation**
	
	![](media/reassociation-response.PNG)
	
	- Négociation de la méthode d’authentification entreprise (TLS?, TTLS?, PEAP?, LEAP?, autre?)
	
	  **Proposition de la méthode par le serveur au client (EAP-TLS)**
	
	  ![](media/negoAuth-request.PNG)
	
	  **Refus du client de la méthode proposée et demande de EAP-PEAP**
	
	  ![](media/negoAuth-response.PNG)
	
	  **Proposition de EAP-PEAP de la part du serveur**
	
	  ![](media/negoAuth-response_2.PNG)
	
	  
	
	- Phase d’initiation
	
	  ![](media/initiationPhase.PNG)
	
	- Phase hello :
		
		**Hello du client**
		
		![](media/hello-client.PNG)
		
		**Hello du serveur**
		
		![](media/hello-server.PNG)
		
		- **Version TLS**
		
		  Les encadrés bleus sur les 2 captures ci-dessus montrent que la version TLS proposée par le client est la 1.2, le serveur quant à lui indique la version 1.0 pour les prochains échanges. 
		
		- **Suites cryptographiques et méthodes de compression proposées par le client et acceptées par l’AP**
		
		  Les encadrés rouges sur les 2 captures ci-dessus montrent que le client propose 31 suites cryptographiques, la méthode choisie est **TLS_RSA_WITH_AES_256_CBC_SHA**.
		
		- **Nonces**
		
		  Les encadrés oranges sur les 2 captures ci-dessus montrent le champ *Random* correspondant au nonce du client et respectivement du serveur. 
		
		- **Session ID**
		
		  Les encadrés noirs sur les 2 captures ci-dessus montrent le champ *Session ID* correspondant à l'ID de session du client et respectivement du serveur.
		
	- Phase de transmission de certificats
	
	  **Certificats transmis par le serveur (3 certificats sont envoyés)**
	
	  ![](media/certificat-server.PNG)
	
	 	- Echanges des certificats
		- Change cipher spec
		
		  ![](media/certificat-server-changeCipher.PNG)
		
	- Authentification interne et transmission de la clé WPA (échange chiffré, vu par Wireshark comme « Application data »)
	
	  **Voici les différents paquets *Application data***
	
	  ![](media/auth-interne.PNG)
	
	  
	
	  **Et le détail du premier paquet**
	
	  ![](media/auth-interne-details.PNG)
	
	- 4-way handshake
	
	  ![](media/4way-handshake.PNG)

### Répondez aux questions suivantes :

> **_Question :_** Quelle ou quelles méthode(s) d’authentification est/sont proposé(s) au client ?
> 
> **_Réponse :_** En regardant la capture au point *Négociation de la méthode d’authentification entreprise*, on peut constater que la méthode proposée au client est EAP-TLS mais que celui-ci refuse et propose à la place EAP-PEAP. 

---

> **_Question:_** Quelle méthode d’authentification est finalement utilisée ?
> 
> **_Réponse:_** La méthode finalement utilisée est EAP-PEAP (Visible sur la dernière capture du point *Négociation de la méthode d’authentification entreprise*)

---

> **_Question:_** Arrivez-vous à voir l’identité du client dans la phase d'initiation ? Oui ? Non ? Pourquoi ?
> 
> **_Réponse:_** Il est effectivement possible de voir l'identité du client (*einet\joel.gonin* sur la capture au point *Phase d’initiation*). Cela est dû au fait que le paquet n'est pas chiffré.

---

> **_Question:_** Lors de l’échange de certificats entre le serveur d’authentification et le client :
>
> - a. Le serveur envoie-t-il un certificat au client ? Pourquoi oui ou non ?
>
> **_Réponse:_**
>
> Le serveur envoie 3 certificats au client. Cela lui permet de prouver son identité auprès du client afin d'éviter des attaques de type *Man In The Middle*.
>
> - b. Le client envoie-t-il un certificat au serveur ? Pourquoi oui ou non ?
>
> **_Réponse:_**
>
> Le client n'envoie pas de certificat car la méthode d'authentification utilisée est EAP-PEAP, cette dernière ne nécessite pas l'envoie d'un certificat par le client afin de s'authentifier mais utilise à la place des identifiants. 

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

> Pour réaliser cette attaque, il a été nécessaire de modifier dans le fichier de configuration `hostapd-wpe.conf` les lignes suivantes afin de mettre le SSID de notre choix ainsi que le canal.
>
> ![](media/config-host.PNG)
>
> Ensuite, une fois l'attaque lancée à l'aide de la commande `sudo hostapd-wpe hostapd-wpe.conf`, on va essayer de se connecter à l'AP dont le *ssid* est celui renseigné plus haut avec des credentials (username = `temp` / mot de passe = `root`) et on va avoir l'affichage suivant :
>
> ![](media/hostwpe.PNG)
>
> On constate alors que le `challenge / response` s'affiche. Dans notre cas, on va utiliser `John` afin de pouvoir craquer le mot de passe. Pour ce faire, on crée un fichier `txt` avec la ligne `jtr NETNTLM` de la capture précédente.
>
> On lance ensuite `John` afin d'obtenir les différents crédentials
>
> ![](media/john.PNG)
>
> On remarque alors qu'on a réussi à retrouver les credentials

### Répondez aux questions suivantes :

> **_Question :_** Quelles modifications sont nécessaires dans la configuration de hostapd-wpe pour cette attaque ?
> 
> **_Réponse :_** Il est nécessaire de modifier le nom du SSID ainsi que le canal.

---

> **_Question:_** Quel type de hash doit-on indiquer à john ou l'outil que vous avez employé pour craquer le handshake ?
> 
> **_Réponse:_**  Les informations d'identification MSCHAPv2 sont produites au format NETNTLM (v1) pour `John the ripper`. 

---

> **_Question:_** Quelles méthodes d’authentification sont supportées par hostapd-wpe ?
> 
> **Réponse:** EAP, WPA, WPA2, RADIUS


### 3. GTC Downgrade Attack avec [EAPHammer](https://github.com/s0lst1c3/eaphammer) 

[EAPHammer](https://github.com/s0lst1c3/eaphammer) est un outil de nouvelle génération pour les attaques WPA Entreprise. Il peut en particulier faire une attaque de downgrade GTC, pour tenter de capturer les identifiants du client __en clair__, ce qui évite le besoin de l'attaque par dictionnaire.

- Installer ```EAPHammer```. Lire la documentation du site de l’outil ou d’autres ressources sur Internet pour comprendre son utilisation
- Modifier la configuration de ```EAPHammer``` pour proposer un réseau semblable au réseau de l’école ou le réseau de votre préférence. Le but est de réaliser une GTC Downgrade Attack.
- Lancer une capture Wireshark
- Tenter une connexion au réseau

> Pour réaliser cette attaque, il a été nécessaire en premier lieu de générer un certification
>
> ![](media/hammer-certi.PNG)
>
> On peut ensuite lancer `eaphammer` avec les différents paramètres
>
> ![](media/attack3-1.PNG)
>
> On essaie de s'identifier avec un appareil sur l'AP correspondant
>
> ![](media/attack3-2.PNG)
>
> On constate alors immédiatement que les credentials utilisés sont affichés.


### Répondez aux questions suivantes :

> **_Question :_** Expliquez en quelques mots l'attaque GTC Downgrade
>
> **_Réponse :_** Pour cette attaque, l'adversaire met en place un Evil Twin pour un réseau d'entreprise. Il suggère ensuite l'utilisation de EAP-GTC durant la négociation EAP avec le client qui tente de se connecter au réseau. 
>
> Si le client accepte la méthode d'authentification suggérée, soit une fenêtre va s'ouvrir et inviter l'utilisateur à entrer un mot de passe à usage unique, soit le mot de passe va être transmis en clair automatiquement.

---

> **_Question:_** Quelles sont vos conclusions et réflexions par rapport à la méthode hostapd-wpe ?
> 
> **_Réponse:_**  L'attaque est plus facile à mettre en place que `hostapd-wpe` pour un utilisateur non expérimenté donc elle peut être faite quasiment par n'importe qui (vu que l'on a directement les credentials en "clair"). De plus, avec la création du certificat, il peut paraitre à un utilisateur non-averti qu'il est bien réele (de l'entreprise / institution que l'on essaie de plagier) donc les smartphones / PC / etc.. nous affichant ces infos ont pourrait les prendre pour réeles.


### 4. En option, vous pouvez explorer d'autres outils comme [eapeak](https://github.com/rsmusllp/eapeak) ou [crEAP](https://github.com/W9HAX/crEAP/blob/master/crEAP.py) pour les garder dans votre arsenal de pentester.

(Il n'y a pas de rendu pour cette partie...)

## Livrables

Un fork du repo original . Puis, un Pull Request contenant :

-	Captures d’écran + commentaires
-	Réponses aux questions

## Échéance

Le 2 juin 2022 à 23h59
