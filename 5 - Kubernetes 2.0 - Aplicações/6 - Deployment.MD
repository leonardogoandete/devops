### Deployment

### O que é um Deployment?
Deployment é um controlador do Kubernetes que gerencia a criação e atualização de Pods e ReplicaSets. Ele permite que você defina o estado desejado da sua aplicação, como o número de réplicas, a imagem do container e as configurações de rede. O Deployment garante que o estado atual da aplicação corresponda ao estado desejado, realizando atualizações contínuas e rollback fácil em caso de falhas.

O Deployment é usado para implantar e gerenciar aplicações no Kubernetes, permitindo que você execute várias versões de uma aplicação em paralelo, faça atualizações sem tempo de inatividade e reverta para versões anteriores se necessário. Ele é uma abstração de nível mais alto que facilita o gerenciamento de aplicações complexas no Kubernetes.   


### Como criar um Deployment?
Para criar um Deployment, você pode usar um arquivo de configuração YAML. 

Lembrando que nao precisamos criar o ReplicaSet, pois o Deployment já cria e gerencia o ReplicaSet para nós.

Aqui está um exemplo básico de um Deployment que mantém 3 réplicas de um Pod com a imagem `nginx:latest`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-container
          image: nginx:latest
          ports:
            - containerPort: 80
```
Neste exemplo, estamos criando um Deployment chamado `my-deployment` que mantém 3 réplicas de um Pod com a label `app: my-app`. O template do Pod define o container que será executado, que neste caso é o `nginx:latest` na porta 80.
Para criar o Deployment, você pode usar o comando `kubectl apply -f <nome-do-arquivo>.yaml`. Isso criará o Deployment no cluster Kubernetes e iniciará as réplicas do Pod conforme especificado.

### Como funciona o Deployment?
O Deployment funciona monitorando o estado dos Pods e ReplicaSets que ele gerencia. Ele verifica se o número de Pods em execução corresponde ao número desejado de réplicas e se a imagem do container corresponde à imagem especificada no template do Pod. Se algum Pod falhar ou for excluído, o Deployment cria um novo Pod para substituí-lo, garantindo que o número de réplicas permaneça constante.

O Deployment usa um seletor de labels para identificar quais Pods ele deve gerenciar. No exemplo acima, o seletor `matchLabels` especifica que o Deployment deve gerenciar todos os Pods com a label `app: my-app`. Isso permite que o Deployment mantenha o controle sobre os Pods que ele deve monitorar e gerenciar.

Além disso, o Deployment permite atualizações contínuas e rollback fácil em caso de falhas. Quando você atualiza a imagem do container ou outras configurações do Pod, o Deployment cria um novo ReplicaSet com as novas configurações e inicia a atualização dos Pods. Se a atualização falhar, você pode reverter facilmente para a versão anterior do Deployment usando o comando `kubectl rollout undo deployment/my-deployment`. Isso garante que sua aplicação permaneça disponível e resiliente, mesmo durante atualizações ou falhas.   

### Como ver o histórico de versões de um Deployment?
Para ver o histórico de versões de um Deployment, você pode usar o comando `kubectl rollout history deployment/<nome-do-deployment>`. Isso exibirá uma lista das revisões do Deployment, incluindo informações sobre cada revisão, como a data de criação e as alterações feitas.
Por exemplo, para ver o histórico do Deployment `my-deployment`, você pode usar o seguinte comando:
```bash
kubectl rollout history deployment/my-deployment
```
Isso mostrará uma lista das revisões do Deployment, permitindo que você veja as versões anteriores e as alterações feitas em cada uma delas. Você também pode usar o comando `kubectl describe deployment/<nome-do-deployment>` para obter informações detalhadas sobre o Deployment, incluindo o histórico de revisões e o estado atual dos Pods e ReplicaSets.


### Como fazer rollback de um Deployment?
Para fazer rollback de um Deployment, você pode usar o comando `kubectl rollout undo deployment/<nome-do-deployment>`. Isso reverte o Deployment para a versão anterior, restaurando o estado anterior dos Pods e ReplicaSets.

Por exemplo, para reverter o Deployment `my-deployment` para a versão anterior, você pode usar o seguinte comando:
```bash
kubectl rollout undo deployment/my-deployment
```
Isso fará com que o Deployment volte para a versão anterior, restaurando os Pods e ReplicaSets para o estado anterior à última atualização. O rollback é uma operação rápida e fácil, permitindo que você mantenha a disponibilidade da sua aplicação mesmo em caso de falhas durante atualizações. 

### Como voltar uma versão específica de um Deployment?
Para reverter um Deployment para uma versão específica, você pode usar o comando `kubectl rollout undo deployment/<nome-do-deployment> --to-revision=<número-da-revisão>`. Isso reverte o Deployment para a revisão especificada, restaurando o estado dos Pods e ReplicaSets para a versão correspondente.
Por exemplo, para reverter o Deployment `my-deployment` para a revisão 2, você pode usar o seguinte comando:
```bash
kubectl rollout undo deployment/my-deployment --to-revision=2
```
Isso fará com que o Deployment volte para a revisão 2, restaurando os Pods e ReplicaSets para o estado correspondente a essa revisão. Essa funcionalidade é útil quando você precisa reverter para uma versão específica do Deployment, em vez de apenas reverter para a última versão anterior.    