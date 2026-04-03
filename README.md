# Docker Swarm - Résilience & Continuité d'Activité

## Architecture

```mermaid
flowchart LR
	U[Utilisateur] --> N[Nginx / PHP :80]
	N --> M[(MariaDB sur NFS)]
	N --> R[Registry :5000]
	U --> C[code-server :8443]

	subgraph Swarm
		MAN[Manager 192.168.10.10]
		W1[Worker1 192.168.10.11]
		W2[Worker2 192.168.10.12]
	end

	subgraph Stockage
		NFS[NFS 192.168.10.20]
	end

	MAN --- W1
	MAN --- W2
	MAN --- NFS
	W1 --- NFS
	W2 --- NFS
```

- Manager : 192.168.10.10
- Worker1 : 192.168.10.11
- Worker2 : 192.168.10.12
- NFS : 192.168.10.20

## Services déployés

- Registry : 5000
- Nginx/PHP : 80
- MariaDB sur NFS
- code-server : 8443

## Déploiement

```bash
git clone https://github.com/prenom-nom/swarm
cd swarm
docker stack deploy -c docker-stack.yml prod
```

## Tests PCA/PRA

Voir la documentation dédiée : [docs/08-pca-pra.md](docs/08-pca-pra.md)

## Objectif du projet

Ce projet met en place un cluster Docker Swarm sur 4 VM Debian pour démontrer la résilience, la reprise de service et la persistance des données via NFS.

## Références

- Roadmap complète : [docs/ROADMAP DOCKER SWARM.md](docs/ROADMAP%20DOCKER%20SWARM.md)
- Phases du projet : [docs/Phase 1 Docker Swarm.md](01-architecture.md)
