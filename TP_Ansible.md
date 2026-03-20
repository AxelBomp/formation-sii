# TP Ansible

## I. TP1 - Installation + premier inventaire

### 1. Installation d'Ansible, et création du ansible.cfg

```bash
apt update
apt install ansible

sudo mkdir -p /etc/ansible/inventory
sudo chown -R docker:docker /etc/ansible

vi /etc/ansible/ansible.cfg
```

```ini
[defaults]
remote_user = docker
```

### 2. Configuration SSH

```bash
ssh-keygen -b 2048
ssh-copy-id docker@<ip voisin>
```

### 3. Modification du sudoers

```bash
sudo vi /etc/sudoers.d/docker
```

```
docker ALL=(ALL) NOPASSWD: ALL
```

### 4. Création du premier inventaire

```bash
cd /etc/ansible/inventory
vi inventory.ini
```

```ini
[web]
webServer ansible_host=xxxxx

[db]
dbServer ansible_host=xxxx
```

### 5. Vérifications de l'inventaire

```bash
ansible -m ping all -i inventory.ini
ansible -m ping web -i inventory.ini
ansible -m ping db -i inventory.ini
```

### 6. Création de l'inventaire en YAML

```bash
vi inventory.yml
```

```yaml
all:
  hosts:
    webServer:
      ansible_host: 192.168.0.204
    dbServer:
      ansible_host: 192.168.0.204

  children:
    web:
      hosts:
        webServer:
    db:
      hosts:
        dbServer:
```

### 7. Refaire les mêmes commandes en utilisant l'inventaire yml

### 8. Modification de l'ansible.cfg pour automatiser l'utilisation du fichier d'inventaire

---

## II. TP2 - Jouons avec les variables

### 1. Lancer la commande `ansible -m debug -a "var=ansible_user"`

### 2. Modifier le fichier ansible.cfg pour configurer la variable remote_user

### 3. Relancer la commande de debug

### 4. Modifier l'inventaire pour y ajouter des users "web" et "db" et relancer la commande de debug

### 5. Ajouter maintenant une variable "port_apache" uniquement pour le serveur web et relancer la commande debug

### 6. Tester la commande

```bash
ansible -m debug -a "var=hostvars['webServer'].port_apache"
```

### 7. Premier playbook de debug

```bash
mkdir /etc/ansible/playbooks
```

```yaml
---
- name: premier playbook debug
  hosts: web
  vars:
    port_apache: 8080
  tasks:
    - name: debug
      debug:
        var: port_apache
```

### 8. Lancer la commande `ansible-playbook debug.yml`. Corriger l'erreur de user dans l'inventaire

### 9. Expliquer les différentes étapes d'exécution du playbook (gathering facts)

### 10. Lancer les commandes

```bash
ansible -m setup web
ansible -m setup web -a "filter=ansible_distribution"
```

### 11. Modifier le playbook pour afficher tous les ansible facts, puis seulement la distribution

---

## III. TP3 - host_vars et group_vars

### 1. Créer deux répertoires dans /etc/ansible/hosts nommés host_vars et group_vars

### 2. Dans group_vars, faire un fichier `web.yml` avec une variable `dns_server: 1.1.1.1`, puis faire un fichier `db.yml` avec `2.2.2.2`

### 3. Modifier le playbook `debug.yml` pour afficher cette variable pour toutes les machines (all)

### 4. Dans host_vars, faire un fichier `dbServer.yml` avec `dns_server: 1.2.3.4` et relancer le playbook

---

## IV. TP4 - Déploiement d'un serveur web et d'un serveur mariadb

### 1. Créer un playbook ciblant le serveur web pour y installer le paquet apache2, puis activer le service apache au démarrage. Corriger l'erreur.

### 2. Modifier le playbook pour ajouter un `index.html` dans le dossier `/var/www/html/` qui contient le texte "HELLO WORLD!"

### 3. Tester le résultat avec un curl

### 4. Créer un 2e playbook ciblant le serveur db pour y installer mariadb (paquets `mariadb-server` et `python3-mysqldb`). Activer le service au démarrage.

---

## V. TP5 - Register, when, loop

### 1. Dans le playbook `debug.yml`, utiliser un module qui va lire le fichier `/etc/resolv.conf`, l'enregistrer dans une variable puis afficher cette variable.

### 2. Modifier le playbook pour n'afficher que le contenu du fichier

### 3. Imaginons que le fichier `/etc/resolv.conf` n'existe pas, il faut tester sa presence et n'executer la tache de lecture que s'il existe. Modifier le playbook en consequence.

### 4. Tester un la présence d'un fichier qui n'existe pas, et relancer le playbook pour observer ce qu'il se passe.

