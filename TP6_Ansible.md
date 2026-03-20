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
