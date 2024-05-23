  

Documentação do Trivy: [https://aquasecurity.github.io/trivy/](https://aquasecurity.github.io/trivy/)

Analisando Dockerfile:

```Bash
trivy config <caminho-dockerfile>
```

  

Scan de imagem default (vuln, secret):

```Bash
trivy image <imagem-docker>
```

  

Scan de imagem utilizando scanner especifico(full):

```Bash
trivy image --scanners vuln,misconfig,secret,license <imagem-docker>
```