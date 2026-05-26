# Parando um container:

### docker stop [id]
(parando imediatamente: adicione -t = 0, pois o container geralmente demora 10 segundos para parar)

# Startando um container:

### docker start [id]

# Roda um comando no container já ativo:

### docker exec
### docker exec -it [id] bash
-it, nesse caso, é de i: interativo, t: terminal.

# Pausar um container:

### docker pause [id]

# Excluir container: 

### docker rm [id]

# Executar um comando e manter o container em segundo plano: 

### docker run -d dockersamples/static-site 

# Excluindo um container imediatamente mesmo que ele esteja rodando:

### docker rm [id] --force

# Checando mapeamento de portas de um container

### docker port [id]

# Remapeando portas

### docker run -d -p 8080:80 dockersamples/static-site 
Nesse caso, queremos que a porta 8080 da nossa máquina (localhost) reflita na porta 80 do container. Ou seja, quando eu acessar a porta 8080 do meu localhost, quero que me leve a porta 80 do dockersamples.

### flag -P: O docker escolhe quais portas livres da sua máquina ele irá mapear.
Ou seja, ao rodar o comando docker run -P nginx, ele irá mapear em alguma porta alta e disponível, que podemos descobrir ao utilizar docker ps.

### Uma imagem é um conjunto de camadas, que ao serem unificadas, formam imagens.
Elas são independentes, cada uma tem o seu respectivo ID. Quando são unificadas, criam uma espécie de regra de execução dentro de um container.

Quando criamos um container, criamos uma **camada extra por cima de imagens.**
Quando imagens são criadas, elas são apenas R/O, ou seja, não podemos modificá-las após sua criação.
Os containers nos permitem criar uma camada de R/W, ou seja, quando criamos um container, criamos essa camada temporária que nos permite alterar essas imagens. 
Quando esse container é parado, essa camada extra é encerrada.

### Por que containers são tão leves?
Além deles serem apenas processos no nosso sistema, podemos dizer que quando um container entra em execução, estamos sempre reaproveitando a mesma imagem.
Como a imagem é apenas de leitura, podemos ter vários containers baseados na mesma imagem. A diferença é que cada um desses containers terá apenas uma camada diferente de read-write, e como essa camada é leve, para manter a rápida performance temos uma reutilização de imagens para múltiplos containers.

### docker history (imagem)
Comando que nos possibilita visualizar camadas de uma imagem.

# Criando a primeira imagem

*Dockerfile*: Arquivo de texto contendo instruções passo a passo para construção de imagem docker.
Ele automatiza a instalação de pacotes, configuração de ambiente e cópia de arquivos.
É compilado pelo comando *docker build*

FROM node:14 - Definir que queremos pegar a partir do node na versão 14.

COPY . /app-node - Queremos pegar todo o conteúdo do nosso diretório atual e copiar para a pasta /app-node.

A partir disso, queremos executar o comando *npm install*, mas esse comando deve ser executado dentro do nosso diretório /app-node , para que possamos instalar TODAS as dependências necessárias da nossa aplicação.

RUN npm install

Por fim, queremos que o ponto de entrada (ENTRYPOINT) do nosso container ao executar essa imagem e começar a ter seu container devidamente em execução, seja iniciar a aplicação.
Para isso, usamos o npm start:
*ENTRYPOINT npm start*

WORKDIR /app-node - Definimos o diretório padrão como /app-node para evitar ter que definir ele em todo comando.

FROM node:14
WORKDIR /app-node
COPY . .
RUN npm install
ENTRYPOINT npm start

Agora, com o dockerfile definido, conseguimos gerar uma imagem. Dentro do diretório, digitamos o comando: 

### docker build -t ana/app-node:1.0 .

*-t*: Criando um nome para a nossa imagem (meio que etiquetando ela);
*:1*: Versão da nossa imagem criada;
*.*: Fazendo isso no diretório atual.

Ao executarmos esse comando, o Docker vai ao Docker Hub buscar a imagem do node na versão que definimos para baixá-la.

### docker run -d -p 8081:3000 ana/app-node:1.0

-d: Modo detached;
-p: Definimos que vamos usar a porta 8081 da nossa máquina para refletir na porta 3000, que é onde nossa aplicação vai ficar em execução dentro do nosso container.

Quando acessamos no nosso navegador (localhost:8081), vemos que conseguimos acessar nossa aplicação de forma containerizada!

### docker stop $(docker container ls -q)

Aqui, damos um comando como parâmetro para o nosso docker stop. o -q é de quiet, e só pega o ID.

ARG vs ENV:

FROM node:14
WORKDIR /app-node
COPY . .
ARG PORT_BUILD=6000
ENV PORT=$PORT_BUILD
EXPOSE $PORT_BUILD
RUN npm install
ENTRYPOINT npm start

O *ARG* e o *ENV* são ambos variáveis de ambiente. O ARG roda apenas no tempo de criação da nossa imagem (docker build), enquanto o ENV é utilizado dentro do container posteriormente.

A instrução *EXPOSE* explicita em qual porta a aplicação está sendo executada.




