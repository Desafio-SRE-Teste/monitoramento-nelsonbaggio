# Como rodar o SRE Lab

## Com docker

- Inicializar o projeto
```console
docker run --name sre-lab -v {path do projeto}/lab/src:/usr/share/nginx/html -p 40636:80 -d nginx
```
- Acessar localmente em [localhost:40636/](http://localhost:40636/)
- Encerrar o projeto
```console
docker rm -f sre-lab
```

## Com docker-compose

- Inicializar o projeto
```console
docker-compose -f lab/docker-compose.yaml up -d
```
- Acessar localmente em [localhost:40636/](http://localhost:40636/)
- Encerrar o projeto
```console
docker-compose -f lab/docker-compose.yaml down
```
