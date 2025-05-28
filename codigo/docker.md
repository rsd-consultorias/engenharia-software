## Conhecendo a command line interface do Docker (Docker CLI)

### Pasta raíz de todos os códigos: aula-docker

Digitar no terminal

```bash

# verificação se o docker foi devidamente instalado
docker

# verificação da versão atual do docker
docker -v

# coletando informações básicas do docker
docker info

# puxando a primeira imagem do registry (Docker Hub)
docker pull hello-world

# listando as imagens que estão na máquina
docker images

# rodando o primeiro container docker da imagem que acabou de ser puxada do registry
docker run hello-world

# listando todos os containers e suas informações básicas
docker ps -a

# listando apenas os containers ativos e suas informações básicas
docker ps

```

## Criando uma imagem via Dockerfile e levantando um container baseado nessa imagem

Digitar no terminal

```bash
# entrar na pasta 1-criar-imagem-script
cd 1-criar-imagem-script

# gerar a imagem a partir do Dockerfile com o nome imagem-script
docker build . -t imagem-script

# subir o container da imagem com o nome imagem-script
docker run imagem-script

# excluindo o container que acabou de ser criado através do seu id
# verificando o id do container
docker ps -a

# removendo o container utilizando o seu id, já que o container não tem nome
docker rm `ID_DO_CONTAINER`

# verificando a situação dos containers após a remoção do container com a imagem imagem-script
docker ps -a

# subir o container da imagem imagem-script e eliminar ele após rodar todas as linhas do Dockerfile
docker run --rm imagem-script

# verificando a situação dos containers após a remoção do container com a imagem imagem-script
docker ps -a
```

Alterar arquivo

```python
# alterar a linha 2 do arquivo script.py
print("Esse script foi alterado")
```

Digitar no terminal

```bash
# subir novamente o script do imagem-script com o argumento --rm
docker run --rm imagem-script

# gerar a imagem a partir do Dockerfile com o nome imagem-script-alterado
docker build . -t imagem-script-alterado

# subir o script do imagem-script-alterado com o argumento --rm
docker run --rm imagem-script-alterado
```

## Criando um container que não é interrompido após rodar todas as linhas do Dockerfile

Digitar no terminal

```bash

# acessar o diretório anterior ao atual
cd ..

# entrar na pasta 2-criar-imagem-ininterrupta
cd 2-criar-imagem-ininterrupta

# gerar a imagem ininterrupta com o nome imagem-ininterrupta com adição de tag de versão
docker build . -t imagem-ininterrupta:1.0

# subir o container imagem-ininterrupta com o nome imagem-ininterrupta
docker run -d --name imagem-ininterrupta imagem-ininterrupta:1.0

# verificar os containers que estão rodando na máquina
docker ps -a

# acessar o terminal docker container via linha de comando
docker exec -it imagem-ininterrupta bash

# acessar o kernel do python dentro do container
python

```

Digitar no kernel do Python

```python

# testando o python dentro do container
print("python dentro do container")

# sair do kernel do python
exit()

```

Digitar no terminal

```bash
# verificar as pastas e os arquivos do container usando o terminal do container
ls

# criar um arquivo dentro do container
touch teste

# parar o container
docker stop imagem-ininterrupta

# remover o container
docker rm imagem-ininterrupta

# o container também pode ser removido diretamento usando a flag -f
# docker rm -f imagem-ininterrupta

# subir novamente o container para verificar se o arquivo criado ainda está lá
docker run -d --name imagem-ininterrupta imagem-ininterrupta:1.0

# acessar o terminal docker container via linha de comando
docker exec -it imagem-ininterrupta bash

# verificar as pastas e os arquivos do container usando o terminal do container
ls

# sair do container
exit

```

## Criando volumes compartilhados host:container

Digitar no terminal

