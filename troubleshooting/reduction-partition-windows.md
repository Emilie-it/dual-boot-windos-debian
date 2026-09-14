# Troubleshooting — Réduction de la partition Windows

## Symptôme

L'objectif était de réduire la partition Windows `C:` d'environ 50 Go afin de libérer de l'espace pour l'installation de Debian 13.

Malgré un espace libre largement suffisant sur le volume, l'outil de gestion des disques de Windows ne proposait qu'environ **5,9 Go de réduction maximale**.

Le problème ne semblait donc pas provenir d'un manque d'espace de stockage disponible.

---

## Diagnostic

L'hypothèse était que des éléments non déplaçables présents sur la partition NTFS empêchaient Windows de déplacer suffisamment la limite de la partition.

### Vérification de l'hibernation et du fichier d'échange

L'état de l'hibernation et du fichier d'échange Windows a été examiné afin de déterminer s'ils pouvaient limiter le redimensionnement.

Les commandes utilisées comprenaient notamment :

```powershell
powercfg /a
Test-Path C:\hiberfil.sys
Get-CimInstance Win32_PageFileUsage
Test-Path C:\pagefile.sys
```

### Analyse des événements Windows

L'analyse des événements Windows liés à la défragmentation a permis d'identifier comme élément non déplaçable :

```text
\$Mft::$BITMAP
```

Cet élément appartient aux métadonnées de la **MFT (Master File Table)** du système de fichiers NTFS.

La MFT est une structure essentielle de NTFS utilisée pour conserver les informations permettant de référencer les fichiers et répertoires présents sur le volume.

---

## Tentative de consolidation de l'espace libre

Une consolidation de l'espace libre a été tentée avec :

```powershell
defrag C: /X
```

L'option `/X` demande notamment à Windows de consolider l'espace libre du volume.

Après cette opération, la réduction disponible restait insuffisante.

---

## Vérification du système de fichiers

L'intégrité du système de fichiers NTFS a ensuite été contrôlée avec :

```powershell
chkdsk C: /scan
```

La vérification n'a détecté aucune erreur nécessitant une réparation.

Le blocage du redimensionnement ne semblait donc pas provenir d'une corruption du système de fichiers.

---

## Solution retenue : GParted Live

Après ces vérifications, la réduction souhaitée restant impossible depuis Windows, j'ai choisi d'effectuer le redimensionnement **hors ligne** avec GParted Live.

Une clé USB bootable GParted Live a été créée avec Rufus.

Après démarrage de l'ordinateur sur cette clé, GParted a permis de réduire la partition Windows `C:` d'environ **50 Go**.

L'espace ainsi libéré pouvait ensuite être utilisé pour l'installation de Debian 13.

---

## Vérifications

GParted a terminé l'opération sans signaler d'erreur.

Windows a ensuite été redémarré afin de vérifier :

* que le système démarrait toujours correctement ;
* que la modification de la partition avait bien été prise en compte ;
* que l'organisation du disque était cohérente après le redimensionnement.

---

## Résultat

La limitation rencontrée avec l'outil Windows a été contournée en effectuant le redimensionnement hors ligne avec GParted Live.

L'espace nécessaire à l'installation de Debian 13 a ainsi pu être libéré sans supprimer l'installation existante de Windows 11.

---

## Ce que j'ai appris

Ce dépannage m'a permis de comprendre plusieurs points :

* espace libre sur un volume et espace réductible ne désignent pas nécessairement la même chose ;
* certains éléments non déplaçables peuvent limiter la réduction d'une partition NTFS depuis Windows ;
* la MFT est une structure essentielle du système de fichiers NTFS ;
* `chkdsk` permet notamment de contrôler l'intégrité d'un système de fichiers ;
* GParted Live permet d'intervenir sur les partitions sans démarrer le système installé sur celles-ci ;
* après une modification de partition, il est important de vérifier que les systèmes concernés démarrent toujours correctement.
