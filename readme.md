## Docker Swarm

O **Docker Swarm** ele funciona como se fosse um orquestrador, onde em um *cluster*, ele irá definir por meio de um *dispatcher* qual máquina irá rodar um determinado container, fazendo isso de forma autônama. Caso um certo container falhe, o docker swarm irá tratar isso e reiniciar esse container.

### Docker Machine

A **Docker Machine** irá criar diversas máquinas virtuais, dentro de uma determinada máquina que atráves delas irá ser feita o cluster.

```shell
docker-machine create -d virtualbox vm1
docker-machine ssh vm1
```

O **docker-machine create** irá criar uma nova máqui virtual do docker, onde a flag *-d* irá indicar o driver que será utilizado e por último indicar o nome da máquina virtual do docker. O **docker-machine ssh** irá se conectar com a máquina virtual criada através do nome informado.

```shell
docker swarm init --advertise-addr 192.168.99.100
```

O **docker swarm init** serve para iniciar o docker swarm na máquina em que desejamos ser rodado, é sempre indicado iniciar com o *--advertise-addr* para informarmos o IP no qual desejamos trabalhar.

#### Brincando com Worker

Para entender o que é o **Worker**, vamos fazer uma analogia, digamos que a máquina virtual em que incializamos o docker swarm seja um rei, e que ele deseja ter suditos para trabalhar com ele, então ele irá chamar novas pessoas para trabalharem para ele, do mesmo modo é como docker swarm, ele irá chamar novas máquinas virtuais para serem os seus "suditos" e assim aumentar o cluster. E leais suditos do swarm é o que chamamos de *worker's*.

```shell
docker swarm join --token <token>

docker swarm join-token worker

docker swarm join-token manager
```

O **docker swarm join** é o comando de adicionar o worker, e o **docker swarm join-token worker** é para informar qual é o comando para adicionar o worker todando assim não necessário decorar o comando de join com o tokem gigante.

```shell
docker node ls
docker node ls --format "{{.Hostname}} {{.ManagerStatus}}"
```

O **docker node ls** lista quais são os nós dentro do cluster, porém ele só pode rodar esse comando no manager, mas não poderá ser execultado nos worker's. É importante observar que o *MANAGER STATUS : Leader* indicar o lider do cluster propriamente dito.

```shell
docker node rm <ID NODE>
docker swarm leave
```

O **docker swarm rm** servirá para remover um nó do cluster, porém é importante observar que só ele só poderá ser removido caso não esteja ativo. E o **docker swarm leave** servirá justamente para desativar o nó caso o comando seja feito dentro da determinada máquina no qual desejamos desativar/sair do swarm.

```shell
docker node inspect vm2
docker service create -p 8080:3000 aluracursos/barbearia
```

O **docker node inspect** exibe as expecificações de determinado worker. O **docker service create** cria um container no escopo do swarm, diferentimente do *docker container* que iria criar um container apenas no escopo local.

```shell
docker service ls
docker service ps
```

O **docker service ls** lista os serviços em execução no momento. O **docker service ps** lista os serviços em exercução. Um service feito pelo o swarm será automaticamente roteado para a máquina certa atráves do *routing mesh*.Para fazer backup dp swarm basta copiar os dados de */var/lib/docker/swarm* para uma pasta de backup.

```shell
docker node demote vm1
```
O **docker node demote** rebaixa o nível da máquina no cluster

#### Limitando Nós de Rodarem Serviços

```shell
docker node update --availability drain vm2
```

O **docker node update --availability drain** irá atualizar e desabilitar o status de *active* do *Availability* para o determinada máquina, sendo assim incapaz de rodar serviços.

```shell
docker service update --constraint-add node.role == worker
```

O docker **docker service update --constraint-add** ele irá limitar até quais nós, o serviço poderá trabalhar. Porém, também podemos impor outros tipos de restrições, como id, hostname e o próprio role. Vamos ver alguns exemplos!

#### Serviços Globais e Relicados

O mode replicado é o tipo padrão do serviço no swarm.

```shell
docker service update --replicas 4 [id]
docker service scale ci10k3u7q6ti=5
```

O comando **docker service update --replicas** criar replicas de um determinado serviço em um é informado a _quantidade_ de replicas juntamente com o _id_ do service. O **docker service scale ci10k3u7q6ti=5**, nesse caso definimos 5 réplicas para o serviço. Os dois comandos produzem o mesmo resultado, o segundo é apenas uma forma resumida do primeiro comando.

```shell
docker service create -p 8080:3000 --mode global  aluracursos/barbearia
```

Com a flag **--mode** pode ser definido o modo como o serviço será criado, e o global definirá que todos os nós carregará uma instância desse determinado serviço.

#### Driver Overlay

A rede ingress utiliza o driver overlay, com o escopo swarm como podemos ver atráves do comando *docker network ls*, e qualquer nó dentro do swarm está dentro de uma mesma rede ingress e podemos observar isso por meio do ID dessa rede.

**Service Discovery** é a ideia de localizar serviços atráves do nome, sem a necessidade de informar o IP, atráves do driver overlay isso será possivel. Porém temos ainda a limitação do ingress, que nele só é possivel se cominucar atráves do IP. Para contornar isso podemos criar uma nova *network* onde devemos informar ao service que criaremos, justemente para eles usarem essa rede em especifico. Pois o *User-Defined Overlay* é criada de forma *lazy* para nós workers. E por fim poderemos udar o conceito de Service Discovery atráves dessas rede "customizada" em que criamos.

Por mais que o driver overlay seja responsável por comunicar múltiplos hosts em uma mesma rede, também podemos conectar containers em escopo local criados com o comando docker container run em redes criadas com esse driver.

Para isso, basta no momento da criação da rede utilizarmos a flag --attachable:
```shell
docker network create -d overlay --attachable my_overlay
```
Com o comando acima, conseguiremos conectar tanto serviços como containers "standalone" em nossa rede my_overlay.

Dica 😉:
```shell
docker stack deploy --compose-file docker-compose.yml vote
```