  

## Docker restart

Quando rodamos um container que termina com erro codigo 0 ou codigo 1, é necessário rodar manualmente o container novamente.

Mesmo caso ocorre quando o serviço do docker é parado manualmente ou por algum sinal.

  

Para isto temos o `docker restart` definindo uma politica para o container poder ser restartado, voltando ao ar normalmente.

Temos 3 tipos de definição são elas:

  

- _**on-failure:**_
    
    Utilizado para reiniciar o contêiner, **somente se for encerrado com erro**. Com o on-failure consigo definir um numero limite de restarts.
    
    Neste caso mesmo o docker estando fora, após voltar, o container será restartado.
    
      
    
    Como definir?
    
    ```Bash
    \#Sem definir numero de restarts
    docker container run -d --restart=on-failure nginx:12.4
    
    \#Definindo numero de restarts
    docker container run -d --restart=on-failure:3 nginx:12.4
    ```
    
    ```YAML
    services:
      nginx:
        image: nginx:latest
        restart: on-failure
    ```
    
- _**unless-stopped:**_
    
    Vai reiniciar o container normalmente mesmo se o código de erro for com 0 ou 1 e com o serviço do docker caindo, a unica exceção é se eu parar de forma manual ou seja com um `docker stop <id_container>` por exemplo ele não vai restartar.
    
      
    
    Como definir:
    
    ```Bash
    docker container run -d --restart=unless-stopped nginx:12.4
    ```
    
    ```YAML
    services:
      nginx:
        image: nginx:latest
        restart: unless-stopped
    ```
    
- _**always:**_
    
    O always vai restartar sempre que o container parar.
    
      
    
    Como definir
    
    ```Bash
    docker container run -d --restart=always nginx:12.4
    ```
    
    ```YAML
    services:
      nginx:
        image: nginx:latest
        restart: always
    ```
    

  

## Healthcheck

É utilizado para quando o estado de saúde dos containers não está saudável.

É um elemento de verificação de saúde, pode ser um ping, usar uma instrução especifica ou um endpoint padrão, geralmente um GET /health.

Podemos configurar o healt check de 3 formas no docker, são elas:

  

O exemplo abaixo é com base em alguma api que contenha um endpoint GET /health configurado.

  

**Linha de comando:**

```Bash
docker container run -d nginx:latest \
  --health-cmd "curl -f http://localhost:3000/health" \  # comando para chamar no endpoint
	--health-timeout 5s \    # tempo que aguardara a resposta do curl  
	--health-retries 5  \    # quantidade de tentativas caso de erro
  --heatlh-interval 10s \  # intervalo de tempo para fazer nova requisição GET
  --health-start-period 30s    # tolerancia de 30seg apos inicio do container para fazer requisições
```

  

**Docker compose:**

```YAML
services:
  nginx:
    image: nginx:latest
    restart: on-failure
    healthcheck:
      test: ["curl", "/health"]  # comando para chamar no endpoint
      interval: 10s              # intervalo de tempo para fazer nova requisição GET
      timeout: 5s                # tempo que aguardara a resposta do curl  
      retries: 5                 # quantidade de tentativas caso de erro
      start_period: 30s          # tolerancia de 30seg apos inicio do container para fazer requisições
```

  

**Direto na imagem:**

Para utilizar direto na imagem precisamos defini isto no Dockerfile, conforme abaixo:

```Docker
\#Geração da imagem do Nginx para rodar o projeto
FROM nginx:stable-alpine3.17-slim
COPY ./config/ngnix.conf /etc/nginx/conf.d/default.conf
WORKDIR /usr/share/nginx/html
RUN rm -rf ./*
COPY --from=builder /app/build .
HEALTHCHECK --interval=10s --timeout=5s --start-period=5s --retries=3 CMD ["curl", "-f", "http://localhost:3000/health" ]
ENTRYPOINT ["nginx", "-g", "daemon off;"]
```

  

  

  

  

### **Gerenciamento de CPU**

Ao começar a criar containers com o Docker, por padrão ele tem acesso a todos os recursos de CPU disponível, vamos começar a controlar isso para limitar a utilização de recursos.

  

Como definir limite de CPU no processo do container do docker.

  

Define o tempo em microssegundos para o processo de scheduler. Faz a distribuição de alocação de CPU em um certo período.

O comando "docker container run -d --cpu-period=100000 --cpu-quota=50000 nginx" é usado para executar um contêiner Docker com restrições de CPU, definindo um período e uma cota para o uso de CPU.

`docker container run -d --cpu-period=100000 --cpu-quota=50000 nginx`

Posso definir qual núcleo quero utilizar, utilizando o parâmetro: `—cpuset-cpus=0`

O formato acima é um pouco complexo para definir por isso podemos utilizar outra abordagem, conforme abaixo:

`docker container run -d --cpus=0.5 nginx`

Ou seja eu informo que quero que utilize somente metade da CPU, para facilitar temos a tabela abaixo:

|   |   |
|---|---|
|valor|qtde CPU|
|0.25|1/4 de cpu|
|0.5|meia cpu|
|1|1 CPU|
|1.5|1 CPU e meia|

Um cuidado que temos que ter é que no linux ele faz o balanceamento no uso da CPU, por exemplo se definirmos 1.5 de CPU e definir para dois núcleos, ou seja o uso de cada núcleo usará 75%.

  

### **Gerenciamento de memória**

Um container nunca irá morrer por uso máximo de CPU, porem por uso máximo de memória sim, irá ocorrer o famoso `out of memory`

Para alterarmos o uso de memória máxima que o container pode ter é utilizarmos a flag `--memory=50M` nesta caso definimos o uso de 50M de memória.

Podemos limitar a memória swap também, mas temos que fazer uma conta, exemplo, se temos 50M de memória RAM, para limitar a swap para 50M temos que somar o que temos na memória RAM + quanto desejo ter na swap, exemplo:

50M ram e desejo ter 70M de swap:

50+70= 120M

devemos definir as flags: `--memory=50M --memory-swap=120M`

  

**Como definir no docker compose:**

```YAML
services:
  nginx:
    image: nginx:latest
    restart: on-failure
    cpuset: "0"          \#Define qual nucleo utilizar, não recomendado utilizar.
    memswap_limit: 450M   \#Define o tamanho da swap(lembrar do calculo)
    deploy:
      resources:
        limits: 
          cpu: "0.5"     # Define a quantidade de CPU para usar.
          memory: 400M   # Define o maximo de memoria a ser usada.
```