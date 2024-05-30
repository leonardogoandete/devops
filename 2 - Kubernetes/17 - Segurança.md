O CNI padrão do KinD não tem suporte a Network Policy, então para usar o kind, precisamos criar ele sem o CNI default, apoś a criação instalamos o CNI desejado como o WaveNet.

Exemplo de arquivo para cluster sem CNI default do KinD:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  disableDefaultCNI: true
  podSubnet: 192.168.0.0/16
```

Instalar o CNI WaveNet:
```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

Há vários Network Policy Provider link para outras versões:
https://kubernetes.io/docs/tasks/administer-cluster/network-policy-provider/

---

#### **NetworkPolicy**

Os PODs por padrão se comunicam sem nenhuma restrição, porem precisamos limitar essas conexões, para fazer essa garantia de controle de comunicação é utilizado o Network Policy.

