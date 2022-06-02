- [Livrables](#livrables)

- [Échéance](#%c3%89ch%c3%a9ance)

- [Quelques éléments à considérer](#quelques-éléments-à-considérer-pour-les-parties-2-et-3-)

- [Travail à réaliser](#travail-%c3%a0-r%c3%a9aliser)

# Sécurité des réseaux sans fil

> Auteurs: Blanc Jean-Luc & Plancherel Noémie

## Laboratoire 802.11 Sécurité WPA Entreprise

__A faire en équipes de deux personnes__

### Objectif :

1.	Analyser les étapes d’une connexion WPA Entreprise avec une capture Wireshark
2.	Implémenter une attaque WPE (Wireless Pwnage Edition) contre un réseau WPA Entreprise
1.  Implémenter une attaque GTC Dowgrade contre un réseau WPA Entreprise


## Quelques éléments à considérer pour les parties 2 et 3 :

Les parties 2 et 3 nécessitent du matériel particulier. Si vous avez travaillé jusqu'ici avec l'interface WiFi interne de votre laptop, il y a des fortes probabilités qu'elle puisse aussi être utilisée pour les attaques Entreprise. Cela dépendra de la capacité de votre interface d'être configurée en mode AP. Ces attaques ne fonctionnent pas avec toutes les interfaces Alfa. Il faudra utiliser le bon modèle.

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
	
	*Pour ce laboratoire, nous n'avons pas réussis à récupérer une capture contenant toutes les informations nécessaires, nous avons donc repris la capture fournie par défaut.*
	
	- Requête et réponse d’authentification système ouvert
	
	![](images/1_1.PNG)
	
	- Requête et réponse d’association (ou reassociation)
	
	![](images/1_2.PNG)
	
	- Négociation de la méthode d’authentification entreprise (TLS?, TTLS?, PEAP?, LEAP?, autre?)
	
	![](images/1_3.PNG)
	
	- Phase d’initiation
	
	![](images/1_4.PNG)
	
	- Phase hello :
	
	  ![](images/1_6.PNG)
	
	  - Version TLS
	
	  ![](images/1_7.PNG)
	
	  - Suites cryptographiques et méthodes de compression proposées par le client et acceptées par l’AP
	
	  ![](images/1_8.PNG)
	
	  - Nonces
	
	  ![](images/1_9.PNG)
	
	  - Session ID
	
	  ![](images/1_10.PNG)
	
	- Phase de transmission de certificats
	
	![](images/1_11.PNG)
	
	- Echanges des certificats
	  
	  ![](images/1_15.PNG)
	  
	  - Change cipher spec
	  
	  ![](images/1_16.PNG)
	  
	- Authentification interne et transmission de la clé WPA (échange chiffré, vu par Wireshark comme « Application data »)
	
	![](images/1_13.PNG)
	
	- 4-way handshake
	
	![](images/1_14.PNG)

### Répondez aux questions suivantes :

Nous nous sommes basés sur la capture Wireshark fournie 

> **_Question :_** Quelle ou quelles méthode(s) d’authentification est/sont proposé(s) au client ?
> 
> **_Réponse :_**  EAP-TLS

---

> **_Question:_** Quelle méthode d’authentification est finalement utilisée ?
> 
> **_Réponse:_**  EAP-PEAP

---

> **_Question: _**Arrivez-vous à voir l’identité du client dans la phase d'initiation ? Oui ? Non ? Pourquoi ?
>
> **_Réponse:_**  Il semble s'agit de Joel Gonin
>
> ![](images/1_5.PNG)

---

> **_Question:_** Lors de l’échange de certificats entre le serveur d’authentification et le client :
> 
> - a. Le serveur envoie-t-il un certificat au client ? Pourquoi oui ou non ?
> 
> **_Réponse:_** Oui, car dans PEAP, le serveur d'authentification doit s'identifier auprès du client, il le fait grâce au certificat.
> 
> - b. Le client envoie-t-il un certificat au serveur ? Pourquoi oui ou non ?
> 
> **_Réponse:_** Non, dans PEAP, le client s'identifie en répondant à un challenge-response.

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
> **_Réponse :_** 

Dans le fichier de configuration `hostpad-wpe`, il est nécessaire de modifier les paramètres suivants:

```
interface=wlp1s0mon
```

Et pour le SSID et le canal du réseau:

```
ssid=heigvd
channel=1
```

On modifie également `eap_fast_a_id_info` pour qu'il corresponde à notre SSID:

```
eap_fast_a_id_info=heigvd
```

---

> **_Question:_** Quel type de hash doit-on indiquer à john ou l'outil que vous avez employé pour craquer le handshake ?
> 
> **_Réponse:_** 

Une fois que nous avons lancé l'attaque avec `hostpad-wpe` de la manière suivante:

```bash
hostapd-wpe /etc/hostapd-wpe/hostapd-wpe.conf
```

Et qu'on essaie de se connecter au réseau `heigvd` avec nos identifiants, nous remarquons que des données ont pu être récupérées:

![2_1](images/2_1.png)

Nous récupérons le username de l'utilisateur, le hash de son mot de passe et finalement le challenge envoyé par le serveur d'authentification.

Le format du hash à indiquer à `john` est de type `NETNLM`.

Ainsi, nous pouvons lancer `john` avec le hash reçu pour trouver le mot de passe de l'utilisateur:

![2_2](images/2_2.png)

Nous voyons que le mot de passe trouvé est `admin123`, ce qui est correct.

---

> **_Question:_** Quelles méthodes d’authentification sont supportées par hostapd-wpe ?
> 
> **_Réponse:_**

D'après [OpenSecurityResearch][https://github.com/OpenSecurityResearch/hostapd-wpe], les méthodes supportées sont les suivantes:

```
1. EAP-FAST/MSCHAPv2 (Phase 0)
2. PEAP/MSCHAPv2
3. EAP-TTLS/MSCHAPv2
4. EAP-TTLS/MSCHAP
5. EAP-TTLS/CHAP
6. EAP-TTLS/PAP
```


### 3. GTC Downgrade Attack avec [EAPHammer](https://github.com/s0lst1c3/eaphammer) 

[EAPHammer](https://github.com/s0lst1c3/eaphammer) est un outil de nouvelle génération pour les attaques WPA Entreprise. Il peut en particulier faire une attaque de downgrade GTC, pour tenter de capturer les identifiants du client __en clair__, ce qui évite le besoin de l'attaque par dictionnaire.

- Installer ```EAPHammer```. Lire la documentation du site de l’outil ou d’autres ressources sur Internet pour comprendre son utilisation
- Modifier la configuration de ```EAPHammer``` pour proposer un réseau semblable au réseau de l’école ou le réseau de votre préférence. Le but est de réaliser une GTC Downgrade Attack.
- Lancer une capture Wireshark
- Tenter une connexion au réseau

Pour lancer l'attaque, on peut utiliser la commande suivante:

```bash
sudo ./eaphammer -i wlan0 --auth wpa-eap --negociate gtc-downgrade -e heigvd --creds
```


### Répondez aux questions suivantes :

> **_Question :_** Expliquez en quelques mots l'attaque GTC Downgrade
> 
> **_Réponse :_** 

L'attaque GTC Downgrade va forcer n'importe quel client qui tente de se connecter à un réseau d'utiliser EAP-GTC durant la négociation EAP. Sur les devices des clients, en général le type d'authentification de mot de passe n'est pas spécifié, ainsi le serveur peut lui même suggérer le type d'authentification. Donc, ceux-ci peuvent présenter un formulaire de connexion générique et les utilisateurs entreront leur credentials en pensant qu'on leur demande les credentials pour se connecter au réseau. Or, ils seront envoyés en clair sur le réseau, dû à la méthode one-time password utilisée dans EAP-GTC et un attaquant pourra les intercepter.

---

> **_Question:_** Quelles sont vos conclusions et réflexions par rapport à la méthode hostapd-wpe ?
> 
> **_Réponse:_** 

Avec `EAPHammer`, le mot de passe récupéré est directement en clair alors qu'avec `hostapd-wpe`, il est nécessaire d'utiliser un outil comme `john` pour récupérer le mot de passe car on reçoit un hash. La configuration de `EAPHammer` est plus simple et rapide que pour `hostapd-wpe`; on peut ajouter tous les arguments directement via la ligne de commande, alors que pour ce dernier, il est nécessaire de modifier tout le fichier de configuration.


### 4. En option, vous pouvez explorer d'autres outils comme [eapeak](https://github.com/rsmusllp/eapeak) ou [crEAP](https://github.com/W9HAX/crEAP/blob/master/crEAP.py) pour les garder dans votre arsenal de pentester.

(Il n'y a pas de rendu pour cette partie...)

## Livrables

Un fork du repo original . Puis, un Pull Request contenant :

-	Captures d’écran + commentaires
-	Réponses aux questions

## Échéance

Le 2 juin 2022 à 23h59
