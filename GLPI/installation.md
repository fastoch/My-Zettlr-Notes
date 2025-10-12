# Installer GLPI 11 sur Debian 13

Tuto : <https://neptunet.fr/install-glpi11/>  
Documentation officielle : 
* <https://www.glpi-project.org/fr/discover-the-new-glpi-help-center/>  
* <https://glpi-install.readthedocs.io/en/latest/>  

---

## Architecture 

**GLPI nécessite 3 composants pour fonctionner** :  
* Un serveur web comme NGINX ou Apache2 pour l'interface utilisateur 
* Un compilateur PHP pour le code qui va gérer les interactions entre GLPI et l'utilisateur 
* Un gestionnaire de Bases de Données (BDD) comme MySQL ou MariaDB pour le stockage et l'accès aux données 

---

## Comment installer GLPI 11 sur Debian 13 ?

### A propos de sudo

* si vous n'avez pas défini de mot de passe **root** lors de l'installation de Debian, vous pourrez utiliser **sudo** 

* sinon, vous devez installer sudo et ajouter votre user au groupe "sudo" : 
  * Log in as root using the root password
  * Install sudo with the command: `apt install sudo` 
  * add your user to the sudo group: `usermod -aG sudo username`

### Pré-requis

* mettez à jour la liste des paquets disponibles dans les dépôts et installer les dernières versions des paquets déjà présents:
```bash
sudo apt update && sudo apt upgrade –y
```

* **installez les 3 composants nécessaires à GLPI** (GLPI 11 est compatible avec PHP 8.4) :  
```bash
sudo apt install apache2 mariadb-server php8.4 –y
```

### Installation des extensions PHP

* **Le dépôt Sury** est la solution la plus fiable pour installer des **extensions PHP** récentes sur Debian 13.
  Voici comment l'ajouter : 
  * installer les dépendances nécessaires : `sudo apt install -y apt-transport-https lsb-release ca-certificates`
  * ajouter la clé GPG du dépôt : `sudo wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg`
  * ajouter le dépôt de Ondřej Surý  :  
```bash
echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/php.list
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

### Préparation de la BDD

* **Connectez-vous au SGBD** précédemment installé : 
```bash
sudo mariadb
```

**IMPORTANT** : La sécurisation du service de BDD n’est plus nécessaire car désormais effectuée par défaut sous Debian 

* **Créez la BDD qui sera utilisée par GLPI** :
```bash
create database glpi_fondasol;
```
Pensez à bien noter le nom donné à votre BDD, ici elle se nomme "glpi_fondasol" 

* **Créez un utilisateur pour cette base de données** et lui octroyer tous les droits : 
```bash
grant all privileges on glpi_fondasol.* to admin@localhost identified by "motdepasse";
```
Ici, nous avons nommé l'utilisateur "admin" et il faut remplacer "motdepasse" par le mot de passe souhaité. 

* Quittez le service mariadb avec la commande `exit` 

### Téléchargement de GLPI

**Téléchargez GLPI** : 
* Trouvez la dernière version stable : <https://github.com/glpi-project/glpi/releases/latest> 
* Placez-vous dans le dossier Téléchargements : `cd ~/Téléchargements/` 
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
Pour des raisons de sécurité, il est recommandé de déplacer certains dossiers sensibles de /var/www/glpi/ car ils sont exposés publiquement sur le web. En particulier le dossier "config" 
qui peut être déplacé vers /etc/glpi/ ou /var/lib/glpi/.  

### Configuration du service web 

Vérifiez la version de php utilisée actuellement : `php –v` 

Créez un **virtualhost** spécialement dédié à GLPI : 
    * Un virtualhost est un fichier configuré sur apache permettant de faire cohabiter plusieurs sites web sur la même machine.  
    * Chaque virtualhost est configuré pour l’un des sites web hébergés sur le serveur. 
    * Créez un fichier **glpi.conf** comme suit : `sudo nano /etc/apache2/sites-available/glpi.conf`
    * Dans ce fichier, insérez le contenu suivant :
```bash
<VirtualHost *:80> 
	# accès serveur GLPI via son nom
    ServerName glpi11-vm 
    # Pour accès serveur via adresse IP au lieu du nom si absence de DNS
    ServerAlias 192.168.0.99 
    DocumentRoot /var/www/html 

    # Accès GLPI depuis http://nom-ou-IP/glpi
    Alias /glpi /var/www/html/glpi/public 

    # Définit les accès et règles de réécriture d'URL pour le dossier public de GLPI
    <Directory /var/www/html/glpi/public> 
        Options -Indexes +FollowSymLinks 
        Require all granted 
        RewriteEngine On 
        RewriteBase /glpi/ 
        RewriteCond %{REQUEST\_FILENAME} !-f 
        RewriteCond %{REQUEST\_FILENAME} !-d 
        RewriteRule ^ index.php [QSA,L] 
    </Directory> 

    # Délègue l'exécution des fichiers .php à PHP-FPM 
    <FilesMatch \.php$> 
    	SetHandler "proxy:unix:/run/php/php8.4-fpm.sock|fcgi://localhost" 
    </FilesMatch> 

    # isole les logs spécifiques à GLPI dans /var/log/apache2
    ErrorLog ${APACHE_LOG_DIR}/glpi_error.log 
    CustomLog ${APACHE_LOG_DIR}/glpi_access.log combined 
