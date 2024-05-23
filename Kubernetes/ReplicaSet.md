Vimos que ao criarmos um POD e se ocorrer do nó cair, excluir o pod ou ele “morrer”, ele não será recriado sozinho!

Então precisamos de um objeto que cuide dessa funcionalidade, para isso existe o **ReplicaSet**.

Ele veio para substituir o Replication Controler em versões mais antigas do kubernetes.

Com o ReplicaSet, conseguimos garantir a quantidade de replicas que desejar, ou o estado da aplicação que se deseja manter para execução.

```YAML
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: cadastro-replicaset
spec:
  selector:
    matchLabels:
      # Tem que ser o mesmo valor declarado no POD.
      app: cadastro
  template:
    metadata:
      labels:
        app: cadastro
    spec:
      containers:
      # Abaixo é utilizado as mesmas informações declarado para criar um pod manualmente.
        - name: cadastro-container
          image: leogoandete/cadastro:latest  # Substitua com a imagem do seu contêiner
          ports:
          - containerPort: 8080
          env:
          - name: QUARKUS_DATASOURCE_USERNAME
            value: "user"
          - name: QUARKUS_DATASOURCE_PASSWORD
            value: "password"
          - name: QUARKUS_DATASOURCE_DB_KIND
            value: "mysql"
          - name: QUARKUS_DATASOURCE_JDBC_URL
            value: "jdbc:mysql://mysql-service:3306/cadastro"
```

  

Após criar o arquivo, aplicar com o comando:

```Bash
kubectl apply -f <nome-arquivo>.yml
```

  

Agora posso fazer a deleção do POD e o ReplicaSet irá garantir que irá criar um novo caso for deletado ou o pod “morrer”.

Veja o exemplo a seguir:

Onde é excluído o pod **cadastro-replicaset-jddnc** e ao listar os pods novamente vemos um novo pod **cadastro-replicaset-l6mtk**

> [!info] Validando ReplicaSet  
> Recorded by leogoandete  
> [https://asciinema.org/a/9tJSN58fHWC3WaiMw5W9A3qYX](https://asciinema.org/a/9tJSN58fHWC3WaiMw5W9A3qYX)
> 
> 
> 

