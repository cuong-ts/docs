# Checking file when build docker



Build this Dockerfile

```text
FROM busybox

RUN mkdir /tmp/build/
COPY . /tmp/build/
RUN du -ah tmp/build | sort -n -r | head -n 100
```

```bash
docker build .
```

