- [Livrables](#livrables)

- [Échéance](#%c3%89ch%c3%a9ance)

- [Quelques éléments à considérer](#quelques-éléments-à-considérer-pour-les-parties-2-et-3-)

- [Travail à réaliser](#travail-%c3%a0-r%c3%a9aliser)

# Sécurité des réseaux sans fil

## Laboratoire 802.11 Sécurité WPA Entreprise

__A faire en équipes de deux personnes__

### Objectif :

1. Analyser les étapes d’une connexion WPA Entreprise avec une capture Wireshark
2. Implémenter une attaque WPE (Wireless Pwnage Edition) contre un réseau WPA Entreprise
3. Implémenter une attaque GTC Dowgrade contre un réseau WPA Entreprise

## Quelques éléments à considérer pour les parties 2 et 3 :

Les parties 2 et 3 nécessitent du matériel particulier. Si vous avez travaillé jusqu'ici avec l'interface WiFi interne de votre laptop, il y a des fortes probabilités qu'elle puisse aussi être utilisée pour les attaques Entreprise. Cela dépendra de la capacité de votra interface d'être configurée en mode AP. Ces attaques ne fonctionnent pas avec toutes les interfaces Alfa. Il faudra utiliser le bon modèle.

En principe, il devrait être possible de démarrer vos machines en Kali natif (à partir d'une clé USB, avec une distro live par exemple) ou d'employer une autre version de Linux si vous voulez utiliser votre propre interface 

## Voici quelques informations qui peuvent vous aider :

- Solution à l’erreur éventuelle « ```Could not configure driver mode``` » :

```
nmcli radio wifi off
rfkill unblock wlan
```

- Pour pouvoir capturer une authentification complète, il faut se déconnecter d’un réseau et attendre 1 minute (timeout pour que l’AP « oublie » le client) 
- Les échanges d’authentification entreprise peuvent être facilement trouvés utilisant le filtre d’affichage « ```eap``` » dans Wireshark
- Il est __impératif__ de bien fixer le cannal lors de vos captures

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
  
    **Requête** : 
  ![open_auth_req.png](./assets/82b5ae9846f52f96915c44decd7da01ef7e6e1c0.png)
  
    **Réponse** : 
  
  ![open_auth_response.png](./assets/20d1efd77c208c5f12b9090a385b055ab705abb0.png)

- Requête et réponse d’association (ou reassociation)
  
  **Requête** :
  
  ![a.png](./assets/c261d1640189b9403a3b6c239f14a384f23c09ec.png)
  
  **Réponse** :

![unknown_a.png](./assets/c3dfe3a97190f0362e2e7183e7301a7784bc2b65.png)


- Négociation de la méthode d’authentification entreprise (TLS?, TTLS?, PEAP?, LEAP?, autre?)
  
  On remarque que le client a directement accepté la proposition du serveur (PEAP) car il n'y a qu'un seul échange de ce type :

![auth.png](./assets/f8b683c3f8754ba2b28f908943cb0b627965ad88.png)


- Phase d’initiation
  
  **Requête identité** :
  
  ![id_req.png](./assets/5dab3e02137756b2a329c1d93aed56db9dd377fd.png)
  
  **Réponse identité** :
  
  ![id_res.png](./assets/0c1348610cc81cb7bfc4678e5d3ba41b2b33d4d1.png)

- Phase hello :
  
  - Version TLS
  - Suites cryptographiques et méthodes de compression proposées par le client et acceptées par l’AP
  - Nonces
  - Session ID
  
  **Client hello** :
  
  ![hello.jpg](./assets/6038432d93a3a1bd70bf05c96d18f13fa69b30de.jpg)
  
  Le session ID ne figure pas sur la capture. Nous pensons que cela est dû au fait qu'il s'agit de la première connexion du client qui n'a pas encore de session.
  
  **Server hello** :  ![server_hello.jpg](./assets/3856b20ca5e0db09eebe17a25e92792045c6b058.jpg)
  

- Phase de transmission de certificats
  
  - Echanges des certificats
    
    ![certif_exch.png](./assets/ed55739ccab7ad5284a577ff8c4e281525c6568f.png)
    
    On peut lire les informations suivantes sur le certificat : <<<<<<< HEAD<<<<<<< HEAD
        ![certif.png](./assets/85513863dda99c0e8a760852a769a92c9fb23c73.png)

- Change cipher spec
  
  ![change_spher_spec.png](./assets/26debd0ca2c06b3f3c3227fc24e2d94985f769f2.png)

- Authentification interne et transmission de la clé WPA (échange chiffré, vu par Wireshark comme « Application data »)
  
  ![wpa_exchange.png](./assets/5d3e8cc54a2a56c8b9c93d61e90c7ee533537e28.png)
  =======

- 4-way handshake
  
  ![4-way.png](./assets/10e820b82a795d4f52c826e3f4f4e9895ea6758a.png)
  
  Le premier message comme exemple :  ![4-way-1.png](./assets/758c9ff5c7431ad6e235117d7e70234c63e66661.png)
  

### Répondez aux questions suivantes :

> **_Question :_** Quelle ou quelles méthode(s) d’authentification est/sont proposé(s) au client ?
> 
> **_Réponse :_** PEAP

---

> **_Question:_** Quelle méthode d’authentification est finalement utilisée ?
> 
> **_Réponse:_** PEAP

---

> **_Question:_** Arrivez-vous à voir l’identité du client dans la phase d'initiation ? Oui ? Non ? Pourquoi ?
> 
> **_Réponse:_** Oui, elle est donnée par le client dans le champ Identity lors de la réponse.

---

> **_Question:_** Lors de l’échange de certificats entre le serveur d’authentification et le client :
> 
> - a. Le serveur envoie-t-il un certificat au client ? Pourquoi oui ou non 
>   
>   **Réponse:**
>   
>   Oui, il envoie une chaîne de certificats : 
>   
>   ![certifs_wesh.png](./assets/1647fde5b219449ade18d3824195a253e18c7f8c.png)
>   
>   Cela lui permet de prouver son identité auprès du client qui va vérifier la chaîne de certificats.
> 
> - b. Le client envoie-t-il un certificat au serveur ? Pourquoi oui ou non ?**Réponse:**
>   
>   Non, dans PEAP le client ne prouve pas son identité par un certificat. Il le fera à l'aide d'un processus d'authentification interne (EAP-MSCHAP-V2).

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

**Résultats de l'attaque**

![ah.png](./assets/fbd8bb81df1285a7725ab785e5294cff11b9f13f.png)

**Résultats après brute-force avec john**

![unknown.png](./assets/4e2f5a1686a8d5801fab5aa5e2c1046d9a82dfee.png)

### Répondez aux questions suivantes :

> **_Question :_** Quelles modifications sont nécessaires dans la configuration de hostapd-wpe pour cette attaque ?
> 
> **_Réponse :_** Il faut modifier le ssid par le nom du réseau de notre evil-twin. 
> 
> ![](https://cdn.discordapp.com/attachments/908358746323972146/976834077875384330/unknown.png)

---

> **_Question:_** Quel type de hash doit-on indiquer à john ou l'outil que vous avez employé pour craquer le handshake ?
> 
> **_Réponse:_** Pas besoin de préciser le type de hash car john peut le deviner. Mais il s'agit d'un NETNTLM

---

> **_Question:_** Quelles méthodes d’authentification sont supportées par hostapd-wpe ?
> 
> **_Réponse:_**
> 
> 1. EAP-FAST/MSCHAPv2 (Phase 0)
> 
> 2. PEAP/MSCHAPv2
> 
> 3. EAP-TTLS/MSCHAPv2
> 
> 4. EAP-TTLS/MSCHAP
> 
> 5. EAP-TTLS/CHAP
> 
> 6. EAP-TTLS/PAP

### 3. GTC Downgrade Attack avec [EAPHammer](https://github.com/s0lst1c3/eaphammer)

[EAPHammer](https://github.com/s0lst1c3/eaphammer) est un outil de nouvelle génération pour les attaques WPA Entreprise. Il peut en particulier faire une attaque de downgrade GTC, pour tenter de capturer les identifiants du client __en clair__, ce qui évite le besoin de l'attaque par dictionnaire.

- Installer ```EAPHammer```. Lire la documentation du site de l’outil ou d’autres ressources sur Internet pour comprendre son utilisation
- Modifier la configuration de ```EAPHammer``` pour proposer un réseau semblable au réseau de l’école ou le réseau de votre préférence. Le but est de réaliser une GTC Downgrade Attack.
- Lancer une capture Wireshark
- Tenter une connexion au réseau

**Résultats de l'attaque**

![](./assets/2022-05-19-15-00-36-image.png)

### Répondez aux questions suivantes :

> **_Question :_** Expliquez en quelques mots l'attaque GTC Downgrade
> 
> **_Réponse :_** EAP-GTC est une méthode d'authentification qui fait passer les credentials en clair dans le tunnel TLS. Si la victime cherche à se connecter à l'AP de l'attaquant (evil-twin), celui-ci peut lui proposer cette méthode d'authentification à la place de PEAP. Si le client accepte, les credentials passent en clair et c'est le jackpot pour l'attaquant qui peut les lire (il contrôle le tunnel TLS).

---

> **_Question:_** Quelles sont vos conclusions et réflexions par rapport à la méthode hostapd-wpe ?
> 
> **_Réponse:_** Downgrade GTC a l'avantage de pouvoir récupérer directement le mot de passe. Si celui-ci est complexe, il est difficile (voire impossible) de le brute-force avec hostapd-wpe. Cependant, pas tous les clients n'acceptent un downgrade GTC...  

### 4. En option, vous pouvez explorer d'autres outils comme [eapeak](https://github.com/rsmusllp/eapeak) ou [crEAP](https://github.com/W9HAX/crEAP/blob/master/crEAP.py) pour les garder dans votre arsenal de pentester.

(Il n'y a pas de rendu pour cette partie...)

## Livrables

Un fork du repo original . Puis, un Pull Request contenant :

- Captures d’écran + commentaires
- Réponses aux questions

## Échéance

Le 2 juin 2022 à 23h59
