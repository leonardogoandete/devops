### **Traefik**

O Traefik é um tipo de ingress controller, como se fosse o Nginx Ingress Controller, porem possui elementos a mais, como route, middlewares.

O Traefik possui CDR mais amplo, pois o do kubernetes é utilizado muita anotations o que pode causar erros de configuração humana e muito verboso.

### Instalação:
Para utiliza-lo basta seguir as recomendações da url abaixo:

[Traefik - Artifact Hub](https://artifacthub.io/packages/helm/traefik/traefik)


1 - Adicionar o repositorio do helm no cluster:
```bash
helm repo add traefik https://traefik.github.io/charts
```

2 - Instalar o chart:
```bash
helm install my-traefik traefik/traefik --version 32.1.1
```

---

### Configuração:

1 - Precisamos ter um serviço jé em execução para poder expor essa rota no Traefik.

2 - Elementos para cria a rota no trefik, 
    
- Endpoint: podemos configurar um tipo de endpoint especifico, pode ser do tipo web, http, udp, tcp.

- Router: Conecta o `endpoint` aos services e tratamento das requisições, ele que adiciona os `middlewares`

- Middleware: Tratamentos aplicado ao `router` antes de encaminhar para o services, exemplo algum header.

- Service: O serviço em execução.

---

### Configurando a rota
1 - Primeiro precisamos criar um ingress route que representa a rota:

```yaml
apiVersion: traefik.contanio.us/v1alpha1
kind: IngressRoute
metadata:
  name: ingressroute
spec:
  entryPoints:
    - web
  routes:
  - match: Host(`meusite.com`)
    kind: Rule
    services:
    - name: nginx-service
      port: 80
```

Podemos validar usando o curl `curl http://meusite.com` se retornar a pagina do nginx, foi configurado com sucesso.

### Configurando Middleware
1 - Criar um middleware.

```yaml
apiVersion: traefik.contanio.us/v1alpha1
kind: Middleware
metadata:
  name: ratelimit
spec:
  rateLimit:
    average: 5
    period: 1s
```

Aplicar o middleware com o `kubectl apply -f` e depois vincular no `route`:

```yaml
apiVersion: traefik.contanio.us/v1alpha1
kind: IngressRoute
metadata:
  name: ingressroute
spec:
  entryPoints:
    - web
  routes:
  - match: Host(`meusite.com`)
    kind: Rule
    services:
    - name: nginx-service
      port: 80
    middleware:
      - name: rateLimit
```


