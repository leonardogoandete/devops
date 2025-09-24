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

Para o uso do Metrics Server, é necessário que ele esteja instalado no cluster. 
Você pode instalar o Metrics Server usando o seguinte comando:
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
O Metrics Server coleta métricas de uso de recursos dos nodes e pods em tempo real. Essas métricas podem ser visualizadas usando o comando `kubectl top`. 

Observação: O Metrics Server não armazena métricas historicamente, ele apenas fornece métricas em tempo real. Para armazenar métricas historicamente, você pode usar soluções como Prometheus.

### Resource Limits - CPU

A CPU em Kubernetes é medida em unidades chamadas "millicores". 1 CPU equivale a 1000 millicores. Por exemplo, se um container tem um limite de CPU de 500m, isso significa que ele pode usar até 50% de uma CPU.

Caso um node atinja o limite de CPU, o nó entrara em `NodePressure Eviction`, o que pode levar à eliminação de pods para liberar recursos, caso tenha outros nodes no cluster, o pod será agendado em outro node.

Para isso podemos definir limits de CPU para evitar que um container consuma todos os recursos do node.
Exemplo de definição de limites de CPU em um Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
    name: meu-pod
spec:
    containers:
    - name: meu-container
      image: nginx
      resources:
        limits:
          cpu: "500m"  # Limite de CPU
```

Quando a aplicação atinge o limite de CPU, o Kubernetes pode limitar o uso da CPU do container, o que pode resultar em desempenho reduzido, mas o container não será encerrado. Geralmente ocorre o "estrangulamento" (throttling) do uso da CPU.

### Resource Limits - Memória
A memória em Kubernetes é medida em bytes. Você pode especificar limites de memória usando sufixos como `Mi` (Mebibytes) e `Gi` (Gibibytes). Por exemplo, se um container tem um limite de memória de `256Mi`, isso significa que ele pode usar até 256 Mebibytes de memória.

Quando um container excede o limite de memória, ele pode ser encerrado pelo sistema operacional (OOMKilled) geralmente com o código de saída 137, para proteger o node e outros containers em execução. Para evitar isso, podemos definir limits de memória para garantir que um container não consuma mais memória do que o permitido.

Exemplo de definição de limites de memória em um Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
    name: meu-pod
spec:
    containers:
    - name: meu-container
      image: nginx
      resources:
        limits:
          memory: "256Mi"  # Limite de Memória
```
Caso um container atinja o limite de memória, ele será encerrado para evitar que o node fique sem memória disponível.

### Resource Requests
Os `requests` são a quantidade mínima de recursos que um container precisa para ser agendado em um node. Se um container não tiver recursos suficientes disponíveis no node, ele não será agendado.

Exemplo de definição de requests em um Pod:

```yaml
apiVersion: v1
kind: Pod   
metadata:
    name: meu-pod
spec:
    containers:
    - name: meu-container
      image: nginx
      resources:    
        requests:
          cpu: "250m"      # Request de CPU
          memory: "128Mi"  # Request de Memória
```

Caso um container não tenha recursos suficientes disponíveis no node, ele não será agendado até que recursos suficientes estejam disponíveis.

Se realizarmos um describe em um pod sem recursos disponíveis, os pods ficaram em status `Pending` e veremos uma mensagem de erro como:
```bash
0/1 nodes are available: 1 Insufficient cpu.
``` 

### QoS (Quality of Service)

O Kubernetes classifica os pods em três classes de QoS com base nas solicitações e limites de recursos:
- **Guaranteed**: Se todos os containers em um pod tiverem `requests` e `limits` definidos e ambos forem iguais, o pod será classificado como `Guaranteed`. Esses pods têm a mais alta prioridade e são os últimos a serem eliminados em caso de pressão de recursos no node.

- **Burstable**: Se um ou mais containers em um pod tiverem `requests` e `limits` definidos, mas não forem iguais, o pod será classificado como `Burstable`. Esses pods têm prioridade média e podem usar recursos adicionais se estiverem disponíveis, mas podem ser eliminados antes dos pods `Guaranteed` em caso de pressão de recursos.

- **BestEffort**: Se nenhum container em um pod tiver `requests` ou `limits` definidos, o pod será classificado como `BestEffort`. Esses pods têm a mais baixa prioridade e são os primeiros a serem eliminados em caso de pressão de recursos no node.

### Limit Ranges
Os Limit Ranges são políticas que podem ser aplicadas a um namespace para definir limites padrão e mínimos para os recursos dos pods e containers. Isso ajuda a garantir que os pods não consumam recursos excessivos e que os recursos sejam distribuídos de maneira justa entre os pods.

Para criar um Limit Range Default, você pode usar o seguinte manifesto YAML:
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: limite-padrao   
  namespace: desenvolvimento # Podemos aplicar em um namespace específico
spec:
  limits:
  - default: # Resources Limits Padrão
      cpu: "500m"
      memory: "256Mi"
    defaultRequest: # Resources Requests Padrão
      cpu: "250m"
      memory: "128Mi"
    type: Container
```

Depois de aplicar esse manifesto, todos os pods criados no namespace `desenvolvimento` que não especificarem `requests` ou `limits` terão os valores padrão definidos no Limit Range aplicados automaticamente.

Podemos definir também limites mínimos e máximos para os recursos dos containers. Por exemplo:
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: limite-minimo-maximo
  namespace: desenvolvimento
spec:
  limits:
  - max: # Limites Máximos
      cpu: "1"
      memory: "512Mi"
    min: # Limites Mínimos
      cpu: "100m"
      memory: "64Mi"
    type: Container
```
Com esse Limit Range, qualquer container criado no namespace `desenvolvimento` deve ter pelo menos `100m` de CPU e `64Mi` de memória, e não pode exceder `1` CPU e `512Mi` de memória. Se um container for criado fora desses limites, o Kubernetes rejeitará a criação do pod.

Os logs do pod mostrarão mensagens de erro indicando que os recursos estão fora dos limites definidos no Limit Range.
Exemplo de mensagem de erro:
```bash
Error creating: pods "meu-pod" is forbidden: exceeded quota: limite-minimo-maximo, requested: cpu=50m,memory=32Mi, used: cpu=0,memory=0, limited: cpu=1,memory=512Mi
```

Podemos definir os Limits Ranges para diferentes types de recursos, como `Container`, `Pod` e `PersistentVolumeClaim`.

### Resource Quotas
As Resource Quotas são usadas para limitar a quantidade total de recursos que podem ser consumidos por todos os pods em um namespace. Isso ajuda a evitar que um único namespace consuma todos os recursos do cluster, garantindo uma distribuição justa dos recursos entre diferentes namespaces.

Os Resource Quotas podem limitar recursos como CPU, memória, número de pods, serviços, volumes persistentes, entre outros, ou seja de recursos computacionais e de objetos.

Como criar uma Resource Quota para recursos computacionais:
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota-computacional
  namespace: desenvolvimento
spec:
  hard:
    requests.cpu: "2"        # Limite total de Requests de CPU
    requests.memory: "1Gi"   # Limite total de Requests de Memória
    limits.cpu: "4"          # Limite total de Limits de CPU
    limits.memory: "2Gi"     # Limite total de Limits de Memória
```

Como criar uma Resource Quota para objetos:
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota-objetos
  namespace: desenvolvimento
spec:
  hard:
    pods: "10"                     # Limite total de Pods
    services: "5"                  # Limite total de Services
    persistentvolumeclaims: "5"    # Limite total de PVCs
    configmaps: "10"               # Limite total de Config
    secrets: "10"                  # Limite total de Secrets
```
