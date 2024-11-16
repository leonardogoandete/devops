### Ansible Templates 

O ansible permite a utilização de templates para a criação de arquivos de configuração. Para isso, é necessário criar um arquivo de template com a extensão .j2.

Documentação do jinja2: [https://jinja.palletsprojects.com/en/stable/templates/](https://jinja.palletsprojects.com/en/stable/templates/)

## Template simples com jinja2

Arquivo de template: `template.j2`
```jinja2
Olá {{ nome }}!
```

Arquivo de variáveis: `vars.yaml`
```yaml
nome: "Leonardo"
```

Arquivo de playbook: `playbook.yml`
```yaml
---
- name: Trabalhando com templates
  hosts: localhost
  vars_files:
    - vars.yaml 
  tasks:
    - name: Criando o template
      ansible.builtin.template:
        src: template.j2
        dest: <CAMINHO DESEJADO>/exemplo.txt
```

Se tudo ocorrer bem, o arquivo `exemplo.txt` será criado com o seguinte conteúdo:
```
Olá Leonardo!
```

## Utilizando estruturas de decisão

Digamos que precisamos exibir uma mensagem com o idioma diferente dependendo da variável `idioma`.

Arquivo de template: `template.j2`
```jinja2
{% if idioma == "pt" %}
Olá {{ nome }}!
{% else %}
Hello {{ nome }}!
{% endif %}
```

Arquivo de variáveis: `vars.yaml`
```yaml
nome: "Leonardo"
idioma: "en"
```

Se tudo ocorrer bem, o arquivo `exemplo.txt` será criado com o seguinte conteúdo:
```
Hello Leonardo!
```

## Utilizando estruturas de repetição

Digamos que precisamos exibir uma lista de nomes.

Arquivo de template: `template.j2`
```jinja2
{% for nome in nomes %}
{% if idioma == "pt" %}
Olá {{ nome }}!
{% else %}
Hello {{ nome }}!
{% endif %}
{% endfor %}
```

## Exemplo aprimorando a configuração do Nginx

Arquivo hosts:
```ini
[maquinas] 
10.253.179.58
10.253.179.87
10.253.179.228
10.253.179.229

[maquinas:vars]
ansible_ssh_private_key_file=~/.ssh/aula_ansible
ansible_user=root
```

Arquivo de variáveis: `vars.yaml`

```yaml
usuario: 
  - leonardo
senha: leonardo
chave_publica: /home/leo/.ssh/aula_ansible.pub
```

Aqui parametrizamos o arquivo de configuração do Nginx para que ele execute em diferentes diretórios home de usuários diferentes.

Arquivo de template: `nginx.conf.j2`
```jinja2

user {{ usuario }} {{ usuario }};

events {
    worker_connections 1024;
}

http
{
    server
    {
        root /home/{{ usuario }}/;
        location / {
            index index.html;
        }
    }
}
```

O arquivo `index.html` pode ser qualquer arquivo html.

Arquivo de playbook: `playbook.yml`
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
        name: "{{ usuario }}"
        password: "{{ senha_criptografada.stdout }}"
        state: present
        shell: /bin/bash
        groups: sudo
        append: yes
    - name: Criar o diretorio ssh 
      ansible.builtin.file:
        path: "/home/{{ usuario }}/.ssh"
        state: directory
        owner: "{{ usuario }}"
        group: "{{ usuario }}"
        recurse: true
    - name: Copiar a chave publica
      ansible.builtin.copy:
        src: "{{ chave_publica }}"
        dest: "/home/{{ usuario }}/.ssh/authorized_keys"

- name: Instalar nginx e executar pagina web
  hosts: all
  vars_files:
    - vars.yaml
  remote_user: "{{ usuario }}"
  tasks:
    - name: Copiar a pagina web
      ansible.builtin.copy:
        src: ./index.html
        dest: /home/{{ usuario }}/index.html
    - name: Instalar o pacote nginx
      become: true
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: true
    - name: Copiar o arquivo de configuração
      become: true
      ansible.builtin.template:
        src: ./nginx.conf.j2
        dest: /etc/nginx/nginx.conf
    - name: Reiniciar o serviço nginx
      become: true
      ansible.builtin.service:
        name: nginx
        state: restarted
```


