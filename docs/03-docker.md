## Phase 3 — Préparation système et installation de Docker

Cette phase sert à rendre toutes les VM Debian prêtes pour le cluster, puis à installer Docker proprement sur les nœuds qui participeront à Swarm. Le sujet repose sur des VM Debian dédiées à l’orchestration, au calcul et au stockage, donc cette étape prépare la base technique avant la création du cluster.[^1]

## Machines concernées

Docker doit être installé au minimum sur le **manager** et les **workers**, car ce sont eux qui exécuteront l’orchestration et les conteneurs applicatifs. Le serveur **NFS** sert au stockage persistant ; tu peux le laisser sans Docker pour garder une architecture plus propre et plus claire à expliquer en soutenance.[^1]


| VM | Docker requis | Raison |
| :-- | :-- | :-- |
| `swarm-manager` | Oui | Orchestration du cluster [^1] |
| `swarm-worker1` | Oui | Exécution des services [^1] |
| `swarm-worker2` | Oui | Exécution des services [^1] |
| `nfs-server` | Non obligatoire | Stockage persistant NFS [^1] |

## Étape 3.1 — Mise à jour système

Sur `swarm-manager`, `swarm-worker1` et `swarm-worker2` :

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y ca-certificates curl gnupg lsb-release apt-transport-https software-properties-common
```

Sur le `nfs-server`, tu peux aussi faire la mise à jour pour garder une base homogène :

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y ca-certificates curl gnupg lsb-release nfs-common
```


### Vérifications

```bash
cat /etc/debian_version
uname -a
hostnamectl
```


## Étape 3.2 — Nettoyer d’anciens paquets Docker

Sur le manager et les workers, supprime les anciennes versions éventuelles avant d’installer la version officielle Docker Engine :

```bash
sudo apt remove -y docker docker-engine docker.io containerd runc
```

Cette étape évite les conflits de paquets entre `docker.io` de Debian et `docker-ce` du dépôt officiel Docker.

## Étape 3.3 — Ajouter le dépôt officiel Docker

Sur le manager et les workers :

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

Puis ajoute le dépôt :

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo $VERSION_CODENAME) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Recharge l’index des paquets :

```bash
sudo apt update
```


## Étape 3.4 — Installer Docker Engine

Toujours sur le manager et les workers :

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Active et démarre Docker :

```bash
sudo systemctl enable docker
sudo systemctl start docker
```


### Vérifications

```bash
sudo systemctl status docker
docker --version
docker info
```

Tu dois voir que le service Docker est actif et que la commande répond correctement.

## Étape 3.5 — Autoriser l’utilisateur à utiliser Docker

Pour éviter d’utiliser `sudo` à chaque commande, ajoute ton utilisateur au groupe Docker sur les trois nœuds :

```bash
sudo usermod -aG docker $USER
newgrp docker
```

Vérifie ensuite :

```bash
docker ps
```

Si la commande répond sans erreur de permission, c’est bon.

## Étape 3.6 — Vérifier les ports et le pare-feu

Pour Swarm, certains ports devront être ouverts entre les nœuds ; même si tu n’actives pas de firewall maintenant, note-les dans ta doc technique. Cette anticipation t’aidera à justifier la connectivité du cluster en soutenance.

Ports à connaître :

- `2377/tcp` → management du cluster
- `7946/tcp` et `7946/udp` → communication entre nœuds
- `4789/udp` → overlay network

Si `ufw` est activé, autorise-les :

```bash
sudo ufw allow 2377/tcp
sudo ufw allow 7946/tcp
sudo ufw allow 7946/udp
sudo ufw allow 4789/udp
```


