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

Quando criamos um ###container, criamos uma camada extra por cima de imagens.
Quando imagens são criadas, elas são apenas R/O, ou seja, não podemos modificá-las após sua criação.
Os containers nos permitem criar uma camada de R/W, ou seja, quando criamos um container, criamos essa camada temporária que nos permite alterar essas imagens. 
Quando esse container é parado, essa camada extra é encerrada.

### Por que containers são tão leves?
Além deles serem apenas processos no nosso sistema, podemos dizer que quando um container entra em execução, estamos sempre reaproveitando a mesma imagem.
Como a imagem é apenas de leitura, podemos ter vários containers baseados na mesma imagem. A diferença é que cada um desses containers terá apenas uma camada diferente de read-write, e como essa camada é leve, para manter a rápida performance temos uma reutilização de imagens para múltiplos containers.

### docker history
Comando que nos possibilita visualizar camadas de uma imagem.


