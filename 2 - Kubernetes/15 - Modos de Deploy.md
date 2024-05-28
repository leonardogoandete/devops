#### **StatefulSet**

Muito utilizado com escalabilidade que contenha volume, exemplo banco de dados ou mensageria. Ele escala pods de forma ordenada e sua identidade se mantem fixa para cada pod.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  minReadySeconds: 10 # by default is 0
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: registry.k8s.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
```

Mais informações: https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/

---
#### **DaemonSet**

Ele garante a replicação de PODs baseado no números de nós existente no cluster kubernetes. Se tivermos 5 workers, vamos ter 5 replicas do pod e um em cada nó. É muito usado em infraestrutura e monitoramento para coletas de log, metricas entre outros.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # these tolerations are to have the daemonset runnable on control plane nodes
      # remove them if your control plane nodes should not run pods
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
```

Mais informações: https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/

---
#### **Job e CronJob**

Temos algumas aplicações que rodam rotinas, muito usado em batch. Para isso o kubernetes tem recursos para suportar essas aplicações.

Para isso temos o JOB, que executa os processos, e ele é acompanhado do CRONJOB que faz a mesma coisa mas de forma agendado. Basicamente ele cria JOB baseado no intervalo de tempo passado.

**Jobs**

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: roleta-job
spec:
  template:
    spec:
      containers:
        - name: job
          image: kubedevio/roleta
      restartPolicy: Never
```

Por padrão o JOB no kubernetes tenta rodar 5 vezes caso de erro nas 5 ele entra em `BackoffLimitExceeded` e o job não executa mais.

Outro parâmetro que podemos incluir é o `parallelism` para definir quantos jobs podem ser executado em paralelo.

O parâmetro `completions` define o numero de JOB que tem que ter sucesso para definir como sucesso.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: roleta-job
spec:
  backoffLimit: 20
  parallelism: 2
  completions: 3
  template:
    spec:
      containers:
        - name: job
          image: kubedevio/roleta
      restartPolicy: Never
```

Como vimos acima o JOB ele executa uma vez com sucesso e nunca mais inicializa, pois não há agendamento, para isso vamos usar o cronjob.

Mais detalhes: https://kubernetes.io/docs/concepts/workloads/controllers/job/

----
**CronJob**

No exemplo acima não teremos a execução do job novamente, por exemplo toda a sexta-feira. Porem podemos resolver isso com o `CronJob` e utiliza como padrão o formato cron do linux, onde temos cinco asteriscos `*` representando um período de tempo diferente. 

Exemplo de `cron`:
```bash
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday)
# │ │ │ │ │                             OR sun, mon, tue, wed, thu, fri, sat
# │ │ │ │ │ 
# │ │ │ │ │
# * * * * *
```

Podemos usar o site https://crontab.guru/ ou https://crontab-generator.org/ para criar um CRON desejado.

Exemplo do `Cronjob`:

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: roleta-job
spec:
  schedule: "*/5 * * * *"
  jobTemplate:
    spec:
      backoffLimit: 20
      parallelism: 2
      completions: 3
      template:
        spec:
          containers:
            - name: job
              image: kubedevio/roleta
          restartPolicy: Never
```

No `schedule` abaixo foi definido que ele irá executar a cada 5 minutos, todas as horas, todos os dias do mês, todos os meses e todos os dias da semana.
```
"*/5 * * * *" 
```

Pode surgir de nosso job estar executando, mais já deu os 5 minutos para executar o próximo job para isso podemos definir com o parâmetro `concurrencyPolicy` .

Mais detalhes: https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/