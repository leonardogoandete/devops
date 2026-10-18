## Exemplo prático completo 

O exemplo a seguir instala o pacote `whois`, cria um usuário, gera uma senha criptografada, cria um diretório `.ssh` e copia uma chave pública para o usuário criado.

Arquivo `playbook.yaml`:
```yaml
---
- name: Criação de um usuario
  hosts: all
  remote_user: root
  vars_files:
    - vars.yaml
  tasks:
    - name: Instalar o pacote whois
      ansible.builtin.apt:
        name: whois
        state: present
        update_cache: true

    - name: Gerar senha criptografada
      ansible.builtin.shell: "mkpasswd --method=SHA-512 --rounds=4096 {{ senha }}"
      register: senha_criptografada

    - name: Criar o usuario
      ansible.builtin.user:
        name: "{{ nome }}"
        password: "{{ senha_criptografada.stdout }}"
        state: present
        shell: /bin/bash
        groups: sudo
        append: yes
    - name: Criar o diretorio ssh 
      ansible.builtin.file:
        path: "/home/{{ nome }}/.ssh"
        state: directory
        owner: "{{ nome }}"
        group: "{{ nome }}"
        recurse: true
    - name: Copiar a chave publica
      ansible.builtin.copy:
        src: "/home/leo/.ssh/aula_ansible.pub"
        dest: "/home/{{ nome }}/.ssh/authorized_keys"
```

Arquivo `vars.yaml`:
```yaml
---
nome: leo
senha: "123456"
```

Arquivo `hosts`:
```ini
[ansible] # Grupo ansible
10.253.179.58
10.253.179.87
10.253.179.228
10.253.179.229

[ansible:vars] # Variaveis para o grupo ansible
ansible_ssh_private_key_file=~/.ssh/aula_ansible
```