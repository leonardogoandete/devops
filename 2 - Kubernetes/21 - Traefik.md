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
    
- Endpoint: podemos configurar um tipo de endpoint especifico, pode ser do tipo http, udp, tcp.

- Router:

- Middleware:

- Service:

