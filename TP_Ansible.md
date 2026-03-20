\# TP Ansible



\## I. TP1 - Installation + premier inventaire



\### 1. Installation d'Ansible, et création du ansible.cfg



```bash

apt update

apt install ansible



sudo mkdir -p /etc/ansible/inventory

sudo chown -R docker:docker /etc/ansible



vi /etc/ansible/ansible.cfg

```



```ini

\[defaults]

remote\_user = docker

```



\### 2. Configuration SSH



```bash

ssh-keygen -b 2048

ssh-copy-id docker@<ip voisin>

```



\### 3. Modification du sudoers



```bash

sudo vi /etc/sudoers.d/docker

```



```

docker ALL=(ALL) NOPASSWD: ALL

```



\### 4. Création du premier inventaire



```bash

cd /etc/ansible/inventory

vi inventory.ini

```



```ini

\[web]

webServer ansible\_host=xxxxx



\[db]

dbServer ansible\_host=xxxx

```



\### 5. Vérifications de l'inventaire



```bash

ansible -m ping all -i inventory.ini

ansible -m ping web -i inventory.ini

ansible -m ping db -i inventory.ini

```



\### 6. Création de l'inventaire en YAML



```bash

vi inventory.yml

```



```yaml

all:

&#x20; hosts:

&#x20;   webServer:

&#x20;     ansible\_host: 192.168.0.204

&#x20;   dbServer:

&#x20;     ansible\_host: 192.168.0.204



&#x20; children:

&#x20;   web:

&#x20;     hosts:

&#x20;       webServer:

&#x20;   db:

&#x20;     hosts:

&#x20;       dbServer:

```



\### 7. Refaire les mêmes commandes en utilisant l'inventaire yml



\### 8. Modification de l'ansible.cfg pour automatiser l'utilisation du fichier d'inventaire



\---



\## II. TP2 - Jouons avec les variables



\### 1. Lancer la commande ansible -m debug -a "var=ansible\_user"



\### 2. Modifier le fichier ansible.cfg pour configurer la variable remote\_user



\### 3. Relancer la commande de debug



\### 4. Modifier l'inventaire pour y ajouter des users "web" et "db" et relancer la commande de debug



\### 5. Ajouter maintenant une variable "port\_apache" uniquement pour le serveur web et relancer la commande debug



\### 6. Tester maintenant la commande ansible -m debug -a "var=hostvars\['webServer'].port\_apache"



\### 7. Premier playbook de debug



```bash

mkdir /etc/ansible/playbooks

```



```yaml

\---

\- name: premier playbook debug

&#x20; hosts: web

&#x20; vars:

&#x20;   port\_apache: 8080

&#x20; tasks:

&#x20;   - name: debug

&#x20;     debug:

&#x20;       var: port\_apache

```



\### 8. Lancer la commande ansible-playbook debug.yml. Corriger l'erreur de user dans l'inventaire



\### 9. Expliquer les différentes étapes d'exécution du playbook (gathering facts)



\### 10. Lancer les commandes



```bash

ansible -m setup web

ansible -m setup web -a "filter=ansible\_distribution"

```



\### 11. Modifier le playbook pour afficher tous les ansible facts, puis seulement la distribution



\---



\## III. TP3 - host\_vars et group\_vars



\### 1. Créer deux répertoires dans /etc/ansible/hosts nommés host\_vars et group\_vars



\### 2. Dans group\_vars, faire un fichier web.yml avec une variable dns\_server: 1.1.1.1, puis faire un fichier db.yml avec 2.2.2.2



\### 3. Modifier le playbook debug.yml pour afficher cette variable pour toutes les machines (all)



\### 4. Dans host\_vars, faire un fichier dbServer.yml avec dns\_server: 1.2.3.4 et relancer le playbook



\---



\## IV. TP4 - Déploiement d'un serveur web et d'un serveur mariadb



\### 1. Créer un playbook ciblant le serveur web pour y installer le paquet apache2, puis activer le service apache au démarrage. Corriger l'erreur.



\### 2. Modifier le playbook pour ajouter un index.html dans le dossier /var/www/html/ qui contient le texte "HELLO WORLD!"



\### 3. Tester le résultat avec un curl



\### 4. Créer un 2e playbook ciblant le serveur db pour y installer mariadb (paquets mariadb-server et python3-mysqldb). Activer le service au démarrage.



\---



\## V. TP5 - Register, when, loop



\### 1. Dans le playbook debug.yml, utiliser un module qui va lire le fichier /etc/resolv.conf, l'enregistrer dans une variable puis afficher cette variable.



\### 2. Modifier le playbook pour n'afficher que le contenu du fichier



\### 3. Imaginons que le fichier /etc/resolv.conf n'existe pas, il faut tester sa présence et n'exécuter la tâche de lecture que s'il existe. Modifier le playbook en conséquence.



\### 4. Tester un la présence d'un fichier qui n'existe pas, et relancer le playbook pour observer ce qu'il se passe.



```yaml

\---

\- name: Playbook de debug

&#x20; hosts: web



&#x20; vars:

&#x20;   mes\_fichiers:

&#x20;     - /etc/resolv.conf

&#x20;     - /etc/crontab



&#x20; tasks:



&#x20;   - name: Tester la presence du fichier resolv.conf

&#x20;     stat:

&#x20;       path: "{{ item }}"

&#x20;     loop: "{{ mes\_fichiers }}"

&#x20;     register: stats\_fichiers



&#x20;   - name: Lire les fichiers

&#x20;     command: cat {{ item.item }}

&#x20;     register: cmd\_return

&#x20;     loop: "{{ stats\_fichiers.results }}"



&#x20;   - name: afficher la variable

&#x20;     debug:

&#x20;       var: item.stdout\_lines

&#x20;     loop: "{{ cmd\_return.results }}"

```



