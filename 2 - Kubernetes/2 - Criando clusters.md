### Configuração do Kubernetes com o KubeAdmin:

[https://highfalutin-vulture-304.notion.site/Instala-o-com-Kubeadm-8ce4f709872342ff848a4df77e53618d](https://www.notion.so/Instala-o-com-Kubeadm-8ce4f709872342ff848a4df77e53618d?pvs=21)

---

### Criando um cluster usando o **kind**:

Documentação: [https://kind.sigs.k8s.io/](https://kind.sigs.k8s.io/)

  

1 - Ter o **docker** e o **kubectl** instalado na máquina.

docker (minimo a engine): [https://docs.docker.com/engine/install/](https://docs.docker.com/engine/install/)

kubectl: [https://kubernetes.io/docs/tasks/tools/](https://kubernetes.io/docs/tasks/tools/)

  

2 - Instalar o **kind** conforme documentação: [https://kind.sigs.k8s.io/docs/user/quick-start#installation](https://kind.sigs.k8s.io/docs/user/quick-start#installation)

  

3 - Rodar comando conforme abaixo:

- Primeiro modo através de um comando
    
    [https://asciinema.org/a/Pn8ZE8qdOQX0Z4SfUEeDKgOuT](https://asciinema.org/a/Pn8ZE8qdOQX0Z4SfUEeDKgOuT)
    
    ```Bash
    kind create cluster --name meu-primeiro-cluster
    ```
    
    > [!info] Instalando o Kind.  
    > Recorded by leogoandete  
    > [https://asciinema.org/a/Pn8ZE8qdOQX0Z4SfUEeDKgOuT](https://asciinema.org/a/Pn8ZE8qdOQX0Z4SfUEeDKgOuT)  
    
- Segundo modo, usando um arquivo para criar o cluster
    
    Obs.: No arquivo abaixo contem a exposição da porta 80 e 443 para conectar ao cluster:
    
    ```YAML
    kind: Cluster
    apiVersion: kind.x-k8s.io/v1alpha4
    nodes:
    - role: control-plane
      kubeadmConfigPatches:
      - |
        kind: InitConfiguration
        nodeRegistration:
          kubeletExtraArgs:
            node-labels: "ingress-ready=true"
      extraPortMappings:
      - containerPort: 80
        hostPort: 80
        protocol: TCP
      - containerPort: 443
        hostPort: 443
        protocol: TCP
    - role: worker
    - role: worker
    ```
    
    Sem exposição:
    
    ```YAML
    ## Sem exposição
    kind: Cluster
    apiVersion: kind.x-k8s.io/v1alpha4
    nodes:
    - role: control-plane
    - role: worker
    - role: worker
    ```
    
    Utilizar o comando para informar o arquivo:
    
    ```Bash
    kind create cluster --config <nome-arquivo.yaml> --name <nome-desejado>
    ```
    
- Terceiro modo
    
    Utilizando serviços de nuvem onde o cluster é gerenciado pelo provedor, exemplos:
    
    EKS - Elastic Kubernetes Service da AWS.
    
    AKS - Azure Kubernetes Service da Microsoft Azure.
    
    GKE - Google Kubernetes Engine da Google
    
    entre outros como Oracle, DigitalOcean, Canonical e etc.
    
    Com o kubernetes gerenciado pelo provedor, não precisamos se preocupar com complexidade de gerenciar os nós(workers) e atualizações de sistemas operacionais e versão do cluster.
    

  

Outros serviços de Kubernetes para usar localmente:

microk8s - [https://microk8s.io/](https://microk8s.io/)

k3s - [https://k3s.io/](https://k3s.io/)

minikube - [https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/)

k3d - [https://k3d.io/](https://k3d.io/)