Essas estratégias de deploy são muito importantes para garantir que o deploy seja feito de forma segura e eficiente. A escolha da estratégia de deploy depende do ambiente e do tipo de aplicação que está sendo feita.

Basicamente quando precisamos trocar a roda do carro em movimento, temos que garantir que a roda nova esteja rodando antes de remover a roda antiga. Isso é o que chamamos de **deploy sem downtime**.

Vamos ver algumas estratégias de deploy:

### Recreate
Esse padrao é o unico que tera downtime, pois ele remove todos os pods antigos e somente quando removido com sucesso, ele irá cria novos pods com a nova versão. É o mais simples de ser implementado, mas não é recomendado para ambientes de produção.

É o mais ideal para ambientes de desenvolvimento e testes ou quando não é necessário garantir resposta continua para o usuário.

Para fazer isso no Kubernetes basta definir o campo `spec.strategy.type` como `Recreate` no arquivo de deployment.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
    replicas: 3
    selector:
        matchLabels:
        app: myapp
    template:
        metadata:
        labels:
            app: myapp
        spec:
        containers:
        - name: myapp
            image: myapp:latest
    strategy:
        type: Recreate
```
---

### Ramped
Essa estratégia é a mais comum e recomendada para ambientes de produção. Ela garante que a nova versão esteja rodando antes de remover a versão antiga.
Podemos definir o campo `spec.strategy.type` como `RollingUpdate` e definir o campo `spec.strategy.rollingUpdate.maxSurge` e `spec.strategy.rollingUpdate.maxUnavailable` para garantir que a nova versão esteja rodando antes de remover a versão antiga.

O que é `maxSurge` e `maxUnavailable`?

- `maxSurge`: Define o número máximo de pods adicionais que podem ser criados acima do número de pods desejados. Pode ser um número absoluto ou um percentual do número de pods desejados.

- `maxUnavailable`: Define o número máximo de pods que podem ser removidos abaixo do número de pods desejados. Pode ser um número absoluto ou um percentual do número de pods desejados.


Cuidados com os valores de `maxSurge` e `maxUnavailable`:

Caso o `maxUnavailable` for muito pode acabar comprometendo a disponibilidade do serviço, então é importante definir um valor que seja seguro para o ambiente.

Da mesma forma, se o `maxSurge` for muito alto, pode acabar comprometendo a capacidade do cluster, então é importante definir um valor que seja seguro para o ambiente.


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
    replicas: 3
    selector:
        matchLabels:
        app: myapp
    template:
        metadata:
        labels:
            app: myapp
        spec:
        containers:
        - name: myapp
            image: myapp:latest
    strategy:
        type: RollingUpdate
        rollingUpdate:
            maxSurge: 1
            maxUnavailable: 1
```
No exemplo acima estamos definindo no `maxSurge` que pode ter 1 pod a mais que o desejado que está definido como "3", e no `maxUnavailable` 1 pod a menos que o desejado.

Por padrão o `maxSurge` é 25% e o `maxUnavailable` é 25%.

---

### Blue/Green
Essa estratégia é muito utilizada para garantir que a nova versão esteja rodando antes de remover a versão antiga. Ela é muito utilizada para garantir que a nova versão esteja rodando antes de remover a versão antiga.

Basicamente o que fazemos é criar um novo ambiente com a nova versão e quando estiver tudo ok, trocamos o ambiente antigo pelo novo.

Nesse cenario é necessário ter o dobro de recursos, pois teremos dois ambientes rodando ao mesmo tempo.

Para fazer isso no kubernetes precisamos criar dois arquivos de deployment para aprendizagem (em ambiente real a versão v1 já estará deployada), um para a versão antiga e outro para a versão nova.

Deployment da versão antiga v1:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
    replicas: 3
    selector:
        matchLabels:
        version: v1
        app: myapp
    template:
        metadata:
        labels:
            version: v1
            app: myapp
        spec:
        containers:
        - name: myapp
            image: myapp:v1
---
apiVersion: apps/v1
kind: Service
metadata:
  name: myapp-service
spec:
    selector:
        app: myapp
        version: v1
    ports:
    - protocol: TCP
        port: 80
        targetPort: 80
    
```

Deployment da versão nova v2:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
    replicas: 3
    selector:
        matchLabels:
        version: v2
        app: myapp
    template:
        metadata:
        labels:
            version: v2
            app: myapp
        spec:
        containers:
        - name: myapp
            image: myapp:v2    
```

Teremos o deployment da versão antiga e da versão nova rodando ao mesmo tempo, e quando a versão nova estiver ok, podemos trocar o serviço para apontar para a versão nova.

Basta utilizar o comando `kubectl apply -f deployment-v2.yaml` para aplicar o deployment da versão nova, depois basta trocar o serviço para apontar para a versão nova usando o comando `kubectl patch service myapp-service -p '{"spec": {"selector": {"version": "v2"}}}'`.

Basicamente ajustamos o selector do serviço para apontar para a versão nova.

Depois de testar e validar a nova versão, podemos remover a versão antiga usando o comando `kubectl delete -f deployment-v1.yaml`.

---

### Canary

A ideia do canary é liberar a nova versão para um grupo de usuários antes de liberar para todos. Isso é muito utilizado para garantir que a nova versão está funcionando corretamente antes de liberar para todos.

Podemos definir que 10% dos usuários recebam a nova versão e depois de testar e validar, liberar para todos.

Para fazer isso no kubernetes precisamos criar dois arquivos de deployment para aprendizagem, um para a versão antiga e outro para a versão nova.

No exemplo abaixo vamos omitir o deployment da versão antiga, pois é o mesmo do exemplo anterior.

Deployment da versão nova v2:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
    replicas: 1
    selector:
        matchLabels:
        version: v2
        app: myapp
    template:
        metadata:
        labels:
            version: v2
            app: myapp
        spec:
        containers:
        - name: myapp
            image: myapp:v2    
---
apiVersion: apps/v1
kind: Service
metadata:
  name: myapp-service
spec:
    selector:
        app: myapp
    ports:
    - protocol: TCP
        port: 80
        targetPort: 80            
```



Vamos supor que temos 10 pods rodando a versão antiga, e queremos liberar a nova versão para 10% dos usuários.

Entao na versão antiga temos 9 pods rodando e na versão nova temos 1 pod rodando.

Para fazer isso atualizamos o deployment da versão antiga para ter 9 replicas e aplicamos o deployment da nova versão para ter 1 replica.

Apos isso vamos fazendo a troca gradualmente, aumentando o numero de replicas da nova versão e diminuindo o numero de replicas da versão antiga.

Exemplo:

- 8 replicas da versão antiga e 2 replicas da versão nova.
- 6 replicas da versão antiga e 4 replicas da versão nova.
- 4 replicas da versão antiga e 6 replicas da versão nova.
- 2 replicas da versão antiga e 8 replicas da versão nova.
- 0 replicas da versão antiga e 10 replicas da versão nova.

