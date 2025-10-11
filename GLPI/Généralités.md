## 1. Qu'est-ce que GLPI ? 

GLPI = Gestionnaire libre de parc informatique 

GLPI est un outil ITSM (gestion des services IT) open source qui peut être utilisé dans divers systèmes opérationnels. 
Il s'agit d'une application web fonctionnant sur un serveur web avec un compilateur PHP et une base de données MySQL. 

GLPI s'exécute donc sur une architecture à 3 niveaux : 
* Niveau de présentation : serveur web, généralement Apache2 ou Nginx 
* Niveau d'application : code PHP 
* Niveau de données : MySQL ou MariaDB 

## 2. Niveau de présentation 

La couche chargée d'afficher les informations à l'utilisateur et qui lui permet d'interagir avec l'outil. 

L'application GLPI génère du code HTML. Pour que ce code ait un sens pour l'utilisateur, nous avons besoin d'un serveur web pour le traduire. 

## 3. Niveau d'application 

La couche métier de l'application. Elle est responsable du traitement des opérations que GLPI effectue sur les données selon les actions de l'utilisateur. Cette logique est codée en PHP. 

## 4. Niveau de données 

La couche responsable du stockage et de l'accès aux données. 
Sur GLPI, ce niveau est géré par un Système de Gestion de Bases de Données basé sur MySQL ou son fork MariaDB. 

## 5. Installation de l'agent GLPI 

L'agent GLPI sert principalement à inventorier automatiquement les équipements (ordinateurs, smartphones, tablettes, périphériques réseau) d'un parc informatique. 

Cet agent s'installe sur les postes clients, à ne pas confondre avec le service GLPI qui lui est à installer sur un serveur web (généralement une VM Linux). 


#### Vérification préalable : 
Aller sur le serveur GLPI et dans le menu Administration > Inventaire, vérifier que l'inventaire est bien activé 

Pour installer l'agent GLPI sur le poste client, il vous faudra : 
* Télécharget l'agent adapté à l'OS du poste client 
* connaître l'URL du serveur GLPI, généralement <http://server/glpi>  
* Remplacer "server" par le nom du serveur ou par son adresse IP. 

Evidemment, les clients et le serveur doivent être sur le même réseau pour pouvoir communiquer. 

Et pour pouvoir utiliser le nom du serveur dans l'URL, il faut que la correspondance adresse IP -> nom du serveur soit enregistrée dans votre serveur DNS. 

Une fois l'agent installé sur le poste client (généralement sous Windows), aller dans les services pour redémarrer le service "glpi-agent". 

Vous pouvez vérifier que l'agent est bien installé en ouvrant un navigateur et en allant sur 127.0.0.1:62354 
Depuis cette page web, il est possible de forcer l'inventaire du poste client sur lequel l'agent a été installé. 
