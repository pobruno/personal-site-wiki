
# Setup Locally
```bash
git clone https://github.com/pobruno/personal-site-wiki
cd pobruno.github.io
bundle exec jekyll serve --port 12001
```
Access http://localhost:12001/ in web browser, enjoy!

Run for production
```sh
JEKYLL_ENV=production bundle exec jekyll serve --port 12001
```
Run for nas
```
JEKYLL_ENV=nas bundle exec jekyll serve --port 12001
```
# Docker
Create image with Dockerfile.
```bash
docker build -t pobruno/pobruno.github.io .
```
## 1. Using Jekyll Image
Create image, build in image.
```sh
FROM jekyll/minimal:4.2.0

WORKDIR /usr/src/app

COPY . .

RUN chmod 777 /usr/src/app -R

RUN jekyll build
```
Create container. `jekyll serve --skip-initial-build` will skip build when container is launched.
```sh
docker run --name pobruno.github.io -p 12080:4000 -d pobruno/pobruno.github.io jekyll serve --skip-initial-build 
```
Access `http://localhost:12080/` in browser.

Check the image size. It is 1.33GB.
```
pobruno-MacBook-Pro:pobruno.github.io bruno$ docker images
REPOSITORY                              TAG                 IMAGE ID            CREATED             SIZE
pobruno/pobruno.github.io         latest              be0228955d5c        2 minutes ago       1.33GB
```

## 1.2 Staging build
Reduce the image size by just copying the output static files.
Two steps:
* Build
* Copy the output `_site` to image
```sh
# https://github.com/pobruno/personal-site-wiki/blob/master/README.md
#
# Builder stage.
# This state compile our jekyll site to get static files
#
FROM jekyll/minimal:4.2.0 AS builder

WORKDIR /srv/jekyll

COPY . /srv/jekyll

RUN jekyll build

#
# Production stage.
# This state compile get back the static files from builder stage
#
FROM jekyll/minimal:4.2.0

COPY --from=builder /srv/jekyll/_site /srv/jekyll/_site
```
Create container. `jekyll serve --skip-initial-build` will skip build when container is launched.
```sh
docker run --name pobruno.github.io -p 12080:4000 -d pobruno/pobruno.github.io jekyll serve --skip-initial-build 
```
Access `http://localhost:12080/` in browser.

Check the new image, size is reduced to 807MB.
```sh
pobruno-MacBook-Pro:pobruno.github.io bruno$ docker images
REPOSITORY                              TAG                 IMAGE ID            CREATED             SIZE
pobruno/pobruno.github.io         latest              2493fd57f4da        2 minutes ago       807MB
```

## 1.3 Deploy to Nginx
Build with Jekyll, deploy with nginx.
```sh
# https://github.com/envygeeks/jekyll-docker/blob/master/README.md
#
# Builder stage.
# This state compile our jekyll site to get static files
#
FROM jekyll/minimal:4.2.0 AS builder

WORKDIR /srv/jekyll

COPY . /srv/jekyll

RUN jekyll build

#
# Production stage.
# This state compile get back the static files from builder stage
#
FROM nginx:1.19.8-alpine

COPY ./nginx.conf /etc/nginx/conf.d/default.conf

## We just need the build to execute the command
COPY --from=builder /srv/jekyll/_site /usr/share/nginx/html/
```
Create container. `jekyll serve --skip-initial-build` will skip build when container is launched.
```sh
docker run --name pobruno.github.io -p 12080:80 -d pobruno/pobruno.github.io
docker run --name pobruno.github.io -p 12080:80 -v=/Users/johnny/Docker/nginx:/etc/nginx/conf.d/ -d pobruno/pobruno.github.io
```
Access `http://localhost:12080/` in browser.

Check the new image, size is reduced to 620MB.
```sh
pobruno-MacBook-Pro:nginx bruno$ docker images
REPOSITORY                              TAG                 IMAGE ID            CREATED             SIZE
pobruno/pobruno.github.io               latest              98cc74b754d8        4 seconds ago       620MB
```

## 1.4 Serve in Container
Create container with compiling and serving.
```sh
docker run --rm \
  --volume="/Users/bruno/pobruno/github/pobruno.github.io:/srv/jekyll" \
  -p 12001:12001 \
  jekyll/jekyll:3.8 \
  jekyll serve --port 12001
```
Access http://localhost:12001/.

## 1.5 Build without Serve
```sh
docker run --rm \
  --volume="/Users/bruno/pobruno/github/pobruno.github.io:/srv/jekyll" \
  -it jekyll/jekyll:3.8 \
  jekyll build
```
Site will be compiled and generated to `_site` folder.

# Deployment
Follow tutorial [Deploying Jekyll Website to Netlify](https://pobruno.github.io/tutorial/deploying-jekyll-website-to-netlify) to continuously deploy this personal website to Netlify.

# Tutorial
Read tutorial [Setting up Jekyll on Ubuntu and macOS](https://pobruno.github.io/tutorial/setting-up-jekyll-on-ubuntu-and-macos) and follow-on tutorials to learn how to build personal website and deploy to GitHub Pages.
