## Formas de deploy

# DaemonSet
Um DaemonSet garante que todos (ou alguns) nós executem uma cópia de um Pod. Quando novos nós são adicionados ao cluster, o DaemonSet garante que os Pods sejam adicionados a esses nós. DaemonSets são frequentemente usados para tarefas de infraestrutura, como coleta de logs ou monitoramento.

Exemplo de YAML para um DaemonSet usando o prometheus-node-exporter:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: prometheus-node-exporter
    labels:
      app: prometheus-node-exporter
spec:
  selector:
    matchLabels:
      app: prometheus-node-exporter
  template:
    metadata:
      labels:
        app: prometheus-node-exporter
    spec:
      containers:
      - name: prometheus-node-exporter
        image: prom/node-exporter:v1.7.0
        ports:
        - containerPort: 9100
```

# Job
Um Job cria um ou mais Pods e garante que um número especificado deles termine com sucesso. Jobs são usados para tarefas de curta duração que precisam ser concluídas, como processamento em lote ou tarefas de manutenção.

Exemplo de YAML para um Job que executa um script simples:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: example-job
spec:   
  template:
    spec:
      containers:
      - name: example
        image: busybox
        command: ["sh", "-c", "echo Hello, Kubernetes! && sleep 30"]
      restartPolicy: Never
  backoffLimit: 4
```

# CronJob
Um CronJob cria Jobs em uma programação específica, semelhante a um cron no Linux. Eles são usados para tarefas recorrentes, como backups ou relatórios periódicos.
Exemplo de YAML para um CronJob que executa um Job a cada minuto:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: example-cronjob
spec:
  schedule: "*/1 * * * *" # Executa a cada minuto
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: example
            image: busybox
            command: ["sh", "-c", "date; echo Hello from the CronJob!"]
          restartPolicy: OnFailure
```

# StatefulSet
Um StatefulSet é usado para gerenciar aplicativos com estado que requerem identidades persistentes e armazenamento estável. Cada Pod em um StatefulSet tem um nome único e persistente, e o armazenamento associado a cada Pod é mantido mesmo que o Pod seja reiniciado ou movido para outro nó.
Exemplo de YAML para um StatefulSet simples:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: example-statefulset
spec:
  serviceName: "example-service"
  replicas: 3
  selector:
    matchLabels:
      app: example
  template:
    metadata:
      labels:
        app: example
    spec:
      containers:
      - name: example
        image: nginx
        ports:
        - containerPort: 80
  volumeClaimTemplates:
  - metadata:
      name: example-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

# Headless Service

