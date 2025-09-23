# Recursos de Deploy no Kubernetes

Este topico ira abordar os recursos de deploy no Kubernetes para manter os serviços estaveis e com alta disponibilidade. Como self-healing, rolling updates, rollbacks, entre outros.


## Image Pull Policy

Esse recurso define quando o Kubernetes deve puxar (pull) a imagem do container a partir do repositório. Existem três opções principais:

- **Always**: O Kubernetes sempre puxa a imagem do repositório, mesmo que a imagem já esteja presente no nó. Isso é útil para garantir que você sempre tenha a versão mais recente da imagem, mas pode aumentar o tempo de inicialização do pod.
- **IfNotPresent**: O Kubernetes puxa a imagem do repositório apenas se ela não estiver presente no nó. Isso pode acelerar o tempo de inicialização do pod, mas pode levar a problemas se a imagem no nó estiver desatualizada.
- **Never**: O Kubernetes nunca puxa a imagem do repositório e sempre usa a imagem presente no nó. Isso pode ser útil em ambientes onde a conectividade com o repositório é limitada, mas pode levar a problemas se a imagem no nó estiver desatualizada.

Por padrão, o Kubernetes usa a política `IfNotPresent` se a tag da imagem for algo diferente de `latest`, e `Always` se a tag for `latest`.

## Restart Policy
A política de reinício define como o Kubernetes deve lidar com a reinicialização dos containers em um pod. Existem três opções principais:

- **Always**: O Kubernetes sempre reinicia o container se ele falhar ou for encerrado. Esta é a política padrão para pods gerenciados por controladores como Deployments, ReplicaSets e StatefulSets.
- **OnFailure**: O Kubernetes reinicia o container apenas se ele falhar (ou seja, retornar um código de saída diferente de zero). Se o container for encerrado com sucesso, ele não será reiniciado.
- **Never**: O Kubernetes nunca reinicia o container, independentemente de ele falhar ou ser encerrado com sucesso.

A política de reinício é definida no nível do pod e se aplica a todos os containers dentro do pod. No entanto, para pods gerenciados por controladores como Deployments, ReplicaSets e StatefulSets, a política de reinício é sempre `Always`, independentemente do valor especificado.

## Command e Args
No Kubernetes, `command` e `args` são usados para definir o comando e os argumentos que serão executados dentro de um container. Eles são especificados na seção `containers` do manifesto do pod.

- **command**: Esta é a lista de strings que define o comando a ser executado no container. Se `command` for especificado, ele substituirá o comando padrão definido na imagem do container (geralmente especificado no Dockerfile com a instrução `ENTRYPOINT`).

- **args**: Esta é a lista de strings que define os argumentos a serem passados para o comando. Se `args` for especificado, ele substituirá os argumentos padrão definidos na imagem do container (geralmente especificado no Dockerfile com a instrução `CMD`).

Se ambos `command` e `args` forem especificados, o Kubernetes executará o comando definido em `command` com os argumentos definidos em `args`.

Aqui está um exemplo de como usar `command` e `args` em um manifesto de pod:

```yaml
apiVersion: v1
kind: Pod   
metadata:
  name: exemplo-pod
spec:
  containers:
  - name: exemplo-container
    image: exemplo-imagem
    command: ["exemplo-comando"]
    args: ["arg1", "arg2"]
```

Neste exemplo, o container `exemplo-container` executará o comando `exemplo-comando` com os argumentos `arg1` e `arg2` quando o pod for iniciado.

Podemos executar diretameente pela linha de comando com o kubectl:

```bash
kubectl run exemplo-pod --image=exemplo-imagem --command -- exemplo-comando arg1 arg2
```

## Self-Healing
O Kubernetes possui um recurso de self-healing que permite que ele monitore o estado dos pods e tome medidas corretivas automaticamente quando um pod falha ou fica indisponível. Isso é feito com o uso de probes de liveness e readiness.

