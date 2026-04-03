## Phase 1 — Cadrage

### Architecture validée

Basé sur le sujet, voici l'architecture officielle de ton projet :[^1]


| VM | Hostname | IP fixe | Rôle | CPU | RAM | Disque |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| VM 1 | `swarm-manager` | `192.168.10.10` | Orchestration Swarm | 2 vCPU | 2 Go | 20 Go |
| VM 2 | `swarm-worker1` | `192.168.10.11` | Exécution conteneurs | 2 vCPU | 2 Go | 20 Go |
| VM 3 | `swarm-worker2` | `192.168.10.12` | Exécution conteneurs | 2 vCPU | 2 Go | 20 Go |
| VM 4 | `nfs-server` | `192.168.10.20` | Stockage persistant NFS | 1 vCPU | 1 Go | 30 Go |

### Services à déployer

Conformément au sujet, la stack contiendra :[^1]


| Service | Image | Port exposé | Rôle |
| :-- | :-- | :-- | :-- |
| Registry | `registry:2` | `5000` | Dépôt local d'images Docker |
| MariaDB | `mariadb:11` | interne | Base de données critique |
| PHP | `php:8.2-fpm` | interne | Serveur applicatif |
| Nginx | `nginx:alpine` | `80` | Proxy + serveur web |
| VSCode Server | `linuxserver/code-server` | `8443` | IDE collaboratif |

### Schéma réseau

```
HOST (hyperviseur)
│
├── réseau bridge : 192.168.10.0/24
│
├── 192.168.10.10  swarm-manager  ←── Docker Swarm Manager (Leader)
│        │
│        ├── 192.168.10.11  swarm-worker1  ←── Worker (conteneurs)
│        ├── 192.168.10.12  swarm-worker2  ←── Worker (conteneurs)
│        │
│        └── réseau overlay interne Swarm ────────────────────────┐
│                                                                  │
└── 192.168.10.20  nfs-server  ←── Volumes persistants (NFS) ◄───┘
```


### Structure GitHub à créer maintenant

Crée le dépôt `swarm` sur ton GitHub avec cette structure :[^1]

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


### Checklist Phase 1

Coche chaque point avant de passer à la Phase 2 :

- [ ] Hyperviseur choisi (VMware / VirtualBox / Proxmox)
- [ ] 4 VM planifiées avec les bons noms, IP et ressources
- [ ] Dépôt GitHub `swarm` créé
- [ ] Structure des dossiers initialisée
- [ ] `README.md` créé avec l'architecture et les IP
- [ ] Schéma réseau noté quelque part (pour la soutenance)


### Ce que tu dois faire maintenant

1. **Crée ton dépôt GitHub** → `github.com/prenom-nom/swarm`[^1]
2. **Note les IP quelque part** (fichier Obsidian, README…) pour ne jamais te tromper ensuite.
3. **Lance la création des VM** dans ton hyperviseur avec les ressources du tableau.


