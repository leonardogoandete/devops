# Labels

Labels são pares chave-valor que podem ser associados a objetos do Kubernetes, como nós (nodes), pods, serviços, etc. Eles são usados para organizar e selecionar subconjuntos de objetos com base em critérios específicos. No contexto de nós (nodes), as labels podem ser usadas para identificar características específicas dos nós, como localização geográfica, tipo de hardware, ambiente (produção, desenvolvimento), etc.

## Gerenciamento de Labels em Nodes

O gerenciamento de labels em nodes envolve a adição, remoção e atualização de labels para refletir as características e o estado atual dos nós. Isso permite que os administradores do Kubernetes organizem e selecionem nós de maneira eficiente para agendamento de pods e outras operações.

### Adicionando Labels a Nodes

Para adicionar uma label a um node, você pode usar o comando `kubectl label`. Por exemplo, para adicionar uma label `environment=production` a um node chamado `node-1`, você pode usar o seguinte comando:

```bash
kubectl label nodes node-1 environment=production
```

## Removendo Labels de Nodes

Para remover uma label de um node, você pode usar o comando `kubectl label` com o sinal de menos (-) para indicar a remoção. Por exemplo, para remover a label `environment` do node `node-1`, você pode usar o seguinte comando:

```bash
kubectl label nodes node-1 environment-
```

# Node Selectors

Node selectors são uma forma de agendar pods em nós específicos com base nas labels associadas a esses nós. Ao definir um node selector em um manifesto de pod, você pode garantir que o pod seja agendado apenas em nós que correspondam às labels especificadas.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: meu-pod
spec:
  containers:
  - name: meu-container
    image: minha-imagem
  nodeSelector:
    environment: production
```

Neste exemplo, o pod `meu-pod` será agendado apenas em nós que tenham a label `environment=production`. Isso é útil para garantir que os pods sejam executados em ambientes específicos, como produção ou desenvolvimento, ou em nós com características específicas, como tipo de hardware ou localização geográfica.

# Node Affinity

Node affinity é uma forma mais avançada de agendamento de pods em nós com base em labels. Ele permite que você defina regras de afinidade para especificar quais nós são elegíveis para agendamento de pods, usando operadores como `In`, `NotIn`, `Exists`, etc.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: meu-pod
spec:
  containers:
  - name: meu-container
    image: minha-imagem
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: environment
            operator: In
            values:
            - production
```

Neste exemplo, o pod `meu-pod` será agendado apenas em nós que tenham a label `environment` com o valor `production`. O uso de node affinity permite uma maior flexibilidade na definição de regras de agendamento, permitindo que você especifique critérios mais complexos para a seleção de nós.

#### Tipos de Node Affinity:
- `requiredDuringSchedulingIgnoredDuringExecution`: O pod só será agendado em nós que atendam aos critérios especificados, mas se um nó deixar de atender aos critérios após o agendamento, o pod continuará a ser executado nesse nó.

- `preferredDuringSchedulingIgnoredDuringExecution`: O pod será preferencialmente agendado em nós que atendam aos critérios especificados, mas se nenhum nó atender aos critérios, o pod ainda poderá ser agendado em outros nós.

- `requiredDuringSchedulingRequiredDuringExecution`: O pod só será agendado em nós que atendam aos critérios especificados, e se um nó deixar de atender aos critérios após o agendamento, o pod será desprogramado e reprogramado em outro nó que atenda aos critérios.


#### Tipos de Operadores:
- `In`: O valor da chave deve estar em uma lista de valores especificada.
- `NotIn`: O valor da chave não deve estar em uma lista de valores especificada.
- `Exists`: A chave deve existir, independentemente do valor.
- `DoesNotExist`: A chave não deve existir.
- `Gt`: O valor da chave deve ser maior que um valor especificado (apenas para valores numéricos).
- `Lt`: O valor da chave deve ser menor que um valor especificado (apenas para valores numéricos).

Podemos definir peso para as regras de afinidade usando `preferredDuringSchedulingIgnoredDuringExecution`, onde podemos especificar a preferência de agendamento para nós que atendam aos critérios, mas ainda permitir que o pod seja agendado em outros nós se nenhum nó atender aos critérios.

```yamlapiVersion: v1
kind: Pod
metadata:
  name: meu-pod
spec:
  containers:
  - name: meu-container
    image: minha-imagem
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: environment
            operator: In
            values:
            - production
```

# Pod Affinity e Pod Anti-Affinity

Pod affinity e pod anti-affinity são mecanismos de agendamento que permitem que você defina regras para agendar pods com base na presença ou ausência de outros pods em nós específicos. Isso é útil para garantir que certos pods sejam agendados juntos (pod affinity) ou separados (pod anti-affinity) com base em critérios específicos, como labels.