Os probes são verificações periódicas que o Kubernetes realiza para determinar se um container está saudável e pronto para receber tráfego. Existem três tipos principais de probes:

- **Startup Probe**: Verifica se a aplicação dentro do container iniciou corretamente. Se a startup probe falhar, o Kubernetes reiniciará o container. Isso é útil para aplicações que podem levar um tempo significativo para iniciar, garantindo que o Kubernetes não considere o container como falho antes que ele tenha tido a chance de iniciar completamente.

O Liveness e Readiness só são aplicados após a Startup Probe ser bem-sucedida.

- **Liveness Probe**: Verifica se o container está em execução. Se a liveness probe falhar, o Kubernetes reiniciará o container. Isso é útil para detectar e corrigir situações em que um container está em um estado de erro, mas ainda está em execução.

- **Readiness Probe**: Verifica se o container está pronto para receber tráfego. Se a readiness probe falhar, o Kubernetes removerá o pod do serviço até que ele esteja pronto novamente. Isso é útil para garantir que o tráfego não seja enviado para um container que ainda não está pronto para processá-lo. O container pode estar em execução, mas não estar pronto para aceitar solicitações.

Os probes podem ser configurados usando diferentes métodos, como HTTP GET, TCP Socket ou execução de comandos. Aqui está um exemplo de configuração de liveness e readiness probes em um manifesto de pod:

Exemplo de configuração de liveness e readiness probes com HTTP GET:
```yaml
apiVersion: v1
kind: Pod       
metadata:
  name: exemplo-pod
spec:
    containers:
    - name: exemplo-container
      image: exemplo-imagem
      livenessProbe:
        httpGet:
          path: /health
          port: 8080
        initialDelaySeconds: 5
        periodSeconds: 10
      readinessProbe:
        httpGet:
          path: /ready
          port: 8080
        initialDelaySeconds: 5
        periodSeconds: 10
```

Exemplo de configuração de liveness e readiness probes com execução de comando:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exemplo-pod
spec:
    containers:
    - name: exemplo-container
      image: exemplo-imagem
      livenessProbe:
        exec:
          command:
          - cat
          - /tmp/healthy
        initialDelaySeconds: 5
        periodSeconds: 10
      readinessProbe:
        exec:
          command:
          - cat
          - /tmp/ready
        initialDelaySeconds: 5
        periodSeconds: 10
```

Exemplo de configuração de liveness e readiness probes com TCP Socket:
```yaml
apiVersion: v1
kind: Pod
metadata:
    name: exemplo-pod
spec:
    containers: 
    - name: exemplo-container
      image: exemplo-imagem
      livenessProbe:
        tcpSocket:
          port: 8080
        initialDelaySeconds: 5
        periodSeconds: 10
      readinessProbe:
        tcpSocket:
          port: 8080
        initialDelaySeconds: 5
        periodSeconds: 10
``` 

Podemos refinar ainda mais as probes com os seguintes parametros:
- **initialDelaySeconds**: Tempo em segundos que o Kubernetes espera antes de iniciar a primeira verificação da probe. Isso é útil para dar tempo ao container para iniciar antes de começar a verificar sua saúde.
- **periodSeconds**: Intervalo em segundos entre as verificações da probe. Isso define com que frequência o Kubernetes verifica a saúde do container.
- **timeoutSeconds**: Tempo em segundos que o Kubernetes espera por uma resposta da probe antes de considerar que ela falhou. Isso é útil para evitar que probes fiquem pendentes por muito tempo.
- **successThreshold**: Número mínimo de verificações consecutivas bem-sucedidas necessárias para considerar que a probe está saudável. Isso é  útil para evitar flutuações temporárias na saúde do container.
- **failureThreshold**: Número mínimo de verificações consecutivas com falha necessárias para considerar que a probe falhou. Isso é útil para evitar flutuações temporárias na saúde do container.
