- 
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
	
	  
	
	**Requête**:
	
	Nous voyons que la station (Huawei) envoi une requête d'authentification à l'AP (Cisco).
	
	![](./images/img1.png)
	
	

  **Réponse**:

  ![](./images/img2.png)

  

  - Requête et réponse d’association (ou reassociation)

  **Requête:**

  ![](./images/img3.png)

  

  **Réponse**:

  ![](./images/img4.png)

  

  - Négociation de la méthode d’authentification entreprise (TLS?, TTLS?, PEAP?, LEAP?, autre?)

    1. L'AP propose EAP-TLS

    ![](./images/img5.png)

    

    2. La station refuse, et propose EAP-PEAP

       ![](./images/img6.png)
  
       
  
    3. L'AP accepte cette méthode
  
       ![](./images/img7.png) 
  
    
  
  - Phase d’initiation
  
    Après que l'AP aie demandé l'identité du client, nous pouvons voir cette dernière dans la réponse du client.
  
    ![](./images/img8.png)
  
    
  
  - Phase hello :

    - Version TLS

    - Suites cryptographiques et méthodes de compression proposées par le client et acceptées par l’AP

    - Nonces
  
    - Session ID
  
      
  
    **Client Hello:**
  
    ![](./images/img9.png)
  
    
  
    Suites cryptographiques proposées par le client
  
    ![](./images/img9b.png)
  
    
  
    Méthode de compression proposée par le client: aucune
  
    ![](./images/img9c.png)
  
    
  
    **Server hello:**
  
    Nous pouvons voir la cipher suite choisie par le serveur, ainsi que le fait qu'il utilise une version obsolète de TLS (1.0).
  
    ![](./images/img10.png)
  
    
  
  - Phase de transmission de certificats
  
    Le serveur envoie ses certificats lors du message `server hello`
  
    ![](./images/img11.png)
  
    
  
    - Echanges des certificats
  
    - Change cipher spec
  
    ![](./images/img12.png)
  
    
  
  - Authentification interne et transmission de la clé WPA (échange chiffré, vu par Wireshark comme « Application data »)
  
  ![](./images/img13.png)
  
  
  
  - 4-way handshake
  
  ![](./images/eapol.png)

### Répondez aux questions suivantes :

> **_Question :_** Quelle ou quelles méthode(s) d’authentification est/sont proposé(s) au client ?
> 
> **_Réponse :_** EAP-TLS, puis EAP-PEAP après son refus de EAP-TLS

---

> **_Question:_** Quelle méthode d’authentification est finalement utilisée ?
> 
> **_Réponse:_** EAP-TLS

---

> **_Question:_** Arrivez-vous à voir l’identité du client dans la phase d'initiation ? Oui ? Non ? Pourquoi ?
> 
> **_Réponse:_**  Oui, elle est passée en clair, car le chiffrement n'a pas encore démarré. 

---

> **_Question:_** Lors de l’échange de certificats entre le serveur d’authentification et le client :
> 
> - a. Le serveur envoie-t-il un certificat au client ? Pourquoi oui ou non ?
> 
> **_Réponse:_** Oui, pour pouvoir être identifié.
> 
> - b. Le client envoie-t-il un certificat au serveur ? Pourquoi oui ou non ?
> 
> **_Réponse:_** Non, le client est authentifié grâce à un challenge-response

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
> **_Réponse :_** Il faut ajuster le SSID et l'interface réseau si nécessaire.

---

> **_Question:_** Quel type de hash doit-on indiquer à john ou l'outil que vous avez employé pour craquer le handshake ?
> 
> **_Réponse:_** netntlm

---

> **_Question:_** Quelles méthodes d’authentification sont supportées par hostapd-wpe ?
>
> **_Réponse:_** EAP-FAST/MSCHAPv2, PEAP/MSCHAPv2, EAP-TTLS/MSCHAPv2, EAP-TTLS/MSCHAP, EAP-TTLS/CHAP, EAP-TTLS/PAP.



Capture de la connexion à l'AP

![](./images/l3b.png)



Données reçues par *hostapd-wpe:*

![](./images/l3.png)



Cracking du hash récupéré pour retrouver le mot de passe, ici *password*

![](./images/l3c.png)



### 3. GTC Downgrade Attack avec [EAPHammer](https://github.com/s0lst1c3/eaphammer) 

[EAPHammer](https://github.com/s0lst1c3/eaphammer) est un outil de nouvelle génération pour les attaques WPA Entreprise. Il peut en particulier faire une attaque de downgrade GTC, pour tenter de capturer les identifiants du client __en clair__, ce qui évite le besoin de l'attaque par dictionnaire.

- Installer ```EAPHammer```. Lire la documentation du site de l’outil ou d’autres ressources sur Internet pour comprendre son utilisation
- Modifier la configuration de ```EAPHammer``` pour proposer un réseau semblable au réseau de l’école ou le réseau de votre préférence. Le but est de réaliser une GTC Downgrade Attack.
- Lancer une capture Wireshark
- Tenter une connexion au réseau


### Répondez aux questions suivantes :

> **_Question :_** Expliquez en quelques mots l'attaque GTC Downgrade
>
> **_Réponse :_** On créer un evil twin qui proposera un niveau de sécurité bas: EAP-GTC qui demande un one-time password si utilisé avec EAP-PEAP. Ce password est envoyé en clair à l'AP, et si le client ne fait pas attention, ou si le type de mot de passe demandé n'est pas précisé dans le message de demande affiché au client, il est possible qu'il donne ses identifiants de connexion au réseau.

---

> **_Question:_** Quelles sont vos conclusions et réflexions par rapport à la méthode hostapd-wpe ?
>
> **_Réponse:_**  C'est un outils très puissant. Le fait que la connexion au faux AP échoue (ce qui est normal) peut être un problème, mais probablement que les utilisateurs lambda ne comprendront pas d'où vient le problème et on pourra tout de même récupérer plusieurs identifiants avant d'être découvert.




### 4. En option, vous pouvez explorer d'autres outils comme [eapeak](https://github.com/rsmusllp/eapeak) ou [crEAP](https://github.com/W9HAX/crEAP/blob/master/crEAP.py) pour les garder dans votre arsenal de pentester.

(Il n'y a pas de rendu pour cette partie...)

## Livrables

Un fork du repo original . Puis, un Pull Request contenant :

-	Captures d’écran + commentaires
-	Réponses aux questions

## Échéance

Le 2 juin 2022 à 23h59
