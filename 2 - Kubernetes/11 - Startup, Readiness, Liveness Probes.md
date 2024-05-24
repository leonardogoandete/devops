Documentação: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#use-a-named-port

Com o Kubernetes conseguimos ter escalabilidade e resiliência utilizando o `ReplicaSet`.

Mas temos também temos recursos de `Self Healing`, o Kubernetes consegue auto recuperar algum processo caso de algum problema, basicamente ele restarta o contêiner dentro do pod.

Para fazer isso precisamos ensinar o Kubernetes, qual parâmetro ele deverá ter e quando dizer quando a aplicações estiver pronta.

Estes cuidados é muito importante ter, para usar o auto desempenho do Kubernetes.

Para isso precisamos configurar 3 recursos fundamentais: 

`Startup Probe`: utilizado em contêineres que iniciam de forma demorada, caso não esteja configurado o `Liveness` vai sempre reiniciar o pod, nunca estando inicializado.

`Readiness Probe`: ele verifica se o contêiner esta pronto para receber requisições. Geralmente quando processando algo e não pode atender requisições.
Quando o `Liveness` está em condições, porem o `Readiness` estando com problema, ele simplesmente não vai ter um endpoind registrado no service.

`Liveness Probes`: utilizado valida a saúde do contêiner, ele irá sempre chamar o endpoint **`/health`** , caso a saúde não esteja saudável, o pod irá ficar reiniciando até ficar saudável.

A Seguir veremos a configuração de cada probe.
podemos utilizar vários tipos de teste em cada um deles, sendo eles o `httpGet`,`tcpSocket`,`exec`, `gRPC`.

---
**Configurando Liveness Probe**

Adicionamos as seguintes informações na declaração do contêiner:
```yaml
containers:
...
- name: api-exemplo
  livenessProbe:
    httpGet:
      path: /q/health
      port: 8080
      scheme: HTTP
    initialDelaySeconds: 10 # Tempo para iniciar o teste apos o container iniciar  
    periodSeconds: 15 # Periodo de tempo para cada teste
    timeoutSeconds: 10 # tempo limite para espera da resposta
    failureThreshold: 3 # Numero de erros permitido, antes de reiniciar o pod
    successThreshold: 1 # Numero de sucessos para considerar verdadeiro
```

---
**Configurando Readiness Probe**
Adicionamos as seguintes informações na declaração do contêiner:
```yaml
containers:
...
- name: api-exemplo
  readinessProbe:
    httpGet:
      path: /q/health/ready
      port: 8080
      scheme: HTTP
    initialDelaySeconds: 10 # Tempo para iniciar o teste apos o container iniciar  
    periodSeconds: 15 # Periodo de tempo para cada teste
    timeoutSeconds: 10 # tempo limite para espera da resposta
    failureThreshold: 3 # Numero de erros permitido, antes de reiniciar o pod
```

**Configurando Startup**
Adicionamos as seguintes informações na declaração do contêiner:
```yaml
containers:
...
- name: api-exemplo
  startupProbe:
    httpGet:
      path: /q/health/started
      port: 8080
      scheme: HTTP
    initialDelaySeconds: 10 # Tempo para iniciar o teste apos o container iniciar  
    periodSeconds: 15 # Periodo de tempo para cada teste
    timeoutSeconds: 10 # tempo limite para espera da resposta
    failureThreshold: 3 # Numero de erros permitido, antes de reiniciar o pod
```
