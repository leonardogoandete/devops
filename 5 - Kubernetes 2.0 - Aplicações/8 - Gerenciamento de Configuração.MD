### Gerenciamento de Configuração

Será abordado de forma geral as configurações que uma aplicação precisa.
Geralmente se usa como variaveis de ambientes, e podemos guardar tokens de API, valores que serão parametrizaveis por ambientes! 

Como configurar? 

O kube fornece diverçãs formas para configurar, podemos colocar no manifesto do Deployment.yaml por exemplo:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-configuracao
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app-configuracao
  template:
    metadata:
      labels:
        app: app-configuracao
    spec:
      containers:
      - name: app-configuracao
        image: kubedevio/app-variaveis-ambiente:v1
        env: ## Aqui é o ponto onde definimos as variaveis de ambiente
        - name: APP_NAME # Nome 
          value: "Minha aplicação" # Valor
        - name: APP_VERSION
          value: "1.0.0"
        - name: APP_AUTH
          value: "xpto"
        ports:
        - containerPort: 3000
```
Seção onde configuramos as variáveis de ambiente
```yaml
env: ## Aqui é o ponto onde definimos as variaveis de ambiente
  - name: APP_NAME # Nome 
    value: "Minha aplicação" # Valor
  - name: APP_VERSION
    value: "1.0.0"
  - name: APP_AUTH
    value: "xpto"
```

O formato acima pode ser feito, porem não é muito recomendavel, para isso o ideal é utilizar os recursos do Kubernetes para isso que são ConfigMap e Secrets.

## ConfigMap

É um objeto do cluster utilizado para armazenar elementos chave-valor. Podemos criar ele via linha de comando e através de arquivos de manifesto YAML.

Uma observação importante é que o ConfigMap não deve conter informações sensíveis, como senhas ou chaves de API pois o valor fica legivel. Para esses casos, devemos utilizar o recurso de Secrets do Kubernetes.

Criando ConfigMap via linha de comando:

```bash
kubectl create configmap app-config --from-literal=APP_NAME="Minha aplicação" --from-literal=APP_VERSION="1.0.0" --from-literal=APP_AUTH="xpto"
```
Criando ConfigMap via linha de comando com arquivo:

```bash
kubectl create configmap app-config --from-file=app.properties
```

Criando ConfigMap através de manifesto YAML:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_NAME: "Minha aplicação"
  APP_VERSION: "1.0.0"
  APP_AUTH: "xpto"
```
Como utilizamos os ConfigMaps? 
Podemos utilizar de duas formas, por referência ou por definição de valor!

#### Utilizando o ConfigMap no Deployment por referência: 

Primeiro precisamos criar a estrutura `envFrom` nos containers, lembrando que o nome das variáveis de ambiente utilizada no app deve corresponder aos nomes das chaves no ConfigMap.

```yaml
    ...
    spec:
      containers:
      - name: app-configuracao
        image: kubedevio/app-variaveis-ambiente:v1
        envFrom:
        - configMapRef:
            name: app-config # Nome do ConfigMap
```

#### Utilizando o ConfigMap no Deployment por valor: 

Primeiro precisamos adicionar a estrutura `env` nos containers. Nesse formato o nome da variável de ambiente não precisa corresponder ao nome da chave no ConfigMap, pois podemos fazer a referência de forma mais flexível.

```yaml
    ...
    spec:
      containers:
      - name: app-configuracao
        image: kubedevio/app-variaveis-ambiente:v1
        env:
        - name: APP_NAME # Nome da variável de ambiente do app
          valueFrom:
            configMapKeyRef:
              name: app-config # Nome do ConfigMap
              key: APP_NAME_CONFIG # Nome da chave no ConfigMap de onde vamos pegar o valor
        - name: APP_VERSION
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_VERSION_CONFIG
        - name: APP_AUTH
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_AUTH_CONFIG
```

## Secret

Os Secrets são semelhantes aos ConfigMaps, mas são utilizados para armazenar informações sensíveis, como senhas, chaves de API e certificados. Os Secrets são codificados em base64 e, por padrão, não são exibidos em texto claro quando você executa comandos `kubectl`.

Criando Secret via linha de comando:

```bash
kubectl create secret generic app-secret --from-literal=APP_PASSWORD="minha-senha-secreta"
```

Criando Secret através de manifesto YAML:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  APP_PASSWORD: bWluaGEtc2VuaGEtc2VjcmV0YQ== # "minha-senha-secreta" em base64
```

Como utilizamos os Secrets? 
Assim como os ConfigMaps, podemos utilizar os Secrets por referência ou por valor.

#### Utilizando o Secret no Deployment por referência: 

```yaml
    ...
    spec:
      containers:
      - name: app-configuracao
        image: kubedevio/app-variaveis-ambiente:v1
        envFrom:
        - secretRef:
            name: app-secret # Nome do Secret
```

#### Utilizando o Secret no Deployment por valor: 

```yaml
    ...
    spec:
      containers:
      - name: app-configuracao
        image: kubedevio/app-variaveis-ambiente:v1
        env:
        - name: APP_PASSWORD # Nome da variável de ambiente do app
          valueFrom:
            secretKeyRef:
              name: app-secret # Nome do Secret
              key: APP_PASSWORD_CONFIG # Nome da chave no Secret de onde vamos pegar o valor 
```



