É errado o uso de variáveis de ambientes 


**ConfigMap** é utilizado para guardar elementos de chave-valor, de forma que poderá ser utilizado nos arquivos de manifestos e setar as variáveis de ambientes.

Vamos extrair as variáveis de ambiente do arquivo `cadastro-deployment.yml` por exemplo:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cadastro-deployment
spec:
  selector:
    matchLabels:
      app: cadastro
  template:
    metadata:
      labels:
        app: cadastro
    spec:
      containers:
        - name: cadastro-container
          image: leogoandete/cadastro:latest
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


Exemplo de um manifesto de **ConfigMap**:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: meu-configmap
data:
  QUARKUS_DATASOURCE_USERNAME: user
  QUARKUS_DATASOURCE_PASSWORD: password
  QUARKUS_DATASOURCE_DB_KIND: mysql
  QUARKUS_DATASOURCE_JDBC_URL: jdbc:mysql://mysql-service:3306/cadastro
```

Basta aplicar e listar:
```bash
# Criar:
kubectl apply -f <manifesto-configmap>.yaml

# Listar:
kubectl get configmap
```

Para usarmos no `deployment` alteramos o arquivo e utilizar o `envFrom`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cadastro-deployment
spec:
  selector:
    matchLabels:
      app: cadastro
  template:
    metadata:
      labels:
        app: cadastro
    spec:
      containers:
        - name: cadastro-container
          image: leogoandete/cadastro:latest
          ports:
          - containerPort: 8080
          envFrom:
            - configMapRef:
                name: meu-configmap  #Aqui usamos o nome do configmap criado
```

É possível pegar somente o valor do configmap caso a `key` seja diferente, conforme abaixo:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cadastro-deployment
spec:
  selector:
    matchLabels:
      app: cadastro
  template:
    metadata:
      labels:
        app: cadastro
    spec:
      containers:
        - name: cadastro-container
          image: leogoandete/cadastro:latest
          ports:
          - containerPort: 8080
          env:
            - name: USER_BANCO
              valueFrom:
                configMapKeyRef:
                  key: QUARKUS_DATASOURCE_USERNAME
                  name: meu-configmap
```

`valueFrom` informa que o valor da chave para ser definida, será obtida de um configmap.
`configMapKeyRef` informa que sera de uma chave especifica dentro do configmap.
`key` declara o nome da chave onde está o valor desejado.
`name` informa o nome do configmap onde deseja obter o valor.

---

**Secret** é a mesma situação do ConfigMap, o diferencial é que os valores serão mascarados.