```bash

# acessar o diretório anterior ao atual
cd ..

# entrar na pasta 3-containers-e-volumes
cd 3-containers-e-volumes

# gerar a imagem containers-e-volumes com o nome containers-e-volumes na versão 1.0
docker build . -t containers-e-volumes:1.0

# subir o container 3-containers-e-volumes com o nome containers-e-volumes e o volume host:container
docker run -d --name containers-e-volumes -v ${PWD}/volume:/app containers-e-volumes:1.0 # Windows -> PowerShell
# docker run -d --name containers-e-volumes -v ./volume:/app containers-e-volumes:1.0 # Linux -> bash

# verificar os containers que estão rodando na máquina
docker ps -a

# acessar o terminal docker container via linha de comando
docker exec -it containers-e-volumes bash

# verificar arquivos dentro do container
ls

# criar arquivos teste1.txt e teste2.txt fora do container, na pasta volume

# criar o arquivo teste3.txt dentro do container
touch teste4.txt

# caso dê algum erro de permissão, é só deletar e criar novamente a pasta com o mesmo nome.

# sair do container
exit

# verificar e inspecionar o container utilizanco o comando inspect
docker inspect containers-e-volumes

```

## Operações básicas como parar, rmover, iniciar e reiniciar containers via Docker CLI

```bash

# parar o container com o nome imagem-ininterrupta
docker stop imagem-ininterrupta

# parar o container com o nome containers-e-volumes
docker stop containers-e-volumes

# verificando situação dos containers
docker ps -a

# iniciando novamente o container com o nome imagem-ininterrupta
docker start imagem-ininterrupta

# iniciando novamente o container com o nome containers-e-volumes
docker start containers-e-volumes

# verificando situação dos containers
docker ps -a

# parando todos os containers que estão em execução
docker stop $(docker ps -a -q)

# verificando situação dos containers
docker ps -a

# removendo todos os containers que estão parados
docker rm $(docker ps -a -q)

# verificando situação dos containers
docker ps -a
```

## Operações básicas como parar, rmover, iniciar e reiniciar containers via extensão Docker do VS Code

```bash
# inicializando dois containers da mesma imagem
docker run -d imagem-ininterrupta:1.0
docker run -d containers-e-volumes:1.0

# parar container imagem-ininterrupta
# parar container containers-e-volumes
# inicializar container imagem-ininterrupta
# inicializar container containers-e-volumes
# acessar terminal do container imagem-ininterrupta
# sair do terminal do container imagem-ininterrupta
# acessar terminal do conntainer containers-e-volumes
```

## Mapeamento de portas entre host e container

Digitar no terminal

```bash

# acessar o diretório anterior ao atual
cd ..

# entrar na pasta 4-container-api
cd 4-container-api

# gerar a imagem do container-api  com o nome container-api com uma tag de versão 1.0
docker build . -t container-api:1.0

# subir o container do container-api com o nome container-api compartilhando o diretório fonte da aplicação
docker run -d --name container-api -v ${PWD}/app:/api/app container-api:1.0 # Windows -> PowerShell
# docker run -d --name container-api -v ./app:/api/app container-api:1.0 # Linux -> bash

# tentar acessar a aplicação que está rodando dentro do container no browser pela porta 8000
# http://localhost:8000

# remover o container via extensão Docker

# adicionar o mapeamento de portas máquina:container para acessar o container
docker run -d -p 8000:8000 --name container-api -v ${PWD}/app:/api/app container-api:1.0 # Windows -> PowerShell
# docker run -d -p 8000:8000 --name container-api -v ./app:/api/app container-api:1.0 # Linux -> bash

# resposta da requisição
# JSON -> {message: "Estou dentro do container.."}

# alterar o texto da requisição para "A aplicação foi alterada..." no arquivo main.py

# acessar novamente o browser na porta 8000 e receber a resposta da requisição
# JSON -> {"message": "A aplicação foi alterada..."}

# remover o container container-api via extensão docker

# alterando o mapeamento das portas para que a porta 9000 redirecione as requisições para a porta 8000 do container
docker run -d -p 9000:8000 --name container-api -v ${PWD}/app:/api/app container-api:1.0 # Windows -> PowerShell
# docker run -d -p 9000:8000 --name container-api -v ./app:/api/app container-api:1.0 # Linux -> bash

# acessar a porta 9000 pelo browser e ser redirecionado para a porta 8000 do container
# receber a resposta da requisição da aplicação
# JSON -> {"message": "A aplicação foi alterada..."}

# remover o container container-api via extensão docker

# alterando o mapeamento das portas para que a porta 8000 redirecione as requisições para a porta 9000 do container
docker run -d -p 8000:9000 --name container-api -v ${PWD}/app:/api/app container-api:1.0 # Windows -> PowerShell
# docker run -d -p 8000:9000 --name container-api -v ./app:/api/app container-api:1.0 # Linux -> bash

# verificar mapeamento de portas com o comando inspect
docker inspect container-api

# a aplicação não tem mais o acesso da porta do servidor do container via mapeamento de portas

# remover o container container-api via extensão docker

```

