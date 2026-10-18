
Para conseguimos usar imagens docker privada no contêiner precisamos fazer um ajuste no deployment e criar uma secret para login no registry.

Primeiro criamos o secret:

```bash
kubectl create secret docker-registry acr-registry-pvd --docker-server=kubedev.azureecr.io --docker-username=kubedev --docker-password=JA&!FDYhab54d4fA --docker-email=email@email.com

```

`--docker-server` esta flag não é necessária, se não for declarado o Registry padrão é o dockerhub.

Depois basta usar a credencial no deployment na mesma seção onde é definido o contêiner:
```yaml
containers:
  - name: cadastro
      image: leogoandete/cadastro:latest  
imagePullSecrets:
- name: acr-registry-pvd
```
