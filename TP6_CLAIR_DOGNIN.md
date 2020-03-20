# CPE Lyon - 4ETI - Année 2018/19 Administration Système 
## TP 6 - Gestion des disques, Boot, Gestion des logs 

## Exercice 1. Disques et partitions

**1. Dans l’interface de configuration de votre VM, créez un second disque dur, de 5 Go dynamiquement alloués ; puis démarrez la VM**

**2. Vérifiez que ce nouveau disque dur est bien détecté par le système**

```
serveur@serveur:~$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0    7:0    0 54,7M  1 loop /snap/lxd/12181
loop1    7:1    0 89,1M  1 loop /snap/core/7917
loop2    7:2    0 54,6M  1 loop /snap/lxd/11985
loop3    7:3    0   89M  1 loop /snap/core/7713
sda      8:0    0   10G  0 disk
├─sda1   8:1    0    1M  0 part
└─sda2   8:2    0   10G  0 part /
sdb      8:16   0    5G  0 disk
sr0     11:0    1 1024M  0 rom
serveur@serveur:~$

```

> Le disque est bien détecté sous le nom sdb.

**3. Partitionnez ce disque en utilisant fdisk : créez une première partition de 2 Go de type Linux (n°83), et une seconde partition de 3 Go en NTFS (n°7)**
>On utilise la commande :
 ```sudo fdisk /dev/sda```
```
Device Boot   Start      End Sectors Size Id Type
sdb1           2048  4196351 4194304   2G 83 Linux
sdb2        4196352 10485759 6289408   3G  7 HPFS/NTFS/exFAT

```

**4. A ce stade, les partitions ont été créées, mais elles n’ont pas été formatées avec leur système de fichiers. A l’aide de la commande mkfs, formatez vos deux partitions ( pensez à consulter le manuel !)**

```
serveur@serveur:/dev$ sudo mkfs.ext4 sdb1
mke2fs 1.44.6 (5-Mar-2019)
Creating filesystem with 524288 4k blocks and 131072 inodes
Filesystem UUID: 297479ef-95a0-46fe-954d-480e82e72ef5
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

serveur@serveur:/dev$ sudo mkfs.ntfs sdb2
Cluster size has been automatically set to 4096 bytes.
Initializing device with zeroes: 100% - Done.
Creating NTFS volume structures.
mkntfs completed successfully. 
```

**5. Pourquoi la commande df -T, qui affiche le type de système de fichier des partitions, ne fonctionne-telle pas sur notre disque ?**

>Parce que  le disque n'est pas connecté au système principal. Celui-ci n'est pas monté sur l'OS.

**6. Faites en sorte que les deux partitions créées soient montées automatiquement au démarrage de la machine, respectivement dans les points de montage /data et /win (vous pourrez vous passer des UUID en raison de l’impossibilité d’effectuer des copier-coller)**

```
UUID=146057e5-0fa9-47f8-94f0-eff90b97421f / ext4 defaults 0 0
/swap.img       none    swap    sw      0       0
UUID=297479ef-95a0-46fe-954d-480e82e72ef5 /data ext4 defaults 0 0
UUID=77957BE920BF54E1 /win ntfs defaults 0 0

```

**7. Utilisez la commande mount puis redémarrez votre VM pour valider la configuration**

```
serveur@serveur:/$ sudo mount /dev/sdb1
serveur@serveur:/$ sudo mount /dev/sdb2

```

_reboot_

```
serveur@serveur:~$ df -aTh
Filesystem     Type        Size  Used Avail Use% Mounted on
/dev/sdb1      ext4        2,0G  6,0M  1,8G   1% /data
/dev/sdb2      fuseblk     3,0G   16M  3,0G   1% /win
serveur@serveur:~$           

```
>On a supprimé une partie des partitions pour éviter une copie trop volumineuse

**8. Montez votre clé USB dans la VM**

