#### **Node Selector**

Podemos definir em qual nó do cluster o container vai executar fixamente através das labels.

Primeiro precisamos definir o label no nó do custer:
```bash
kubectl label node <nome-do-no> database=mongodb
```

Depois definimos no deployment do POD o `nodeSelector`:
```yaml
nodeSelector:
  database: mongodb
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cadastro-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      # Tem que ser o mesmo valor declarado no POD.
      app: cadastro
  template:
    metadata:
      labels:
        app: cadastro
    spec:
      nodeSelector:
        database: mongodb
      containers:
        - name: cadastro
          image: leogoandete/cadastro:latest
```

Essa definição de relacionamento de nodeSelector com o nó, pois caso o nó é excluído ou não há o nó com a definição da label o pod não vai ser criado. Ou seja perde a resiliência das informações.

---

#### **Node Affinity**

Com o Affinity podemos aplicar o mesmo conceito do Node Selector, porem vamos ter preferencia por usar um label, se ele n'ao existir ele vai criar mesmo assim o deployment.

A afinidade do nó é conceitualmente semelhante a `nodeSelector`, permitindo restringir em quais nós seu pod pode ser programado com base nos rótulos dos nós. Existem dois tipos de afinidade de nó:

- `requiredDuringSchedulingIgnoredDuringExecution`: o agendador não pode agendar o pod a menos que a regra seja atendida. Funciona como `nodeSelector`, mas com uma sintaxe mais expressiva.
- `preferredDuringSchedulingIgnoredDuringExecution`: o escalonador tenta encontrar um nó que atenda à regra. Se um nó correspondente não estiver disponível, o agendador ainda agendará o pod.

Mais detalhes: https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#node-affinity

---

