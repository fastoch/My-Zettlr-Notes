- Dernière version de GLPI : <https://github.com/glpi-project/glpi/releases/>  
- Documentation officielle : <https://help.glpi-project.org/faq/fr/glpi/installation_update>  
- Tuto : <https://www.it-connect.fr/tuto-mettre-a-jour-glpi/>  

---

## 1. Vérification préalable 

Si vous envisagez d'effectuer une mise à jour majeure sur votre serveur GLPI, par exemple, vérifiez au préalable que les extensions 
que vous utilisez soient bien prises en charge par la nouvelle version.  

Notez que le plugin **FusionInventory** est pris en charge uniquement jusqu'à la version **10.0.3** de GLPI.  
Pour les versions supérieures, GLPI a développé un **agent natif** ainsi qu'un **plugin** "GLPI Inventory" qui remplacent FusionInventory. 

## 2. Sauvegarder vos données (BDD + dossier GLPI) 

Avant de mettre à jour GLPI, il est fortement recommandé de sauvegarder vos données. 
Si votre serveur GLPI est une VM, prendre un snapshot afin de pouvoir facilement faire machine arrière en cas de pépin.  

Dans tous les cas, les sauvegardes qui suivent sont fortement recommandées.

Tout d'abord, exportez la base de données (BDD) GLPI via un dump SQL avec la commande suivante :
```bash
mysqldump -u \[utilisateur] -p \[nom\_BDD] > /chemin/vers/sauvegarde/glpi\_backup.sql
```
Notez que la commande `mysqldump` n'est pas native à Debian, mais fait partie du paquet client MySQL ou MariaDB. 
Elle devient disponible après l'installation de l'un de ces paquets, comme `mysql-client` ou `mariadb-client`

C'est la BDD qui contient toutes les données de l'application (tickets, utilisateurs, inventaire, configurations, etc.) 
Si vous avez oublié le nom de la BDD : 
- connectez-vous à votre instance mySQL : `mysql -u root –p` 
- Puis listez les bases de données : `show databases;` 

Sauvegarder les **dossiers importants** de GLPI : config, files, marketplace, plugins.
- Les dossiers de l'instance GLPI, notamment plugins et marketplace, sont souvent situés dans /var/www/glpi
- Le dossier contenant les fichiers uploadés par GLPI (documents, images, etc.) se trouve généralement dans /var/lib/glpi/files
- Les configurations peuvent être déplacées dans /etc/glpi, notamment le dossier config  

## 3. Mettre à jour le dossier GLPI 

Si votre GLPI est installé dans le répertoire "/var/www/glpi/" du serveur Web, c'est ce dossier que vous devez supprimer.  
```bash
sudo rm -rf /var/www/glpi/
```

À partir du GitHub officiel de GLPI, vous devez copier le lien de l'archive tar.gz correspondante à la version que vous souhaitez installer. 
Puis, à partir de la ligne de commande, initiez le téléchargement avec la commande wget : 
```bash
cd /tmp \
wget https://github.com/glpi-project/glpi/releases/download/10.0.12/glpi-10.0.12.tgz
```

Une fois que c'est fait, décompressez l'archive tar.gz : 
```bash
tar -xzvf glpi-10.0.12.tgz
```

Une fois l'archive décompressée, vous obtenez un dossier "glpi" qui contient l'ensemble des fichiers de cette nouvelle version. 
Vous devez déplacer ce dossier vers la racine de votre application web GLPI :  
```bash
sudo mv glpi /var/www/glpi 
```

## 4. Récupérer les données de l'instance GLPI 

Il s'agit de récupérer les données contenues dans les dossiers "marketplace", "plugins", "config" et "files", celles sauvegardées précédemment. 

Pour ce faire, nous pouvons simplement les copier un par un depuis le dossier de sauvegarde vers le dossier d'installation de GLPI: 
```bash
sudo cp -Rf /home/\<username>/backup\_glpi/files /var/www/glpi/ 
sudo cp -Rf /home/\<username>/backup\_glpi/plugins /var/www/glpi/ 
sudo cp -Rf /home/\<username>/backup\_glpi/config /var/www/glpi/ 
sudo cp -Rf /home/\<username>/backup\_glpi/marketplace /var/www/glpi/
```
 
Puis, il faut modifier les permissions sur les données du répertoire "/var/www/glpi" pour que l'utilisateur "www-data", utilisé par Apache2, soit propriétaire (Apache2 est le serveur web qui héberge l'appli GLPI) : 
```bash
sudo chown -R www-data:www-data /var/www/glpi/ 
```

## 5. Mettre à jour la BDD GLPI 

Si vous accédez à l'interface de votre GLPI, vous allez tomber sur une page qui liste l'ensemble des prérequis et à la fin de cette page, 
il y a un bouton nommé "Upgrade" qui permet de déclencher la mise à jour.

Mais, l'éditeur de GLPI recommande plutôt d'effectuer la mise à jour à partir de la ligne de commande.  
Pour cela, vous devez ouvrir un terminal sur votre serveur via une connexion SSH. 

Assurez-vous que votre serveur dispose bien de tous les prérequis nécessaires pour accueillir GLPI : 
```bash
cd /var/www/glpi 
sudo php bin/console glpi:system:check\_requirements 
```

En principe, ce sera le cas, car GLPI était déjà installé. 
Si tout est bon, vous pouvez lancer la mise à jour de la base de données GLPI : 
```bash
sudo php bin/console db:update 
```

Vous devrez répondre à quelques questions et ensuite recevoir le message "Migration effectuée" en vert.  
Connectez-vous à votre GLPI pour vérifier que tout fonctionne. 

Une fois la mise à niveau terminée et après avoir vérifié que tout fonctionne, vous devez supprimer ce fichier par sécurité : 
```bash
sudo rm /var/www/glpi/install/install.php 
```

Ce fichier pourrait être exploité pour réinitialiser ou compromettre l'installation de GLPI. 

Si vous utilisez des plugins, vous devez également vérifier leur état et éventuellement les mettre à jour à partir du menu "Configuration" puis "Plugins". 

Enfin, si vous l'aviez activé, vous pouvez désactiver le mode maintenance de GLPI : 
```bash
cd /var/www/glpi  
sudo php bin/console glpi:maintenance:disable
```
 
