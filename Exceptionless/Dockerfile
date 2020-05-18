FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS build
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

RUN set -x \
  && if [ "${TARGETPLATFORM}" = "linux/arm/v7" ] ; then dotnet restore -r linux-arm ; fi

RUN set -x \
  && if [ "${TARGETPLATFORM}" = "linux/arm64" ] ; then dotnet restore -r linux-arm64 ; fi

RUN set -x \
  && if [ "${TARGETPLATFORM}" = "linux/amd64" ] ; then dotnet restore ; fi

# Copy everything else and build app
COPY . .
RUN set -x \
  && if [ "${TARGETPLATFORM}" = "linux/arm/v7" ] ; then dotnet build -c Release -r linux-arm ; fi

RUN set -x \
  && if [ "${TARGETPLATFORM}" = "linux/arm64" ] ; then dotnet build -c Release -r linux-arm64 ; fi

RUN set -x \
  && if [ "${TARGETPLATFORM}" = "linux/amd64" ] ; then dotnet build -c Release ; fi


# job-publish

FROM build AS job-publish
WORKDIR /app/src/Exceptionless.Job

RUN dotnet publish -c Release -o out

# job

FROM mcr.microsoft.com/dotnet/core/aspnet:3.1 AS job
WORKDIR /app
COPY --from=job-publish /app/src/Exceptionless.Job/out ./
ENTRYPOINT [ "dotnet", "Exceptionless.Job.dll" ]

# api-publish

FROM build AS api-publish
WORKDIR /app/src/Exceptionless.Web

RUN dotnet publish -c Release -o out

# api

FROM mcr.microsoft.com/dotnet/core/aspnet:3.1 AS api
WORKDIR /app
COPY --from=api-publish /app/src/Exceptionless.Web/out ./
ENTRYPOINT [ "dotnet", "Exceptionless.Web.dll" ]
