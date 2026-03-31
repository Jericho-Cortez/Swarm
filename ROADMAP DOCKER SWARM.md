---
title: "ROADMAP Docker Swarm"
tags:
  - docker
  - swarm
  - debian
  - infrastructure
  - devops
  - nfs
  - pca-pra
  - cluster
  - projet-scolaire
created: 2026-03-31
updated: 2026-03-31
status: en-cours
type: roadmap
project: HexaLab
aliases:
  - Docker Swarm Projet
  - Roadmap Swarm
---

> [!abstract] Résumé rapide
> Déploiement d'un cluster **Docker Swarm** sur 4 VM Debian (1 manager, 2 workers, 1 NFS).
> Services ciblés : Registry interne, MariaDB, PHP, Nginx, VSCode Server.
> Rendu : dépôt GitHub + PCA/PRA documentés + présentation orale.

## 🎯 Objectif final

Le projet consiste à déployer un cluster Docker Swarm sur plusieurs VM Debian avec un nœud manager, des workers et une VM NFS pour la persistance, puis à héberger plusieurs services critiques comme un registry interne, MariaDB, PHP, Nginx et VSCode Server.

Le rendu doit inclure un dépôt GitHub, des procédures de test de fonctionnement, des simulations de perte de conteneur, une vérification PCA/PRA, puis une présentation orale avec support.

---

## 🗺️ Roadmap projet

| Phase | But | Livrable attendu | Statut |
| :---: | :--- | :--- | :---: |
| 1 | Cadrage — Définir l'architecture et les ressources | Schéma + plan IP + liste des VM | ⬜ |
| 2 | Préparation — Installer Debian CLI et configurer le réseau | 4 VM prêtes et joignables | ⬜ |
| 3 | Base système — Installer Docker et sécuriser un minimum | Docker opérationnel sur toutes les VM | ⬜ |
| 4 | Cluster — Créer le Swarm et intégrer les nœuds | `docker node ls` fonctionnel | ⬜ |
| 5 | Stockage — Déployer NFS et connecter les volumes | Volumes persistants disponibles | ⬜ |
| 6 | Services — Déployer la stack applicative | Registry, DB, PHP, Nginx, VSCode Server | ⬜ |
| 7 | Résilience — Tester les pannes et la reprise | Preuves PCA/PRA documentées | ⬜ |
| 8 | Documentation — Structurer le dépôt GitHub | README + procédures + captures | ⬜ |
| 9 | Soutenance — Préparer la présentation | Slides + démonstration | ⬜ |

---

## Phase 1 — Cadrage

> [!info] Contexte
> Le PDF impose une architecture avec une VM Debian manager, plusieurs workers et une VM dédiée au stockage NFS, donc il faut partir sur une maquette simple, stable et démontrable.
> La meilleure base pour un projet étudiant solide est : **1 manager, 2 workers et 1 serveur NFS**, tous en Debian CLI, sur le même réseau privé.

### Architecture recommandée

| VM | Rôle | IP |
| :--- | :--- | :--- |
| `swarm-manager` | Orchestration du cluster | `192.168.10.10` |
| `swarm-worker1` | Exécution des services | `192.168.10.11` |
| `swarm-worker2` | Exécution des services | `192.168.10.12` |
| `nfs-server` | Persistance des volumes | `192.168.10.20` |

### Ressources conseillées

| VM | vCPU | RAM | Disque |
| :--- | :---: | :---: | :---: |
| Manager | 2 | 2 Go | 20 Go |
| Worker1 | 2 | 2 Go | 20 Go |
| Worker2 | 2 | 2 Go | 20 Go |
| NFS | 1 | 1-2 Go | 30 Go |

### Ce que tu dois produire à cette phase

- [ ] Un schéma réseau simple
- [ ] Le nom de chaque VM
- [ ] Les IP fixes
- [ ] Le rôle de chaque machine

---

## Phase 2 — Installation Debian

> [!tip] Objectif
> À la fin de cette phase, toutes les machines doivent démarrer proprement, avoir une IP fixe, répondre au ping et être joignables en SSH.

### À faire sur chaque VM

1. Installer Debian minimal (mode CLI, sans interface graphique)
2. Définir le hostname
3. Configurer l'IP statique
4. Installer SSH
5. Ajouter les entrées `/etc/hosts`

### Vérifications

- `ping` entre toutes les VM
- `ssh` depuis le manager vers les workers et le NFS
- `hostnamectl`
- `ip a`

### Captures à prévoir

- [ ] Console d'une VM Debian CLI
- [ ] Fichier réseau configuré
- [ ] Test ping entre les machines
- [ ] Connexion SSH réussie

---

## Phase 3 — Préparation système

> [!warning] Important
> Cette étape est importante car elle évite les erreurs au moment du `join Swarm` ou des montages NFS.

