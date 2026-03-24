## III. TP3 - host_vars et group_vars

### 1. Créer deux répertoires dans /etc/ansible/inventory nommés host_vars et group_vars

### 2. Dans group_vars, faire un fichier `web.yml` avec une variable `dns_server: 1.1.1.1`, puis faire un fichier `db.yml` avec `2.2.2.2`

### 3. Modifier le playbook `debug.yml` pour afficher cette variable pour toutes les machines (all)

### 4. Dans host_vars, faire un fichier `dbServer.yml` avec `dns_server: 1.2.3.4` et relancer le playbook

---
