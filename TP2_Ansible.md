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