### Paquets utiles

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget vim git sudo ca-certificates gnupg lsb-release nfs-common
```

### Docker à installer sur manager + workers

- Docker Engine
- Docker CLI
- containerd
- plugin compose

### Vérifications

```bash
docker --version
docker info
docker run hello-world
```

### Durcissement conseillé

- [ ] Créer un utilisateur admin non root
- [ ] Ajouter la clé SSH
- [ ] Désactiver le login root SSH
- [ ] Vérifier que Docker démarre au boot (`systemctl enable docker`)

### Captures à prévoir

- [ ] `docker --version`
- [ ] `docker info`
- [ ] `systemctl status docker`

---

## Phase 4 — Création du cluster Swarm

> [!important] Cœur du projet
> Le cluster Swarm permet la **haute disponibilité**, la **scalabilité** et la **reprise rapide** en cas de défaillance.

### Étapes

**1. Initialiser le Swarm sur le manager :**

```bash
docker swarm init --advertise-addr 192.168.10.10
```

**2. Récupérer la commande d'adhésion worker**

**3. Rejoindre le cluster depuis chaque worker**

**4. Vérifier depuis le manager :**

```bash
docker node ls
```

### Ce que tu dois comprendre

| Rôle | Fonction |
| :--- | :--- |
| Manager | Pilote le cluster, orchestre les services |
| Worker | Exécute les tâches conteneurisées |
| Scheduler | Planifie automatiquement selon l'état du cluster |

### Bonus — Étiqueter les nœuds

```bash
docker node update --label-add role=app swarm-worker1
docker node update --label-add role=app swarm-worker2
docker node update --label-add role=db swarm-manager
```

### Captures à prévoir

- [ ] Résultat de `docker swarm init`
- [ ] Résultat de `docker node ls`
- [ ] Vue claire des rôles manager / workers

---

## Phase 5 — Mise en place du NFS

> [!info] Pourquoi NFS ?
> Le sujet demande explicitement une VM dédiée à l'hébergement des volumes Docker pour garantir la conservation des données et leur récupération. Cette phase est **essentielle pour la démo PCA/PRA**.

### Sur le serveur NFS

```bash
sudo apt install -y nfs-kernel-server
sudo mkdir -p /srv/nfs/docker
echo "/srv/nfs/docker 192.168.10.0/24(rw,sync,no_subtree_check,no_root_squash)" | sudo tee -a /etc/exports
sudo exportfs -rav
sudo systemctl restart nfs-kernel-server
```

### Sur manager et workers

- Installer `nfs-common`
- Tester un montage manuel
- Préparer les volumes NFS pour Swarm

### Vérifications

```bash
showmount -e 192.168.10.20
mount -t nfs 192.168.10.20:/srv/nfs/docker /mnt
```

### Captures à prévoir

- [ ] Contenu de `/etc/exports`
- [ ] Résultat de `showmount -e`
- [ ] Montage réussi côté client

---

## Phase 6 — Déploiement des services critiques

> [!note] Stratégie
> Les services sont déployés dans une **stack Swarm** via un fichier YAML pour un rendu propre, reproductible et professionnel.

### Ordre conseillé de déploiement

1. Réseau overlay
2. Registry interne
3. Nginx + PHP
4. MariaDB
5. VSCode Server
6. Tests d'accès

### Pourquoi cet ordre

| Service | Raison |
| :--- | :--- |
| Registry | Peut servir pour des images personnalisées plus tard |
| Nginx/PHP | Permet de montrer une application accessible |
| MariaDB | Ajoute la persistance et la criticité |
| VSCode Server | Service complémentaire concret pour la démo |

### Livrables de cette phase

- [ ] `docker-stack.yml`
- [ ] Fichiers de config Nginx
- [ ] Volume mapping NFS
- [ ] Réseau overlay fonctionnel

### Commandes de vérification

```bash
docker stack deploy -c docker-stack.yml prod
docker stack ls
docker stack services prod
docker service ls
docker ps
```

### Captures à prévoir

- [ ] `docker stack services`
- [ ] Accès web Nginx
- [ ] Accès VSCode Server
- [ ] Conteneurs répartis sur plusieurs nœuds

---

## Phase 7 — Tests de fonctionnement

> [!check] Le PDF demande explicitement d'écrire les procédures de tests de fonctionnement.
> Tu dois prévoir des tests normaux avant de faire les tests de panne.

### Tests à documenter

- [ ] Tous les nœuds sont `Ready`
- [ ] Tous les services sont `Running`
- [ ] L'application Nginx répond
- [ ] MariaDB est accessible
- [ ] VSCode Server est accessible
- [ ] Les volumes NFS sont bien utilisés

### Format conseillé pour chaque test

| Champ | Description |
| :--- | :--- |
| Objectif | Ce que le test vérifie |
| Préconditions | État requis avant le test |
| Commandes | Commandes à exécuter |
| Résultat attendu | Ce qui doit se passer |
| Résultat observé | Ce qui s'est réellement passé |
| Capture | Screenshot à joindre |

### Exemple — Test état nominal du cluster

**Objectif :** Vérifier que tous les nœuds du cluster sont opérationnels.

```bash
docker node ls
```

---

## Phase 8 — Tests PCA / PRA

> [!danger] Phase critique
> Le sujet attend une **simulation de perte de conteneur** et une **vérification PCA/PRA**. C'est probablement la partie qui fera la différence entre un projet "fonctionnel" et un projet "maîtrisé".

### Scénarios à exécuter

- [ ] Suppression d'un conteneur applicatif
- [ ] Arrêt d'un service Swarm
- [ ] Redémarrage forcé d'un service
- [ ] Extinction d'un worker
- [ ] Vérification de la persistance après incident
- [ ] Vérification de l'accessibilité du service après reprise

### Scénario 1 — Perte d'un conteneur

```bash
docker ps
docker rm -f <container_id>
docker service ps prod_nginx
```

> **Attendu :** Swarm recrée automatiquement une tâche pour maintenir le nombre de réplicas.

### Scénario 2 — Panne d'un worker

Sur `worker1` :

```bash
sudo systemctl stop docker
```

Sur le manager :

```bash
docker node ls
docker service ps prod_php
```

> **Attendu :** Les tâches sont replanifiées sur un autre nœud disponible si la redondance le permet.

### Scénario 3 — Persistance NFS

1. Créer une donnée dans MariaDB ou un fichier dans un volume
2. Redémarrer ou recréer le service
3. Vérifier que la donnée est toujours présente

> **Attendu :** Les données persistent grâce au stockage NFS.

### Définitions PCA / PRA

> [!example] Glossaire
> - **PCA** (Plan de Continuité d'Activité) : continuité de service *pendant* l'incident, avec redondance et relance automatique.
> - **PRA** (Plan de Reprise d'Activité) : récupération du service et des données *après* l'incident, grâce à la réorchestration et au stockage persistant.

---

## Phase 9 — Documentation GitHub

> [!info] Objectif
> Ton dépôt doit être clair, propre et exploitable par quelqu'un qui n'a jamais vu ton cluster.

### Structure recommandée

```text
swarm/
├── README.md
├── docker-stack.yml
├── docs/
│   ├── 01-architecture.md
│   ├── 02-installation-vm.md
│   ├── 03-installation-docker.md
│   ├── 04-configuration-swarm.md
│   ├── 05-configuration-nfs.md
│   ├── 06-deploiement-services.md
│   ├── 07-tests-fonctionnement.md
│   ├── 08-tests-pca-pra.md
│   └── images/
└── scripts/
    ├── install-docker.sh
    ├── init-swarm.sh
    └── join-worker.sh
