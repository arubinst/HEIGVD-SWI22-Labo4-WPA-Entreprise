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

  #### Authentification système ouvert

  Requête et réponse d’authentification système ouvert

  Lien de notre capture Wireshark :  [SWI_Labo4_AutentificationWPA-Enterprise.pcapng](./files/SWI_Labo4_AutentificationWPA-Enterprise.pcapng) 

  ##### Client

  ![OpenSystemAuth_client](./assets/OpenSystemAuth_client.PNG)

  ##### AP Cisco

  ![OpenSystemAuth_cisco](./assets/OpenSystemAuth_cisco.PNG)

  #### Requête et réponse d’association (ou reassociation)

  ##### Requête

  ![request-peap](./assets/request-peap.PNG)

  ##### Réponse

  ![response-peap](./assets/response-peap.PNG)

  #### Négociation

  Négociation de la méthode d’authentification entreprise (TLS?, TTLS?, PEAP?, LEAP?, autre?)

  ##### Requête

  ![request-peap](./assets/request-peap.PNG)

  ##### Réponse

  ##### ![response-peap](./assets/response-peap.PNG)

  #### Phase d’initiation

  ##### Requête d'identité

  ![identity-request](./assets/identity-request.PNG)

  ##### Réponse d'identité

  ![response-identity](./assets/response-identity.PNG)

  #### Phase hello

  ##### Client 

  - Version TLS

  ![version](./assets/client-hello/version.PNG)

  - Suites cryptographiques et méthodes de compression proposées par le client et acceptées par l’AP

  ![ciphersuite](./assets/client-hello/ciphersuite.PNG)

  - Nonces

  ![nonce](./assets/client-hello/nonce.PNG)

  - Session ID

    Avec notre capture, nous avions aucun id de session, ce qui est étrange.

  ![session-id](./assets/client-hello/session-id.PNG)

  Avec la capture `auth`, nous avons pu avoir l'id de session suivant :

  ![auth-session-id](./assets/auth/auth-session-id.PNG)

  ##### AP Cisco

  Aucun serveur hello n'a pas été capturée avec wireshark. On a utilisé alors les captures du fichier `auth`.

  - Version TLS

  ![version-tls](./assets/server-hello/version-tls.PNG)

  - Suites cryptographiques et méthodes de compression proposées par le client et acceptées par l’AP

  ![ciphersuite](./assets/server-hello/ciphersuite.PNG)

  - Nonces

  ![nonce](./assets/server-hello/nonce.PNG)

  - Session ID

  ![session-id](./assets/server-hello/session-id.PNG)

  #### Phase de transmission de certificats

  ##### Echanges des certificats

  On n'avait pas de certificats échangés avec notre capture, alors on a pris celle pour `auth`.
  
  ![server-certificate](./assets/auth/server-certificate.PNG)
  
  ##### Change cipher spec
  
  - Client -> AP
  

![change-cipher-client](./assets/auth/change-cipher-client.PNG)

  - AP -> client


  ![changier-cipher-ap](./assets/auth/changier-cipher-ap.PNG)

#### Authentification interne et transmission de la clé WPA 

Echange chiffré, vu par Wireshark comme « Application data »

Du fichier `Auth`car on n'avait pas de data dans notre capture wireshark,

![data-encrypt](./assets/auth/data-encrypt.PNG)



#### 4-way handshake

##### Notre fichier

On peut constater que le Message 2 n'a pas été capturé.

![4way-handsake-us](./assets/4way-handsake-us.PNG)



##### Fichier auth

Avec le fichier `auth`, voici un 4 way handshake complet.

![4way-handshake](./assets/auth/4way-handshake.PNG)

### Répondez aux questions suivantes :

> **_Question :_** Quelle ou quelles méthode(s) d’authentification est/sont proposé(s) au client ?
>
> **_Réponse :_** 
>
> Dans notre cas, EAP-PEAP

---

> **_Question:_** Quelle méthode d’authentification est finalement utilisée ?
> 
> **_Réponse:_** EAP-PEAP

---

> **_Question:_** Arrivez-vous à voir l’identité du client dans la phase d'initiation ? Oui ? Non ? Pourquoi ?
>
> **_Réponse:_** 
>
> Oui, car la communication n'est pas encore chiffrée à ce moment là
>
> ![identity-response](./assets/identity-response.PNG)

---

