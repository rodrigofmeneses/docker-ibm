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

A flag -p mapeia em qual porta que o container irá rodar em seu host. Nesse caso, estamos mapenaod que o Python app rode na porta 5000 dentro do container para a porta 5001 dentro do host. Note que se a porta 5001 já estive sendo usada por outra aplicação no host, precisaremos trocar o 5001 para outro valor, como 5002 por exemplo.

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