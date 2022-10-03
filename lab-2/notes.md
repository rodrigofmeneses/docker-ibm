# Lab 2 Overview.

## Overview

Vamos construir conhecimento a partir do que foi feito no lab 1, onde aprendemos a rodar containers.
Iremos construir uma imagem Docker a partir de um Dockerfile.
Em seguida, iremos subir a imagem para o DockerHub.

## Prerequisites

Docker instalado

# Parte 1

## Criar uma aplicação Python Flask.

crie um arquivo app.py e copie:

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "hello world!"

if __name__ == "__main__":
    app.run(host="0.0.0.0")
```

# Parte 2

## Construir uma imagem Docker

Se você não tem Python instalado localment, não tem problema. Essa é uma das vantagens de usar containers Docker. Voce pode usar o Python em seus containers sem te-lo instalado na sua máquina.

1. Crie um arquivo chamado Dockerfile e adicione o seguinte conteúdo:
```
FROM python:3.10.5
RUN pip install flask
CMD ["python","app.py"]
COPY app.py /app.py
```
 - **FROM python:3.10.5**
    Esse é o ponto de partida de um Dockerfile. Está indicando a partir de onde a imagem será criada.
    Essa imagem python:3.10.5 está especificando que no repositório do python deve buscar essa versão.
 - **RUN pip install flask**
    Está indicando para executar esse comando na hora de construir o container.
 - **CMD ["python","app.py"]**
    CMD é o comando que é executado ao iniciar o container. Esse comando está mandando rodar minha aplicação python.
 - **COPY app.py /app.py**
    O comando COPY está copiando o arquivo local *app.py* para a imagem Docker.
    Pode parecer contra intuitivo por CMD ["python", "app.py"] antes dessa linha, porém a linha CMD só será executada depois que o container iniciar.

2. Construir a imagem Docker.

Basta digitar o seguinte comando:

```
docker imagem build -t python-hello-world .
```

Se tudo der certo, você verá a imagem criada ao utilizar o comando:
```
docker image ls
```

# Parte 3

## Rodar a imagem Docker

1. Agora que criamos a imagem, vamos bota-la pra funcionar.

```
docker run -p 5001:5000 -d python-hello-world
```

A flag -p mapeia em qual porta que o container irá rodar em seu host. Nesse caso, estamos indicando que o Python app rode na porta 5000 dentro do container para a porta 5001 dentro do host. Note que se a porta 5001 já estive sendo usada por outra aplicação no host, precisaremos trocar o 5001 para outro valor, como 5002 por exemplo.

2. Vamos na porta http://localhost:5001 em nosso browser pra ver se deu certo. Deve aparecer um "Hello world!".

3. Checar o log do container.
    Para ver o log basta digitar ```docker logs <container_id>```, para verificar o id usamos ```docker container ls```.
    Com isso vamos ver a saída padrão do terminal.

# Parte 4

## Enviar para o repositório central

1. Criar uma conta no [Docker Hub](https://hub.docker.com/).

2. Logar usando o docker no terminal:
   ```docker login```

3. Criar um nome de usuário:
   a convenção é [dockerhub username]/[image name].
```
docker tag python-hello-world [dockerhub username]/python-hello-world
```

4. Agora é só enviar para o respositóri utilizando:
```
docker push [dockerhub username]/python-hello-world
```

5. Agora só ir na conta do Docker Hub e ver se a imagem está la.

# Parte 5

## Mudanças no deploy

1. Vamos modificar o arquivo app.py trocando a string "Hello World" para "Hello Beutiful World".
    Agora que a aplicação está atualizada, é preciso rebuildar e atualizar no repositório.

2. Devemos então rebuildar utilizando o comando:
```docker image build -t [dockerhub username]/python-hello-world .```

3. Agora é só enviar para o dockerhub:
```
docker push [dockerhub username]/python-hello-world
```

# Parte 6

## Entendendo camadas da imagem

Considerando o que foi usado:

```
FROM python:3.10.5
RUN pip install flask
CMD ["python","app.py"]
COPY app.py /app.py
```

Cada linha é uma camada. Cada camada contém apenas o que há de diferente das camadas antes delas. Para por essas camadas em um único container, o Docker usa o 'union file system' para aninhar essas camadas em uma única visão.

Cada camada da imagem é READ-ONLY exceto a camada do topo, no qual é criada para o container. A camada do container READ/WRITE implementa "copy-on-write", isso signiffica que os arquivos guardados nas camadas inferiores apenas são enviados para ser read/write quando são editados.

A função "copy-on-write" é muito rápida em quase todos os casos. Podemos inspecionar quais arquivos foram enviados para o container com o comando docker diff.

Como as camadas são read-only, elas podem compartilhar imagens que se mantiveram inalterada. Tudo isso faz parte do mecanismo de 'caching' do docker.

Você consegue ver isso de perto quando vai atualizar uma imagem, perceba que várias camadas aparecerão a seguinte mensagem "Layer already exists", e apenas a ultima tem um código criptografado doido. Isso significa que só a ultima camada foi modificada.

# Parte 7

## Removendo containers

1. Para isso inicialmente devemos ver a lista de containers com:
```docker container ls```

2. Executar:
   ```docker container stop <container id>```
3. Remover containers parados com:
   ```docker system prune```

# Resumão
 - Usamos o Dockerfile para criar imagems reutilizaveis para nossa aplicação.
 - Imagens Docker podem ser disponibilizadas no Docker Hub.
 - Uma imagem docker contém todas as dependencias necessárias para a aplicação rodar. Isso é útil pois não precisaremos lidar com diferenças de ambiente.
 - Docker usa um sistema de união de camadas reutilizaveis, aquela "copy-on-write", que utiliza o sistema de caching e deixa tudo uma bala de rápido.
 - Cada linha do Dockerfile cria uma nova camada, e a última é a que será modificada frequentemente, então lembrar de sempre botar o código fonte no fim do arquivo.