### Ansible Module
O que é ? 

São blocos de código que é executado nas maquinas remotas, geralmente utilizado para executar uma ação.

Os modulos são distribuido em collections no ansible e pode ser acessado no link abaixo os diversos modulos existentes.

https://docs.ansible.com/ansible/latest/collections/index.html


### **Modulo Reboot**

```bash
ansible all -i hosts -m reboot
```

### **Modulo Copy**
```bash
ansible all -i hosts -m copy -a "src=./arquivo.txt dest=/home/vagrant/arquivo.txt"
```

### **Modulo Apt (debian) / Yum (Redhat)**
1. Instalando pacote
```bash
ansible ansible all -i hosts -m apt -a "name=nginx update_cache"
```

2. Desabilitando pacote
```bash
ansible ansible all -i hosts -m apt -a "name=curl absent"
```

3. Habilitando pacote
```bash
ansible ansible all -i hosts -m apt -a "name=curl present"
```

### **Modulo Service**

1. Parando o serviço do Nginx
```bash
ansible ansible all -i hosts -m service -a "name=nginx state=stopped"
```

1. Iniciando o serviço do Nginx
```bash
ansible ansible all -i hosts -m service -a "name=nginx state=started"
```
