---
description: Best practices for building containers
---

# Best practices for building containers

### Dockerfile tips and tricks

Consider the example below.

```yaml
FROM debian

# Copy application files
COPY . /app

# Install required system packages
RUN apt-get update
RUN apt-get -y install imagemagick curl software-properties-common gnupg vim ssh
RUN curl -sL https://deb.nodesource.com/setup_10.x | bash -
RUN apt-get -y install nodejs

# Install NPM dependencies
RUN npm install --prefix /app
EXPOSE 80
CMD ["npm", "start", "--prefix", "app"]
```

It takes **127.8 seconds** to build the image and it is **554MB**. Let's improve this result by following some good practices!!

#### 1- Order matters for caching

The least frequently changing statements should be appear first. This is because when you change or modify a line in the dockerfile and its cache gets invalidated, the subsequent line will break due to these changes. Hence, you need to keep the most frequently changing lines as last as possible.

```yaml
FROM debian

# Install required system packages
RUN apt-get update
RUN apt-get -y install imagemagick curl software-properties-common gnupg vim ssh
RUN curl -sL https://deb.nodesource.com/setup_10.x | bash -
RUN apt-get -y install nodejs

# Copy application files
COPY . /app

# Install NPM dependencies
RUN npm install --prefix /app
EXPOSE 80
CMD ["npm", "start", "--prefix", "app"]
```

Rebuild the image using the same command, but avoiding the installation of the system packages. This is the result: it takes **5.8 seconds** to build!! The improvement is huge!!

#### 2- ****You **should be more specific about the files you copy** to make sure that you are not invalidating the cache with changes that do not affect the application.

```yaml
FROM debian

# Install required system packages
RUN apt-get update
RUN apt-get -y install imagemagick curl software-properties-common gnupg vim ssh
RUN curl -sL https://deb.nodesource.com/setup_10.x | bash -
RUN apt-get -y install nodejs

# Copy application files
COPY package.json server.js /app

# Install NPM dependencies
RUN npm install --prefix /app
EXPOSE 80
CMD ["npm", "start", "--prefix", "app"]
```

> NOTE: Use "COPY" instead of "ADD" when possible. Both commands do basically the same thing, but "ADD" is more complex: it has extra features like extracting files or copying them from remote sources. From a security perspective too, using "ADD" increases the risk of malware injection in your image if the remote source you are using is unverified or insecure.

#### 3- Avoiding packaging dependencies that you do not need

The current Dockerfile includes the **ssh** system package. However, you can access your containers using the `docker exec` command instead of ssh'ing into them. Apart from that, it also includes **vim** for debugging purposes, which can be installed when required, instead of packaged by default. Both packages are removable from the image.

In addition, you can configure the package manager to avoid installing packages that you don't need. To do so, use the `--no-install-recommends` flag on your `apt-get` calls:

```yaml
FROM debian

# Install required system packages
RUN apt-get update
RUN apt-get -y install --no-install-recommends imagemagick curl software-properties-common gnupg
RUN curl -sL https://deb.nodesource.com/setup_10.x | bash -
RUN apt-get -y install --no-install-recommends nodejs

# Copy application files
COPY package.json server.js /app

# Install NPM dependencies
RUN npm install --prefix /app
EXPOSE 80
CMD ["npm", "start", "--prefix", "app"]
```

#### 4- Try to chain all the cacheable units.

```yaml
FROM debian

# Install required system packages
RUN apt-get update && apt-get -y install --no-install-recommends imagemagick curl software-properties-common gnupg \
 && curl -sL https://deb.nodesource.com/setup_10.x | bash - 
 && apt-get -y install --no-install-recommends nodejs

# Copy application files
COPY package.json server.js /app

# Install NPM dependencies
RUN npm install --prefix /app
EXPOSE 80
CMD ["npm", "start", "--prefix", "app"]
```

#### 5- Finally, remove the package manager cache to reduce the image size

```yaml
FROM debian

# Install required system packages
RUN apt-get update && apt-get -y install --no-install-recommends imagemagick curl software-properties-common gnupg \
 && curl -sL https://deb.nodesource.com/setup_10.x | bash - 
 && apt-get -y install --no-install-recommends nodejs 
 && rm -rf /var/lib/apt/lists/*

# Copy application files
COPY package.json server.js /app

# Install NPM dependencies
RUN npm install --prefix /app
EXPOSE 80
CMD ["npm", "start", "--prefix", "app"]
```

If you rebuild the image again…

```text
$ docker build . -t express-image:0.0.3
```

… The image was reduced to **340MB!!** That's almost half of its original size.







