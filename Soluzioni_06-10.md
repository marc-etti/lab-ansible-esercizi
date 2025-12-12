# Soluzioni agli esercizi di Ansible

## 6. Copiare una directory intera su tutti i nodi gestiti
Struttura delle cartelle:
```
playbook_copy_dir.yml
files/
 └── mia_cartella/
      ├── file1.txt
      └── file2.txt
```
playbook_copy_dir.yml
```
---
- name: Copiare una directory su tutti i nodi
  hosts: ubuntu_nodes
  become: true

  tasks:
    - name: Copiare directory ricorsivamente
      ansible.builtin.copy:
        src: "files/mia_cartella/"
        dest: "/opt/mia_cartella/"
        owner: root
        group: root
        mode: "0755"
        recursive: yes
```

## 7. Utilizzare un template Jinja2
Struttura file:
```
playbook_template.yml
templates/
 └── config.j2
```
Esempio template (templates/config.j2):
```
Applicazione: {{ app_name }}
Ambiente: {{ env }}
Versione: {{ version }}
```
playbook_template.yml
```
---
- name: Usare un template Jinja2
  hosts: ubuntu_nodes
  become: true

  vars:
    app_name: "DemoApp"
    env: "development"
    version: "1.0.0"

  tasks:
    - name: Generare config da template
      ansible.builtin.template:
        src: "templates/config.j2"
        dest: "/etc/demoapp.conf"
        mode: "0644"
```
Per verificare, eseguire `cat /etc/demoapp.conf` su uno dei nodi gestiti.

## 8. Eseguire un comando e salvare l’output
playbook_command_output.yml
```
---
- name: Eseguire comando e salvare output
  hosts: ubuntu_nodes

  tasks:
    - name: Eseguire ls /root
      ansible.builtin.command: "ls -l /root"
      register: risultato_ls

    - name: Mostrare l'output
      ansible.builtin.debug:
        var: risultato_ls.stdout
```

## 9. Playbook con più "play" separati
playbook_multi_play.yml
```
---
# Play 1 - Aggiornamento pacchetti
- name: Aggiornare i pacchetti
  hosts: ubuntu_nodes
  become: true

  tasks:
    - name: Update apt
      ansible.builtin.apt:
        update_cache: yes
        upgrade: yes

# Play 2 - Installare un'applicazione
- name: Installare Apache
  hosts: ubuntu_nodes
  become: true

  tasks:
    - name: Installare Apache2
      ansible.builtin.apt:
        name: apache2
        state: present
```

## 10. Creare un ruolo Ansible personalizzato
Nuovo file inventario:
```
[web_servers]
node1 ansible_host=127.0.0.1 ansible_port=2222 ansible_user=root ansible_password=rootpass ansible_python_interpreter=/usr/bin/python3 ansible_ssh_common_args='-o StrictHostKeyChecking=no'

[system_servers]
node2 ansible_host=127.0.0.1 ansible_port=2223 ansible_user=root ansible_password=rootpass ansible_python_interpreter=/usr/bin/python3 ansible_ssh_common_args='-o StrictHostKeyChecking=no'

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

Struttura delle cartelle:
```
.
├── inventory.ini
├── playbook_principale.yml
└── roles/
    ├── ruolo_web/
    │   ├── tasks/
    │   │   └── main.yml
    │   ├── handlers/
    │   │   └── main.yml
    │   ├── templates/
    │   │   └── index.html.j2
    │   ├── files/
    │   │   └── logo.png
    │   └── vars/
    │       └── main.yml
    └── ruolo_system/
        ├── tasks/
        │   └── main.yml
        └── vars/
            └── main.yml
```
### RUOLO 1: ruolo_web
roles/ruolo_web/vars/main.yml
```
app_title: "Benvenuto nel mio server"
```

roles/ruolo_web/templates/index.html.j2
```
<h1>{{ app_title }}</h1>
<p>Questa pagina è stata generata tramite Ansible con un ruolo dedicato.</p>
<img src="logo.png" alt="Logo" />
```
roles/ruolo_web/tasks/main.yml
```
---
- name: Installare Apache
  ansible.builtin.apt:
    name: apache2
    state: present
    update_cache: yes

- name: Copiare il template index.html
  ansible.builtin.template:
    src: index.html.j2
    dest: /var/www/html/index.html
    mode: "0644"
  notify: Restart Apache

- name: Copiare il file logo.png
  ansible.builtin.copy:
    src: logo.png
    dest: /var/www/html/logo.png
    mode: "0644"
```

roles/ruolo_web/handlers/main.yml
```
---
- name: Restart Apache
  ansible.builtin.service:
    name: apache2
    state: restarted
```
### RUOLO 2: ruolo_system
roles/ruolo_system/vars/main.yml
```
packages:
  - curl
  - htop
  - vim
```
roles/ruolo_system/tasks/main.yml
```
---
- name: Aggiornare il sistema
  ansible.builtin.apt:
    update_cache: yes
    upgrade: yes

- name: Installare pacchetti utili
  ansible.builtin.apt:
    name: "{{ packages }}"
    state: present
```
### Playbook principale che utilizza i ruoli
playbook_principale.yml
```
---
# Play 1 - gestisce node2
- name: Configurazione di sistema
  hosts: system_servers
  become: true

  roles:
    - ruolo_system

# Play 2 - gestisce node1
- name: Installazione server web
  hosts: web_servers
  become: true

  roles:
    - ruolo_web
```