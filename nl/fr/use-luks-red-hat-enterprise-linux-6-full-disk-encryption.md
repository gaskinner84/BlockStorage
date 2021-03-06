---

copyright:
  years: 2014, 2019
lastupdated: "2019-02-05"

---
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Chiffrement de disque complet avec LUKS dans Red Hat Enterprise Linux
{: #LUKSencryption}

Vous pouvez chiffrer des partitions sur votre serveur Red Hat Enterprise Linux 6 avec Linux Unified Key Setup-on-disk-format (LUKS), ce qui est important lorsqu'il s'agit d'ordinateurs portables et de support amovible. LUKS permet à plusieurs clés d'utilisateur de déchiffrer une clé principale utilisée pour le chiffrement en bloc de la partition.

Cette procédure suppose que le serveur peut accéder à un nouveau volume {{site.data.keyword.blockstoragefull}}, non chiffré, qui n'a été ni formaté, ni monté. Pour plus d'informations sur la connexion de {{site.data.keyword.blockstorageshort}} à un hôte Linux, voir [Connexion à des numéros d'unité logique (LUN) iSCSI sous Linux](/docs/infrastructure/BlockStorage?topic=BlockStorage-mountingLinux).

{site.data.keyword.blockstorageshort}} qui mis à disposition dans des [centres de données sélectionnés](/docs/infrastructure/BlockStorage?topic=BlockStorage-news) est automatiquement doté du chiffrement au repos géré par le fournisseur. Pour plus d'information, voir [Sécurisation des données - Chiffrement au repos géré par le fournisseur](/docs/infrastructure/BlockStorage?topic=BlockStorage-encryption).
{:note}

## Opérations possibles avec LUKS

- Chiffrement de la totalité des unités par bloc et parfaite adaptation à la protection du contenu des périphériques mobiles tels qu'un support de stockage amovible ou des unités de disque d'ordinateur portable.
- Le contenu sous-jacent de l'unité par bloc chiffrée est arbitraire, ce qui le rend utile pour le chiffrement des unités de permutation. Le chiffrement peut également être utile avec certaines bases de données qui utilisent des unités par bloc spécialement formatées pour le stockage des données.
- Utilisation du sous-système de noyaux d'associateur de périphériques existant.
- Renforcement de la sécurité avec une phrase passe, qui fournit une protection contre les ajouts de dictionnaire sous forme de pièces jointes.
- Possibilité pour les utilisateurs d'ajouter des clés de sauvegarde ou des phrases passe car les unités LUKS contiennent plusieurs emplacements de clé.


## Opérations impossibles avec LUKS

- Possibilité pour les applications nécessitant un grand nombre d'utilisateurs (plus de huit) d'avoir des clés distinctes pour accéder aux mêmes unités.
- Utilisation d'applications nécessitant un chiffrement au niveau fichier. Pour plus d'informations, voir le document [RHEL Security Guide ![Icône de lien externe](../../icons/launch-glyph.svg "Icône de lien externe")](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Security_Guide/sec-Encryption.html){:new_window}.

## Configuration d'un volume chiffré LUKS avec {{site.data.keyword.blockstorageshort}} Endurance

Le processus de chiffrement de données crée une charge sur l'hôte, qui risque d'impacter les performances.
{:note}

1. Saisissez la commande suivante à une invite shell en tant que root pour installer le package requis :   <br/>
   ```
   # yum install cryptsetup-luks
   ```
   {: pre}
2. Obtenez l'ID de disque :<br/>
   ```
   # fdisk –l | grep /dev/mapper
   ```
   {: pre}
3. Localisez votre volume dans la liste.
4. Chiffrez l'unité par bloc :

   1. Cette commande initialise le volume et vous permet de définir une phrase passe. <br/>

      ```
      # cryptsetup -y -v luksFormat /dev/mapper/3600a0980383034685624466470446564
      ```
      {: pre}

   2. Répondez par YES (tout en majuscules).

   3. Le périphérique apparaît maintenant sous forme de volume chiffré :

      ```
      # blkid | grep LUKS
      /dev/mapper/3600a0980383034685624466470446564: UUID="46301dd4-035a-4649-9d56-ec970ceebe01" TYPE="crypto_LUKS"
      ```

5. Ouvrez le volume et créez un mappage.<br/>
   ```
   # cryptsetup luksOpen /dev/mapper/3600a0980383034685624466470446564 cryptData
   ```
   {: pre}
6. Entrez la phrase de passe.
7. Vérifiez le mappage et affichez le statut du volume chiffré.   <br/>
   ```
   # cryptsetup -v status cryptData
   /dev/mapper/cryptData is active.
     type:  LUKS1
     cipher:  aes-cbc-essiv:sha256
     keysize: 256 bits
     device:  /dev/mapper/3600a0980383034685624466470446564
     offset:  4096 sectors
     size:    41938944 sectors
     mode:    read/write
     Command successful
   ```
8. Ecrivez des données aléatoires sur l'unité chiffrée `/dev/mapper/cryptData`. Ainsi, le monde extérieur les considérera comme des données aléatoires, ce qui signifie qu'elles seront protégées contre la divulgation des modèles d'utilisation. Cette étape peut prendre un certain temps.<br/>
    ```
    # shred -v -n1 /dev/mapper/cryptData
    ```
    {: pre}
9. Formatez le volume.<br/>
   ```
   # mkfs.ext4 /dev/mapper/cryptData
   ```
   {: pre}
10. Montez le volume.<br/>
   ```
   # mkdir /cryptData
   ```
   {: pre}
   ```
   # mount /dev/mapper/cryptData /cryptData
   ```
   {: pre}
   ```
   # df -H /cryptData
   ```
   {: pre}

### Démontage et fermeture du volume chiffré en toute sécurité
   ```
   # umount /cryptData
   # cryptsetup luksClose cryptData
   ```
   {: codeblock}

### Remontage et montage d'une partition chiffrée LUKS existante
   ```
   # cryptsetup luksOpen /dev/mapper/3600a0980383034685624466470446564 cryptData
      Enter the password previously provided.
   # mount /dev/mapper/cryptData /cryptData
   # df -H /cryptData
   # lsblk
   NAME                                       MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
   xvdb                                       202:16   0    2G  0 disk
   └─xvdb1                                    202:17   0    2G  0 part  [SWAP]
   xvda                                       202:0    0   25G  0 disk
   ├─xvda1                                    202:1    0  256M  0 part  /boot
   └─xvda2                                    202:2    0 24.8G  0 part  /
   sda                                          8:0    0   20G  0 disk
   └─3600a0980383034685624466470446564 (dm-0) 253:0    0   20G  0 mpath
   └─cryptData (dm-1)                       253:1    0   20G  0 crypt /cryptData
   sdb                                          8:16   0   20G  0 disk
   └─3600a0980383034685624466470446564 (dm-0) 253:0    0   20G  0 mpath
   └─cryptData (dm-1)                       253:1    0   20G  0 crypt /cryptData
   ```
