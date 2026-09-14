# Dual Boot Windows 11 / Debian 13

## Objectif

Mettre en place un dual boot Debian 13 / Windows 11 afin de travailler dans des environnements Windows et Linux.

L'installation est réalisée sur une machine disposant initialement de Windows 11, configurée en UEFI avec un disque utilisant une table de partitions GPT.

## Environnement

### Systèmes d'exploitation

* Windows 11 (système existant)
* Debian 13

### Configuration du démarrage et du disque

* Firmware : UEFI
* Secure Boot : activé
* Table de partitions : GPT

### Outils et supports d'installation

* Rufus
* Clé USB bootable Debian 13 créée à partir de l'image ISO Debian
* Clé USB bootable GParted Live

### Systèmes de fichiers

* Partition Windows : NTFS
* Partition Debian : ext4

---

## Préparation

Avant l'installation de Debian :

1. Vérification des caractéristiques de la machine et de la configuration Windows avec `msinfo32`.
2. Vérification de l'espace de stockage disponible pour accueillir Debian.
3. Téléchargement de l'image ISO Debian 13 adaptée à l'architecture de la machine.
4. Création d'une clé USB bootable Debian avec Rufus.
5. Préparation d'environ 50 Go d'espace pour Debian par réduction de la partition Windows `C:`.

### Problème rencontré lors du redimensionnement

La gestion des disques Windows ne permettait de réduire la partition `C:` que d'environ 5,9 Go, malgré un espace libre largement supérieur.

Des vérifications ont donc été effectuées afin d'identifier la cause de cette limitation.

L'analyse des événements Windows liés à la défragmentation a notamment permis d'identifier des métadonnées NTFS non déplaçables associées à la MFT comme limite au redimensionnement.

Une consolidation de l'espace libre a été tentée avec :

```powershell
defrag C: /X
```

L'intégrité du système de fichiers a également été vérifiée avec :

```powershell
chkdsk C: /scan
```

Aucune erreur du système de fichiers n'a été détectée.

La réduction souhaitée restant impossible depuis Windows, j'ai choisi d'effectuer le redimensionnement hors ligne avec GParted Live.

> Le détail du diagnostic est disponible dans `troubleshooting/reduction-partition-windows.md`.

---

## Redimensionnement avec GParted Live

1. Création d'une clé USB bootable GParted Live à partir de son image ISO avec Rufus.
2. Redémarrage de l'ordinateur et accès au menu de démarrage UEFI.
3. Démarrage sur la clé GParted Live.
4. Réduction de la partition Windows `C:` d'environ 50 Go afin d'obtenir de l'espace non alloué destiné à Debian.
5. Application du redimensionnement.

GParted a confirmé la fin du redimensionnement sans erreur.

Windows a ensuite été redémarré afin de vérifier que le système restait fonctionnel après la modification de sa partition.

---

## Installation de Debian 13

L'installation de Debian 13 a été réalisée depuis la clé USB bootable.

Le partitionnement a été réalisé manuellement afin de conserver les partitions Windows existantes.

Pour Debian :

* système de fichiers : `ext4` ;
* point de montage : `/` ;
* utilisation d'une partition principale pour le système Debian.

La partition EFI existante a été conservée sans formatage afin de préserver la configuration de démarrage existante.

Pendant l'installation, les composants suivants ont notamment été sélectionnés :

* environnement de bureau GNOME ;
* utilitaires usuels du système ;
* serveur SSH.

---

## Démarrage des systèmes

GRUB permet de sélectionner le système à démarrer.

Les deux systèmes ont été testés :

* Debian 13 : démarrage fonctionnel ;
* Windows 11 : démarrage fonctionnel.

```text
Démarrage
    |
   GRUB
  /    \
 /      \
Debian   Windows 11
  |          |
  OK         OK
```

---

## Vérifications sous Windows

Après l'installation, un retour sous Windows a permis de vérifier :

* le démarrage correct de Windows ;
* l'organisation des partitions après l'installation de Debian ;
* l'état du volume Windows ;
* l'état du chiffrement du volume `C:`.

Le volume Windows a notamment été contrôlé avec `manage-bde`.

### Hibernation et démarrage rapide

L'hibernation Windows a volontairement été laissée désactivée.

Ce choix maintient également le démarrage rapide de Windows indisponible et évite de laisser les volumes Windows dans un état lié à l'hibernation susceptible de poser problème lors d'un accès depuis Linux.

L'état actuel peut être vérifié avec :

```powershell
powercfg /a
```

Le fichier d'échange Windows (`pagefile.sys`) est quant à lui actif.

---

## Post-installation Debian

Après l'installation, le serveur SSH a été vérifié afin de confirmer :

* que le service était actif ;
* qu'il était configuré pour démarrer automatiquement ;
* qu'il écoutait sur le port TCP 22.

Un test de connexion SSH locale a également été réalisé.

---

## Ce que j'ai appris

Ce lab m'a permis de travailler notamment sur :

* la différence entre UEFI, GPT, partition et système de fichiers ;
* le fonctionnement général d'un dual boot ;
* le redimensionnement d'une partition NTFS ;
* l'utilisation de GParted Live pour intervenir hors ligne ;
* le rôle de GRUB dans le démarrage des systèmes ;
* l'importance de vérifier l'intégrité d'un système de fichiers avant une modification de partition ;
* l'importance des vérifications après intervention ;
* certains effets de l'hibernation et du démarrage rapide Windows dans un environnement dual boot.
