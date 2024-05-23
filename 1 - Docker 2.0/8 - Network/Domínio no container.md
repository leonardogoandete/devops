Podemos adicionar no arquivo host do container a resolução de DNS e IP da seguinte forma:

  

```Bash
docker run -it --add-host meudominio.com:172.19.171.92 ubuntu
```

Se executarmos um `cat /etc/hosts` no container tera a entrada adicionada no arquivo.