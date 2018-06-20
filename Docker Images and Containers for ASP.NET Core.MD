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
