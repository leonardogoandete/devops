# Persistencia de Dados e Volume

#### Conceito de Stateless e Statefull
- Stateless: Aplicações que não armazenam dados entre sessões. Cada requisição é independente e não depende de dados anteriores.
- Stateful: Aplicações que armazenam dados entre sessões. Elas mantêm o estado e dependem de dados anteriores para funcionar corretamente.

#### Onde isso implica no Kubernetes?
- Em um ambiente Kubernetes, pods são efêmeros e podem ser reiniciados ou movidos entre nós. Portanto, se uma aplicação é stateful, é necessário garantir que os dados persistam mesmo que o pod seja reiniciado ou movido. Geralmente, mapeamos volumes persistentes (Persistent Volumes - PV) para armazenar esses dados.


### Gerenciamento de dados no Kubernetes
- **Volumes**: São diretórios acessíveis aos containers em um pod. Eles podem ser usados para armazenar dados temporários ou persistentes.
- **Persistent Volumes (PV)**: São recursos de armazenamento no cluster que foram provisionados por um administrador ou dinamicamente provisionados usando Storage Classes. Eles são independentes do ciclo de vida dos pods.
- **Persistent Volume Claims (PVC)**: São solicitações de armazenamento feitas pelos usuários. Um PVC pode solicitar um PV com certas características (tamanho, modo de acesso, etc.).
- **Storage Classes**: Permitem a definição de diferentes tipos de armazenamento (por exemplo, SSD, HDD) e políticas de provisionamento dinâmico.


## Volumes no Kubernetes

- Volume nada mais é do que um storage que é anexado ao pod, e o ciclo de vida do volume é o ciclo de vida do pod. Ele é plugado ao pod usando interfaces de volume.
- Normalmente o volume é externo ao cluster, ou seja, ele é um storage que está em outro lugar, e o Kubernetes faz a conexão com esse storage.
 

# Persistent Volume (PV)

- Um Persistent Volume (PV) é um objeto no cluster e forma uma representação de um pedaço de armazenamento(NAS por exemplo) no cluster que foi provisionado por um administrador ou dinamicamente provisionado usando Storage Classes.
- Podemos ter vários tipos de PVs, como NFS, iSCSI, GlusterFS, AWS EBS, GCE PD, Azure Disk, entre outros.
- Uma boa prática é usar labels para fazer a seleção do PV que o PVC deve usar e também usar PVC para fazer a ligação.

#### Como criar um PV?
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: meu-pv
  labels:
    type: nfs
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  volumeMode: Filesystem
  mountOptions:
    - hard
    - nfsvers=4
  nfs:
    path: /mnt/data # Caminho no servidor NFS
    server: nfs-server.example.com # Endereço do servidor NFS
```

O `accessModes` define como o volume pode ser montado pelos pods. Existem quatro modos principais:
- ReadWriteOnce: O volume pode ser montado como leitura e escrita por um único nó
- ReadWriteMany: O volume pode ser montado como leitura e escrita por muitos nós
- ReadOnlyMany: O volume pode ser montado como somente leitura por muitos nós
- ReadWriteOncePod(kubernetes 1.29+): O volume pode ser montado como leitura e escrita por um único pod

O `persistentVolumeReclaimPolicy` define o que acontece com o volume quando ele é liberado de um PVC. Pode ser Retain, Recycle ou Delete.
- Retain: O volume é mantido e precisa ser manualmente limpo.
- Recycle: O volume é reciclado (No NFS e HostPath).
- Delete: O volume é deletado.

O `volumeMode` define o modo do volume, que pode ser Filesystem ou Block.
- Filesystem: O volume é formatado com um sistema de arquivos, mais comum.
- Block: O volume é usado como um dispositivo de bloco bruto.

O `mountOptions` são opções específicas do sistema de arquivos que podem ser usadas ao montar o volume.

Tipos de status de um PV:
- Available: O volume está disponível para ser reclamado por um PVC.
- Bound: O volume está ligado a um PVC.
- Released: O volume foi liberado de um PVC, mas ainda não foi reciclado ou deletado.
- Failed: O volume falhou na provisão ou está inacessível.


# Persistent Volume Claim (PVC)

- Um Persistent Volume Claim (PVC) é uma solicitação de armazenamento feita por um usuário. Ele é usado para reclamar um PV com certas características (tamanho, modo de acesso, etc.). O PVC é ligado a um PV que satisfaz os requisitos do PVC.
- Uma boa pratica é usar labels para fazer a seleção do PV que o PVC deve usar e tambem criar Storage Class para provisionamento dinâmico.

Como criar um PVC?
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: meu-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem  
  resources:
    requests:
      storage: 5Gi
  storageClassName: "" # Nome da Storage Class, se estiver usando
  selector:
    matchLabels:
      type: nfs
```

# Storage Class

- Serve para classificar os tipos de armazenamento disponíveis no cluster. Ela permite a provisão dinâmica de volumes persistentes (PVs) com base nas necessidades dos usuários.
- Cada Storage Class define um provisionador, que é o plugin responsável por criar o volume físico. Exemplos de provisionadores incluem AWS EBS, GCE PD, Azure Disk, entre outros.

Como criar uma Storage Class?
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: meu-storage-class
provisioner: csi.example.com # Nome do provisionador
reclaimPolicy: Retain # Política de recuperação
allowVolumeExpansion: true # Permite expansão do volume
mountOptions:
  - discard
volumeBindingMode: WaitForFirstConsumer # Pode ser Immediate ou WaitForFirstConsumer
parameters:
  guaranteedReadWriteLatency: "true" # Parâmetros específicos do provisionador