**9. Créez un dossier partagé entre votre VM et votre système hôte (rem. il peut être nécessaire d’installer les Additions invité de VirtualBox**


## Exercice 2. Personnalisation de GRUB

**1. Commencez par changer l’extension du fichier /etc/default/grub.d/50-curtin-settings.cfg s’il est présent dans votre environnement (vous pouvez aussi commenter son contenu).**

```
serveur@serveur:/etc/default/grub.d$ sudo mv 50-curtin-settings.cfg 50-curtin-settings.cfg.back

```

**2. Modifiez le fichier /etc/default/grub pour que le menu de GRUB s’affiche pendant 10 secondes ; passé ce délai, le premier OS du menu doit être lancé automatiquement.**

```
GRUB_TIMEOUT=10

```

**3. Lancez la commande update-grub**

```
serveur@serveur:/etc/default$ sudo update-grub
Sourcing file `/etc/default/grub'
Sourcing file `/etc/default/grub.d/init-select.cfg'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.0.0-31-generic
Found initrd image: /boot/initrd.img-5.0.0-31-generic
Found linux image: /boot/vmlinuz-5.0.0-29-generic
Found initrd image: /boot/initrd.img-5.0.0-29-generic
Found linux image: /boot/vmlinuz-5.0.0-27-generic
Found initrd image: /boot/initrd.img-5.0.0-27-generic
done

```

**4. Redémarrez votre VM pour valider que les changements ont bien été pris en compte**

Les changements ont bien été pris en compte.

**5. On va augmenter la résolution de GRUB et de notre VM. Cherchez sur Internet le ou les paramètres à rajouter au fichier grub.**

```
# The resolution used on graphical terminal
# note that you can use only modes which your graphic card supports via VBE
# you can see them in real GRUB with the command `vbeinfo'
GRUB_GFXMODE=1280x800 

```

**6. On va à présent ajouter un fond d’écran. Il existe un paquet en proposant quelques uns : grub2-splash-images (après installation, celles-ci sont disponibles dans /usr/share/images/grub).**

```
root@serveur:/etc/default# apt-get install grub2-spashimages

```

**7. Il est également possible de configurer des thèmes. On en trouve quelques uns dans les dépôts (grub2-themes-*). Installez-en un.**

```
root@serveur:/etc/default# apt-get install grub2-themes-ubuntu-mate

```

**8. Ajoutez une entrée permettant d’arrêter la machine, et une autre permettant de la redémarrer.**

**9. Configurer GRUB pour que le clavier soit en français**
```
sudo grub-kbdcomp -o /boot/grub/bepo.gkb fr
nano /etc/default/grub
#GRUB_HIDDEN_TIMEOUT=0 GRUB_TERMINAL_INPUT="at_keyboard"
nano /etc/grub.d/40_custom
#!/bin/sh 
exec tail -n +3 $0
insmod keylayouts 
keymap /boot/grub/bepo.gkb
sudo update-grub
```
## Exercice 3. Noyau

Dans cet exercice, on va créer et installer un module pour le noyau.

**1. Commencez par installer le paquet build-essential, qui contient tous les outils nécessaires (compilateurs, bibliothèques) à la compilation de programmes en C (entre autres).**

**2. Créez un fichier hello.c contenant le code suivant :**

```
#include <linux/module.h>
#include <linux/kernel.h>

MODULE_LICENSE("GPL");
MODULE_AUTHOR("John Doe");
MODULE_DESCRIPTION("Module hello world");
MODULE_VERSION("Version 1.00");

int init_module(void)
{
printk(KERN_INFO "[Hello world] - La fonction init_module() est appelée.\n");
return 0;
}

void cleanup_module(void)
{
printk(KERN_INFO "[Hello world] - La fonction cleanup_module() est appelée.\n");
}

```

**3. Créez également un fichier Makefile :**

```
obj-m += hello.o

all:
make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean

install:
cp ./hello.ko /lib/modules/$(shell uname -r)/kernel/drivers/misc

```

**4. Compilez le module à l’aide de la commande make, puis installez-le à l’aide de la commande make install.**

```
root@serveur:~# make
make -C /lib/modules/5.0.0-29-generic/build M=/root modules
make[1]: Entering directory '/usr/src/linux-headers-5.0.0-29-generic'
  CC [M]  /root/hello.o
  Building modules, stage 2.
  MODPOST 1 modules
  CC      /root/hello.mod.o
  LD [M]  /root/hello.ko
make[1]: Leaving directory '/usr/src/linux-headers-5.0.0-29-generic'
root@serveur:~# make install
cp ./hello.ko /lib/modules/5.0.0-29-generic/kernel/drivers/misc

```

**5. Chargez le module ; vérifiez dans le journal du noyau que le message ”La fonction init_module() est appelée” a bien été inscrit, synonyme que le module a été chargé ; confirmez avec la commande lsmod.**

```
root@serveur:/lib/modules/5.0.0-29-generic/kernel/drivers/misc# journalctl | grep "init_module"
oct. 21 09:24:52 serveur kernel: [Hello world] - La fonction init_module() est appelée.
root@serveur:/lib/modules/5.0.0-29-generic/kernel/drivers/misc# lsmod
Module                  Size  Used by
hello                  16384  0

```

**6. Utilisez la commande modinfo pour obtenir des informations sur le module hello.ko ; vous devriez notamment voir les informations figurant dans le fichier C.**

```
root@serveur:/lib/modules/5.0.0-29-generic/kernel/drivers/misc# modinfo hello.ko
filename:       /lib/modules/5.0.0-29-generic/kernel/drivers/misc/hello.ko
version:        Version 1.00
description:    Module hello world
author:         John Doe
license:        GPL
srcversion:     4398A2271F215E3A6F58078
depends:
retpoline:      Y
name:           hello
vermagic:       5.0.0-29-generic SMP mod_unload

```

**7. Déchargez le module ; vérifiez dans le journal du noyau que le message ”La fonction cleanup_module() est appelée” a bien été inscrit, synonyme que le module a été déchargé ; confirmez avec la commande lsmod.**

```
root@serveur:/lib/modules/5.0.0-29-generic/kernel/drivers/misc# rmmod hello
root@serveur:/lib/modules/5.0.0-29-generic/kernel/drivers/misc# journalctl | grep "cleanup_module"
oct. 21 09:31:18 serveur kernel: [Hello world] - La fonction cleanup_module() est appelée.

```

**8. Pour que le module soit chargé automatiquement au démarrage du système, il faut l’inscrire dans le fichier /etc/modules. Essayez, et vérifiez avec la commande lsmod après redémarrage de la machine.**

## Exercice 4. Exécution de commandes en différé : at et cron
**1. echo 'echo "Réunion" ' | at now +3 minutes**

**2. non**

**3. crontab -e */5 * * * * echo "Il faut réviser l'examen"**

**4. crontab -e */15 * * * * echo tâche**

> Mon collègue et moi même avons décidé de ne pas faire de mise en page pour cet exercice, pourquoi me direz vous ? Pour une raison bien simple. Aujourd'hui tout les comptes-rendus, les rapports sont polis lissés, peu d'originalité dans l'écriture ou même redondant. Avec mon collègue, nous vous proposons ici une expérience rafraîchissante, vierge de toute modification, produit brut de notre intellect,peut être un peu fatigué en cette fin de semaine de confinement certes mais intellect tout de même. Avec cet exercice vous retrouvez la fraîcheur des premiers jours, le frisson de la découverte, de l'inconnu ! Des réponses obscures qui peuvent avoir des significations pour le moins incertaines. C'est ça l'essence même de l'aventure des comptes-rendus
> Après ce léger craquage, je vous souhaite un bon week end de repos
> Cordialement, 
> L'équipe technique
## Exercice 5. Surveillance de l’activité du système

  

**1. Dans tty1, lancez la commande htop, puis tapez la commande w dans tty2. Qu’affiche cette commande ?**

  

>La commande affiche la liste des utilisateurs connectés avec d'autres informations supplémentaires, tel qu'une IP associés et les processus démarrer par l'utilisateur.

  

**2. Comment afficher l’historique des dernières connexions à la machine ?**

  

>Il est possible d'accéder à l'historique des connexion à la machine via la commande last

  

**3. Quelle commande permet d’obtenir la version du noyau ?**

  

>Il est possible d'obtenir la version actuel du noyaux via la commande uname -r

  

**4. Comment récupérer toutes les informations sur le processeur, au format JSON ?**

  

>Les informations du processeur sont accessible via la commande lshw -class processor. Ont peut définir le format json en rajoutant l'argument -json à la fin.

  

**5. Comment obtenir la liste des derniers démarrages de la machine avec la commande journalctl ? Comment afficher tout ce qu’il s’est passé sur la machine lors de l’avant-dernier boot ?**

  

>la commande **journalctl** peut afficher les logs des démarrages de la machine via la commande:

```

journalctl --list-boots

```

> Il est possible de spécifié un numéro de boot, par exemple la commande suivante sélectionne l'avant-dernier boot:

```

journalctl -b -1

```

**6. Comment obtenir la liste des derniers démarrages de la machine avec la commande journalctl ?**

  

> Il est possible d'obtenir la liste des derniers démarrages via la commande: **journalctl --list-boots**

  

**7. Faites en sortes que lors d’une connexion à la machine, les utilisateurs soient prévenus par un message à l’écran d’une maintenance le 26 mars à minuit.**

  

> Il est possible de définir le "message of the day" dans le fichier /etc/motd. J'ai donc écris le message dans ce fichier.

  

**8. Ecrivez un script bash qui permet de calculer le k-ième nombre de Fibonacci : Fk = Fk−1 + Fk−2, avec F0 = F1 = 1. Lancez le calcul de F100 puis lancez la commande tload depuis un autre terminal virtuel. Que constatez-vous ? Interrompez ensuite le calcul avec CTRL+C et observez la conséquence sur l’affichage de tload.**
