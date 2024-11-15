## Register

O que é um Register?

O registro é uma maneira de armazenar o resultado de uma tarefa em uma variável para uso posterior. Isso é útil quando você deseja usar o resultado de uma tarefa em outra tarefa.

Como usar um Register?

Para usar um registro, você precisa adicionar a variável `register` ao final de uma tarefa. Por exemplo:

```yaml
- name: Resultado de um comando
  hosts: localhost
  tasks:
    - name: Executar um comando
      ansible.builtin.shell: ps
        register: resultado
    - name: Exibir o resultado
      ansible.builtin.debug:
        msg: "{{ resultado }}"
```