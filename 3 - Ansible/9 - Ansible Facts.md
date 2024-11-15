### Ansible Facts

O Ansible Facts é um sistema de descoberta de informações sobre os hosts gerenciados pelo Ansible. O Ansible Facts coleta informações sobre o sistema operacional, hardware, software, rede e muito mais. O Ansible Facts é útil para coletar informações sobre os hosts gerenciados pelo Ansible e usar essas informações em tarefas de playbook.

# Como listar os facts do Ansible?

Ha duas maneiras de listar os facts do Ansible:

## 1. Usando o módulo setup ad-hoc

```shell
ansible all -i hosts -m setup
```

Com o comando acima temos uma saída muito grande, para filtrar a saída podemos usar o comando abaixo:

```shell
ansible all -i hosts -m setup -a "filter=ansible_processor"
```



## 2. Usando facts no playbook

```yaml
---
- name: Ver os dados de Facts
  hosts: all
  remote_user: root
  tasks:
    - name: Ver o processador
      ansible.builtin.debug:
        msg: "{{ ansible_processor }}"
    - name: Ver o ip
      ansible.builtin.debug:
        msg: "{{ ansible_ens3.ipv4.address }}"
```

As vezes nao precisamos usar o Facts, podemos usar o comando abaixo para desativar o Facts:

Dessa forma o Ansible não coleta informações sobre os hosts gerenciados pelo Ansible tornando a execução do playbook mais rápida e eficiente.

```yaml
---
- name: Teste sem Facts
  hosts: all
  gather_facts: no
  tasks:
    - name: Exibir uma mensagem
      ansible.builtin.debug:
        msg: "Olá Mundo"
```