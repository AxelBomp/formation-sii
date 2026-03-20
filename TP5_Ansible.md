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
