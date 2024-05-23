- docker info
    
    Responsável para mostrar informações sobre o `docker engine` e do sistema operacional.
    
    Ex.:
    
    ```Bash
    docker info
    ```
    

---

- docker events
    
    Mostra todos os evento que ocorre no docker, como criação de container, conexão de network entre outros.
    
    Ao rodar o comando o terminal trava pois está em tempo real, podemos usar parametros para consulta.
    
    Ex.:
    
    ```Bash
    docker events —filter event=create
    ```
    

---

- docker logs
    
    Exibe os logs de um container em execução.
    
      
    
    Ex.:
    
    ```Bash
    docker container logs <id_container>
    OU
    docker container logs <nome_container>
    ```
    

---

- docker inspect
    
    Mostra varias informações sobre objetos do docker.
    
    ```Bash
    docker container inspect <id_recurso>
    docker image inspect <id_recurso>
    docker network inspect <id_recurso>
    ```
    
      
    
      
    

---

- docker top
    
    Mostra todos os processos que estão em execução no container.
    
    Ex.:
    
    ```Bash
    docker top <id_container>
    docker top <nome_container>
    ```
    
      
    
      
    

---

- docker stats
    
    Mostra a utilização de recursos de containers como cpu, memória, network e I/O.
    
    Ex.:
    
    ```Bash
    \#Utilizar --no-strem para não sincronizar a todo momento
    docker stats
    
    \#para pegar o momento atual(foto)
    docker stats --no-stream
    
    \#Somente de um container
    docker stats <id_container>
    ```
    
      
    
      
    

---

- docker exec
    
    Utilizado para executar um comando no container.
    
    Ex.:
    
    ```Bash
    docker exec <id_container> ls   \#executa no diretorio corrente
    
    \#modo iterativo
    docker exec -it <id_container> /bin/bash
    ```