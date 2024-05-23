Como garantir que a minha imagem é a que está rodando realmente?

Para este tipo de problema temos a assinatura da imagem usando o **Cosign**.

Documentação: [https://docs.sigstore.dev/](https://docs.sigstore.dev/)

Instalação do Cosign: [https://docs.sigstore.dev/system_config/installation/](https://docs.sigstore.dev/system_config/installation/)

  

![[imagens/Untitled 15.png|Untitled 15.png]]

1- Neste passo realizamos o build da imagem, e enviamos ao nosso registry.

2- Local onde a imagem é armazenado.

3- Local de fato onde a imagem será executada.

4- Neste passo está o problema, alguma pessoal mal intencionada, poderá criar uma imagem adulterada e sobrescrever a que já existe no registry. Fazendo com que não tenhamos garantia de que a imagem que está rodando é a de fato no passo 1.

  

Como fazemos isso?

Geramos um par de chaves similar ao SSH e utilizaremos elas para realizar a assinatura.

  

Gerando par de chaves:

```Bash
\#Nome default será cosign.key e cosign.pub
cosign generate-key-pair

\#Nome das chaves sera definido com o valor entre < >.
cosign generate-key-pair --output-key-prefix <nome_desejado_chave>
```

Obs.: Senha opcional.

  

Assinando uma imagem:

```Bash
\#Utilizar a chave privada para assinar. No exemplo abaixo o nome é cosing.key
cosign sign --key cosign.key <nome-imagem>
```

Utilizando chave como variavel de ambiente:

```Bash
export COSIGN_KEY=$(cat cosign.key)
\#Utilizando chave como variavel de ambiente
cosign sign --key env://COSIGN_KEY <nome-imagem>
```

  

Verificando se a imagem está assinada:

```Bash
\#Utilizar a chave publica para verificar a assinatura.
cosign verify --key cosign.pub <nome-imagem>
```