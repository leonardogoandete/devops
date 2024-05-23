## Elementos básicos de um deploy no Kubernetes.

POD: O pod é onde é executado os containers, podemos ter mais de um container em um pod.

Não podemos confundir com rodar toda a aplicação em um POD.

O cenário correto para ter mais de um container em um POD, é utilizar um pattern chamado Side Car, onde é utilizado a aplicação principal em um container, e dentro do pod temos outros container auxiliares para coisas básicas.

Ex.: Coleta de logs, redirecionamento de rota e coleta de métricas.

Quando utilizamos mais de um container em um POD, ele irá utilizar a mesma rede e o mesmo file system.

Nunca podemos usar o pod para deploy da aplicação puro (naked pod), pois ele sozinho não garante escalabilidade e resiliência. Será melhor descrito no ReplicaSet e Deployment.

  

**Criando um pod declarativo:**

Salvar o código abaixo em um arquivo com o final .yml ou .yaml

```YAML
# Para descobrir a versão seguir o passo do "Descobrindo API Version"
apiVersion: v1

# Tipo do objeto a ser criado: Pod, Deployment, Secret...
kind: Pod

# Pode ser colocado varios valores como nome, namespaces, label e etc.
metadata: 
  name: meu-primeiro-pod
  app: nginx
  versao: verde
# Especificação de um container
spec:
   containers:
     - name: meucontainer
       image: leogoandete/doasanguepoa-front:v1.2.1
```

- Descobrindo API Version
    
    ```Bash
    kubectl api-resources
    ```
    
    ![[imagens/Untitled 6.png]]
    

  

Para aplicar basta rodar o comando abaixo:

```Bash
kubectl apply -f <nome-do-arquivo>.yaml
```


---

**Labels e Selectors:**

**Label**: é a utilização de elementos chave e valor nos objetos do kubernetes, ficam no mesmo local do metadata.

**Selector**: Forma de selecionar certo objetos baseado nos labels declarados. Muito utilizado para ReplicSet para selecionar os pods, Services, para saber quais serviços serão expostos.