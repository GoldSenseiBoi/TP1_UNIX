# Compte Rendu TP 01 : Installation Serveurs
De Ibrahima Sory DIALLO

## Partie 1 : 



## Partie 2 : Post-Installation

### 2.1 Configuration SSH
Pour configurer le serveur SSH, voici les étapes suivies :
1. Utilisation de la commande `apt search openssh-server` pour vérifier si le serveur SSH est disponible.
2. Installation de SSH avec la commande `apt install openssh-server`.
3. Modification de la configuration SSH pour autoriser la connexion root avec mot de passe en modifiant le fichier `/etc/ssh/sshd_config` :
   - Remplacer `PermitRootLogin prohibit-password` par `PermitRootLogin yes`.
   - Redémarrage du service SSH avec `systemctl restart sshd`.
### 2.2 Connexion SSH
Je me suis connecté à la machine virtuelle depuis ma machine hôte avec la commande SSH :
```bash
ssh root@10.0.2.15
```

### 2.3 Vérification du nombre de paquets installés
Pour vérifier le nombre de paquets après l'installation minimale, j'ai utilisé :
```bash
dpkg -l | wc -l
```
Le résultat était de **354 paquets**, ce qui est proche de l'attendu (320).

### 2.4 Utilisation de l'espace disque
J'ai vérifié l'utilisation de l'espace disque avec :
```bash
df -h
```
La partition `/` utilise 1,2 Go, ce qui est bien en dessous de 1 Go, comme attendu pour une installation minimale.

### 2.5 Résumé des commandes utilisées et résultats

- **Locales :** Vérification de la locale avec la commande :
  ```bash
  echo $LANG
  ```
  Sortie : `fr_FR.UTF-8`
  
- **Nom de la machine :**
  ```bash
  hostname
  ```
  Sortie : `serveur1`

- **Domaine :**
  ```bash
  hostname -d
  ```
  Sortie : `ufr-info-p6.jussieu.fr`

- **Dépôts APT :**
  ```bash
  cat /etc/apt/sources.list | grep -v -E '^#|^$'
  ```
  Sortie :  
  ```
  deb http://ftp.fr.debian.org/debian/ bookworm main
  deb http://security.debian.org/debian-security bookworm-security main
  deb http://ftp.fr.debian.org/debian/ bookworm-updates main
  ```

- **Comptes utilisateurs :**
  ```bash
  cat /etc/shadow | grep -vE ':\*:|:!\*:'
  ```

- **Informations sur les partitions (fdisk) :**
  ```bash
  fdisk -l
  ```

---

## Partie 3 : Aller plus loin

### 3.1 Installation automatique avec Preseed
Pour automatiser l'installation d'une Debian, on peut utiliser un fichier `preseed`. Ce fichier contient les réponses aux questions posées pendant l'installation. Exemple d'un fichier minimal :
```bash
d-i debian-installer/locale string fr_FR
d-i netcfg/get_hostname string serveur1
d-i passwd/root-password password mypassword
```



### 3.2 Mode de secours : Changer le mot de passe root avec un Live CD/USB

Si le mot de passe root est perdu, il est possible de le réinitialiser en utilisant un système Live (Live CD ou clé USB). Voici les étapes détaillées :

1. **Démarrer sur un Live CD/USB :** Démarrer l'ordinateur sur un environnement de récupération ou un système d'exploitation en mode Live.

2. **Identifier la partition contenant le système :** Pour lister les partitions disponibles, exécuter la commande suivante :

   ```bash
   fdisk -l
   ```

   Cette commande permet d'identifier la partition contenant le système root (en général `/dev/sda1` ou une autre partition Linux).

3. **Monter la partition root :** Une fois la partition correcte identifiée, la monter sur le point de montage `/mnt` :

   ```bash
   mount /dev/sdXY /mnt
   ```

   Remplacer `sdXY` par la partition correspondante (par exemple, `/dev/sda1`).