\### 5. Désinstaller mariadb du serveur cible. Modifier ensuite le playbook d'installation mariadb pour variabiliser les paquets à installer, puis utiliser une loop pour tous les installer dans une même task



\### 6. Modifier le playbook d'installation Apache pour changer le contenu de la page index.html. Enregistrer le résultat du module dans une variable. Ajouter une task de restart du service apache s'il y a eu une modif dans le fichier index.html.



\---



\## VI. TP6 - Handlers et notify



\### 1. Supprimer la condition "when" du playbook d'installation apache pour y ajouter un handler de rédémarrage du service s'il y a un change sur la tâche de copy du fichier index.html



\### 2. Modifier le playbook install\_mariadb pour y ajouter les tasks suivantes, ajouter un handler de redémarrage du service mariadb s'il y a un change quelque part.



```yaml

&#x20;   - name: config mariadb

&#x20;     lineinfile:

&#x20;       path: /etc/mysql/mariadb.conf.d/50-server.cnf

&#x20;       regexp: '^bind-address'

&#x20;       line: 'bind-address = 0.0.0.0'

&#x20;       state: present



&#x20;   - name: config root

&#x20;     mysql\_user:

&#x20;       name: root

&#x20;       host: localhost

&#x20;       password: toto

&#x20;       login\_user: root

&#x20;       login\_password: toto

&#x20;       priv: '\*.\*:ALL,GRANT'

&#x20;       state: present

&#x20;       login\_unix\_socket: /var/run/mysqld/mysqld.sock



&#x20;   - name: config database

&#x20;     mysql\_db:

&#x20;       name: ma\_base

&#x20;       state: present

&#x20;       login\_user: root

&#x20;       login\_password: toto



&#x20;   - name: create table users

&#x20;     mysql\_query:

&#x20;       login\_db: ma\_base

&#x20;       login\_user: root

&#x20;       login\_password: toto

&#x20;       query: |

&#x20;         CREATE TABLE IF NOT EXISTS mes\_users (

&#x20;           id INT AUTO\_INCREMENT PRIMARY KEY,

&#x20;           name VARCHAR(255) NOT NULL

&#x20;         );



&#x20;   - name: add users

&#x20;     mysql\_query:

&#x20;       login\_db: ma\_base

&#x20;       login\_user: root

&#x20;       login\_password: toto

&#x20;       query: |

&#x20;         INSERT INTO mes\_users (name) VALUES

&#x20;         ('axel'),('jean'),('alfred'),('gaston');

```



\### 3. Variabiliser tous les logins et les mots de passe dans les bons fichiers host\_vars ou group\_vars, ainsi que la liste des users à créer dans la base.



\---



\## VII. TP7 - Vaultisation



\### 1. Créer un fichier pass.txt et y inscrire une passe phrase de votre choix



\### 2. Vaultiser le mot de passe root avec cette passe phrase et l'inclure dans votre inventaire. Modifier le playbook d'installation en conséquence et le lancer



\---



\## VIII. TP8 - Rôles et collections



\### 1. Créer un répertoire roles dans /etc/ansible/playbooks



\### 2. Se placer dans ce répertoire et initialiser un role apache. Examiner l'arborescence créée.



\### 3. Déplacer les tâches du playbook d'install\_apache dans ce rôle, puis modifier le playbook pour qu'il fasse appel au rôle.



\### 4. Faire de même pour mariadb



\---



\## IX. TP9 - Templates



\### 1. Créer un fichier de Template index.php.j2 au bon endroit avec le contenu suivant (à vous de remplacer les variables)



```php

<?php



$servername = ????

$username = ???

$password = ???

$dbname = ???



$conn = new mysqli("localhost", $username, $password, $dbname);



if ($conn->connect\_error) {

&#x20;   die("Connection failed:" . $conn->connect\_error);

}



$sql = "SELECT name FROM mes\_users";

$result = $conn->query($sql);



if ($result->num\_rows > 0) {

&#x20;   while($row = $result->fetch\_assoc()) {

&#x20;       echo "Hello " . $row\["name"]. "<br>";

&#x20;   }

} else {

&#x20;   echo "0 result";

}



$conn->close();

?>

```



\### 2. Testez le résultat !



\### 3. Modifier votre index.php pour faire une vraie page HTML



```html

<!DOCTYPE html>

<html lang="fr">

<head>

&#x20;   <meta charset="UTF-8">

&#x20;   <title> A VARIABILISER </title>

&#x20;   <link rel="stylesheet" href="style.css">

</head>

<body>



&#x20;   <h1>Bienvenue</h1>

&#x20;   <p>Ceci est un exemple de page HTML avec un fichier CSS externe.</p>



</body>

</html>

```



\### 4. Templatiser le fichier style.css pour modifier les paramètres suivants (tout doit être variabilisé)



```css

body {

&#x20;   background-color: ????

&#x20;   font-family: Arial, sans-serif;

&#x20;   font-size: 14px;

}

```



\### 5. Testez à nouveau !





