**Documentation complète** de ce qu'on vient de faire, prête pour `docs/06-deploiement-services.md`.[^1]

## Objectif validé

Déployer les services critiques demandés par le sujet :[^1]

- **Registry interne** (dépôt local)
- **MariaDB** (base critique avec persistance NFS)
- **PHP-FPM** (serveur applicatif)
- **Nginx** (frontal web accessible)
- **code-server** (IDE web collaboratif)

**Architecture déployée** :

```
swarm-manager (Leader)
├── Registry (port 5000)
├── MariaDB (persistance NFS)
└── code-server (port 8443)

swarm-worker1 + swarm-worker2
├── PHP-FPM (2 réplicas)
└── Nginx (2 réplicas)
```


## Arborescence déployée

```
~/swarm-app/
├── docker-stack.yml
├── app/
│   └── index.php
└── nginx/
    └── default.conf
```


## Fichier docker-stack.yml final

```yaml
version: "3.8"

services:
  registry:
    image: registry:2
    ports:
      - "5000:5000"
    volumes:
      - registry_data:/var/lib/registry
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.hostname == swarm-manager
      restart_policy:
        condition: on-failure
    networks:
      - backend

  mariadb:
    image: mariadb:11
    environment:
      MARIADB_ROOT_PASSWORD: rootpass
      MARIADB_DATABASE: appdb
      MARIADB_USER: appuser
      MARIADB_PASSWORD: apppass
    volumes:
      - mariadb_data:/var/lib/mysql
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.hostname == swarm-manager
      restart_policy:
        condition: on-failure
    networks:
      - backend

  php:
    image: php:8.2-fpm
    volumes:
      - ./app:/var/www/html
    deploy:
      replicas: 2
      restart_policy:
        condition: on-failure
    networks:
      - backend

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./app:/var/www/html:ro
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
    deploy:
      replicas: 2
      restart_policy:
        condition: on-failure
    networks:
      - backend

  code-server:
    image: lscr.io/linuxserver/code-server:latest
    environment:
      - PASSWORD=Admin123!
      - TZ=Europe/Paris
    ports:
      - "8443:8443"
    volumes:
      - code_data:/config
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.hostname == swarm-manager
      restart_policy:
        condition: on-failure
    networks:
      - backend

networks:
  backend:
    driver: overlay

volumes:
  registry_data:
    driver: local
    driver_opts:
      type: nfs
      o: addr=192.168.10.20,nolock,soft,rw
      device: ":/srv/nfs/registry"

  mariadb_data:
    driver: local
    driver_opts:
      type: nfs
      o: addr=192.168.10.20,nolock,soft,rw
      device: ":/srv/nfs/mariadb"

  code_data:
    driver: local
    driver_opts:
      type: nfs
      o: addr=192.168.10.20,nolock,soft,rw
      device: ":/srv/nfs/code"
```


## Fichiers déployés

### `app/index.php`

```php
<?php
echo "<h1>Docker Swarm OK</h1>";
echo "<p>Hostname: " . gethostname() . "</p>";
echo "<p>Date: " . date('Y-m-d H:i:s') . "</p>";
phpinfo();
?>
```


### `nginx/default.conf`

```nginx
server {
    listen 80;
    server_name _;
    root /var/www/html;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME /var/www/html$fastcgi_script_name;
    }
}
```


## Commandes de déploiement

```
cd ~/swarm-app
docker stack deploy -c docker-stack.yml prod
docker stack services prod
```

![[Pasted image 20260402162025.png]]

## Statut services (après correction bind mounts)

```
prod_nginx         2/2   nginx:alpine      *:80->80/tcp
prod_php           2/2   php:8.2-fpm       
prod_mariadb       1/1   mariadb:11        
prod_registry      1/1   registry:2        *:5000->5000/tcp
prod_code-server   1/1   code-server       *:8443->8443/tcp
```


## Tests validés

### 1. Nginx + PHP

```
curl http://192.168.10.10
```

```
<h1>Docker Swarm OK</h1>
Hostname: swarm-worker1
Date: 2026-04-02 16:xx:xx
```
![[Pasted image 20260402160955.png]]

### 2. Registry

```
curl http://192.168.10.10:5000/v2/
```

```
{}
```
![[Pasted image 20260402161134.png]]

### 3. Code-server

```
http://192.168.10.10:8443
```

Login : `Admin123!`
![[Pasted image 20260402161326.png]]
## Volumes NFS validés

```
docker volume ls
prod_code_data
prod_mariadb_data  
prod_registry_data

docker volume inspect prod_mariadb_data
```

```
"DriverOpts": {
    "device": ":/srv/nfs/mariadb",
    "o": "addr=192.168.10.20,nolock,soft,rw",
    "type": "nfs"
}
```


## Captures prises

- `docker stack services prod`
- Page Nginx (`http://192.168.10.10`)
- Registry (`curl :5000/v2/`)
- Code-server login
- `docker volume ls`
- ![[Pasted image 20260402162412.png]]
- `docker volume inspect prod_mariadb_data`
- ![[Pasted image 20260402162451.png]]


## Point technique important

**Bind mounts locaux** (`./app`, `./nginx`) :

- Doivent exister sur **tous les workers**
- Copiés via `scp` depuis manager
- Fonctionnent pour dev/test
- En prod → utiliser **configs** Docker Swarm


## Checklist Phase 6 ✓

- [x] Stack déployée 5/5 services
- [x] Nginx + PHP accessibles
- [x] MariaDB persistante NFS
- [x] Registry fonctionnel
- [x] code-server accessible
- [x] Volumes NFS montés
- [x] Documentation + captures


