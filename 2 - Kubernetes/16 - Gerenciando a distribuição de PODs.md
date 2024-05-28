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

