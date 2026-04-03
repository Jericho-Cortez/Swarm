## Phase 5 — Configuration NFS (Stockage persistant)

**Objectif** : Configurer le serveur NFS pour les volumes Docker (MariaDB, Registry) et tester la persistance.[^1]

***

## Étape 5.1 — Installation NFS Server

**Sur `nfs-server`** :

```bash
apt update
apt install -y nfs-kernel-server
```


***

## Étape 5.2 — Créer les répertoires partagés

**Sur `nfs-server`** :

```bash
mkdir -p /srv/nfs/{docker,registry,mariadb,code}
chown -R nobody:nogroup /srv/nfs
chmod -R 777 /srv/nfs
ls -la /srv/nfs
```


***

## Étape 5.3 — Configurer /etc/exports

**Sur `nfs-server`** :

```bash
cat > /etc/exports << 'EOF'
# NFS pour Docker Swarm
/srv/nfs/docker        192.168.10.0/24(rw,sync,no_subtree_check,no_root_squash)
/srv/nfs/registry      192.168.10.0/24(rw,sync,no_subtree_check,no_root_squash)
/srv/nfs/mariadb       192.168.10.0/24(rw,sync,no_subtree_check,no_root_squash)
/srv/nfs/code          192.168.10.0/24(rw,sync,no_subtree_check,no_root_squash)
EOF
```

Appliquer :

```bash
exportfs -rav
systemctl restart nfs-kernel-server
systemctl enable nfs-kernel-server
systemctl status nfs-kernel-server
```


***

## Étape 5.4 — Installer client NFS (manager + workers)

**Sur manager** :

```bash
apt install -y nfs-common
```

**Copier vers workers** :

```bash
ssh devops@swarm-worker1 'sudo apt update && sudo apt install -y nfs-common'
ssh devops@swarm-worker2 'sudo apt update && sudo apt install -y nfs-common'
```


***

## Étape 5.5 — Test NFS depuis manager

**Sur manager** :

```bash
showmount -e 192.168.10.20
mkdir /mnt/nfs-test
mount -t nfs 192.168.10.20:/srv/nfs/docker /mnt/nfs-test
df -h | grep nfs
echo "Test NFS $(date)" > /mnt/nfs-test/testfile.txt
cat /mnt/nfs-test/testfile.txt
ls -la /mnt/nfs-test
umount /mnt/nfs-test
```

![[Pasted image 20260402153239.png]]

***

## Étape 5.6 — Test NFS depuis workers

**Depuis manager** :

```bash
ssh devops@swarm-worker1 'sudo mkdir /mnt/nfs-test && sudo mount -t nfs 192.168.10.20:/srv/nfs/docker /mnt/nfs-test && ls /mnt/nfs-test'
ssh devops@swarm-worker2 'sudo mkdir /mnt/nfs-test && sudo mount -t nfs 192.168.10.20:/srv/nfs/docker /mnt/nfs-test && ls /mnt/nfs-test'
```


***

## Étape 5.7 — Vérification finale NFS

**Sur manager** :

```bash
# Tous les exports visibles
showmount -e 192.168.10.20

# Fichier persistant visible
ssh root@192.168.10.20 'cat /srv/nfs/docker/testfile.txt'
```

**Résultat attendu** :

```
/srv/nfs/docker        192.168.10.0/24
/srv/nfs/registry      192.168.10.0/24
Test NFS Thu Apr  2 ...
```

![[Pasted image 20260402155430.png]]
![[Pasted image 20260402155353.png]]

***

## Captures Phase 5

| Test | Commande |
| :-- | :-- |
| Exports | `showmount -e 192.168.10.20` |
| Montage | `df -h | grep nfs` |
| Fichier test | `cat /mnt/nfs-test/testfile.txt` |
| Worker1 | `ssh worker1 'mount nfs'` |



