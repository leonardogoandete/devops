# Repositórios Privados
Em muitos casos, as aplicações que você deseja implantar em um cluster Kubernetes podem estar armazenadas em repositórios privados, seja no Docker Hub, GitHub Container Registry, Amazon ECR ou outros serviços de registro de contêineres. Para que o Kubernetes possa acessar essas imagens privadas, é necessário configurar o acesso adequado.

## Criando um Secret para Acesso ao Repositório Privado
Para permitir que o Kubernetes acesse as imagens privadas precisamos criar uma credencial de acesso, que será armazenada em um Secret do tipo `docker-registry`. Esse Secret conterá as informações necessárias para autenticar no repositório privado.

### Passo 1: Criar o Secret
Use o comando `kubectl create secret docker-registry` para criar o Secret. Substitua `<nome-do-secret>`, `<seu-usuario>`, `<seu-email>`, `<seu-token-ou-senha>` e `<url-do-registro>` pelos valores apropriados.

```bash
kubectl create secret docker-registry <nome-do-secret> \
  --docker-username=<seu-usuario> \
  --docker-password=<seu-token-ou-senha> \
  --docker-email=<seu-email> \
  --docker-server=<url-do-registro>
```

### Passo 2: Criar o Deployment Usando o Secret
Depois de criar o Secret, você pode referenciá-lo no manifesto do Deployment para que o Kubernetes use essas credenciais ao puxar a imagem do repositório privado.

Exemplo de manifesto de Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:   
  name: minha-aplicacao
spec:
  replicas: 2
  selector: 
    matchLabels:
      app: minha-aplicacao
  template:
    metadata:
      labels:
        app: minha-aplicacao
    spec:
      containers:
      - name: meu-container
        image: <url-do-registro>/<seu-usuario>/minha-aplicacao
        ports:
        - containerPort: 80
      imagePullSecrets:
      - name: <nome-do-secret>
```

