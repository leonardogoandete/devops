### **Cert Manager**

O Cert-manager é um gerenciador de certificado para o kubernetes, ele opera com os principais players do mercado. Como letsencrypt, venafi, vault e os auto assinados.

Ele nao só ajuda a genrenciar como pode fazer a renovação proximo da sua expiração.


### Arquitetura 
![Arquitetura do Cert Manager](https://cert-manager.io/images/high-level-overview.svg)



### Issuer e Cluster Issuer
Cluster issue:
É utilizado para reaproveitar o certificado para o cluster todo.

Issue:
É utilizado por namespaces separado.

### Certificates
É o certificado que sera usado e renovado, sempre que criamos ele tera junto um `certificate request` e um `Issuer/ClusterIssue`.

### Kubernetes Secret
Onde ficará as chaves TLS do certificado.

### Instalação
O processo de instalação é simples, basta instalar conforme o manual https://cert-manager.io/docs/installation 

### **Criando Issuers**

Para criar um Issue(objeto da entidade certificadora) basta utilizar o arquivo abaixo:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencript-homolog
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: email@email.com
    privateKeySecretRef:
      name: letsencrypt-homolog
    solvers:
      - http01:
          ingress:
            class: traefik # Aqui utilizar o tipo de ingress desejado nginx, traefik, haproxy...
```
Obs.: No exemplo acima ao final do processo a pagina vai dizer que tem um certificado não seguro apenas para teste, para usar um valido precisa alterar o server para o de produção.

>Homologação: https://acme-staging-v02.api.letsencrypt.org/directory

>Produção: https://acme-v02.api.letsencrypt.org/directory


Agora vamos criar o objeto Certificado para interagir com o ClusterIssue criado anteriormente e obter os certificados com a certificadora.

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: meu-certificado
spec:
  secretName: meu-certificado-secret
  issuerRef: 
    kind: ClusterIssuer
    name: <letsencript-homolog> #Utilizar o mesmo nome do ClusterIssuer
  dnsNames:
    - "meu-dominio.com"
```

Podemos validar se o certificado foi criado com sucesso usando o comando abaixo, a coluna `READY` deverá estar como `True`.

```bash
kubectl get certificates
```

Para utilizar o certificado no `Traefik` basta criar um `IngressRoute` com TLS conforme exemplo abaixo:

```yaml
apiVersion: traefik.contanio.us/v1alpha1
kind: IngressRoute
metadata:
  name: ingressroute-tls
spec:
  entryPoints:
    - websecure  # Definido para usar HTTPS
  routes:
  - match: Host(`meu-dominio.com`)
    kind: Rule
    services:
    - name: nginx-service
      port: 80
    middleware:
      - name: rateLimit
  tls:
    secretName: meu-certificado-secret # Utilizar o nome definido no certificado
```

