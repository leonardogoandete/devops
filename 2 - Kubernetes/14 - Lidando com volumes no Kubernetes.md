
Aplicações statefull necessitam armazenar os dados de forma persistente, por exemplo banco de dados, e para resolvermos a questão da persistência que sabemos que os containers são efemeros, temos o docker volume para guardar essas informações.

Temos diversos tipos de volumes: 
Documentação: https://kubernetes.io/docs/concepts/storage/volumes/

Para representar o objeto volume dentro do cluster existe o Persistent Volume (pv).

Exemplo de arquivo utilizando o `PersistentVolume`:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

Agora para vincularmos o POD com o volume, precisamos de um `PersistentVolumeClaim`, ele solicita o uso do volume.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

Este modo é ruim pois os volumes são montados em nós do kubernetes, então se ele for trocado ou excluído, ele será perdido.