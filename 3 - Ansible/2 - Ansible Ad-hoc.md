## Ansible Ad-hoc

O ansible ad-hoc é a execução mais simples de executar uma tarefa, pode ser feito em varios hosts gerenciados porem será executado somente uma instrução.

Ex.: Instalar o NGINX, reiniciar, upload de arquivo, cada um dessas ações deverá ser feito com um unico comando separada.

Onde é utilizado? 

Para execuxões pontuais, como desligar a maquina, executar um ping, geralmente para tarefas mais simples.

## Executando comandos

No exemplo executamos o comando para rodar na maquina local.

```bash
ansible localhost -m ping
```

Lista de elementos para utilizar objetos do ansible:
https://docs.ansible.com/ansible/latest/collections/index.html

#### Utilizando maquina remota

No exemplo abaixo foi utilizado duas maquinas de exemplo:

- O problema abaixo toda vez que for adicionado uma nova maquina é preciso adicionar no `know hosts`
- Utilização de inventário para não passar mais os IPs no comando.

```bash
ansible all -i '192.168.0.12,192.168.0.13,' -m ping --private-key ~/.ssh/<nome-da-chave> -u <usuario-da-maquina-remota>
```

