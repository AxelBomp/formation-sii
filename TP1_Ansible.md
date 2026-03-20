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