allowTopologies:
  - matchLabelExpressions:
      - key: topology.kubernetes.io/zone
        values:
          - us-central1-a
          - us-central1-b
```

O `provisioner` define o plugin que será usado para provisionar o volume. Cada provisionador tem seus próprios parâmetros específicos.

O `reclaimPolicy` define o que acontece com o volume quando ele é liberado de um PVC. Pode ser Retain, Recycle ou Delete.

O `volumeBindingMode` define quando o volume será ligado ao PVC. Pode ser Immediate (imediatamente) ou WaitForFirstConsumer (espera até que um pod consuma o PVC).

O `parameters` são parâmetros específicos do provisionador que podem ser usados para configurar o volume.

O `allowTopologies` define as topologias onde o volume pode ser provisionado, como zonas ou regiões específicas.

# E como usamos o volume no Pod?

- Pode ser de duas formas:
1. Criado manualmente (PV e PVC)
2. Criado dinamicamente (Storage Class)

Exemplo criando um volume manualmente (PV e PVC) e usando no Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: meu-pod
spec:
  containers:
    - name: meu-container
      image: nginx
      volumeMounts:
        - mountPath: /mnt/data
          name: meu-volume
  volumes:
    - name: meu-volume
      nfs:
        server: nfs-server.example.com
        path: /mnt/nfs
        readOnly: true
```

Exemplo criando um volume dinamicamente (Storage Class) e usando no Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: meu-pod
spec:
  containers:
    - name: meu-container
      image: nginx
      volumeMounts:
        - mountPath: /mnt/data
          name: meu-volume
  volumes:
    - name: meu-volume
      persistentVolumeClaim:
        claimName: meu-pvc
```

# Exemplos de volumes

### Exemplo - HostPath
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-hostpath
spec:
  containers:
    - name: meu-container
      image: nginx
      volumeMounts:
        - mountPath: /usr/share/nginx
          subPath: html
          name: meu-volume
  volumes:
    - name: meu-volume
      hostPath:
        path: /mnt/data
        type: DirectoryOrCreate
```

### Exemplo - NFS Storage
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: db-pv-nfs
  labels:
    type: nfs
spec:
  capacity:
    storage: 4Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: nfs-static-storage
  nfs:
    server: nfs-server.example.com
    path: /mnt/nfs_shared
  mountOptions:
    - nfsvers=4.2
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem  
  resources:
    requests:
      storage: 4Gi
  storageClassName: nfs-static-storage
  selector:
    matchLabels:
      type: nfs
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-nfs
spec:
  containers:
    - name: meu-container
      image: nginx
      volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: db-vol
  volumes:
    - name: db-vol
      persistentVolumeClaim:
        claimName: db-pvc
```

### Exemplo - NFS Storage - Deploy com Volume (Digital Ocean)
- Basta alterar o `storageClassName` para o nome da Storage Class do Digital Ocean, que geralmente é `do-block-storage`. Não é necessário criar o PV, pois o Digital Ocean faz o provisionamento dinâmico.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem  
  resources:
    requests:
      storage: 4Gi
  storageClassName: do-block-storage # Nome da Storage Class do Digital Ocean
  selector:
    matchLabels:
      type: nfs
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-nfs
spec:
  containers:
    - name: meu-container
      image: nginx
      volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: db-vol
  volumes:
    - name: db-vol
      persistentVolumeClaim:
        claimName: db-pvc
```

# Limit Ranges
- Limit Ranges são políticas que podem ser aplicadas a namespaces para limitar os recursos que os pods e containers podem consumir. Eles ajudam a garantir que os recursos do cluster sejam usados de maneira eficiente e evitam que um único pod consuma todos os recursos disponíveis.

Exemplo de Limit Range:
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: storage-limit-range
  namespace: meu-namespace
spec:
  limits:
    - type: PersistentVolumeClaim
      max:
        storage: 10Gi
      min:
        storage: 1Gi
      default:
        storage: 5Gi
      defaultRequest:
        storage: 2Gi
```

# Resource Quota
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: storage-quota
  namespace: meu-namespace
spec:
  hard:
    persistentvolumeclaims: 5
    requests.storage: 20Gi
    limits.storage: 30Gi
```

# ConfigMap como Volume

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: meu-configmap # Nome do ConfigMap
data:
    meu-arquivo.conf: |
        chave1=valor1
        chave2=valor2
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-com-configmap
spec:
  containers:
    - name: meu-container
      image: nginx
      volumeMounts:
        - mountPath: /etc/config
          name: config-volume # Nome do volume definido abaixo
    volumes:
      - name: config-volume # Nome do volume
        configMap:
          name: meu-configmap # Nome do ConfigMap
          items:
            - key: meu-arquivo.conf
              path: meu-arquivo.conf
            defaultMode: 420 # Permissões do arquivo (opcional)
```

# Secret como Volume
Primeiro criamos o Secret:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: meu-secret
type: Opaque
data:
  meu-arquivo.conf: |
    chave1=valor1
    chave2=valor2
```
Depois usamos o Secret como Volume no Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-com-secret
spec:
  containers:
    - name: meu-container
      image: nginx
      volumeMounts:
        - mountPath: /etc/config
          name: secret-volume
  volumes:
    - name: secret-volume
      secret:
        secretName: meu-secret
        items:
          - key: meu-arquivo.conf
            path: meu-arquivo.conf
        defaultMode: 420 # Permissões do arquivo (opcional)
```