4. **Vérifier le contenu de la partition :** Pour s'assurer que la bonne partition a été montée, lister son contenu :

   ```bash
   ls /mnt
   ```

   Si la partition est correcte, on devrait voir des répertoires habituels comme `/bin`, `/dev`, `/root`, etc.

5. **Changer le mot de passe root :** Si la partition est correcte, exécuter la commande suivante pour changer le mot de passe root :

   ```bash
   passwd root
   ```

   Le système demandera de saisir le nouveau mot de passe deux fois.

6. **Modification du fichier /etc/shadow (optionnel) :**

   Si nécessaire, il est possible de modifier manuellement le fichier `/etc/shadow`. Voici comment procéder :

   1. Ouvrir le fichier `/etc/shadow` avec un éditeur de texte (vim ou nano) :

      ```bash
      nano /mnt/etc/shadow
      ```

   2. Chercher la ligne correspondant à l'utilisateur root, elle devrait ressembler à ceci :

      ```perl
      root:$6$g9LWR1MJXjm0$IRP3uil/aSsDVR/HoCqXvTMUbp9.91z58MkiZSoHfFv3AuB54xQetmTP6E9Y6k2Wku:xxxxxx:0:99999:7:::
      ```

   3. Si une copie de la ligne de hash de mot de passe d'un autre système (ou sauvegarde) est disponible, elle peut être remplacée ici. Il est essentiel de changer tout ce qui vient après le premier `:` par le nouveau hash.

7. **Démonter la partition et redémarrer :** Après avoir changé le mot de passe ou modifié le fichier `/etc/shadow`, démonter la partition :

   ```bash
   umount /mnt
   ```

   Puis redémarrer le système avec la commande suivante :

   ```bash
   reboot
   ```

8. **Connexion avec le nouveau mot de passe root :** Une fois le redémarrage effectué, il est possible de se connecter avec le nouveau mot de passe root défini.

### 3.3 Redimensionnement de la partition en ligne de commande
Je souhaite redimensionner la partition racine sans réinstaller Debian et sans utiliser d’interface graphique. Voici les étapes pour y parvenir, tout en ligne de commande.

#### Étapes à suivre :

1. **Sauvegarde des données :**
   Avant toute chose, il faut faire une sauvegarde des fichiers importants car manipuler les partitions peut causer une perte de données.

2. **Redémarrer en mode Live CD/USB :**
   
   - Redémarre et accède au BIOS ou au menu de démarrage (touche `F12`, `Esc`, ou autre).
   - Boot sur le Live CD/USB en mode terminal (pas besoin d’interface graphique).

3. **Lister les partitions et vérifier l’état :**
   - Tape la commande suivante pour voir les partitions présentes sur le disque :
     ```bash
     fdisk -l
     ```
   - Prends note du nom de la partition à redimensionner (par exemple, `/dev/sda1`).

4. **Démonter la partition racine :**
   Pour redimensionner, il faut d'abord démonter la partition :
   ```bash
   umount /dev/sda1



### La suite 

- Pour redimensionner la partition à une taille spécifique (ex. 10 Go) :

    ```bash
    resize2fs /dev/sda1 10G
    ```

- Si vous souhaitez utiliser tout l'espace disponible sur la partition :

    ```bash
    resize2fs /dev/sda1
    ```



1. Lancez `fdisk` pour ajuster la table de partition :

    ```bash
    fdisk /dev/sda
    ```

2. Supprimez la partition existante sans effacer les données :

    - Tapez `d` pour supprimer la partition.

3. Créez une nouvelle partition avec le même numéro de début :

    - Tapez `n` pour créer une nouvelle partition.
    - **ATTENTION :** Ne modifiez pas le numéro de début des secteurs, sinon vous risquez de perdre les données.

4. Sauvegardez les changements et quittez :

    - Tapez `w` pour sauvegarder et quitter `fdisk`.



Après avoir modifié la taille de la partition et ajusté la table de partition, redémarrez le système avec la commande :

```bash
reboot
```



Une fois le système redémarré, vérifiez la nouvelle taille de la partition avec la commande suivante :

```bash
df -h
```