```yaml
---
- name: Playbook de debug
  hosts: web

  vars:
    mes_fichiers:
      - /etc/resolv.conf
      - /etc/crontab

  tasks:

    - name: Tester la presence du fichier resolv.conf
      stat:
        path: "{{ item }}"
      loop: "{{ mes_fichiers }}"
      register: stats_fichiers

    - name: Lire les fichiers
      command: cat {{ item.item }}
      register: cmd_return
      loop: "{{ stats_fichiers.results }}"

    - name: afficher la variable
      debug:
        var: item.stdout_lines
      loop: "{{ cmd_return.results }}"
```

### 5. Désinstaller mariadb du serveur cible. Modifier ensuite le playbook d'installation mariadb pour variabiliser les paquets à installer, puis utiliser une loop pour tous les installer dans une même task

### 6. Modifier le playbook d'installation Apache pour changer le contenu de la page `index.html`. Enregistrer le resultat du module dans une variable. Ajouter une task de restart du service apache s'il y a eu une modif dans le fichier `index.html`.

---

## VI. TP6 - Handlers et notify

### 1. Supprimer la condition `when` du playbook d'installation apache pour y ajouter un handler de redemarrage du service s'il y a un change sur la tache de copy du fichier `index.html`

### 2. Modifier le playbook install_mariadb pour y ajouter les tasks suivantes, ajouter un handler de redémarrage du service mariadb s'il y a un change quelque part.

```yaml
    - name: config mariadb
      lineinfile:
        path: /etc/mysql/mariadb.conf.d/50-server.cnf
        regexp: '^bind-address'
        line: 'bind-address = 0.0.0.0'
        state: present

    - name: config root
      mysql_user:
        name: root
        host: localhost
        password: toto
        login_user: root
        login_password: toto
        priv: '*.*:ALL,GRANT'
        state: present
        login_unix_socket: /var/run/mysqld/mysqld.sock

    - name: config database
      mysql_db:
        name: ma_base
        state: present
        login_user: root
        login_password: toto

    - name: create table users
      mysql_query:
        login_db: ma_base
        login_user: root
        login_password: toto
        query: |
          CREATE TABLE IF NOT EXISTS mes_users (
            id INT AUTO_INCREMENT PRIMARY KEY,
            name VARCHAR(255) NOT NULL
          );

    - name: add users
      mysql_query:
        login_db: ma_base
        login_user: root
        login_password: toto
        query: |
          INSERT INTO mes_users (name) VALUES
          ('axel'),('jean'),('alfred'),('gaston');
```

### 3. Variabiliser tous les logins et les mots de passe dans les bons fichiers host_vars ou group_vars, ainsi que la liste des users à créer dans la base.

---

## VII. TP7 - Vaultisation

### 1. Creer un fichier `pass.txt` et y inscrire une passe phrase de votre choix

### 2. Vaultiser le mot de passe root avec cette passe phrase et l'inclure dans votre inventaire. Modifier le playbook d'installation en conséquence et le lancer

---

## VIII. TP8 - Rôles et collections

### 1. Créer un répertoire roles dans /etc/ansible/playbooks

### 2. Se placer dans ce répertoire et initialiser un role apache. Examiner l'arborescence créée.

### 3. Déplacer les tâches du playbook d'install_apache dans ce rôle, puis modifier le playbook pour qu'il fasse appel au rôle.

### 4. Faire de même pour mariadb

---

## IX. TP9 - Templates

### 1. Creer un fichier de Template `index.php.j2` au bon endroit avec le contenu suivant (a vous de remplacer les variables)

```php
<?php

$servername = ????
$username = ???
$password = ???
$dbname = ???

$conn = new mysqli("localhost", $username, $password, $dbname);

if ($conn->connect_error) {
    die("Connection failed:" . $conn->connect_error);
}

$sql = "SELECT name FROM mes_users";
$result = $conn->query($sql);

if ($result->num_rows > 0) {
    while($row = $result->fetch_assoc()) {
        echo "Hello " . $row["name"]. "<br>";
    }
} else {
    echo "0 result";
}

$conn->close();
?>
```

### 2. Testez le résultat !

### 3. Modifier votre index.php pour faire une vraie page HTML

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title> A VARIABILISER </title>
    <link rel="stylesheet" href="style.css">
</head>
<body>

    <h1>Bienvenue</h1>
    <p>Ceci est un exemple de page HTML avec un fichier CSS externe.</p>

</body>
</html>
```

### 4. Templatiser le fichier `style.css` pour modifier les parametres suivants (tout doit etre variabilise)

```css
body {
    background-color: ????
    font-family: Arial, sans-serif;
    font-size: 14px;
}
```

### 5. Testez à nouveau !
