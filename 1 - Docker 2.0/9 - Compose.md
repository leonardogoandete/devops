O que é?

O Docker Compose é uma forma simples de criar e gerenciar containers. Ele permite definir vários containers em um único arquivo YAML, facilitando a criação e execução de ambientes complexos.

Com o Docker Compose, você pode definir cada container como um serviço, especificando sua imagem, portas, variáveis de ambiente e outros detalhes.

O Docker Compose também oferece recursos para trabalhar com volumes e redes, permitindo que você defina volumes de dados persistentes e conecte containers em redes específicas.

  

Resumo: Forma declarativa para criação e gerenciamento de containers, volumes e redes.

  

**Convenção para criar um compose:**

[https://compose-spec.io](https://compose-spec.io)

  

_**comando para rodar manualmente no terminal:**_

```Bash
docker container run -d -p 8080:80 nginx:latest
```

  

_**compose para criar o container com nginx:**_

```YAML
version: "3.8"

#cada container é um service no compose
services:
#todo container possui um nome. Ex.: nginx
  nginx:
    #para dar um nome no container
    container_name: nginx
	#todo container usa uma imagem. Ex.: nginx:latest
    image: nginx:latest
	#mapeando port bind
	ports:
      - "8080:80"
```

  

_**compose para criar o container com banco de dados:**_

```YAML
version: "3.8"

#cada container é um service no compose
services:
#todo container possui um nome. Ex.: nginx
  banco:
    #para dar um nome no container
    container_name: banco_dados
	#todo container usa uma imagem. Ex.: nginx:latest
    image: postgres:12.17
	#mapeando port bind
	ports:
      - "5432:5432"
    #para definir variavéis de ambiente
    envoriment:
      POSTGRES_PASSWORD: Pg123
      POSTGRES_USER: usuario
      POSTGRES_DB: database
```

  

para rodar o arquivo acima usamos o comando no caminho `docker compose up` do arquivo.

outras formas de executar:

```Bash
# para não prender o terminal
docker compose up -d

# para definir um arquivo compose manual
docker compose -f <arquivo.yml> up -d
```

  

Para realizar a remoção dos container basta utilizar o comando:

```Bash
#para remover 
docker compose down

# para definir um arquivo de remoção
docker compose -f <arquivo.yml> down
```

  

Para iniciar ou parar a execução dos containers basta utilizar o comando:

```Bash
# para startar
docker compose start
# para startar com um arquivo compose manual
docker compose -f <arquivo.yml> start 


#O comando abaixo não exclui os containers
# para pausar
docker compose stop
# para pausar com um arquivo compose manual
docker compose -f <arquivo.yml> stop 
```

  

---

## Trabalhando com volumes no compose

Adicionar ao service conforme exemplos:

_**usando bind mount:**_

```YAML
version: "3.8"
services:
  banco:
    container_name: banco_dados
    image: postgres:12.17
	ports:
      - "5432:5432"
    envoriment:
      POSTGRES_PASSWORD: Pg123
      POSTGRES_USER: usuario
      POSTGRES_DB: database
	#mapeando o volume maquina host para o container
    # nesse exemplo é usado o diretório local, se não 
    # existir a pasta "postgre_vol", será criada automaticamente
    volumes: 
      - ./postgre_vol:var/lib/postgresql/data
```

  

_**usando volume gerenciado pelo docker:**_

Desta forma preciso declarar ao compose que será criado um volume, essa declaração é feita no nível do `services`. Desta forma posso executar `docker compose down` e os dados não serão perdidos, podendo usar em nova execução.

```YAML
version: "3.8"
services:
  banco:
    container_name: banco_dados
    image: postgres:12.17
	ports:
      - "5432:5432"
    envoriment:
      POSTGRES_PASSWORD: Pg123
      POSTGRES_USER: usuario
      POSTGRES_DB: database
	#mapeando o volume gerenciado para o container
    # nesse exemplo é usado o volume criado na propriedade "volumes"
    # mais abaixo do código, e referenciado no volumes dentro do service.
    volumes: 
      - postgre_docker_vol:var/lib/postgresql/data

volumes:
  #criação do volume
  postgre_docker_vol:
    #definir nome de exibição para o volume.
    name: postgre_exemplo_vol
    
#Posso referenciar volumes já criados na maquina, conforme abaixo:
#Ex.: docker volume create meu_volume_favorito
#Obs.: se não existir dará erro.
  meu_volume_favorito:
    # aqui declaro que não foi criado pelo compose.
    external: true

```

---

## Trabalhando com network no compose

Similar ao `volumes` basta adicionar no compose da seguinte forma:

- **bridge:**
    
    ```YAML
    #Obs.: no mesmo nivel de services.
    networks:
      #Nome da rede
      minha_rede_favorita:
        #especificar o tipo
        driver: bridge
    
    #Após podemos adicionar no service desejado.
    services:
      banco:
        container_name: banco_dados
        image: postgres:12.17
        #Definindo a rede do container banco.
        networks:
          - minha_rede_favorita
        #Obs.: Todos os container que utilizarem o banco de dados,
        # deverá ter declarado a rede na sua declaração
           
    ```
    
    Consigo definir `networks` criadas fora do compose, basta fazer da seguinte forma:
    
    Ex.: `docker network create minha_rede`
    
    ```YAML
    version: "3.8"
    services:
      ...
    networks:
      minha_rede:
        external: true
    ```
    
- **host:**
    
    Para utilizar o modo `host` para ficar com o mesmo IP da maquina pessoal basta configurar o service com o `network_mode: host` (sem isolamento de rede)
    
    ```YAML
    version: "3.8"
    services:
      nginx:
        image: nginx:latest
        network_mode: "host"
    ```
    

  

---

## Trabalhando com profiles no compose

Caso precisamos declarar serviços no docker compose que deve subir somente com certos perfis, podemos utilizar profiles no serviços declarados conforme abaixo:

```YAML
services:
  postgre:
    image: postgres:latest
    profiles: 
      - dev
  frontend:
    image: meu_front:latest
```

Se rodar somente com `docker compose up` o único serviço que executará será o `frontend` .

Para subir os outros devemos passar o profile usando `docker compose --profile dev up` será criado o restante dos serviços.

Tomar cuidado com `depends_on` se o serviço que depende tiver um profile declarado irá dar erro.

---

## Variáveis no compose

Podemos parametrizar valores de variavéis de ambiente no compose.yml, conforme abaixo:

```YAML
services:
  banco_dados:
    image: postgres:${POSTGRES_TAG}
    envoriment:
      POSTGRES_PASSWORD: ${PG_PASS}
      POSTGRES_USER: ${PG_USER}
      POSTGRES_DB: database

# Posso definir um valor default 
# caso não for passado nenhuma variavel desta forma
services:
  banco_dados:
    image: postgres:${POSTGRES_TAG:-12.15}
    envoriment:
      POSTGRES_PASSWORD: ${PG_PASS:m1nh4S3nh4}
      POSTGRES_USER: ${PG_USER:user_run}
      POSTGRES_DB: database
```

Para passarmos os valores para o compose podemos utilizar três formas:

- 1 - Definido as variáveis junto com o comando `docker compose`
    
    ```Bash
    POSTGRES_TAG=12.17 PG_PASS=Pg@123 PG_USER=user_run docker compose up -d
    ```
    
- 2 - Definir nas variáveis de ambiente na maquina:
    
    ```Bash
    export POSTGRES_TAG=12.17 
    export PG_PASS=Pg@123 
    export PG_USER=user_run
    ```
    
- 3 - Usando arquivo `.env`
    
    Este formato ele identifica por padrão.
    
    ```Plain
    export POSTGRES_TAG=12.17 
    export PG_PASS=Pg@123 
    export PG_USER=user_run
    ```
    
    Depois executar: `docker compose up`
    
- 4 - Usando arquivo `desenv.env`
    
    Criar o arquivo no mesmo modelo que o `.env` , porem precisamos definir manualmente com o comando:
    
    ```Bash
    docker compose --env-file desenv.env up -d
    ```
    
- 5 -

  

  

  

---

## Facilidades

Umas das facilidades ao uso do compose é permitir a alteração do yaml e aplicar o compose novamente, ele alterar a execução do container(como se fosse um refresh)

  

- **_Posso alterar o command ou entrypoint do container:_**

```YAML
version: "3.8"

services:
  nginx:
    container_name: nginx
    image: nginx:latest
	ports:
      - "8080:80"
    command:
      - echo
      - hello world 
```

---

  

- _**Posso referenciar services para vincular através do nome do service.**_

```YAML
version: "3.8"

services:
  backend:
    image: minha_api:latest
    networks:
      - bridge
  frontend:
    image: meu_front:latest
    networks:
      - bridge
    envoriments: 
      # Estou dizendo para meu front que a url da minha api é o serviço de cima,
      # automaticamente o DNS resolve o nome. Lembrar de colocar na mesma rede.
      - ENV_API_URL: backend:8080

networks:
  rede_servico:
    driver: bridge
```

---

  

- **_==Dependência de serviços:==_**

  

Caso tenha um componente, exemplo uma api que usa banco de dados,

para resolver esse problema podemos corrigir adicionando a declaração `depends_on` e adiciono no container que precisa da dependência. Podemos ter uma lista de dependências.

  

Exemplo:

```YAML
service:
  banco:
    image: mongo:latest
  api:
    image: minha_api:latest
    depends_on:
      - banco
```

---

- _**Utilizar compose para construir uma imagem de container.**_

  

Normalmente costumamos a fazer o a construção de uma imagem usando um Dockerfile e utilizar o comando `docker build -t leogoandete/login:latest .` , conseguimos reproduzir isso dentro do compose.

  

Exemplo:

Arquivo Dockerfile.

```Docker
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json .
RUN npm install
COPY . .
RUN npm run build

#Geração da imagem do Nginx para rodar o projeto
FROM nginx:stable-alpine3.17-slim
COPY ./config/ngnix.conf /etc/nginx/conf.d/default.conf
WORKDIR /usr/share/nginx/html
RUN rm -rf ./*
COPY --from=builder /app/build .
ENTRYPOINT ["nginx", "-g", "daemon off;"]
```

Tendo o arquivo `Dockerfile` no diretorio, podemos contruir o compose da seguinte forma:

  

```YAML
services:
  projeto_front:
    image: meu-projeto:v1.0.0
    build:          # Definição que sera construido a imagem
      context: /src     # Caminho do diretorio para enviar para imagem. Ex: pasta onde está o codigo
      dockerfile:   # Caminho do Dockerfile
    ports:
      -  8080:80
```

Posso forçar um novo build usando `docker compose up -d --build` ou posso apenas fazer o build usando `docker compose build`

---