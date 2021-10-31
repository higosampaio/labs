# Docker Labs

<details open="open">
  <summary>Table of Contents</summary>
  <ol>
    <li><a href="#comandos-básicos">Comandos básicos</a></li>
    <li><a href="./hello-world/index.md">Rodando um Hello World</a></li>
    <li><a href="./ubuntu/index.md">Executando o Ubuntu</a></li>
    <li><a href="./publishing-doors/index.md">Publicando portas com nginx</a></li>
    <li><a href="./commands-outside/index.md">Executar comandos diretamente em um container</a></li>
    <li><a href="./bind-mounts/index.md">Bind Mounts</a></li>
    <li><a href="./volumes/index.md">Trabalhando com volumes</a></li>
    <li><a href="./imagens/index.md">Trabalhando com imagens</a></li>
    <li><a href="./entrypoint-vs-cmd/index.md">ENTRYPOINT vs CMD</a></li>
    <li><a href="#publicando-imagem-no-dockerhub">Publicando imagem no DockerHub</a></li>
    <li><a href="#networks">Networks</a></li>
    <li><a href="#otimizando-imagens-utilizando-multistage-building">Otimizando imagens utilizando Multistage Building</a></li>
  </ol>
</details>

## Comandos básicos

```docker
docker ps # Listar containers ativos
docker ps -a # Listar todos os containers ativos e não ativos
docker start container_id # Levantar um container
docker stop container_id # Parar um container
docker rm container_id # Remover um container
docker rm container_id -f # Remover com o -f (force) para o caso do container rodando
docker rm $(docker ps -a -q) -f # Remover todos os containers de uma vez
```

## Publicando imagem no DockerHub

Criando uma imagem baseada no nginx e modificando a página index.html

```docker
FROM nginx:latest

# Copiando o dir html da máquina e sobrescrevendo o dir html do nginx
COPY html /usr/share/nginx/html

# Copiando o entrypoint e o cmd do nginx para deixar a imagem rodando
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["nginx", "-g", "daemon off;"]
```

Testando a imagem localmente antes de subir

```docker
# Buildando o imagem
docker build -t higosampa/nginx-fullcycle .

# Rodando a imagem
docker run --rm -d -p 8080:80 higosampa/nginx-fullcycle
```

Publicando a imagem no DockerHub

```docker
# Primeiro precisamos fazer login no DockerHub
docker login

# Subindo a imagem
docker push higosampa/nginx-fullcycle
```

## Networks

As networks possibilitam a comunicação entre containers.

1. Network do tipo **_bridge_**: É o tipo de network padrão e mais comum. Normalmente utilizadas para fazer um container se comunicar com outro.

2. Network do tipo **_host_**: Permite mesclar a network do docker com a network do host do docker (minha máquina). A minha máquina terá a capacidade de acessar uma porta direta no container sem precisar fazer exposição de porta.

3. Network do tipo **_overlay_**: Permite a comunicação de containers entre máquinas diferentes. Geralmente utilizado em ambientes de maior escalabilidade.

4. Network do tipo macvlan: Permite setar um macaddress em um container e pode fazer parecer que é uma network que está plugada na sua rede.

5. Network do tipo none: Define que não terá nenhuma rede e garante que o container irá rodar de forma isolada.

### Trabalhando com bridge

Verificando os comandos disponíveis para trabalhar com networks

```docker
docker networks
```

Identificando os containers que fazem parte de uma network

```docker
docker network inspect bridge
```

Criando uma nova network

```docker
docker network create --driver bridge minharede
```

Criando containers na mesma rede

```docker
docker run -dit --name ubuntu1 --network minharede bash
docker run -dit --name ubuntu2 --network minharede bash
```

Conectando um container já existente em uma rede

```docker
docker network connect minharede container_name
```

## Otimizando imagens utilizando Multistage Building

A otimização de imagens é comumente utilizada para colocar imagens em ambiente de produção. Quanto mais enxuta a imagem ficar, mais leve e menos vulnerável a falhas ela fica. Geralmente utiliza-se imagens baseadas no Alpine Linux para reduzir o tamanho de uma imagem que você queira otimizar.

O Multistage Build é uma forma de se trabalhar o build de uma imagem em duas ou mais etapas, e dessa forma podemos fazer um build principal e um secundário baseado no Apine Linux para otimizar. Neste <a href="laravel/Dockerfile.prod">Dockerfile</a> há um exemplo.
