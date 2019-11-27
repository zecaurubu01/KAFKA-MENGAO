*Tutorial Apache Kafka*

*Alunos: Humberto B. e Marcos T.*

# Serviço de mensageria com Kafka - Utilizando Ubuntu 18.04

Esse é um tutorial basico que consiste em fazer trocas de mensagens entre cliente e produtor.


### 1. Faça o download do código

Utilize o link abaixo para fazer o download do arquivo com o código do serviço de mensagens com Kafka.

[Download](http://ftp.unicamp.br/pub/apache/kafka/2.3.0/kafka_2.12-2.3.0.tgz)

Apos fazer o download do código, extraia o arquivo.

```console
tar -xzf kafka_2.12-2.3.0.tgz
```

Entre no diretório do Kafka.

```console
cd kafka_2.12-2.3.0
```

### 2. Iniciando servidor

OBS: Para dar inicio essa etapa, é necessario ter o JAVA instalado.

Verifique se você tem o JAVA instalado e qual a sua versão. Para verifcar sua versão execute o seguinte comando.

```console
java --version
```

Quando executamos esse tutorial, a versão utilizada foi:

	 openjdk 11.0.4 2019-07-16                                                                                               
	 OpenJDK Runtime Environment (build 11.0.4+11-post-Ubuntu-1ubuntu218.04.3)                                               
	 OpenJDK 64-Bit Server VM (build 11.0.4+11-post-Ubuntu-1ubuntu218.04.3, mixed mode, sharing)

Caso não tenha o java instalado, execute os seguintes comandos:

```console
sudo apt install default-jre

sudo apt install openjdk-11-jre-headless

sudo apt install openjdk-8-jre-headless

```

### 2.1 Iniciando servidor ZooKepeer

Agora que já temos o JAVA instalado, iremos iniciar o servidor ZooKeeper.

```console
bin/zookeeper-server-start.sh config/zookeeper.properties
```

### 2.2 Iniciando servidor Kafka

```console
bin/kafka-server-start.sh config/server.properties
```

### 3. Criando um tópico

Vamos criar um tópico chamado "boot" com uma única partição e apenas uma réplica:

```console
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic boot
```
Agora podemos ver esse tópico se executarmos o comando list topic:


```console
bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

### 4. Enviando algumas mensagens

O Kafka vem com um cliente de linha de comando que recebe as entradas de um arquivo ou da entrada padrão e as envia como mensagens para o cluster Kafka. Por padrão, cada linha será enviada como uma mensagem separada.

Execute o produtor e digite algumas mensagens no console para enviar ao servidor.

```console
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic boot
```
### 5. Iniciando um consumidor

Kafka também tem um consumidor de linha de comando que enviará mensagens para a saída padrão.


```console
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic boot --from-beginning
```
### 6. Configurando um cluster Multi-Broker

Até agora, temos competido com um único broker, mas isso não é divertido. Para Kafka, um único broker é apenas um cluster de tamanho um, portanto, nada muda muito além de iniciar mais algumas instâncias do broker. Mas apenas para ter uma ideia, vamos expandir nosso cluster para três nodes (ainda todos em nossa máquina local).

Primeiro, criamos um arquivo de configuração para cada um dos intermediários:

```console
cp config/server.properties config/server-1.properties

cp config/server.properties config/server-2.properties
```
Agora vamos editar nossos arquivos e definir as seguinter propriedades:

 	config/server-1.properties:
       broker.id=1
       listeners=PLAINTEXT://:9093
       log.dirs=/tmp/kafka-logs-1
		
	config/server-2.properties:
       broker.id=2
       listeners=PLAINTEXT://:9094
       log.dirs=/tmp/kafka-logs-2

O broker.id é o nome exclusivo e permanente de cada node no cluster. Temos que substituir o diretório das portas e logs apenas porque estamos executando tudo isso na mesma máquina e queremos impedir que os corretores tentem se registrar na mesma porta ou substituir os dados uns dos outros.

Já temos o Zookeeper e nosso node único iniciado, portanto, precisamos apenas iniciar os dois novos nodes:


```console
bin/kafka-server-start.sh config/server-1.properties

bin/kafka-server-start.sh config/server-2.properties
```
Agora vamos criar um novo tópico com fator de replicação 3.

```console
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 3 --partitions 1 --topic triboot
```
Agora que temos um cluster, como podemos saber qual broker está fazendo o que? Para ver isso, execute o comando para descrever os tópicos:

```console
bin/kafka-topics.sh --describe --bootstrap-server localhost:9092 --topic triboot
```
### 7. Matando o Lider

Agora que já identificamos quem o lider, iremos derruba-lo. Assim, podemos verificar quem assume a nova liderança.
