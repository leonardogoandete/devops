### Labels e Selectors

O Kubernetes utiliza **labels** e **selectors** para organizar e selecionar recursos dentro do cluster. Labels são pares chave-valor que podem ser atribuídos a objetos como Pods, Services, Deployments, etc. Elas permitem categorizar e identificar recursos de forma flexível.

### O que são Labels?
Labels são metadados que podem ser aplicados a objetos do Kubernetes. Elas ajudam a identificar e agrupar recursos de acordo com critérios específicos. Por exemplo, você pode usar labels para identificar a versão de uma aplicação, o ambiente (produção, desenvolvimento, teste) ou a equipe responsável pelo recurso.

### Exemplo de Labels
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
    version: v1
spec:
  containers:       
    - name: my-container
      image: nginx:latest
      ports:
        - containerPort: 80
```
Neste exemplo, o Pod `my-pod` possui duas labels: `app: my-app` e `version: v1`. Essas labels podem ser usadas para identificar o Pod em consultas e operações subsequentes.    

### O que são Selectors?
Selectors são expressões usadas para filtrar objetos com base em suas labels. Eles permitem que você selecione um subconjunto de recursos que atendem a critérios específicos. Os selectors são amplamente utilizados em operações como listagem, exclusão e atualização de recursos.   

### Exemplo de Selectors
Para selecionar Pods com base em labels, você pode usar o comando `kubectl get pods` com a opção `-l` (ou `--selector`). Por exemplo, para listar todos os Pods com a label `app=my-app`, você pode usar o seguinte comando:
```bash
kubectl get pods -l app=my-app
```
Isso retornará todos os Pods que possuem a label `app` com o valor `my-app`. Você também pode combinar múltiplos critérios usando operadores lógicos, como `,` (AND) e `!=` (NOT). Por exemplo:
```bash
kubectl get pods -l app=my-app,version!=v2
```
Este comando listará todos os Pods com a label `app=my-app`, mas excluirá aqueles que possuem a label `version` com o valor `v2`.       