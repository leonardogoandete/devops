## ReplicaSet

### O que é um ReplicaSet?
ReplicaSet é um controlador do Kubernetes que garante que um número específico de réplicas de um Pod esteja sempre em execução. Ele monitora os Pods e, se algum deles falhar ou for excluído, o ReplicaSet cria novos Pods para manter o número desejado de réplicas.
O ReplicaSet é usado para garantir a alta disponibilidade e a escalabilidade dos Pods, permitindo que você execute várias instâncias de um Pod em paralelo. Isso é especialmente útil para aplicações que precisam lidar com alta carga de trabalho ou que precisam ser resilientes a falhas.


Como criar um ReplicaSet?

Para criar um ReplicaSet, você pode usar um arquivo de configuração YAML. Aqui está um exemplo básico de um ReplicaSet que mantém 3 réplicas de um Pod:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-container
          image: nginx:latest
          ports:
            - containerPort: 80
```
Neste exemplo, estamos criando um ReplicaSet chamado `my-replicaset` que mantém 3 réplicas de um Pod com a label `app: my-app`. O template do Pod define o container que será executado, que neste caso é o `nginx:latest` na porta 80.

Para criar o ReplicaSet, você pode usar o comando `kubectl apply -f <nome-do-arquivo>.yaml`. Isso criará o ReplicaSet no cluster Kubernetes e iniciará as réplicas do Pod conforme especificado.

### Como funciona o ReplicaSet?
O ReplicaSet funciona monitorando os Pods que ele gerencia. Ele verifica se o número de Pods em execução corresponde ao número desejado de réplicas. Se algum Pod falhar ou for excluído, o ReplicaSet cria um novo Pod para substituí-lo, garantindo que o número de réplicas permaneça constante.
O ReplicaSet usa um seletor de labels para identificar quais Pods ele deve gerenciar. No exemplo acima, o seletor `matchLabels` especifica que o ReplicaSet deve gerenciar todos os Pods com a label `app: my-app`. Isso permite que o ReplicaSet mantenha o controle sobre os Pods que ele deve monitorar e gerenciar. 

Porem como o ReplicaSet é um controlador de baixo nível ou seja o ReplicaSet junto com o pod da escalabilidade e resiliencia, porem se quiser fazer atualizações, como por exemplo atualizar a imagem do container, precisamos de um controlador de mais alto nível, como o Deployment, que gerencia ReplicaSets e Pods. O Deployment permite atualizações contínuas e rollback fácil em caso de falhas.