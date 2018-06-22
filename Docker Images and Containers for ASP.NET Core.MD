# Docker and ASPNETCORE

# Running ASPNETCORE in a Container

## ASPNETCORE

Basic commands
```
dotnet new
dotnet restore
dotnet build
```

Test projects
```
dotnet test
```

Library projects
```
dotnet pack
dotnet nuget push
```

Web/App projects
```
dotnet run
dotnet publish
```

## Docker
* **Container** is an isolated process
* **Image** provides a file system for container

Running the dotnet cli
```
docker run --rm -it microsoft/aspnetcore:2
```

Mounting host folder into a container
```
docker run --rm -it -v ${PWD}:C:\api microsoft/aspnetcore:2
```

Find container ip adrress
```
docker inspect [containerid]
```

Publishing to a host port and accessing remotely
```
docker run --rm -it -v ${PWD}:C:\api -p 8080:80 microsoft/aspnetcore:2
```

# Picking a base image

## Multi arch image suport (Linux and Windows)
* .NET Core Docker images move to multi-arch based tags
    * Linux -> tag-jessie
    * Windows -> tag-nanoserver

## .NET Core or ASP.NET Core
* What app model?

## OS base image
* Windows or Linux 

## Component
* Runtime vs SDK
* Prod vs Dev

## Versions
* Runtime version
* SDK version
* OS version

# Distributed a pre published app in an image

Create a Dockerfile
```
FROM microsoft/aspnetcore:2

WORKDIR /app
COPY bin/Debug/netcoreapp2.0/publish .

ENTRYPOINT ["dotnet", "api.dll"]
```
Build an image from dockerfile
```
dotnet build -t aspnetcore/generator .
```
Run a container
```
docker run --rm -it -p 8080:80 aspnetcore/generator
```
Pushing to docker hub
```
docker push aspnetcore/generator
```
Running on another machine
```
docker run --rm -it -p 8080:80 aspnetcore/generator
```
Create .dockerignore
```
*
!bin/Debug/netcoreapp2.0/publish
```

# Build the app in a container

Building .NET Core in a container
```
docker run --rm -it -v ${PWD}:/api microsoft/dotnet:2-sdk
```

Inside a contianer shell
```
dotnet restore
dotnet build
dotnet publish
```

Move to host shell and goto publish folder
```
dotnet api.dll
```
-- So you can run an app with only sdk is installed on host machine

Setting port with environment variable
```
docker run --rm -it -v ${PWD}:/api -p 8080:80 microsoft/dotnet:2-sdk
```

Inside a container shell
```
cd /api
export ASPNETCORE_URLS=http://+:80
dotnet restore
dotnet run
```
-- You can access app from host

Change image
```
docker run --rm -it -v ${PWD}:/api -p 8080:80 microsoft/aspnet-core-build:2
```

# Automate building with a docker file

Traditional way
```
docker run --rm -it -v ${PWD}:/api -p 8080:80 microsoft/aspnetcore-build:2
cd /api
rm -rf bin/ obj/
dotnet restore
dotnet publish
```

With a dockerfile
```
FROM microsoft/aspnetcore-build:2

WORKDIR /api
COPY . .
RUN dotnet restore
RUN dotnet publish -o /publish

WORKDIR /publish
ENTRYPOINT ["dotnet", "/publish/api.dll"]
```

Running the dockerfile
```
docker build -t aspnetcore/generator:build .
docker run --rm -it -p 8080:80 aspnetcore/generator:build
```

-- Avoid cache invalidation when cahnging dockerfile
Inside .dockerignore
```
bin/
obj/
Dockerfile
.dockerignore
```

# Using multi state build to create runtime optimized image

Docker with multi stages
```
# Build stage
FROM microsoft/aspnetcore-build:2 AS build-env
WORKDIR /api

# Optimize build
COPY *.csproj .
RUN dotnet restore

COPY . .
RUN dotnet publish -c Release -o /publish

# Runtime Stage
FROM microsoft/aspnetcore:2
WORKDIR /publish
COPY --from=build-env /publish .
ENTRYPOINT ["dotnet", "api.dll"]
```

Running
```
docker build -t aspnetcore/generator:multi .
docker run --rm -it -p 8080:80 aspnetcore/generator:multi
```

# Composing an ASPNET CORE app

Instead run docker build and run we create a docker-compose.yml
```
version: '3.1'

services:
    generator:
        image: aspnetcore/generator:multi
        ports:
            - "8080:80"
```

Running
```
docker-compose up
```

Starting container in a background
```
docker-compose up -d
```

Cleaning
```
docker-compose down
```

Building 
```
version: '3.1'

services:
    generator:
        build:
            context: .
            dockerfile: Dockerfile
        image: aspnetcore/generator:multi
        ports:
            - "8080:80"
```