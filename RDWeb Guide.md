# Installation du client RDWeb

## Installer le client RDWeb HTML5 sur RDS Windows Server 2016

Prérequis :
• Serveur RD Web Access doit exécuter à minima Windows Server 2016
• Le déploiement RDS 2016 doit comporter au moins une passerelle RDS
• Les CAL RDS installées doivent être de type « Par-Utilisateur »
• Les certificats SSL configurés pour la passerelle RDS et le Serveur RD Web Access doivent être délivrés et signés par une CA (Certification Authority) valide Publique : les certificats SSL auto-signés empêchent l’utilisation du client RD Web
• Seules les connexions provenant des OS suivants seront acceptées par le Client RD Web :
• Windows 10
• Windows Server 2008 R2 (ou ultérieur)
Instructions :
• Ouvrir une Session Windows sur le serveur RD Web Access
• Lancer Windows PowerShell en Administrateur
• Saisir la commande suivante pour mettre à jour le module « PowerShellGet » :

``Install-Module -Name PowerShellGet -Force``

ATTENTION : avant d’utiliser « Install-Module » il faut vérifier :
• La version de PowerShell est bien 5.1 ou ultérieure avec la commande :

``Get-Host | Select-Object Version
$PSVersionTable``

• Le TLS 1.2 seulement est utilisé :
• Pour vérifier quels protocoles sont utilisés, entrer la commande :

``[Net.ServicePointManager]::SecurityProtocol``

• Pour réassigner manuellement le protocole de sécurité :

