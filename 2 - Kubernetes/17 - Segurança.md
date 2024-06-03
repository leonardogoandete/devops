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

Exemplo de um arquivo de NetworkPolicy:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: nginx-policy
spec:
  podSelector:
    matchLabels:
      app: teste
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
      - podSelector:
          matchLabels:
            app: teste-cliente
    ports:
      - protocol: TCP
        port: 80
  
  egress:
  - to:
      - podSelector:
          matchLabels:
            app: teste-cliente
    ports:
    - protocol: TCP
      port: 80
```

No exemplo acima todos os pods com a label `app: teste` vão ter regras de entrada e saída para a porta 80.

Na `spec` da policy definimos que tanto trafego de entrada e saída na porta 80 só vai ocorrer com pod que tiver com label `teste-cliente`.

Para validar podemos executar abaixo os seguintes comandos: 
Sem acesso, deverá dar time-out:
```bash
kubectl run -i --tty --image kubedevio/ubuntu-curl ping-test --restart=Never --rm -- curl http://nginx
```

Com acesso, deverá carregar a pagina do NGINX:
```bash
kubectl run -i --tty -l app=teste-cliente --image kubedevio/ubuntu-curl ping-test --restart=Never --rm -- curl http://nginx
```

Podemos aplicar bloqueios por `namespaces` e por range de ip, mais informações:
https://kubernetes.io/docs/concepts/services-networking/network-policies/

---

#### **ServiceAccount e RBAC**

