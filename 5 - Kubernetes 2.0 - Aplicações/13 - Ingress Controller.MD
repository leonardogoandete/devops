# Ingress Controller
Um Ingress Controller é um componente essencial em um cluster Kubernetes que gerencia o tráfego de entrada para os serviços dentro do cluster. Ele atua como um ponto de entrada para solicitações externas, roteando-as para os serviços apropriados com base em regras definidas.

## O que é um Ingress?
Um Ingress é um recurso do Kubernetes que define como o tráfego externo deve ser roteado para os serviços dentro do cluster. Ele permite que você configure regras de roteamento, como URLs e domínios, para direcionar o tráfego para diferentes serviços.


## Configuração de um Ingress Controller Localmente
Para configurar um Ingress Controller localmente, você pode usar o kind (Kubernetes IN Docker) para criar um cluster Kubernetes local. A seguir estão os passos básicos para configurar um Ingress Controller usando o NGINX como exemplo:

1. **Criar um Cluster com kind**:
   Primeiro, crie um arquivo de configuração para o kind que inclua a configuração do Ingress Controller.

   ```yaml
   # kind-config.yaml
   kind: Cluster
   apiVersion: kind.x-k8s.io/v1alpha4
   nodes:
   - role: control-plane
     extraPortMappings:
     - containerPort: 30000
       hostPort: 80
       listenAddress: "0.0.0.0"
       protocol: TCP
   - role: control-plane
   - role: control-plane
   - role: worker
   - role: worker
   - role: worker
   ```

   Em seguida, crie o cluster usando o comando:

   ```bash
   kind create cluster --config kind-config.yaml
   ```
2. **Instalar o Ingress Controller NGINX**:
    Use o Helm para instalar o Ingress Controller NGINX no cluster:
    ```bash
    helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
    helm repo update
    helm install ingress-nginx ingress-nginx/ingress-nginx
    ```
3. **Verificar a Instalação**:
   Verifique se o Ingress Controller está em execução:
    ```bash
    kubectl get pods -n ingress-nginx
    ```
4. **Criar um Ingress Resource**:

    Crie um arquivo YAML para definir um recurso Ingress que roteia o tráfego para um serviço específico:
    ```yaml
    # ingress-resource.yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: example-ingress
      annotations:
         nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
      rules:
      - host: example.local
         http:
            paths:
            - path: /
              pathType: Prefix
              backend:
                service:
                  name: example-service
                  port:
                    number: 80
    ```
    Aplique o recurso Ingress:
    ```bash
    kubectl apply -f ingress-resource.yaml
    ```
5. **Configurar o arquivo hosts**:
   Adicione uma entrada no arquivo hosts do seu sistema para mapear o domínio `example.local` para o endereço IP do cluster kind (geralmente `127.0.0.1`):

   ```
   127.0.0.1 example.local
   ```
6. **Testar o Ingress**:
    Agora você pode testar o Ingress acessando `http://example.local` no seu navegador ou usando curl:
    ```bash
    curl http://example.local
    ```

## Documentação Adicional
- [Ingress NGINX](https://kubernetes.github.io/ingress-nginx/)


## Ingress Simples
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-blue
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-blue
  template:
    metadata:
      labels:
        app: web-blue
    spec:
      containers:
      - name: web-blue
        image: fabricioveronez/web-color:blue
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: web-blue
spec:
  selector:
    app: web-blue
  ports:
  - port: 80
    targetPort: 80
```

- Ingress para rotear o tráfego para o serviço web-blue:

```
apiVersion: networking.k8s.io/v1
kind: Ingress   
metadata:
  name: web-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-blue
            port:
              number: 80
```


### Ingress basado em path
- Deploy de duas aplicações web (blue e green) com seus respectivos Services:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-blue
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-blue
  template:
    metadata:
      labels:
        app: web-blue
    spec:
      containers:
      - name: web-blue
        image: fabricioveronez/web-color:blue
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: web-blue
spec:
  selector:
    app: web-blue
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-green
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-green
  template:
    metadata:
      labels:
        app: web-green
    spec:
      containers:
      - name: web-green
        image: fabricioveronez/web-color:green
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: web-green
spec:
  selector:
    app: web-green
  ports:
  - port: 80
    targetPort: 80
```

- Ingress para rotear o tráfego baseado no caminho:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress   
metadata:
  name: web-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /blue
        pathType: Prefix
        backend:
          service:
            name: web-blue
            port:
              number: 80
      - path: /green
        pathType: Prefix
        backend:
          service:
            name: web-green
            port:
              number: 80
```

Com essa configuração, ao acessar `http://<INGRESS_IP>/blue`, você será direcionado para a aplicação web azul, e ao acessar `http://<INGRESS_IP>/green`, você será direcionado para a aplicação web verde.

## Prefix vs Exact Path
No Kubernetes Ingress, os tipos de caminho "Prefix" e "Exact" determinam como as solicitações são roteadas com base no caminho da URL.

- **Prefix**: Quando você usa o tipo de caminho "Prefix", qualquer solicitação que comece com o caminho especificado será roteada para o backend correspondente. Por exemplo, se você definir um caminho como "/app" com o tipo "Prefix", as solicitações para "/app", "/app/page1", "/app/page2", etc., serão todas roteadas para o mesmo serviço backend.

- **Exact**: Quando você usa o tipo de caminho "Exact", apenas as solicitações que correspondem exatamente ao caminho especificado serão roteadas para o backend correspondente. Por exemplo, se você definir um caminho como "/app" com o tipo "Exact", apenas as solicitações para "/app" serão roteadas para o serviço backend, enquanto solicitações para "/app/page1" ou "/app/page2" não serão roteadas para esse serviço.

Link para a documentação:
https://kubernetes.io/docs/concepts/services-networking/ingress/#examples


## Default Backend
Um Default Backend em um Ingress Controller é um serviço que lida com solicitações que não correspondem a nenhuma das regras definidas no recurso Ingress. Ele atua como um fallback para garantir que todas as solicitações recebidas pelo Ingress Controller sejam tratadas, mesmo que não haja uma correspondência específica para o caminho ou host solicitado.

### Configuração de um Default Backend

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: default-backend
spec:
    replicas: 1
    selector:
        matchLabels:
            app: default-backend
    template:
        metadata:
            labels:
                app: default-backend
        spec:
            containers:
            - name: default-backend
                image: nginx
                ports:
                - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
    name: default-backend
spec:
    selector:   
        app: default-backend
    ports:
    - port: 80
      targetPort: 80
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
    ingressClassName: nginx
    name: example-ingress
    annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
spec:
    defaultBackend:
        service:
            name: default-backend
            port:
                number: 80
``` 

## Ingress baseado em Dominio

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-blue
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-blue
  template:
    metadata:
      labels:
        app: web-blue
    spec:
      containers:
      - name: web-blue
        image: fabricioveronez/web-color:blue
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: web-blue
spec:
  selector:
    app: web-blue
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-green
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-green
    template:
      metadata:
        labels:
          app: web-green
      spec:
        containers:
        - name: web-green
          image: fabricioveronez/web-color:green
          ports:
          - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: web-green
spec:
  selector:
    app: web-green
  ports:
  - port: 80
    targetPort: 80
```
- Ingress para rotear o tráfego baseado no domínio:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: blue.example.local
    http:
      paths:
      - path: /blue
        pathType: Prefix
        backend:
          service:
            name: web-blue
            port:
              number: 80
    - host: green.example.local
      http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: web-green
              port:
                number: 80
```