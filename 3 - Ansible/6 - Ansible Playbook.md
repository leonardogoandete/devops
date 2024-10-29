### **Ansible Playbook**

O que é ?

Os Playbooks do Ansible são um conjunto de instruções que especificam como um determinado sistema deve ser configurado ou gerenciado. Eles podem ser usados para automatizar tarefas simples ou complexas e podem ser facilmente compartilhados entre equipes de operações.

Resumindo:
É um conjunto de plays(execuções) utilizando arquivos `yaml`.

## Como configurar

1. Definir o inventário

```.ini
[nginx]
192.168.0.19
192.168.0.20
192.168.0.21
192.168.0.22

[nginx:vars]
ansible_user=root
ansible_ssh_private_key_file=~/.ssh/aula_vagrant
```

2. Arquivo `index.html` para subtituição.
```html
<!DOCTYPE html>
<html lang="pt-br">
    <head>
        <title>Titulo da Pagina</title>
        <meta charset="utf-8">
    </head>
    <body>
        <h1>Conteudo do HTML.</h1>
    </body>
</html>
```

3. Arquivo `playbook.yaml`
```yaml
---
# Primeiro play
- name: Intalação do nginx e configuração de uma pagina estatica
  # No hosts pode ser o nome do grupo ou para todos `all`
  hosts: all
  # Lista de tarefas
  tasks:
      # nome da tarefa
    - name: Instalar o NGINX
      # Define o modulo a ser utilizado
      ansible.builtin.apt:
        name: nginx
        state: present
    - name: Copiar o arquivo index.html
      ansible.builtin.copy:
        src: ./index.html
        dest: /var/www/html/index.nginx-debian.html
```

4. Execução.

Agora basta executar o playbook usando o comando abaixo:

```bash
ansible-playbook -i hosts playbook.yaml
```

Se tudo der certo veremos a seguinte saida:

![alt text](../imagens/primeiro-playbook.png)
      


