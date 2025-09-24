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

### Comunicação entre namespaces

Para fazer a comunicação entre namespaces, podemos usar o formato `<nome-do-servico>.<nome-do-namespace>.svc.cluster.local`. Por exemplo, se temos um serviço chamado `meu-servico` no namespace `desenvolvimento`, podemos acessá-lo a partir de outro namespace usando `meu-servico.desenvolvimento.svc.cluster.local`.

### Objetos Namespaced

Podem existir objetos que não são namespaced, ou seja, que não pertencem a um namespace específico. Exemplos desses objetos incluem:
- Nodes
- PersistentVolumes
- StorageClasses

Podemos consultar quais objetos são namespaced ou não usando o comando:
```bash
kubectl api-resources
```
O resultado mostrará uma coluna chamada `NAMESPACED` que indica se o recurso é namespaced (`true`) ou não (`false`).

## Resources

O Kubernetes permite a alocação de recursos como CPU e memória para os containers, garantindo que eles tenham os recursos necessários para funcionar corretamente.

Basicamente definimos limites e o minimo de recursos que um container pode usar. Isso ajuda a evitar que um container consuma todos os recursos do node, afetando outros containers.



### Resources - Metrics Server
O Metrics Server é um componente essencial para coletar métricas de uso de recursos (CPU e memória) dos nodes e pods em um cluster Kubernetes. Ele é usado por outros componentes do Kubernetes, como o Horizontal Pod Autoscaler, para tomar decisões baseadas em métricas.