``[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12``

• Une fois le TLS 1.2 forcé, relancer l’installation du module

``Install-Module -Name PowerShellGet -Force``

• Si une erreur est retournée, utiliser le proxy (voir plus bas)
• Une autre solution en cas d’erreur :

``Register-PSRepository -Name PSGallery -SourceLocation https://www.powershellgallery.com/api/v2/``

• Saisir Y pour confirmer l’exécution de la commande
• Fermer la console PowerShell et en rouvrir une pour que l’Update du module PowerShellGet soit prise en compte
• Exécuter la commande suivante pour installer le Module de Gestion du client RD Web : 

``Install-Module -Name RDWebClientManagement``

• Voir au-dessus si une erreur est retournée
• Saisir A pour accepter et valider avec entrée
• Exécuter la commande suivante pour installer la dernière version du client RD (Remote Desktop) Web

``Install-RDWebClientPackage``

• Si une erreur est retournée, configurer le proxy comme ceci :

````powershell 
$proxyUrl = “myproxy:port”
$proxyUsername = “username”
$proxyPassword = “password”
$proxyCredentials = New-Object System.Net.NetworkCredential($proxyUsername, $proxyPassword)
$proxy = New-Object System.Net.WebProxy($proxyUrl)
$proxy.Credentials = $proxyCredentials

[System.Net.WebRequest]::DefaultWebProxy = $proxy

Install-RDWebClientPackage

# Une fois le package du Client RD Web installé saisir la commande suivante et noter le résultat :

Get-RDWebClientPackage
````

- Maintenant, il faut exporter le certificat SSL utilisé par le service Broker. 
Ouvrir une session Windows sur le Serveur Broker et lancer la console (snap-in) 
> MMC > Ajouter « Certificats » > pour le « Compte d’Ordinateur » > Développer « Personal » Certificats >

- Clic droit sur le certificat utilisé par le service Broker
> All Tasks > Export 

- Le fichier .Cer a été exporté et placé dans C:
- Le fichier .Cer exporté précédemment sera utilisé lors de la prochaine opération pour prendre en charge l’authentification SSO depuis le client RD Web.
- Exécuter la commande suivante en spécifiant l’emplacement vers lequel le .Cer a été exporté et placé :

``Import-RDWebClientBrokerCert C:\MYCERT.cer``

- Erreur lors de la sélection du certificat RD Gateway
“could not configure the certificate on one or more servers”

- Désinstaller le rôle RD Gateway puis le réinstaller
Lors de l'installation du rôle, créer un certificat pour la RD Gateway via PowerShell :
````powershell
$Password = ConvertTo-SecureString -String “password” -AsPlainText -Force New-RDCertificate -Role RDGateway -DNSName MY.FQDN -Password $Password -ExportPath “C:\CertRdWebNew\name.pfx” -ConnectionBroker MY.FQDN
````

Puis, vérifier les certificats avec cette cmdlet : 

``Get-RDCertificate``

- Enfin, exécuter la commande PS suivante pour publier le nouveau client RD Web :

``Publish-RDWebClientPackage -Type Production –Latest``

- Si une erreur est retournée lors de l’import du certificat : retirer le proxy :

``netsh winhttp reset proxy``

- Note importante : cette commande doit être exécutée si vous devez déployer le client RD Web dans un environnement de production, si vous souhaitez simplement « PoKé » le client RD Web sur un environnement de Test/Dev/Hom, exécutez plutôt la commande suivante :

``Publish-RDWebClientPackage -Type Test –Latest``

Pour se connecter à votre client RD Web, la syntaxe de l’URL à utiliser est la suivante :

- Si Installation en Production (paramètre -Type Production)

<https://FQDN-de-votre-RDWebAccess.com/RDWeb/WebClient>

- Si Installation en environnement de Test (paramètre -Type Test)

<https://FQDN-de-votre-RDWebAccess.com/RDWeb/WebClient-Test>


- RDS error “A Remote Desktop Services deployment does not exist in the server pool”.

Avant toute chose, l’erreur peut venir du proxy : exécuter ces cmdlets:

```` powershell
netsh winhttp show proxy
netsh winhttp reset proxy
````

- Tout d’abord, vérifier les rôles de serveur RDS sont déployés sur les serveurs RDS de la topologie et que la base de données hautement disponible RDS CB est en ligne.
Ensuite, à partir du point de terminaison de gestion RDS (management endpoint), importer le module RDS Powershell en utilisant cette cmdlet :

``Import-Module RemoteDesktop``

- Exécuter les cmdlets suivantes (déploiement de session/déploiement de bureau virtuel ou les deux) :

```` powershell
new-rdsessiondeployment
new-rdvirtualdesktopdeployment
````

- Après ces cmdlets, il devrait être possible d’ajouter une RDS Session Hosts collections and executant cette cmdlet :

``new-rdsessioncollection``

- Dans certains cas, en particulier sur les vieilles versions RDS, cette erreur est attribuée à l’utilisation du protocole IPv6 (qui doit être désactivé)
Il faut également s’assurer que PS Remoting est activé sur tous les serveurs RDS.
Essayer en se connectant en administrateur du domaine ainsi qu’en administrateur local.

- La vue de la topologie RDS devrait maintenant être visible dans le Server Manager.

## Installer Remote Desktop Services

1. - Ouvrir le Server Manager 
> Manage > Add Roles and Features

2. - Remote Desktop Services Installation
> Select Deployment Type > Quick Start
Select Deployment Scenario > Session-Based Desktop Deployment

Info : Quick Start va installer les rôles Connection Broker, Web Access et Session Host sur le même serveur

3. - Cocher Restart the destination server automatically if required > Deploy

- Après redémarrage du système vérifier que les services sont bien configurés “Succeeded”

- On peut maintenant accéder a Remote Desktop Services sur le panneau de gauche du Server Manager
- Lorsqu'on clique, on accède à RDS Manager
- Comme on a choisi Quick Deployment, Collection(QuickSessionCollection) et Remote App Programs sont déjà configurés
- Collection sépare RD Session Hosts and fermes séparés et permet aux admins d'organiser les ressources.
- On constate qu'il manque une RD Gateway ainsi qu'un RD Licensing Server

> Add RD Licensing Server (bouton vert)

- Sélectionner le serveur
- Confirmer la séléction > add. Attendre jusqu'à ce que le service soit déployé > close
- Ensuite, il faut ajouter une RD Gateway > Add RD Gateway (bouton vert)
- Selectionner le serveur
- Lorsqu'on passe par cet outil, il va créer un certificat SSL auto signé. Comme nom, entrer le FQDN du serveur RDS.
- Next > Add > Configure Certificate
- On note que niveau du certificat a un statut Not Configured. Le certificat RD Gateway est utilisé pour la communication client vers gateway et doit être approuvé par les clients.

- Avant de créer un nouveau certificat, il faut configurer le DNS de façon à ce que les utilisateurs externes puissent résoudre le nom de la RD Gateway à la bonne adresse IP.
- Il faudra le configurer sur le DNS externe (DNs d'hote ou FAI), quelqu'un dont on a pas le contrôle mais qui accède à internet.
-Dans cet exemple, le “DNS externe” (Routeur-machine sur un réseau externe) va servir de DNS pour le réseau externe.
- Si l'on esssaye de pinger depuis une machine Windows 10 externe cela ne fonctionne pas. Mais en interne, ça fonctionne.
- Pour ce faire : ouvrir DNS Manager et aller jusqu'à Forward Lookup Zones > Clic droit > New Zone
- Zone Type > accept defaults > next.
- Zone Name > entrer le nom de zone
- Zone File, Dynamic Update > accept defaults > finish
- Clic droit sur la nouvelle zone > New Host (A or AAAA)
- Ici, on devrait indiquer l'IP externe du routeur NAT ou du firwall, l'IP publique la plus proche de la gateway.
- l'IP CA doit être également ajoutée
- Maintenant, le ping depuis une machine externe fonctionne.

- Essayons de se connecter à RDCB avec RDP.
- Windows + R > mstsc > entrer le nom RDCB > Advanced Tab
- Advanced > Settings > spécifiez ici la RD Gateway > OK > Connect
- Une fenêtre de sécurité Windows apparait, > Tapez les identifiants > OK > erreur
- Si il y a une erreur, alors le certificat n'est pas configuré.
- En réalité, il faudrait acheter un certificat après d'une CA publique (GoDaddy, VeriSign, etc.). Ce certificat doit contenir le FQDN utilisé comme URL RD Web Access. - Il doit être au format .pfx et doit contenir une clé privée.

- Open Server Manager > Tools > Certification Authority
- CA > clic droit Certificate Template > Manage
- Clic droit sur Web Server Template > Duplicate Template
- Changer le nom
- Request Handling Tab > Allow private key to be exported
- Security Tab > Authenticated Users > autoriser Enroll et Autoenroll. (Vérouiller le certificat à certaines personnes).
- Domain Computers > autoriser Read, Enroll et Autoenroll. > Clic OK
- Clic droit sur Certificate Templates > New > Certificate Template to Issue
- Selectionner le certificat créé > OK

- La dernière étape est d'inscrire le certificat. Changer de machine et ouvrir MMC (Windows + R > mmc)
- Clic droit sur Personal > All Tasks > Request New Certificate
- Before you begin and Select Certificate Enrollment Policy > Next
- Request Certificate page > Selectionner le certiciat SSL > cliquer sur More information is required…
- Changer le Subject Name Type en Comon Name et ajouter le nom exacte du serveur ou site web que vous allez utiliser.
- On peut ajouter le nom simple puis le FQDN > OK
- Enroll > Finish

- Ensuite, dans Personal > Certificates > le certificat apparait, il faut maintenant l'exporter avec la clé privée et configurer la Gateway, RDWeb Acces, RDCB pour qu'ils l'utilisent.
- Clic droit sur celui ci > All Tasks > Export
- Yes, Export the private key > next
Next (.PFX)
- Entrer un mot de passe
- Taper le nom et l'endroit où l'enregistrer

- Retourner sur Deployment Properties > RD Gateway > Select Existing certificat
- Add certificate > OK
- Clic Apply
- Faire la même chose pour RDWA et RDCB

- On peut tester de se connecter a RDWA, si tout va bien, il n'y a pas de message d'erreur de certificat.


## Installer CAL RDS : 
null

### Sources : 
<https://stefanos.cloud/kb/rds-error-a-remote-desktop-services-deployment-does-not-exist-in-the-server-pool/>
<https://www.delftstack.com/howto/powershell/installing-the-nuget-package-in-powershell/>
<https://hichamkadiri.wordpress.com/2018/10/24/installer-le-client-remote-desktop-web-html5-sur-rds-windows-server-2016/>
<https://stefanos.cloud/kb/rds-error-a-remote-desktop-services-deployment-does-not-exist-in-the-server-pool/ https://www.delftstack.com/howto/powershell/installing-the-nuget-package-in-powershell/>
<https://mehic.se/2017/02/08/how-to-install-remote-desktop-services-2016-quick-start-deployment/>