# 🦞 Stack-Stream — BigData stream stack (docker)

<p align="center">
    <picture>
        <source media="(prefers-color-scheme: light dark)" srcset="images/archi_globale.drawio.png">
        <img src="images/archi_globale.drawio.png" alt="BigData stream stack (docker)" width="500">
    </picture>
</p>

<p align="center">
  <strong>EXPLOIT DATA FROM POSTGRES DATABASE</strong>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge" alt="MIT License"></a>
</p>

**Stack-Stream** is a simple *BigData stream stack* running over *Docker*.
It will help you to deploy and test a simple **streaming** pipeline using **Docker**.

**Components:**
- **[Docker](https://www.docker.com/)** (safer container ecosystem)
- **[Postgres DB](https://www.postgresql.org/)** (Advanced Open Source Relational Database)
- **[Kafka](https://kafka.apache.org/)** (open-source distributed event streaming platform)
- **[Confluent Connectors](https://www.confluent.io/product/connectors/)** (open-source connectors & sinks for streaming platform)
- **[Elasticsearch](https://www.elastic.co/elasticsearch/)** (open source, distributed search and analytics engine)
- **[Kibana](https://www.elastic.co/kibana/)** (open source interface to query, analyze, visualize, and manage your data stored in Elasticsearch)
- **[PgAdmin](https://www.pgadmin.org/)** (PostgreSQL Admin Tool)
- **[Portainer](https://www.portainer.io/)** (Kubernetes, Docker & Podman management.)

## Versions

* SE: **Ubuntu 22.04.5 LTS**
* Runtime enviromment for test: **Docker version 29.1.5**.
* Browser: **Opera 126.x**
* Others versions: in **compose.yml**

## Install

### Docker (if not already installed)

- **[Install](https://docs.docker.com/engine/install/)**



```bash
# prerequises
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release -y

# Add GPG keys
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

# Update repository & install docker
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

- **<u>Docker service</u>** 
```bash
# start & stop & status
sudo systemctl status docker
sudo systemctl start docker
sudo systemctl stop docker
```

- **<u>User access Config</u>**
```bash
# configure user
sudo groupadd docker # add group if not exist
sudo usermod -aG docker $USER
newgrp docker # Or restart session
```
---

### Portainer
```bash
# Create Docker Volume 'portainer_data'
sudo docker volume create --driver local --opt type=none --opt device={path_to_your_local_folder} --opt o=bind portainer_data

# Inspect the volume (check)
docker volume inspect portainer_data

# Start Portainer
sudo docker run -d -p 7003:8000 -p 7443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /run/docker.sock:/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

#### Access from browser **`https://localhost:7443`**

<p align="center">
    <picture>
        <source media="(prefers-color-scheme: light dark)" srcset="images/Screenshot_portainer_1.png">
        <img src="images/Screenshot_portainer_1.png" alt="Portainer" width="700">
    </picture>
</p>

---

### Clone the project

- **<u>GIT (SSH)</u>** 
```bash
# clone
git clone git@github.com:ngoupatrick/stack-stream.git
```
---

### Run project & some cleaning ops

```bash
# Be sure to be in the folder with compose.yml file
# start all
docker compose up -d

# stop all and clean some volume
docker compose down -v --remove-orphans

# logs (you can also have access to logs on portainer)
docker logs -f {container}

# Bash access of specific container
docker exec -it {container} bash

# remove unused images
docker image prune -a

# remove all container not used for the last 24h
docker container prune --filter "until=24h"

# remove unused volume
docker volume prune
```

#### **result** (You can have access to **logs, console, inspect and stack** of each container throw **Portainer**)
<p align="center">
    <picture>
        <source media="(prefers-color-scheme: light dark)" srcset="images/Screenshot_portainer_2.png">
        <img src="images/Screenshot_portainer_2.png" alt="Portainer2" width="700">
    </picture>
</p>

---
## Source

### Postgres
```bash
# you must first acces to postgres container
docker exec -it postgres bash

# some postgres commands
# 1- connect to a database: psql -h [hôte] -p [port] -U [utilisateur] -d [nom_base] -W
psql -h postgres -p 5432 -U user -d db_tickets -W # password in compose.yml
```

| **<span style="color:green">Commands</span>** | **<span style="color:green">Description</span>** |
| :--- | :--- | 
| **\l** | `Lister toutes les bases de données.` |
| **\dn** | `Lister les schemas` |
| **\c [nom_base]** | `Changer de base de données (se reconnecter).` |
| **\dt** | `Lister les tables de la base actuelle.` |
| **\d [table]** | `Décrire la structure d'une table spécifique.` |
| **\conninfo** | `Afficher les détails de la connexion actuelle.` |
| **\q** | `Quitter psql.` |
`

---

### PgAdmin

```sql
-- create schema 'test'
CREATE SCHEMA IF NOT EXISTS test;

-- create table 'test.utilisateurs'
CREATE TABLE IF NOT EXISTS test.utilisateurs (
    id SERIAL PRIMARY KEY,              -- Identifiant auto-incrémenté et clé primaire
    nom VARCHAR(100) NOT NULL,          -- Texte limité à 100 caractères, obligatoire
    email VARCHAR(255),
    date_inscription DATE DEFAULT CURRENT_DATE, -- Valeur par défaut (date du jour)
    genre VARCHAR(10) CHECK (genre IN ('M', 'F')),
    val1 INTEGER CHECK (val1 >= 0) NOT NULL,
    val2 REAL
);

-- Insert some data
INSERT INTO test.utilisateurs (nom, email, genre, val1, val2) VALUES 
('Jean Dupont', 'jean@email.com', 'M', 17, 15.4),
('Lucie Terre', 'lucie@email.com', 'F', 14, 17.4),
('Marc Fox', 'marc@email.com', 'M', 18, 6.9),
('Paul Dupont', 'paul@email.com', 'M', 13, 13.1),
('Cathy Terre', 'cathy@email.com', 'F', 16, 11.7),
('Marie Fox', 'marie@email.com', 'F', 19, 16.9);

-- Select
SELECT * FROM test.utilisateurs;

-- DELETE
DELETE FROM test.utilisateurs WHERE id > 6;

-- average 'val1' and 'val2' group by gender
select genre, round(avg(val1)::numeric,2) as m_v1, round(avg(val2)::numeric, 2) as m_v2 from test.utilisateurs group by genre;
```
RESULT
<p align="center">
    <picture>
        <source media="(prefers-color-scheme: light dark)" srcset="images/Screenshot_pgadmin.png">
        <img src="images/Screenshot_pgadmin.png" alt="pgadmin2" width="700">
    </picture>
</p>

---

## Streaming ingestion

### kafka

---

### Connect

---