```

### README minimum

- [ ] Présentation du projet
- [ ] Objectifs
- [ ] Architecture
- [ ] Plan IP
- [ ] Technologies utilisées
- [ ] Procédure de déploiement
- [ ] Procédure de test
- [ ] Résultats
- [ ] Difficultés et solutions

---

## Phase 10 — Présentation orale

> [!tip] Conseil clé
> Évite les slides trop théoriques : **montre surtout ce que tu as conçu, déployé, testé et validé.**

### Plan de soutenance conseillé

1. Contexte du projet
2. Objectifs techniques
3. Architecture du cluster
4. Rôle du manager, workers et NFS
5. Déploiement des services critiques
6. Fonctionnement normal
7. Simulation de panne
8. Vérification PCA/PRA
9. Difficultés rencontrées
10. Bilan et compétences acquises

### Démonstrations idéales

- [ ] `docker node ls`
- [ ] `docker service ls`
- [ ] Accès à Nginx
- [ ] Suppression d'un conteneur → recréation automatique
- [ ] Arrêt d'un worker → redistribution des services
- [ ] Persistance des données après incident

---

## 📅 Organisation du travail

### Séquences de travail

| Séquence | Contenu |
| :---: | :--- |
| Séquence 1 | VM + réseau + Docker |
| Séquence 2 | Swarm + NFS |
| Séquence 3 | Stack applicative |
| Séquence 4 | Tests + doc + présentation |

### Planning conseillé

| Jour | Tâches |
| :---: | :--- |
| Jour 1 | Architecture + création VM |
| Jour 2 | Docker + Swarm |
| Jour 3 | NFS + volumes |
| Jour 4 | Déploiement des services |
| Jour 5 | Tests PCA/PRA |
| Jour 6 | Documentation GitHub |
| Jour 7 | Soutenance et répétition |

---

## ✅ Check-list finale

> [!check] Avant de considérer le projet terminé, vérifie :

- [ ] Les 4 VM Debian sont prêtes et joignables
- [ ] Le cluster Swarm est opérationnel (`docker node ls`)
- [ ] Le NFS fonctionne et stocke les données
- [ ] Les services critiques sont déployés et accessibles
- [ ] Les tests de fonctionnement sont rédigés
- [ ] Les scénarios PCA/PRA sont démontrés et documentés
- [ ] Le dépôt GitHub est structuré avec toutes les docs
- [ ] Les captures d'écran sont prêtes et légendées
- [ ] La présentation est prête et répétée

---

## 🔗 Liens internes utiles

- [[Docker Swarm - Commandes essentielles]]
- [[NFS - Configuration et dépannage]]
- [[PCA PRA - Concepts et définitions]]
- [[HexaLab - Architecture générale]]
- [[GitHub - Bonnes pratiques de documentation]]

---

```dataview
TABLE status, tags, updated
FROM #docker OR #swarm
SORT updated DESC
```
