## Phase 8 — Tests PCA / PRA (Simulations de panne)

**Phase critique pour la note !** Le sujet demande explicitement : *"procédures de tests de fonctionnement, simulation perte d'un container, vérification PCA/PRA"*.[^1]

**Objectif** : Prouver la **haute disponibilité** et la **reprise automatique**.

**Toutes les commandes sur MANAGER.**

***

## Test 8.1 — Perte d'un conteneur Nginx

**Avant** :

```bash
echo "=== AVANT PANNE ==="
docker service ps prod_nginx --no-trunc
```
![[Pasted image 20260402165010.png]]
**Panne** :

```bash
CONTAINER_ID=$(docker ps | grep nginx | awk '{print $1}' | head -1)
docker rm -f $CONTAINER_ID
```
![[Pasted image 20260402165022.png]]
**Après** (attends 30s) :

```bash
echo "=== APRÈS PANNE (30s) ==="
docker service ps prod_nginx --no-trunc
curl http://192.168.10.10
```

**Attendu** : Swarm recrée le conteneur, site toujours accessible.
![[Pasted image 20260402165048.png]]
***

## Test 8.2 — Arrêt forcé service Nginx

**Avant** :

```bash
echo "=== SERVICE NGINX AVANT ==="
docker service ps prod_nginx
```
![[Pasted image 20260402165106.png]]
**Panne** :

```bash
docker service update --force prod_nginx
```
![[Pasted image 20260402165137.png]]
**Après** :

```bash
echo "=== SERVICE NGINX APRÈS (1min) ==="
docker service ps prod_nginx
curl http://192.168.10.10
```

**Attendu** : Nouveau conteneur créé, service toujours UP.
![[Pasted image 20260402165209.png]]
***

## Test 8.3 — Panne d'un worker

**Avant** :

```bash
echo "=== WORKERS AVANT ==="
docker node ls
docker service ps prod_nginx
```
![[Pasted image 20260402165304.png]]
**Panne worker1** :
**Sur worker1** : `sudo systemctl stop docker`

**Depuis manager** :

```bash
echo "=== WORKER1 DOWN ==="
docker node ls
docker service ps prod_nginx
curl http://192.168.10.10
```
![[Pasted image 20260402165358.png]]
**Après** (attends 1min) :

```
Worker1 → Down
Services replanifiés sur worker2/manager
Site toujours accessible
```


***

## Test 8.4 — Persistance NFS (MariaDB)

**Insérer donnée** :

```bash
docker exec $(docker ps | grep mariadb | awk '{print $1}') mariadb -u root -e "CREATE DATABASE test_pca; SHOW DATABASES;"
```

**Panne MariaDB** :

```bash
docker service update --force prod_mariadb
```

![[Pasted image 20260403092645.png]]

**Vérification persistance** :

```bash
sleep 30
docker exec $(docker ps | grep mariadb | awk '{print $1}') mariadb -u root -e "SHOW DATABASES;"
```

**Attendu** : `test_pca` toujours présent (NFS).
![[Pasted image 20260403092752.png]]
***

## Test 8.5 — Reprise worker

**Sur worker1** :

```bash
sudo systemctl start docker
```

**Sur manager** :

```bash
docker node ls
docker node update --availability active swarm-worker1
```
![[Pasted image 20260403092827.png]]

