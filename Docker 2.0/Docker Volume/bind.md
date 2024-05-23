O comando "bind" no Docker permite mapear um diretório da máquina host para um diretório dentro do container. No entanto, é importante notar que ao criar um arquivo na máquina host, ele terá as permissões do usuário que o criou, o que pode comprometer o funcionamento do container.

resumo: Mapeia o diretório da maquina host com o diretório do container.

  

![[imagens/Untitled 8.png|Untitled 8.png]]

### ==Obs.: FileSystem é o sistema de arquivos da maquina Host.==

  

Exemplo de uso do comando `docker run --mount`:

```Bash
docker run --mount type=bind,source=/caminho/da/minha/maquina,target=/caminho/do/container nome-da-imagem

OR

docker run -v /caminho/da/minha/maquina:/caminho/do/container nome-da-imagem
```

O comando acima mapeia um diretório da sua máquina host para um diretório dentro do container.

---

Particularidades do modo **`bind`** .

Ao criar um arquivo na maquina host ele ficará com o usuário de onde foi criado.

Ex.:

![[imagens/Untitled 1 4.png|Untitled 1 4.png]]

O arquivo da linha 1 foi criado dentro do container.

O arquivo da linha 2 foi criado na maquina(host) onde o container está executando.

  

Isso pode comprometer o funcionamento do container, pois qualquer um tem permissão de escrita e exclusão do arquivo.