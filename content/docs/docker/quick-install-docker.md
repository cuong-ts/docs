# Quick install docker

install docker with yum:

```
$ yum install docker
```

add your user to docker group to able run `docker` command

```
$ sudo usermod -a -G docker your_user
```

 make sure docker run on start up

```bash
systemctl enable docker
service docker start
```



