# Installer GLPI 11 sur Debian 13

Tuto : <https://neptunet.fr/install-glpi11/>  
Documentation officielle : 
* <https://www.glpi-project.org/fr/discover-the-new-glpi-help-center/>  
* <https://glpi-install.readthedocs.io/en/latest/>  

---

#### Architecture 

**GLPI nécessite 3 composants pour fonctionner** :  
* Un serveur web comme NGINX ou Apache2 pour l'interface utilisateur 
* Un compilateur PHP pour le code qui va gérer les interactions entre GLPI et l'utilisateur 
* Un Système de Gestion de Bases de Données (SGBD) comme MySQL ou MariaDB pour le stockage et l'accès aux données 

---

## Voyons comment installer GLPI 11 sur Debian 13

* si vous n'avez pas défini de mot de passe **root** lors de l'installation de Debian, vous pourrez utiliser **sudo** 

* sinon, vous devez installer sudo et ajouter votre user au groupe "sudo" : 
  * Log in as root using the root password
  * Install sudo with the command: `apt install sudo` 
  * add your user to the sudo group: `usermod -aG sudo username`

* mettez à jour la liste des paquets disponibles dans les dépôts et installer les dernières versions des paquets déjà présents:
```bash
sudo apt update && sudo apt upgrade –y
```

* **installez les 3 composants nécessaires à GLPI** (GLPI 11 est compatible avec PHP 8.4) :  
```bash
sudo apt install apache2 mariadb-server php8.4 –y
```

* **Le dépôt Sury** est la solution la plus fiable pour installer des **extensions PHP** récentes sur Debian 13.
  Voici comment l'ajouter : 
  * installer les dépendances nécessaires : `sudo apt install -y apt-transport-https lsb-release ca-certificates`
  * ajouter la clé GPG du dépôt : `sudo wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg`
  * ajouter le dépôt de Ondřej Surý  :  
```bash
echo "deb https://packages.sury.org/php/ $(lsb\_release -sc) main" | sudo tee /etc/apt/sources.list.d/php.list
```

* installez un gestionnaire de processus PHP nommé **FastCGI Process Manager** (FPM) : 
```bash
sudo apt install php8.4-fpm –y
```
**FPM** permet d’exécuter le code PHP dans un service séparé d’Apache. 
Cela évite de surcharger le service web et améliore les performances, la stabilité et la sécurité du site. 

* installez les **16 modules PHP** de base dont pourrait avoir besoin GLPI : 
```bash
sudo apt install php8.4-{dom,simplexml,redis,cli,imap,mysql,mbstring,curl,gd,xml,intl,apcu,xmlrpc,zip,bz2,bcmath} -y
```

* Pour permettre à GLPI de communiquer avec un annuaire **LDAP**, comme Active Directory, il faut aussi installer `php8.4-ldap` 

* **Connectez-vous au SGBD** précédemment installé : 
```bash
sudo mariadb
```

**IMPORTANT** : La sécurisation du service de BDD n’est plus nécessaire car désormais effectuée par défaut sous Debian 

* **Créez la BDD qui sera utilisée par GLPI** :
```bash
create database glpi\_fondasol;
```
Pensez à bien noter le nom donné à votre BDD, ici elle se nomme "glpi_fondasol" 

* **Créez un utilisateur pour cette base de données** et lui octroyer tous les droits : 
```bash
grant all privileges on glpi_fondasol.* to admin@localhost identified by "motdepasse";
```
Ici, nous avons nommé l'utilisateur "admin" et il faut remplacer "motdepasse" par le mot de passe souhaité. 

* Quittez le service mariadb avec la commande `exit` 

* **Téléchargez GLPI** : 
  * Trouvez la dernière versoin stable : <https://github.com/glpi-project/glpi/releases/latest> 
  * Placez-vous dans le dossier Téléchargement : `cd ~/Téléchargements/` 
  * Téléchargez la dernière version : `wget https://github.com/glpi-project/glpi/releases/download/11.0.1/glpi-11.0.1.tgz`

* Décompressez l’archive de GLPI directement dans le répertoire par défaut du service web (apache2) : 
```bash
sudo tar -xvzf glpi-11.0.1.tgz -C /var/www/html
```

* Rendez l’utilisateur des services web (nommé www-data) propriétaire de ces nouveaux fichiers : 
```bash
sudo chown -R www-data /var/www/html
```

* Vous pouvez vérifier que tout est OK en listant le contenu du répertoire :
```bash
ls -l /var/www/html
```
Vous pourrez constater la présence d’un répertoire nommé « glpi » dont le propriétaire est bien l’utilisateur nommé « www-data ». 

**IMPORTANT**:
Pour des raisons de sécurité, il est recommandé de déplacer certains dossiers sensibles de /var/www/glpi/ car ils sont exposés publiquement sur le web. En particulier le dossier "config" qui peut être déplacé vers /etc/glpi/ ou /var/lib/glpi/.  

* **Configurez le service web** : 
  * vérifiez la version de php utilisée actuellement : `php –v` 
  * créez un **virtualhost** spécialement dédié à GLPI : 
    * Un virtualhost est un fichier configuré sur apache permettant de faire cohabiter plusieurs sites web sur la même machine.  
    * Chaque virtualhost est configuré pour l’un des sites web hébergés sur le serveur. 
    * Créez un fichier **glpi.conf** comme suit : `sudo nano /etc/apache2/sites-available/glpi.conf`
    * Dans ce fichier, insérez le contenu suivant :
```bash
<VirtualHost \*:80> 
    ServerName glp11-vm 
    ServerAlias 192.168.0.99 
    DocumentRoot /var/www/html 
    Alias /glpi /var/www/html/glpi/public 
    <Directory /var/www/html/glpi/public> 
        Options -Indexes +FollowSymLinks 
        Require all granted 
        RewriteEngine On 
        RewriteBase /glpi/ 
        RewriteCond %{REQUEST\_FILENAME} !-f 
        RewriteCond %{REQUEST\_FILENAME} !-d 
        RewriteRule ^ index.php \[QSA,L] 
    </Directory> 
    <FilesMatch \\.php$> 
    	SetHandler "proxy:unix:/run/php/php8.4-fpm.sock|fcgi://localhost" 
    </FilesMatch> 
    ErrorLog ${APACHE\_LOG\_DIR}/glpi\_error.log 
    CustomLog ${APACHE\_LOG\_DIR}/glpi\_access.log combined 
</VirtualHost>
```
Remplacez "ServerName" par le nom de votre serveur (ou l’IP si vous n'avez pas configuré de DNS)  
Dans la section "FilesMatch", indiquez la version de PHP installée sur votre système (ici 8.4) 

Activez les modules nécessaires pour que PHP-FPM fonctionne correctement avec apache : 
```bash
sudo a2enmod proxy\_fcgi setenvif
```

Activez un module apache qui permet de gérer les URL dynamiques et les redirections comme celles utilisées par GLPI : 
