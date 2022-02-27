# Remove unused docker resource

## TL;DR

remove all the things, such as images, container, network:

```
$ docker system prune
```

stop all container running:

```text
$ docker stop $(docker ps -q) 
```

remove all container:

```text
$ docker rm $(docker ps -a -q)
```

remove unused volumes:

```text
$ docker volume prune
```

remove unused images:

```text
$ docker image prune
```



