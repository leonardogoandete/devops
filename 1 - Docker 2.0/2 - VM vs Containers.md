## Maquina Virtual:

Contem as camadas:

![](../imagens/arq-maquina-virtual.png)

VMs: Cada vm acessa um pouco da infraestrutura de forma virtual e criar uma máquina.

Hypervisor : virtualização do recursos e gerenciamento das vms criadas

Infraestrutura: memoria, processador e etc.

  

### Características:

- Alto isolamento, pois há uma máquina virtual para cada aplicaçãoç
- Alto consumo de recurso, ex cada S.O usa 2GiB de memória.

## Conteiners:

![](../imagens/arq-container.png)

  

Container Runtime: docker

Sistema Operacional: linux

Infraestrutura: memoria, processador e etc.

  

### Características:

- Baixo isolamento, pois roda o mesmo kernel do host;
- Alta economia de recursos;