
O Helm lhe auxilia no gerenciamento de aplicações para Kubernetes — Charts do Helm ajudam a definir, instalar, e atualizar até a mais complexa aplicação para o Kubernetes.

O deploy no Kubernetes é bem complexo e podemos acabar esquecendo algum arquivo para aplicar/configurar .

Para isso temos um gerenciador de pacotes, similar com o apt, o helm ele é baseado em templates, que é chamado de charms.

Ele funciona com uma ferramenta CLI, e a partir dela criamos algum charts, aplicar algum chats. Sempre utilizamos algum repositório para escolher as ferramentas.

Toda instalação de um chart é uma release.

### **Instalação**

Link: https://helm.sh/docs/intro/install/

### **Manipulando repositórios**

Adicionando repositório: 
```bash
helm repo add stable https://charts.helm.sh/stable
```

Atualizando lista de pacotes:
```bash
helm repo update
```

Listando repositórios:
```bash
helm repo list
```

Removendo repositórios:
```bash
helm repo remove <nome-repositorio>
```


### ** Instalando aplicações**

Instalando o helm para instalar o nginx ingress controller.
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```

Podemos executar um `inspect` para ver as informações do helm.

```bash
helm inspect all ingress-nginx/ingress-nginx
```

Podemos ver somente os `values` de um chart:

```bash
helm inspect values ingress-nginx/ingress-nginx
```

Instalando de fato o chart:
```bash
helm install <nome-release> ingress-nginx/ingress-nginx

# Em um namespace especifico:
helm install <nome-release> ingress-nginx/ingress-nginx --namespace <nome-namespace>
```

Output de saída da instalação de uma release:

![](../imagens/instalacao-helm-ingress.png)

Desinstalando uma release:
```bash
helm uninstall <nome-release>
# Em um namespace especifico:
helm uninstall <nome-release> --namespace <nome-namespace>
```

Output: 
```bash
release <nome-releasse> uninstalled
```

Atualizando uma release:
```bash
helm upgrade <nome-release> ingress-nginx/ingress-nginx --values <caminho-do-arquivo>

# Em um namespace especifico:
helm upgrade <nome-release> ingress-nginx/ingress-nginx --values <caminho-do-arquivo> --namespace <nome-namespace>
```

Visualizar histórico de uma release: 
```
helm history <nome-release>

# Em um namespace especifico:
helm history <nome-release> --namespace <nome-namespace>
```

Podemos fazer um rollback, ou seja voltar a versão anterior:
```bash
helm rollback <nome-release>

# Em um namespace especifico:
helm rollback <nome-release> --namespace <nome-namespace>
```

### **Primeiro Helm Chart**

Para criarmos um template de Helm, executamos o comando 
```bash
helm create <nome-chart>
```

Ele irá criar uma pasta com o nome definido no `create`:

![](../imagens/primeiro-helm.png)

#### **Criando um template de mongo usando helm:**

Criar o arquivo abaixo com o nome desejado porem ele deverá estar dentro da pasta "Templates".
Pode ser criado vários arquivos separados, para criar os objetos no cluster, como service, hpa entre outros.

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-mongodb-deployment
spec:
  selector:
    matchLabels:
      # Tem que ser o mesmo valor declarado no POD.
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
        - name: {{ .Release.Name }}-container
          image: mongodb:{{ .Values.mongodb.tag }}
          ports:
          - containerPort: 27017
          env:
          - name: QUARKUS_DATASOURCE_USERNAME
            value: {{ .Values.mongodb.credenciais.userName }}
          - name: QUARKUS_DATASOURCE_PASSWORD
            value: {{ .Values.mongodb.credenciais.userPassword }}
```

Adicionar no arquivo `values.yml` os valores.

``` yaml
mongodb:
  tag: 4.2.8
  credenciais:
    userName: "meu_usuario"
    userPassword: "minha_senha"
```

Após os ajustes acima para validarmos se será criado os objetos corretamente sem executar no cluster pode ser usado o comando abaixo:

```bash
helm install <nome-desejado> ./<diretorio-templates> --dry-run --debug

#Com output para um arquivo
helm install <nome-desejado> ./<diretorio-templates> --dry-run --debug > arquivo.yml
```

Para instalar podemos rodar o comando:
Lembrando que o deployment ficara com o nome passado que foi definido \<nome-desejado> 

```bash
helm install <nome-desejado> <diretorio-helm> 
```

#### **Estrutura if/else**

Podemos utilizar operadores condicionais no template.