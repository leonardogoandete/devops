# Gerenciamento de Ambiente e Recurso
Em um ambiente Kubernetes, é essencial gerenciar adequadamente os recursos e ambientes para garantir que as aplicações funcionem de maneira eficiente e segura. Isso inclui a criação de namespaces, a alocação de recursos e a configuração de políticas de segurança.

## Namespaces
Namespaces são uma forma de dividir um cluster Kubernetes em múltiplos ambientes virtuais. Eles permitem a organização de recursos e a aplicação de políticas específicas para diferentes equipes ou projetos.

### Criando um Namespace
Você pode criar um namespace usando o comando `kubectl create namespace`. Por exemplo, para criar um namespace chamado `desenvolvimento`:
```bash
kubectl create namespace desenvolvimento
```

### Listando Namespaces
Para listar todos os namespaces no cluster, use:
```bash
kubectl get namespaces  
```
### Usando Namespaces em Recursos
Ao criar recursos como Pods, Services ou Deployments, você pode especificar o namespace onde eles devem ser criados. Por exemplo, para criar um Pod no namespace `desenvolvimento`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: meu-pod     
  namespace: desenvolvimento
spec:
    containers:
    - name: meu-container
      image: nginx
```
### Definindo o Namespace Padrão
Você também pode definir um namespace padrão para o seu contexto atual usando o comando:
```bash
kubectl config set-context --current --namespace=desenvolvimento
```


