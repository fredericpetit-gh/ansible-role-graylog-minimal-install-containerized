# Rôle Ansible "graylog-commpleted"

Rôle Ansible pour déployer Graylog avec MongoDB et OpenSearch en conteneurs.

## Description

Ce rôle lance trois conteneurs :
- MongoDB 6-jammy
- OpenSearch 1
- Graylog 6 (exposé sur le port 9009)

## Variables par défaut

| Variable                    | Description                         | Valeur par défaut                |
|-----------------------------|-------------------------------------|----------------------------------|
| `mongo_image`               | Image Docker MongoDB                | `mongo:6-jammy`                  |
| `opensearch_image`          | Image Docker OpenSearch             | `opensearchproject/opensearch:1` |
| `graylog_image`             | Image Docker Graylog                | `graylog/graylog:6`              |
| `graylog_admin_password`    | Mot de passe administrateur Graylog | `root`                           |
| `graylog_password_secret`   | Clé secrète pour Graylog            | `root`                           |
| `graylog_http_external_uri` | URL externe Graylog                 | `http://localhost:9009/`         |

## Prérequis

Ce rôle nécessite :
- Docker ou Podman installé sur l'hôte cible.
- La collection Ansible `community.docker` installée localement (`ansible-galaxy collection install community.docker`).

## Installation

À ajouter dans le fichier requirements.yaml :

```yaml
- src: git+https://gitlab.com/fredericpetit/ansible-role-graylog-completed.git
  name: graylog-completed
  version: main
```

Puis faire :

`ansible-galaxy role install -r requirements.yaml`

## Utilisation

Ajouter le rôle dans un playbook :

```yaml
- hosts: serveur
  become: yes
  roles:
    - graylog-commpleted
```