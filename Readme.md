# ISS_TP3_Serge_Noah_2582510

# Exercice 19 – Ansible - Playbook

<br></br>

## But du travail

Dans ce laboratoire, nous démontrons l'utilisation d'**Ansible** en mode **Playbook** afin d'automatiser le déploiement d'un conteneur d'infrastructure (**Apache Web Server via Docker**) sur un nœud distant, tout en sécurisant nos secrets d'architecture à l'aide d'**Ansible Vault**.

Principalement, nous avons réalisé :

- L'initialisation d'un espace de projet (`webapp`) sous l'utilisateur de déploiement;
- La transition d'un inventaire classique vers le format structuré **YAML**;
- La rédaction d'un **Playbook Ansible** complet gérant l'orchestration des couches (dépendances, clés GPG, installation du moteur Docker et cycle de vie du conteneur HTTPD);
- La mise en œuvre d'une isolation stricte des identifiants et privilèges d'exécution (`become`) chiffrés via **AES-256** avec **Ansible Vault**.

<br></br>

---

# Table des matières

1. [But du travail](#but-du-travail)
2. [Section 1 : Historique des configurations et commandes](#section-1--historique-des-configurations-et-commandes)
    - 1.1 Initialisation de l'environnement de projet
    - 1.2 Configuration de l'inventaire YAML
    - 1.3 Sécurisation avec Ansible Vault
    - 1.4 Création du Playbook
    - 1.5 Exécution du Playbook
3. [Section 2 : Validation sur le serveur client](#section-2--validation-sur-le-serveur-client)
4. [Conclusion](#conclusion)
5. [Auteurs](#auteurs)

---

# Section 1 : Historique des configurations et commandes

## 1. Initialisation de l'environnement de projet

Nous avons débuté par basculer sur l'utilisateur dédié **deploy** et structuré notre espace de travail en y copiant la configuration de base d'Ansible.

```bash
su -l deploy
mkdir webapp
cp ansible.cfg webapp/
cd webapp/
```

---

## 2. Configuration de l'inventaire YAML

Nous avons créé un inventaire moderne au format **YAML** (`hosts.yaml`) regroupant les différents hôtes administrés ainsi que les paramètres de connexion SSH.

Création du fichier :

```bash
vim hosts.yaml
```

Contenu du fichier :

```yaml
---
all:
  vars:
    ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
  hosts:
    control:
      ansible_connection: local

prod:
  hosts:
    srv-apache-1:
      ansible_host: 10.100.2.33
```

Modification du fichier `ansible.cfg` :

```ini
[defaults]
inventory = hosts.yaml
```

Validation de la connectivité :

```bash
ansible -m ping all
```

Les deux hôtes (`control` et `srv-apache-1`) ont répondu avec l'état **SUCCESS**, confirmant que l'inventaire et la connexion SSH étaient correctement configurés.

---

## 3. Sécurisation avec Ansible Vault

Afin de protéger les informations sensibles, nous avons utilisé **Ansible Vault** pour chiffrer les variables contenant les privilèges d'administration.

Création du répertoire sécurisé :

```bash
mkdir -p vars
ansible-vault create vars/secret-variables.yaml
```

Contenu du coffre :

```yaml
---
ansible_sudo_pass: S0l&il01
```

---

## 4. Création du Playbook

Le fichier `deploy.yaml` contient toutes les tâches nécessaires à l'installation de Docker ainsi qu'au déploiement automatique du serveur Apache.

Création du Playbook :

```bash
vim deploy.yaml
```

Le Playbook réalise notamment :

- l'installation des dépendances;
- l'ajout de la clé GPG de Docker;
- l'ajout du dépôt officiel Docker;
- l'installation du moteur Docker;
- le démarrage du service Docker;
- le déploiement automatique d'un conteneur **Apache HTTPD**.

---

## 5. Exécution du Playbook

Exécution :

```bash
ansible-playbook deploy.yaml --ask-vault-pass
```

### Analyse du résultat

L'exécution s'est terminée sans erreur (`failed=0`).

Le rapport affichait :

- **ok** : tâches déjà conformes;
- **changed=0** : aucune modification supplémentaire n'était nécessaire.

Cela démontre l'**idempotence** d'Ansible : plusieurs exécutions du même Playbook produisent le même état sans réinstaller inutilement les composants déjà présents.

---

# Section 2 : Validation sur le serveur client

Afin de vérifier le résultat du déploiement, nous nous sommes connectés directement au serveur distant.

Connexion :

```bash
ssh deploy@10.100.2.33
```

Vérification du conteneur Docker :

```bash
sudo docker ps
```

La commande confirme que le conteneur **Apache HTTPD** est bien démarré et fonctionne correctement.

Enfin, l'accès au serveur Web a été validé à partir d'un navigateur en utilisant l'adresse IP du serveur, ce qui permet d'afficher la page d'accueil par défaut d'Apache.

---

# Conclusion

Ce laboratoire nous a permis de mettre en pratique les principaux concepts d'**Ansible Playbook** dans un contexte réel d'automatisation d'infrastructure.

Les objectifs suivants ont été atteints :

- création d'un inventaire YAML;
- utilisation d'un Playbook pour automatiser le déploiement;
- sécurisation des informations sensibles avec **Ansible Vault**;
- installation automatique de Docker;
- déploiement d'un conteneur Apache;
- validation complète du fonctionnement du service Web.

Cette approche illustre les avantages de l'automatisation des infrastructures, notamment la reproductibilité, l'idempotence, la sécurité des secrets et la simplification des opérations d'administration système.

