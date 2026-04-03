## Phase 4 — Création du cluster Docker Swarm

**Cœur du projet !** On transforme tes 3 nœuds Docker en cluster Swarm haute disponibilité.[^1]

***

## Étape 4.1 — Initialiser le Swarm (manager UNIQUEMENT)

**Sur swarm-manager** :

```bash
docker swarm init --advertise-addr 192.168.10.10
```

**Résultat attendu** :

```
Swarm initialized: current node (xxxx) is now a manager.

To add a worker to this cluster:
docker swarm join --token SWMTKN-1-xxxxxxxx 192.168.10.10:2377

To add a manager to this cluster:
docker swarm join-token manager
```

![Initialisation du Swarm](images/Phase%204%20Screen/Pasted%20image%2020260402151532.png)
**Copie la commande `docker swarm join --token ...`** — tu en auras besoin.

***

## Étape 4.2 — Joindre les workers

**Sur swarm-worker1** :

```bash
docker swarm join --token SWMTKN-1-xxxxxxxx 192.168.10.10:2377
```

**Sur swarm-worker2** :

```bash
docker swarm join --token SWMTKN-1-xxxxxxxx 192.168.10.10:2377
```

**Token identique** pour les 2 workers.

![Jointure des workers](images/Phase%204%20Screen/Pasted%20image%2020260402151659.png)

***

## Étape 4.3 — Vérifier le cluster

**Sur manager** :

```bash
docker node ls
```

**Résultat attendu** :

```
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
xxxxx *                       swarm-manager       Ready               Active              Leader              27.3.1
xxxxx                         swarm-worker1       Ready               Active                          27.3.1
xxxxx                         swarm-worker2       Ready               Active                          27.3.1
```

![Vérification du cluster](images/Phase%204%20Screen/Pasted%20image%2020260402151740.png)

***

## Étape 4.4 — Infos détaillées

**Sur manager** :

```bash
docker node inspect self
docker info | grep -i swarm
```


***

## Étape 4.5 — Étiqueter les nœuds (optionnel mais pro)

**Sur manager** :

```bash
# Étiquettes pour placement services
docker node update --label-add type=app swarm-worker1
docker node update --label-add type=app swarm-worker2
docker node update --label-add type=control swarm-manager

# Vérif
docker node inspect swarm-worker1 | grep -A 5 '"Labels"'
```

![Étiquetage des nœuds](images/Phase%204%20Screen/Pasted%20image%2020260402152053.png)

✅ swarm-worker1 → label "type=app" ajouté
✅ swarm-worker2 → label "type=app" ajouté  
✅ swarm-manager → label "type=control" ajouté
✅ 3 nœuds Ready/Active
✅ Manager Leader

***

## Étape 4.6 — Test réseau Swarm

**Sur manager** :

```bash
docker network ls
```

Tu dois voir le réseau `docker_gwbridge`.

![Réseau Swarm](images/Phase%204%20Screen/Pasted%20image%2020260402152135.png)

