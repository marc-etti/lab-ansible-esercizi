# Soluzioni agli esercizi di Ansible

## 1. Creare un file di testo su tutti i nodi
playbook_file.yml
```
---
- name: Creare un file di testo su tutti i nodi
  hosts: all
  become: true

  tasks:
    - name: Creare un file di testo con contenuto
      ansible.builtin.copy:
        dest: /tmp/prova.txt
        content: "Questo file è stato creato tramite Ansible.\n"
        owner: root
        group: root
        mode: '0644'
```

## 2. Creare un utente su tutti i nodi
playbook_user.yml
```
---
- name: Creare un utente su tutti i nodi gestiti
  hosts: all
  become: true

  tasks:
    - name: Creare l'utente developer
      ansible.builtin.user:
        name: developer
        shell: /bin/bash
        state: present
```

## 3. Cambiare la password di un utente
Nota: la password deve essere hashata con SHA-512.
playbook_change_password.yml
```
---
- name: Cambiare la password di un utente su tutti i nodi
  hosts: all
  become: true

  vars:
    nuova_password: "{{ 'Password123!' | password_hash('sha512') }}"

  tasks:
    - name: Aggiornare la password dell'utente developer
      ansible.builtin.user:
        name: developer
        password: "{{ nuova_password }}"
```

## 4. Installare pacchetti software
playbook_install_packages.yml
```
---
- name: Installare pacchetti software su tutti i nodi
  hosts: all
  become: true

  tasks:
    - name: Aggiornare la lista dei pacchetti
      ansible.builtin.apt:
        update_cache: true

    - name: Installare nginx e curl
      ansible.builtin.apt:
        name:
          - nginx
          - curl
        state: present
```

## 5. Gestire un servizio (start/stop/restart)
playbook_service.yml
```
---
- name: Gestire un servizio su tutti i nodi
  hosts: all
  become: true

  tasks:
    - name: Avviare nginx
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true

    - name: Arrestare nginx
      ansible.builtin.service:
        name: nginx
        state: stopped

    - name: Riavviare nginx
      ansible.builtin.service:
        name: nginx
        state: restarted
```