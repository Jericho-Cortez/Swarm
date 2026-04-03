## Phase 7 — Tests formels (tous sur MANAGER)

**Toutes les commandes s'exécutent sur `swarm-manager`** (sauf mention contraire).

***

## Test 7.1 — État stack complet

```bash
# MANAGER
docker stack services prod --format "table {{.Name}}\t{{.Replicas}}\t{{.Image}}\t{{.Ports}}"
```

**Attendu** : 5 services, tous READY.

![[Pasted image 20260402163952.png]]

## Test 7.2 — Distribution services

```bash
# MANAGER
docker service ps prod_nginx --no-trunc | head -5
docker service ps prod_php --no-trunc | head -5
```

**Attendu** : Nginx/PHP sur différents workers.

![[Pasted image 20260402164022.png]]

## Test 7.3 — Nginx + PHP

```bash
# MANAGER
curl -s http://192.168.10.10 | head -10
curl -I http://192.168.10.10
```

```
**Attendu** : `<h1>Docker Swarm OK</h1>`, `HTTP/1.1 200`.
```

![[Pasted image 20260402164254.png]]
## Test 7.4 — Registry

```bash
# MANAGER
curl -s http://192.168.10.10:5000/v2/
```

**Attendu** : `{}`
![[Pasted image 20260402164331.png]]
## Test 7.5 — Code-server

```bash
# MANAGER
curl -I http://192.168.10.10:8443
```

**Attendu** : `HTTP/1.1 401 Unauthorized`
![[Pasted image 20260402164428.png]]
## Test 7.6 — MariaDB

```bash
# MANAGER
docker service logs prod_mariadb --tail 5
```

**Attendu** : `mysqld: ready for connections`.
![[Pasted image 20260402164503.png]]
## Test 7.7 — Volumes NFS

```bash
# MANAGER
docker volume ls | grep prod
docker volume inspect prod_mariadb_data | grep -A 5 "DriverOpts"
```

**Attendu** : `type: nfs`, `device: ":/srv/nfs/mariadb"`
![[Pasted image 20260402164521.png]]
## Test 7.8 — Réseau overlay

```bash
# MANAGER
docker network ls | grep overlay
docker network inspect prod_backend --format '{{range .Containers}}{{.Name}} {{end}}'
```

**Attendu** : `prod_backend` overlay + noms conteneurs.
![[Pasted image 20260402164543.png]]
***

## Différence Phase 6 vs Phase 7

| Phase 6 | Phase 7 |
| :-- | :-- |
| Déploiement initial | Tests formels documentés |
| `docker service ls` basique | `docker stack services` formaté |
| Tests manuels | Checklist structurée |
| Preuves rapides | Captures pour soutenance |