### docker push anamorone/app-node:1.1

Conseguimos subir uma imagem no Docker Hub. Repare que o nome de usuário deve ser o mesmo que a sua conta no docker hub. Para isso, temos um comando no qual conseguimos copiar uma imagem já existente para outra tag, até com o mesmo ID:

### docker tag ana/app-node:1.2 anamorone/app-node:1.2


# Revisão de alguns comandos importantes:

### docker rm $(docker container ls -aq)
*-a:* Pegar todos os container (até os que estão parados);
*-q:* Pegar apenas os IDs.

### docker rmi $(docker image ls -aq) --force
Para remover imagens. O --force força a exclusão, caso dê algum erro.

### docker ps -s
*-s:* Tamanho do container (real + virtual (da imagem)).

# Persistindo dados (volumes):

Se criarmos um novo container com *docker run -it ubuntu bash* e dermos um *docker ps -s*, podemos perceber que se já tivermos outro container já em execução, teremos um retorno de um novo container com outro ID e seu tamanho zerado (camadas de read-write são isoladas uma das outras, e cada container terá a sua)

## bind mount

### docker run -it --mount type=bind,source=/home/ana/volume-docker,target=/app ubuntu bash

Aqui, definimos que queremos fazer um mount do tipo bind, e que o diretório da nossa máquina será o home/ana/volume-docker. Nosso target vai ser /app também, dentro desse container.

Assim, se tivermos um arquivo de configuração que a nossa aplicação precisa executar, ou dados essenciais desse tipo, conseguimos mantê-los sem problemas.

## volumes

São uma área gerenciada pelo Docker dentro do sistema de arquivos.
OU seja, mesmo que as nossas informações continuem no nosso host original para serem persistidas. nós teremos uma área que o Docker irá gerenciar, o que é muito mais seguro em termos de alguém fazer alterações indesejadas.

Ele geralmente fica armazenado em */var/lib/docker/volumes*
Conseguimos criar volumes, inspecionar eles, remover os que não estão sendo utilizados, etc. já que é tudo gerenciado pelo Docker.

Para *criar um volume:*

### docker volume create meu-volume

Para rodar um container utilizando esse volume:

### docker run -it --mount source=meu-volume,target=/app ubuntu bash

## tmpfs

Esse tipo de armazenamento tem uma ideia de persistir dados na memória do host, mas não na camada de R/W, e sim apenas na memória do host, ficando em memória temporariamente. Ou seja, armazenm dados em memória volátil.

### docker run -it --tmpfs=/app ubuntu bash
ou
### docker run -t --mount type=bash,destination=/app ubuntu bash

# Comunicação entre Containers

Por padrão, containers não se enxergam. Redes Docker são as pontes entre as ilhas, conectando containers entre si.

## 1 - Bridge (rede padrão)

Quando damos um docker run sem especificar nada. o container cai na rede bridge padrão. Se o IP mudar, caso o container reinicie, é ruim em produção.

Podemos criar nossa própria rede bridge:

### docker network create minha-rede

Aqui, o Docker ativa um DNS interno, ou seja, os containers passam a se encontrar pelo *nome*, e não mais pelo IP.

## Criando 2 containers na mesma rede:

### docker run -d --name banco --network minha-rede postgres
### docker run -d --name api --network minha-rede minha-api

## 2 - host (sem isolamento de rede)

Com esse driver, o container abandona seu isolamento de rede e usa diretamente a interface de rede da sua máquina.

### docker run ---network host nginx

Se o ngnix escuta na porta 80, ele vai estar na porta 80 da sua máquina diretamente, sem mapeamento de porto (-p 80:80)

Usamos esse tipo em situações que exigem máxima performance de rede, ou quando você precisa que o container enxergue serviços rodando diretamente no host. Mas, isso faz com que haja *perda de isolamento*.

## 3 - none (sem rede nenhuma)

### docker run --network none ubuntu

O container fica completamente isolado. Sem internet, sem comunicação com nada. Útil para tarefas que processam dados locais e não devem ter acesso externo por segurança.

### Montando uma api que fala com um banco de dados:

*1: Criar a rede*
### docker network create app-network

*2. Subir o banco nessa rede, com um nome*

### docker run -d \
### ---name postgres-db \
### network app-network \
### -e POSTGRESS_PASSWORD=senha123 \
### postgres

*3. Subir a API nessa mesma rede*

### docker run -d \
### --name minha-api \
### --network app-network \
### -p 3000:3000 \
### minha-api-image

No código da API, a string de conexão do banco seria:

### postgresql://user:senha123@postgres-db:5432/meudb

Isso resolve postgres-db para o IP correto automaticamente. Se o container reiniciar e mudar de IP não importa, pois o nome continua funcionando.

## Inspecionando redes:

*Ver detalhes de uma rede (containers que estão nela, IPs, etc.)*
### docker network inspect minha-rede

*Conectar um container que já está rodando a uma rede:*
### docker network connect minha-rede meu-container

*Desconectar*
### docker network disconnect minha-rede meu-container

*Remover redes não utilizadas*
### docker network prune

