# Sysprep - Erreurs courantes 

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

Prêtez attention aux **codes erreur Microsoft** qui commencent par **0x** et utilisez-les pour effectuer des recherches afin d'en savoir plus sur la cause probable de l'erreur rencontrée.

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
