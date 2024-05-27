#### **Resource Request e Resource Limits**

Recurso super importante que protege o cluster e a aplicação, é Request e Limitis.

Quando é configurado o Resource, temos que definir o **request** que é o que defini o minimo de recurso necessário para executar a aplicação, isso é o garantido para executar a aplicação, por isso o termo "proteger a aplicação".

Quando definimos o **limits** já estamos protegendo o cluster, por exemplo temos uma api que tem um bug, e utilize recursos sem parar, inclusive impactando outras api. Com o **limits** garantimos o máximo para ele não ultrapassar o valor.

Exemplo de configuração de resouces:
```yaml
...
template:
    metadata:
      labels:
        app: cadastro
    spec:
      containers:
        - name: cadastro
          image: leogoandete/cadastro:latest
          resources:
            limits:
              memory: "500Mi"
              cpu: "300m"
            requests:
              memory: "300Mi"
              cpu: "100m"
...
```

Obs.: O request é garantido para o POD. O limits ele não é garantido, somente se o cluster tiver recursos para disponibilizar. O ideal é deixar o container sempre com uso abaixo do limitis.


#### **HPA** (Horizontal Pod Autoscaler)

No resource podemos configurar outra funcionalidade, o HPA, ele nada mais é que a escalabilidade elástica do kubernetes automático, baseado nos usos de recursos.
O HPA pode usar mais parâmetros, como memória entre outros.

Podemos habilitar o HPA com manifesto yaml ou via linha de comando, no exemplo abaixo vamos configurar o HPA baseado em CPU.

```
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: cadastro-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: cadastro-deployment
  minReplicas: 1
  maxReplicas: 4
  targetCPUUtilizationPercentage: 50
```

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: cadastro
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: cadastro-deployment
  minReplicas: 1
  maxReplicas: 3
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```
O `targetCPUUtilizationPercentage`/`averageUtilization` foi definido com 50% do uso de `CPU` declarado no requests do deployment, ele irá escalar. 

Ou seja um deployment com CPU Requests de "100m" quando o uso chegar em "50m" ele vai criar um novo POD.


#### **QOS** (Kubernetes Quality of Service)
Os níveis de QOS é para informar o quanto o Kubernetes irá se preocupar para manter o pod ativo.

Quando o Kubernetes cria um Pod, ele atribui uma dessas classes de QoS ao Pod:

`Guaranteed` quando definimos o **resquests** e **limits** iguais.
Quando o kubernetes necessitar eliminar pods o tipo Garantido, será um dos últimos a ser excluído.

`Burstable` Será colocado uma prioridade mais baixa, ele é o meio termo. Para definir ele precisamos do **requests** e **limits** declarado e não tendo valores iguais.

`BestEffort` Será o primeiro a ser baixado. Para definir bastar não incluir o **requests** e **limits**.

### **LimitRange**
Temos o cenário onde não é definido os resources, podemos criar um padrão, ele sera o default para todos os itens criado após o default ser aplicado, caso tenha componentes antigos ele tem que se "recriado" para pegar estes valores.
Caso eu defina um deployment com o resources excedendo o default do limits ele não vai criar o recurso.

O LimitRange pode ser do tipo `Pod`,`Container` e `volume`.

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limit-container
spec:
  limits:
    - type: Container
      max:
        cpu: "800m"
        memory: "1024Mi"
      min:
        cpu: "200m"
        memory: "256Mi"
      default:
        cpu: "200m"
        memory: "256Mi"
      defaultRequest:
        cpu: "200m"
        memory: "256Mi"
```

### **Resource Quota**

Podemos querer limitar os recursos a nível de namespaces. Ele serve para cobrir o gap deixado pelo LimitRange, pois com limite a nivel de POD, basta criarmos mais PODs para roubar recursos.

Com o Resource Quota definimos para cada namespaces quanto terá de consumo nele.

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resource
spec:
  hard:
    requests.cpu: "2"
    requests.memory: "2Gi"
    limits.cpu: "3"
    limits.memory: "4Gi"
```

No exemplo acima, caso eu crie um deployment e queira varias replicas, o limite de CPU que todas as replicas somadas vai ter é 2 CPUs. 
Podemos ter 10 replicas com 200m de CPU cada, totalizando 2000m(2 CPU).