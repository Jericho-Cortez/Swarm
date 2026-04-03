## Phase 2 — Installation Debian CLI sur les 4 VM

Cette phase consiste à installer Debian 12 en mode **CLI pur** sur chaque VM, configurer les IP fixes, les hostnames, SSH, et vérifier que toutes les machines se voient entre elles.[^1]

***

## Étape 2.1 — Télécharger Debian 12 Netinstall

Télécharge l'ISO **minimal** (pas le DVD) :
👉 **https://www.debian.org/distrib/netinst**
Choisis `amd64` → `netinst` (~400 Mo)

> ⚠️ N'utilise **pas** l'ISO DVD complet — il installe des paquets inutiles.

***

## Étape 2.2 — Créer les 4 VM dans ton hyperviseur

Crée **4 VM identiques** avec ces paramètres :


| Paramètre | Manager / Workers | NFS Server |
| :-- | :-- | :-- |
| OS | Debian 12 64-bit | Debian 12 64-bit |
| CPU | 2 vCPU | 1 vCPU |
| RAM | 2048 Mo | 1024 Mo |
| Disque | 20 Go | 30 Go |
| Réseau | Bridge (même réseau que ton host) | Bridge |
| ISO | debian-12-netinst.iso | debian-12-netinst.iso |


***

## Étape 2.3 — Installation Debian (même procédure sur les 4 VM)

Lance chaque VM et suis ces étapes dans l'installeur :

```
1. Install  →  choisir "Install" (pas Graphical Install)
2. Langue   →  English ou French (ton choix)
3. Pays     →  France
4. Clavier  →  Français
5. Hostname →  (voir tableau ci-dessous)
6. Domain   →  laisser vide ou "local"
7. Root password → définis un mot de passe fort (ex: Admin123!)
8. Utilisateur  →  crée un user (ex: devops)
9. Partitionnement → "Guided - use entire disk" → 1 seule partition
10. Mirror  →  deb.debian.org
11. Packages → NE COCHER QUE :
       ✅ SSH server
       ✅ standard system utilities
       ❌ PAS de Desktop / GUI
12. GRUB    →  installer sur /dev/sda
13. Redémarrer
```


### Hostname à utiliser pour chaque VM

| VM   | Hostname à saisir |
| :--- | :---------------- |
| VM 1 | `swarm-manager`   |
| VM 2 | `swarm-worker1`   |
| VM 3 | `swarm-worker2`   |
| VM 4 | `nfs-server`      |


***

## Étape 2.4 — Configurer l'IP fixe sur chaque VM

Connecte-toi en root sur chaque VM, puis configure l'interface réseau.

### Identifier ton interface réseau

```bash
ip a
```

L'interface s'appelle généralement `ens33`, `eth0` ou `enp0s3`.

### Éditer le fichier réseau

```bash
nano /etc/network/interfaces
```

**Sur swarm-manager :**

```
auto lo
iface lo inet loopback

auto ens33
iface ens33 inet static
    address 192.168.10.10
    netmask 255.255.255.0
    gateway 192.168.10.1
    dns-nameservers 8.8.8.8 8.8.4.4
```

**Sur swarm-worker1 :**

```
auto ens33
iface ens33 inet static
    address 192.168.10.11
    netmask 255.255.255.0
    gateway 192.168.10.1
    dns-nameservers 8.8.8.8 8.8.4.4
```

**Sur swarm-worker2 :**

```
auto ens33
iface ens33 inet static
    address 192.168.10.12
    netmask 255.255.255.0
    gateway 192.168.10.1
    dns-nameservers 8.8.8.8 8.8.4.4
```

**Sur nfs-server :**

```
auto ens33
iface ens33 inet static
    address 192.168.10.20
    netmask 255.255.255.0
    gateway 192.168.10.1
    dns-nameservers 8.8.8.8 8.8.4.4
```

Applique le réseau :

```bash
systemctl restart networking
ip a
```

> ⚠️ Remplace `ens33` par le nom réel de ton interface si différent.

***

## Étape 2.5 — Configurer /etc/hosts sur les 4 VM

Sur **chaque VM**, édite `/etc/hosts` :

```bash
nano /etc/hosts
```

Ajoute ces lignes à la fin :

```
192.168.10.10   swarm-manager
192.168.10.11   swarm-worker1
192.168.10.12   swarm-worker2
192.168.10.20   nfs-server
```


***

## Étape 2.6 — Paquets de base utiles

Sur **chaque VM** :

```bash
apt update && apt upgrade -y
apt install -y curl wget vim git sudo gnupg2 \
  ca-certificates lsb-release nfs-common net-tools
```


***

## Étape 2.7 — Tests de connectivité

### Depuis swarm-manager, pinge toutes les machines :

```bash
ping -c 3 swarm-worker1
ping -c 3 swarm-worker2
ping -c 3 nfs-server
ping -c 3 8.8.8.8
```


### Vérifie le hostname de chaque VM :

```bash
hostnamectl
```


### Teste SSH depuis le manager vers chaque nœud :

```bash
ssh devops@swarm-worker1
ssh devops@swarm-worker2
ssh devops@nfs-server
```


***

## Étape 2.8 — Configurer SSH sans mot de passe (bonus pro)

Depuis le **manager**, génère une clé SSH et copie-la vers les autres VM :

```bash
ssh-keygen -t ed25519 -C "swarm-manager"
ssh-copy-id devops@swarm-worker1
ssh-copy-id devops@swarm-worker2
ssh-copy-id devops@nfs-server
```

Résultat : connexion SSH sans mot de passe entre le manager et les workers → utile pour les scripts.

***

## Checklist Phase 2 ✅

Avant de passer à la Phase 3, vérifie :

- [x] 4 VM Debian installées en CLI
- [x] Pas d'interface graphique installée
- [x] SSH actif sur les 4 machines
- [x] IP fixe configurée sur chaque VM
- [x] `/etc/hosts` rempli sur les 4 machines
- [x] Ping entre toutes les VM fonctionne
- [x] Connexion SSH depuis le manager vers les workers et NFS

***

### Captures à faire pour la doc

| Capture | Commande |
| :-- | :-- |
| Hostname OK | `hostnamectl` |
| IP fixe OK | `ip a` |
| Ping OK | `ping -c 3 swarm-worker1` |
| SSH OK | connexion réussie |
| Hosts configuré | `cat /etc/hosts` |


