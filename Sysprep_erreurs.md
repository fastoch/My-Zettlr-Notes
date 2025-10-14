# Sysprep - Erreurs courantes 

src = https://neptunet.fr/error-sysprep/

## Qu'est-ce que Sysprep ?

Sysprep permet de préparer des "masters" de Windows, des images standard facilement réutilisables.
C'est surtout utilisé en entreprise pour effectuer de nombreux déploiements en un minimum de temps.

Cet utilitaire se trouve dans **C:\Windows\System32\Sysprep.**
Sysprep peut s'utiliser en mode graphique (GUI) ou en ligne de commandes (CLI).

## Consulter les logs de Sysprep

Lorsqu'une erreur survient, Sysprep vous invite à consulter les logs (journaux d'évènements).
Ces logs se trouvent dans le dossier **C:\Windows\System32\Sysprep\Panther**.

Dans le dossier "Panther", vous trouverez le fichier "setupact", il contient tous les évènements.
Mais celui qui nous intéresse est "**setuperr**" car il contient uniquement les erreurs.

Ouvrez le fichier "setuperr" (il faut être admin).
Chaque ligne est horodatée, ce qui permet de repérer l'erreur qui vous intéresse.

Prêtez attention aux **codes erreur Microsoft** qui commencent par **0x** et utilisez-les pour effectuer des recherches sur le Web afin d'en savoir plus sur la cause probable de l'erreur rencontrée.

## Erreurs fréquentes et solutions

### Redémarrage en attente suite mise à jour système

**Erreur indiquée dans le fichier setuperr**:
*"There are one or more Windows updates that require a reboot. To run Sysprep, reboot the computer
and restart the application."*

**Solution :**
Vérifiez dans **Windows Update** si un redémarrage est nécessaire, redémarrez la machine et relancez Sysprep.
![sysprep_error1.png](./images/sysprep_error1.png)

### Stockage en cours d'utilisation

**Erreur indiquée dans le fichier setuperr**:
*"Audit mode cannot be turned on if reserved storage is in use. An update or servicing operation may be using reserved storage"*

**Solution:**
Vérifiez dans **Windows Update** que des mises à jour sont en cours de téléchargements et d’installation. Patientez jusqu’à la mise à jour complète du système, redémarrez la machine et relancez Sysprep.

### Bitlocker actif

**Erreur indiquée dans le fichier setuperr**:
*"BitLocker is on for the OS volume. Turn BitLocker off to run Sysprep."*

**Solution 1 : Bitlocker est vraiment activé:**
Allez dans « Ce PC », faites un clic droit sur la partition qui contient le système d’exploitation, 
cliquez sur « Gérer BitLocker » puis sur « Désactiver BitLocker ». 
Une fois désactivé, relancez Sysprep *(un redémarrage avant Sysprep peut être nécessaire)*.

**Solution 2 : Bitlocker n'est pas activé:**
Si BitLocker n’est pas clairement indiqué activé, cela n’empêche pas le **chiffrement du disque**, 
surtout à partir de Windows 11 24h2 qui chiffre automatiquement les données après l’installation de l’OS.

Pour vérifier si le disque est chiffré, ouvrez une invite de commandes et exécutez `manage-bde -status`
Vérifiez que l’état de la conversion ne soit pas « chiffré » ou « en cours de chiffrement ». 
Si c’est le cas, utilisez cette commande pour désactiver ou interrompre le chiffrement:
`manage-bde -off "C:"`

Selon le **pourcentage chiffré**, cette opération d'interruption peut être longue.
Surveillez régulièrement l’état de la conversion avec la première commande jusqu’à ce que ce soit indiqué "**intégralement déchiffre**".

Si vous préférez, vous pouvez faire les **mêmes manipulations** avec des commandes **PowerShell** :
`Get-BitLockerVolume -MountPoint "C:"`
`Disable-BitLocker -MountPoint "C:"`

### Package Microsoft non-provisionné

**Erreur indiquée dans le fichier setuperr**:
*"Package Microsoft.BingSearch_1.0.79.0_x64_8wekyb3d8bbwe was installed for a user, but not provisioned for all users. This package will not function properly in the sysprep image."*

Cette erreur est très courante. Elle indique qu’un package est installé sur la machine mais 
non-provisionné pour tous les utilisateurs. Il s’agit en fait d’un "**bloatware**", une application pré-installée qui émane du Microsoft Store.

***Solution:***
Dans le message d’erreur du fichier setuperr, il y a le **nom du package incriminé**.
Ce qui va nous intéresser dans le nom exact du package, c’est la **suite de 13 caractères** composées de à la fin, dans notre exemple: **8wekyb3d8bbwe**.

Cette suite de caractère correspond au « **PublisherId** », c’est-à-dire à l’identifiant de l’éditeur du package, qui est propre à Microsoft.

Afin d’éviter de corriger les erreurs de package une par une (car en corriger une en fait apparaître une autre, et ainsi de suite...), on va se servir de « 8wekyb3d8bbwe » pour supprimer d’un seul coup tous 
les packages possiblement pré-installées par Microsoft ("**bloatware**").

Ouvrez Powershell en tant qu'admin est exécuter la commande suivante:
```powershell
`Get-AppxPackage -AllUsers | Where {$_.PackageFullName -like "*8wekyb3d8bbwe*"} | Remove-AppxPackage`
```
La commande risque de retourner beaucoup d'erreurs (en rouge) mais ce n'est pas important.
L'exécution sera terminée lorsque vous verrez de nouveau le prompt indiquer le chemin 
« C:\Windows\system32 ». Vous pourrez alors relancer Sysprep.

### Dépassement du nombre de réinitialisations autorisé

**Erreur indiquée dans le fichier setuperr**:
*«  Failure occurred while executing ‘SLReArmWindows’ from C:\Windows\System32\slc.dll »*

Cette erreur est plus rare et plus complexe. Elle signifie que la machine a été réinitialisée de trop nombreuses fois et qu'il  n’est plus possible de la généraliser avec Sysprep.

L’action "généraliser" de Sysprep agit comme la commande `slmgr /rearm` qui vise à réarmer la licence de Windows, et cette opération peut être effectuée un nombre de fois limité selon l’édition du produit.

***Solution:***
Allez dans l’**éditeur de la base de registre** *(appuyez simultanément sur les touches Windows et R du clavier et saisissez « regedit »)*.

Dans l’arborescence de l’éditeur de base de registre, allez dans 
**HKEY_LOCAL_MACHINE > SOFTWARE > Microsoft > Windows NT > CurrentVersion > SoftwareProtectionPlatform**

Dans la partie de droite, double-cliquez sur la valeur nommée «**SkipRearm**».
Dans le champ **"Données de la valeur"**, mettez un **1 à la place du 0** et cliquez sur OK.

Toujours dans l’arborescence de l’éditeur de base de registre, allez dans 
**HKEY_LOCAL_MACHINE > SYSTEM > Setup > Status > SysprepStatus**

Dans la partie de droite, mettez la valeur **"CleanupState" sur 2** et la valeur **"GeneralizationState" sur 7**.

Fermez l’éditeur de la base de registre.

Ouvrez l'invite de commandes en tant qu'admin et saisissez les 2 commandes suivantes l’une après l’autre:
```
msdtc -uninstall
msdtc -install
```
il n’y aura aucun retour, c’est normal
MSDTC = Microsoft Distributed Transaction Coordinator

Vous pouvez relancer Sysprep.

---
EOF