## Orquestrando múltiplos containers usando Docker Compose

Digitar no terminal

```bash

# acessar o diretório anterior ao atual
cd ..

# entrar na pasta 5-intro-docker-compose
cd 5-intro-docker-compose

# gerar as imagens de todos os serviços do arquivo docker-compose.yaml
docker compose build

# verificar as imagens criada depois do build
docker images

# rodar os containers de todos os serviços do arquivo docker-compose.yaml com terminal desatachado
docker compose up -d

# verificar as rotas da API
# / -> {"message": "Estou dentro do container..."}
# /variavel_ambiente -> {"USERNAME": "DOCKER"}

# derrubar o conjunto de containers utilizando o comando down
docker compose down

# verificar a situação atual dos containers
docker ps -a

```

## Desenvolvendo uma aplicação Full Stack (front-end e back-end) utilizando containers que se comunicam em uma mesma rede

Digitar no terminal

```bash

# acessar o diretório anterior ao atual
cd ..

# entrar na pasta 6-ambiente-docker-compose
cd 6-ambiente-docker-compose

# gerar as imagens de todos os serviços do arquivo docker-compose.yaml
docker compose build

# rodar os containers de todos os serviços do arquivo docker-compose.yaml com terminal desatachado
docker compose up -d

# remover todos os container, volumes e redes utilizando o comando down com a tag -v
docker compose down -v

```

## Subindo uma imagem para o Docker Hub (Registry)

Digitar no terminal

```bash

# acessar o diretório anterior ao atual
cd ..

# entrar na pasta 7-subir-imagem-registry
cd 7-subir-imagem-registry

# gerar a imagem com as instruções do arquivo Dockerfile
docker build . -t jogo-par-impar:1.0

# testar a imagem criada localmente
docker run -it jogo-par-impar:1.0

# gerar novamente a imagem com a tag padrão para subir para o seu repositório particular do Docker Hub
docker build . -t `USUARIO_DOCKER_HUB`/jogo-par-impar:1.0

# subir a imagem para o seu repositório particular do Docker Hub (Registry) com o seu usário do Docker Hub
docker push `USUARIO_DOCKER_HUB`/jogo-par-impar:1.0

# verificando a imagem no Docker Hub (Registry)
# https://hub.docker.com

# removendo a imagem local com o mesmo nome
docker rmi `USUARIO_DOCKER_HUB`/jogo-par-impar:1.0

# verificar as imagens que estão disponíveis de forma local no Docker
docker images

# puxando a imagem recem armazenada no Docker Hub para o repositório de imagens local
docker pull `USUARIO_DOCKER_HUB`/jogo-par-impar:1.0

# testar a imagem recém puxada para o repositório local do docker
docker run -it `USUARIO_DOCKER_HUB`/jogo-par-impar:1.0

```
