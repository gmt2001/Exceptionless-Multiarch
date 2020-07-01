ARG DOCKER_USER
ARG PUPPETEER_VERSION
FROM ${DOCKER_USER}/puppeteer:${PUPPETEER_VERSION} AS base
WORKDIR /app

RUN apt update -y \
     && apt install -y --no-install-recommends git --no-install-recommends \
     && apt autoremove -y \
     && apt clean

COPY src/package.json src/package-lock.json src/bower.json src/.bowerrc ./

# dependencies

FROM base AS dependencies
WORKDIR /app

RUN npm set progress=false && \
    npm config set depth 0

RUN npm ci && \
    npx bower --allow-root install

# build

FROM dependencies AS build
WORKDIR /app

ARG EXCEPTIONLESS_UI_VERSION
ENV UI_VERSION=$EXCEPTIONLESS_UI_VERSION

COPY src/. .

USER root
RUN npx grunt build
RUN sed -i 's/<head>/<head <base href="\/" \/>/' /app/index.html

# testrunner

FROM build AS testrunner
WORKDIR /app

ENTRYPOINT [ "npx", "grunt", "test" ]

# ui

FROM nginx:stable as ui
WORKDIR /app
COPY --from=build /app/dist ./
COPY bootstrap /usr/local/bin/bootstrap
COPY nginx-site.conf /etc/nginx/conf.d/default.conf

RUN chmod +x /usr/local/bin/bootstrap && \
  echo "daemon off;" >> /etc/nginx/nginx.conf

ENTRYPOINT [ "bootstrap" ]