</VirtualHost>
```
Remplacez "glpi11-vm" par le nom de votre serveur (ou l’IP si vous n'avez pas configuré de DNS).  
Dans la section "FilesMatch", modifiez la version de PHP s'il ne s'agit pas de la 8.4. 

**La config ci-dessus permet de** :
- orienter les requêtes non-relatives à un fichier/dossier existant vers le routeur PHP principal (`index.php`)
- désactiver l’accès direct à la liste des fichiers pour renforcer la sécurité.
- donner accès à tous les utilisateurs web
- supporter la gestion correcte des URLs dynamiques via le moteur de réécriture

Activez les **modules** nécessaires pour que **PHP-FPM** fonctionne correctement avec apache : 
```bash
sudo a2enmod proxy_fcgi setenvif
```

Activez un module apache qui permet de gérer les **URL dynamiques** et les **redirections** comme celles utilisées par GLPI : 
```bash
sudo a2enmod rewrite
```

**Activez la config de PHP-FPM** pour s'assurer qu'apache va bien déléguer l’exécution des fichiers PHP :
```bash
sudo a2enconf php*-fpm
```

**Désactivez** le fichier de **configuration par défaut** pour éviter les interférences :
```bash
sudo a2dissite 000-default.conf
```

**Activez le virtualhost** spécialement créé pour glpi :
```bash
sudo a2ensite glpi.conf
```

La config du service web est terminée, il ne reste plus qu’à **redémarrer le service apache2** pour appliquer toutes les modifications apportées :
```bash
sudo systemctl restart apache2
```

### Installer GLPI via interface web

Les fichiers (/var/www/html) et services (MariaDB, PHP et Apache2) sont prêts pour GLPI.
L'installation va se poursuivre via un navigateur web.

Accédez à votre service GLPI depuis n'importe quel PC **sur le même réseau** en vous rendant à
l'URL suivante :
**http://ip_ou_nom_du_serveur_glpi/glpi**

Vous arrivez sur la page d'installation de GLPI : 
- Sélectionnez le **Français** dans la liste déroulante et cliquez sur **OK**.
- **Acceptez les conditions d’utilisation** pour poursuivre.
- Cliquez sur le bouton **Installer** pour lancer l'installation.

Une série de tests sera lancée pour s’assurer que tous les prérequis nécessaires au bon fonctionnement de GLPI sont remplis.

Si tout est OK, cliquez sur **Continuer**, sinon corrigez les erreurs signalées et relancez une vérification.

Il reste à saisir les informations sur la BDD destinée à GLPI que nous avons précédemment créée.
- Dans le champ "Serveur SQL", saisissez **localhost** 
    - si la BDD est sur une autre machine, saisissez son adresse IP ou son nom. 
- Saisissez ensuite le nom de votre utilisateur SQL ("admin" dans notre cas), ainsi que son mot de passe.

Un test de connexion à la BDD est effectué. Si réussi, vous pourrez sélectionner la BDD et cliquer sur **Continuer**.

Attendez que l'initialisation de la BDD soit terminée et cliquez à nouveau sur **Continuer**.
La fin de l'installation ne nécessite pas d'instructions particulières.

**ATTENTION**:
Notez bien les identifiants des 4 comptes par défaut qui vous sont fournis en fin d'installation.

### Première connexion

En fin d'installation, après avoir noté les identifiants fournis, cliquez sur "Utiliser GLPI".
Connectez-vous avec le compte administrateur.

Une fois connecté, vous arrivez sur le tableau de bord de GLPI.
Vous pouvez désactiver les données de démonstration en cliquant sur le bouton dédié.

Un **message d’avertissement** vous informe que par sécurité il faudra **changer les mots de passe par défaut des 4 utilisateurs** déjà présents dans GLPI.

Si vous cliquez sur le nom de l’un des utilisateurs, vous arriverez directement sur sa configuration. 
En bas à droite, cliquez sur **Modifier le mot de passe**.

Saisissez un nouveau mot de passe puis **cliquez en dehors de la nouvelle fenêtre** pour valider le 
changement en cliquant sur **Sauvegarder**.

**Répétez l’opération sur les 4 comptes utilisateurs** pour faire disparaître l’avertissement.

**Votre GLPI est désormais fonctionnel ! ![bravo](https://neptunet.fr/wp-content/plugins/kama-wp-smile/packs/qip/bravo.gif)**

Les différents **menus latéraux** vous permettront de gérer :
- votre parc informatique,
- vos tickets d’incidents,
- vos contrats,
- vos fournisseurs,
- vos commandes,
- vos utilisateurs
- les projets du SI
- etc.

---
EOF