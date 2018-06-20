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
