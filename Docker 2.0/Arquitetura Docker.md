## Linux

Docker Daemon(Docker host é quem hospeda o docker Daemon):

Docker Daemon é o componente responsável por executar e gerenciar os containers Docker em um ambiente. Ele é responsável por receber os comandos do Docker Client, gerenciar imagens, redes e volumes, além de monitorar e controlar a execução dos containers.

  

Docker Host:

Docker Host é a máquina física ou virtual que hospeda o Docker Daemon. É nessa máquina que os containers serão executados e gerenciados. O Docker Host deve possuir os recursos necessários, como capacidade de processamento, memória e armazenamento, para suportar a execução dos containers de forma eficiente.

  

Docker Client:

Docker Client é a interface de linha de comando ou aplicativo que permite aos usuários interagirem com o Docker Daemon. Através do Docker Client, os usuários podem enviar comandos para o Docker Daemon, como criar, executar, parar e remover containers, além de gerenciar imagens, redes e volumes.

  

Docker Resgistry:

Docker Registry é um serviço que armazena e distribui imagens Docker. Ele permite que os usuários compartilhem e acessem facilmente imagens para a criação de containers. Existem registros públicos, como o Docker Hub, e registros privados, que podem ser configurados para uso interno em uma organização.

  

![](../../imagens/arq-docker.png)

  

---

  

## Arquitetura docker a fundo

Docke atualmente:

![](../../imagens/arq-docker-interno.png)

  

  

## Windows

## Windows

No Windows, é possível executar o Docker usando o WSL 2 (Windows Subsystem for Linux 2). O WSL 2 é um ambiente de execução do Linux que permite executar aplicativos e comandos do Linux diretamente no Windows.

Com o WSL 2 habilitado, você pode instalar o Docker Desktop no Windows e aproveitar todos os recursos do Docker, como criar e gerenciar containers, imagens e redes, diretamente na sua máquina Windows.

A integração do Docker com o WSL 2 no Windows proporciona uma experiência mais otimizada e eficiente ao utilizar o Docker no ambiente Windows.

  

![](../../imagens/arq-docker-windows.png)

  

Ou seja da mesma forma como o linux é virtualizado o docker é virtualizado da mesma forma.