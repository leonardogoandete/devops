O POD tem um ciclo de vida dentro do Kubernetes.
Ele é criado, cumpre o seu papel e depois é eliminado.

Dentro desse percursos ele tem alguns estágios. Vamos ver a seguida.

##### Estágio dos PODs:
`PENDING`: Este é o estágio inicial do POD quando é aceito no cluster e será criado, esse estágio dura até o download das imagens para a criação dos containers. 

`RUNNING`: Ele entra neste estado depois de ser criado e os container entram em processo de inicialização.

`SUCESS`: Assim que todos os containers estiverem rodando com sucesso.

`FAILED`: Onde algum container falhou.

`UNKNOW`: Será preciso analise.


##### **Estágios dos containers:**
`WAITING`: Está em processo de inicialização, podendo estar baixando uma imagem do registry ou aplicando algumas configurações.

`RUNNING`: Quando não há nenhum problema e ele está rodando perfeitamente.

`TERMINATED`: Quando tem algum problema, ou o POD está sendo encerrado.

---

### **Signal SIGTERM e Signal SIGKILL**

Sempre que um container e encerrado e não importa a razão. Sempre será disparado para o container um `sigterm`(delicado) e em seguida um `sigkill`(agressivo). Por padrão o tempo de execução do comando é de 30seg, podemos alterar usando o parâmetro `terminationGracePeriodSeconds`.

---
### **Post Start e Pré Stop**


Post Start e executado paralelamente após o processo de inicialização do container, mas ele tem que retornar um valor positivo para o container sair do estágio WAITING se retornar erro o container é encerrado.

Podemos usar ele com o `HTTP GET` e o `exec`
```yaml
...
spec:
  containers:
    - name: nginx
      image: nginx
      lifecycle:
        postStart: # comando para executar antes de iniciar
          exec:
            command:
              - "time"
        preStop: # comando para executar antes de encerrar
          exec:
            command:
              - "time"
```

---
### **Init Container**

Cria um container para "preparar" o terreno antes de executar o container pricipal, por exemplo uma migration de um banco ou a instalação de um agente de monitoramento.

```yaml
...
spec:
  initContainers:
    - name: init
      image: ubuntu/ubuntu:latest
      command: ["curl","-X","POST" "url-binario"]
  containers:
    - name: nginx
      image: nginx      
```

Obs.: Caso não exista a url ele irá gerar erro e ficar restartando e o status `Init:Error`

![[../imagens/erro-init-container.png]]