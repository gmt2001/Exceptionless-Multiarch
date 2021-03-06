ARG EXCEPTIONLESS_UI_VERSION
ARG EXCEPTIONLESS_DOTNET_VERSION
ARG EXCEPTIONLESS_DOTNET_SDK_VERSION
ARG DOCKER_USER
FROM ${DOCKER_USER}/exceptionless-ui:${EXCEPTIONLESS_UI_VERSION} AS ui

FROM mcr.microsoft.com/dotnet/sdk:${EXCEPTIONLESS_DOTNET_SDK_VERSION} AS build
WORKDIR /app

ARG TARGETPLATFORM
ARG BUILDPLATFORM

COPY ./*.sln ./NuGet.Config ./
COPY ./build/*.props ./build/

# Copy the main source project files
COPY src/*/*.csproj ./
RUN for file in $(ls *.csproj); do mkdir -p src/${file%.*}/ && mv $file src/${file%.*}/; done

# Copy the test project files
COPY tests/*/*.csproj ./
RUN for file in $(ls *.csproj); do mkdir -p tests/${file%.*}/ && mv $file tests/${file%.*}/; done

RUN dotnet restore

# Copy everything else and build app
COPY . .

RUN dotnet build -c Release

# job-publish

FROM build AS job-publish
WORKDIR /app/src/Exceptionless.Job

RUN dotnet publish -c Release -o out

# job

FROM mcr.microsoft.com/dotnet/aspnet:${EXCEPTIONLESS_DOTNET_VERSION} AS job
WORKDIR /app
COPY --from=job-publish /app/src/Exceptionless.Job/out ./
ENTRYPOINT [ "dotnet", "Exceptionless.Job.dll" ]

# api-publish

FROM build AS api-publish
WORKDIR /app/src/Exceptionless.Web

RUN dotnet publish -c Release -o out

# api

FROM mcr.microsoft.com/dotnet/aspnet:${EXCEPTIONLESS_DOTNET_VERSION} AS api
WORKDIR /app
COPY --from=api-publish /app/src/Exceptionless.Web/out ./
ENTRYPOINT [ "dotnet", "Exceptionless.Web.dll" ]

# app

FROM mcr.microsoft.com/dotnet/aspnet:${EXCEPTIONLESS_DOTNET_VERSION} AS app

WORKDIR /app
COPY --from=api-publish /app/src/Exceptionless.Web/out ./
COPY --from=ui /app ./wwwroot
COPY --from=ui /usr/local/bin/bootstrap /usr/local/bin/bootstrap
COPY ./build/docker-entrypoint.sh ./
COPY ./build/supervisord.conf /etc/

ENV EX_ConnectionStrings__Storage=provider=folder;path=/app/storage \
    EX_RunJobsInProcess=true \
    ASPNETCORE_URLS=http://+:80 \
    EX_Html5Mode=true

EXPOSE 80

ENTRYPOINT ["/app/docker-entrypoint.sh"]
CMD [ "dotnet", "Exceptionless.Web.dll" ]