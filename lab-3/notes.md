# Container orchestration
O que é essa orquestração de containers?
Gerenciamento de containers:
 - Kubernetes, Docker Swarm, MESOS
Empresas também usam seus próprios
 - IBM Cloud Container Service, Amazon ECS, Azure Containers ...

# Lab 3 Overview

## Overview
Agora aprenderemos a utilizar containers em produção.

Utilizaremos a ferramenta Dcoker Swarm.

# Parte 1

# Criando meu primeiro Swarm
Para esse exercício utilizaremos o site [Play-with-Docker](https://labs.play-with-docker.com/)

1. Logar no Play-with-Docker.
2. Criar três nós clican em **Add new instance** três vezes.
3. Inicializar o swarm no nó 1:
```docker swarm init --advertise-addr eth0```
    Uma mensagem deve indicar que esse nó agora é o 'manager'.

4. Para adicionar um 'worker' a esse swarm, rode o seguinte comando:
```
docker swarm join --token SWMTKN-1-2c73v2mn1ahe0e1b68hg0kwse9jtg3psnhozv2ezbvyskfnlaz-4ge05xrpo48vhcd2zj2u4fbri 192.168.0.16:2377
```
esse token foi disponibilizado ao criar o manager
5. Voltando ao nó manager, vamos verificar quem são os workes com:
   ```docker node ls```
   Um * indica quem é o manager.

# Parte 2

## Deploy o primeiro serviço

Agora que temos três nós Swarm inicializados no cluster, vamos deployar um container. Para rodar o container em um Docker Swarm, precisamos criar um serviço. Um serviço é uma abstração que representa multiplos containers com a mesma imagem deployed.

1. Deploy um serviço usando NGINX:
```
docker service create --detach=true --name nginx1 --publish 80:80  --mount source=/etc/hostname,target=/usr/share/nginx/html/index.html,type=bind,ro nginx:1.12
pgqdxr41dpy8qwkn6qm7vke0q
```
2. Inspecionar serviços:
```docker service ls```

3. Verificar se tem o container do serviço rodando:
```docker service ps nginx1```
4. Testar o serviço:
```curl localhost:80```
Ao rodar isso em qualquer um dos nós a resposta será o nome dada pelo nó manager.

# Parte 3

## Escalar o serviço

Em produção pode haver um grande trafego de requisiões na nossa aplicação, por isso é necessário escalar.

1. Atualizar o serviço com um numero de réplicas:
```docker service update --replicas=5 --detach=true nginx1```
Ao ativar isso o seguinte acontecerá:
 - O estado do serviço foi atualizado para 5 replicas, no qual é guardada no armazenamento interno do swarm.
 - Docker swarm reconhece que o numero de replicas que foram escalas agora não dar match com o estado declarado de 5
 - Então o Docker swarm escala mais 5 tasks (containers) a fim de igualar seu estado com o que foi declarado.

2. Verifique as instâncias rodando:
```docker service ps nginx1```
Perceba que várias instâncias estão rodando nos diferentes nós.

3. Envie um monte de requests para http://localhost:80.
```
curl localhost:80
node 1
curl localhost:80
node 1
curl localhost:80
node 2
curl localhost:80
node3
curl localhost:80
node1
```
Dessa vez verá que diferentes nós responderão a request.

4. Verifique os logs para os serviços:
```docker service logs nginx1```
Baseado nesses logs percebe-se que cada requisição foi servida por diferentes containers.

# Parte 4

## Aplicando updates
Agora que temos um serviço em produção, vamos atualizar o serviço do nginx para a versao 1.13.

1. Rodar ```docker service update --image nginx:1.13 --detach=true nginx1```:
Isso engatilhou atualizações no swarm. Rode ```docker service ps nginx1``` para ver as atualizações em tempo real. É possível também indicar quantos nós serão atualizados ou por um delay nisso. 

2. Depois de um tempo rode novamente ```docker service ps nginx1``` para verificar se estão todos atualizados.

# Parte 5
## Conciliar problemas com containers

É possível verificar em tempo real os serviços rodando. 
1. Para isso vamos recriar as instâncias do nginx2 agora na porta 81:
```
docker service create --detach=true --name nginx2 --replicas=5 --publish 81:80  --mount source=/etc/hostname,target=/usr/share/nginx/html/index.html,type=bind,ro nginx:1.12
```
2. Agora vamos ficar vigiando em tempo real o clusters:
```watch -n 1 docker service ps nginx2```

3. Para tirar um nó do cluster, basta ir no em algum worker e digitar:
```docker swarm leave```

Perceba que no terminal do manager o nó que foi desconectado está como 'Shutdown'.

# Parte 6

## Determine quantos nós você precisa
É possível ter vários managers para sua aplicação, uma de suas funções é tolerar falhas de nós. Por exemplo:

- Three manager nodes tolerate one node failure.
- Five manager nodes tolerate two node failures.
- Seven manager nodes tolerate three node failures.

# Resumão

- Tivemos uma breve introdução sobre orquestramento de containers em produção.
- Docker Swarm escala serviços usando uma linguagem declarativa. Você declara o estado e o swarm tenta manter e conciliar os processos.
- Docker Swarm é composto por nós managers e workers. Apenas managers podem modificar estados do swarm. Workers tem como função ter uma alta escalabilidade e são apenas usados para rodas containers.
- No sistema de rotas, qualquer porta pode ser publicada aos serviços, deixando-as expostas para todos os nós do warm. Requests serão automaticamente roteadas e distribuidas para o container onde estão os serviços.
- Tem outras ferramentas.