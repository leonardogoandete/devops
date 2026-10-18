Utilizando sock para se comunicar ao Docker via API:

  

Link para os endpoints:

[https://docs.docker.com/engine/api/v1.45/](https://docs.docker.com/engine/api/v1.45/)

[https://docs.docker.com/engine/api/#api-version-matrix](https://docs.docker.com/engine/api/#api-version-matrix)

  

Obtendo versão do docker:

```Bash
curl --unix-socket /var/run/docker.sock http://localhost/version
```