> **_Question:_** Lors de l’échange de certificats entre le serveur d’authentification et le client :
>
> - a. Le serveur envoie-t-il un certificat au client ? Pourquoi oui ou non ?
>
> On n'a pas capturée la trame dans notre cas. Mais avec le fichier auth, on constate que le serveur s'authentifie bien en envoyant son certificat. Ce qui est demandé par EAP-PEAP. Une fois le certificat accepté par le client, le serveur est alors authentifié auprès du client.
>
> **_Réponse:_**
>
> - b. Le client envoie-t-il un certificat au serveur ? Pourquoi oui ou non ?
>
> **_Réponse:_**
>
> Non avec EAP-PEAP, seul le serveur envoie son certificat au client. Contrairement à EAP-TLS, l 'utilisateur entre et envoie ses informations d'identification au serveur RADIUS. Celui-ci les vérifie et les authentifie pour l'accès au réseau.
>
> Référence : [https://www.securew2.com/blog/eap-tls-vs-peap-mschapv2-which-authentication-protocol-is-superior](https://www.securew2.com/blog/eap-tls-vs-peap-mschapv2-which-authentication-protocol-is-superior)

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
>
> Dans le fichier de configuration, hostapd-wpe.conf, il faut modifier le ssid : ssid=HEIG-VD. Dans notre cas nous avons également fixé le channel à 1.
> 
>![configsHostapd](assets/configsHostapd.PNG)
> 

---

> **_Question:_** Quel type de hash doit-on indiquer à john ou l'outil que vous avez employé pour craquer le handshake ?
>
> **_Réponse:_** 
>
> hashcat -m 5500 -O test::::40ac4dd44436c858e146c64f89d655b6d74d9f4f6a839590:5b142dfaad0383f9 /usr/share/wordlists/rockyou.txt
>
> C'est 5500 / NetNTLMv1 / NetNTLMv1+ESS  
>
> 

![Captureshcat-1](./assets/Captureshcat-1.PNG)

> ****
>
> **_Question:_** Quelles méthodes d’authentification sont supportées par hostapd-wpe ?
>
> **_Réponse:_**
>
> ```
> hostapd-wpe supporte les méthodes suivantes :
> -EAP-FAST/MSCHAPv2 (Phase 0)
> -PEAP/MSCHAPv2
> -EAP-TTLS/MSCHAPv2
> -EAP-TTLS/MSCHAP
> -EAP-TTLS/CHAP
> -EAP-TTLS/PAP
> ```
> Référence : [https://github.com/aircrack-ng/aircrack-ng/tree/master/patches/wpe/hostapd-wpe](https://github.com/aircrack-ng/aircrack-ng/tree/master/patches/wpe/hostapd-wpe)

### 3. GTC Downgrade Attack avec [EAPHammer](https://github.com/s0lst1c3/eaphammer) 

[EAPHammer](https://github.com/s0lst1c3/eaphammer) est un outil de nouvelle génération pour les attaques WPA Entreprise. Il peut en particulier faire une attaque de downgrade GTC, pour tenter de capturer les identifiants du client __en clair__, ce qui évite le besoin de l'attaque par dictionnaire.

- Installer ```EAPHammer```. Lire la documentation du site de l’outil ou d’autres ressources sur Internet pour comprendre son utilisation
- Modifier la configuration de ```EAPHammer``` pour proposer un réseau semblable au réseau de l’école ou le réseau de votre préférence. Le but est de réaliser une GTC Downgrade Attack.
- Lancer une capture Wireshark
- Tenter une connexion au réseau

## Capture

Nous avons eu plusieurs problèmes pour faire fonctionner EAPHammer, nous avons enfin pu le lancer :

![EAPHammer](./assets/EAPHammer.PNG)

Le réseau evil twin a bien été créé :

![fake-evil](./assets/fake-evil.jpg)

Malheureusement il nous était impossible de nous y connecter (Connexion impossible). Et rien n'était capturé par Wireshark :

![WiresharkVideEAPHammer](./assets/WiresharkVideEAPHammer.PNG)

Nous n'avons donc pas pu effectuer l'entièreté de l'attaque pour récupérer les crédentials.


### Répondez aux questions suivantes :

> **_Question :_** Expliquez en quelques mots l'attaque GTC Downgrade
>
> **_Réponse :_** 
>
> L'attaque consiste à créer un faux AP avec le nom d'un SSID cible (evil twin) utilisant EAP-GTC.
>
> Le but est de forcer un appareil client à utiliser la méthode EAP-GTC. Celle-ci consiste à demander au client un mot de passe OTP.
>
> La plupart du temps, les logiciels ou appareils clients vont simplement présenter un formulaire de mot de passe générique. En raison de cela, l'utilisateur peut être confus et peut croire  qu'il doit entrer ses crédentials du réseau plutôt que le mot de passe OTP.
>
> Le problème avec cette transmission, c'est qu'elle se fait en clair. L'attaquant peut alors sniffer les crédentials de l'utilisateur.
>
> 
>
> Référence : [https://solstice.sh/iii-eap-downgrade-attacks/](https://solstice.sh/iii-eap-downgrade-attacks/#:~:text=access%20points%20instead.-,GTC%20Downgrade%20Attacks,the%20access%20point%20in%20plaintext)



---

> **_Question:_** Quelles sont vos conclusions et réflexions par rapport à la méthode hostapd-wpe ?
>
> **_Réponse:_** 

>Avec hostapd-wpe, l'attaque sera plus difficile voire impossible si le mot de passe dont est dérivé la clé est compliqué. En effet, avec cette attaque, on doit encore effectuer un brute-force sur le handshake.
> 
>Avec GTC Downgrade Attack, on berne l'utilisateur pour obtenir son mot de passe. Ainsi, il sera cassé même s'il est compliqué.
> 
>Néanmoins, si l'utilisateur est vigilant alors l'attaque GTC Downgrade pourrait ne pas fonctionner car l'utilisateur ne rentrera alors pas son mot de passe.

### 4. En option, vous pouvez explorer d'autres outils comme [eapeak](https://github.com/rsmusllp/eapeak) ou [crEAP](https://github.com/W9HAX/crEAP/blob/master/crEAP.py) pour les garder dans votre arsenal de pentester.

(Il n'y a pas de rendu pour cette partie...)

## Livrables

Un fork du repo original . Puis, un Pull Request contenant :

-	Captures d’écran + commentaires
-	Réponses aux questions

## Échéance

Le 2 juin 2022 à 23h59
