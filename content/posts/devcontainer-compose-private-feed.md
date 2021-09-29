---
title: "Docker compose + devcontainer + private artifacts feed"
date: 2021-09-29T14:56:21-04:00
draft: false
---

**_TL:DR_** If you want use devcontainers and docker compose but also have a private Azure Artifacts feed you need to set the `network_mode` to `bridge` and add links to the services that need to talk to each other.

## Background

I recently got the chance to do some greenfield Azure Functions development and I wanted to use this opportunity to go all in on devcontainers. Getting up and running was super simple and there is plenty of articles out there that cover it. My situation where things started to deviate was that I wanted to seperate out my services using `docker-compose.yml` instead of a single `Dockerfile`. 

Simple enough, I make a few quick edits to the `devcontainer.json`:

```diff
@@ -2,7 +2,9 @@
 // https://github.com/microsoft/vscode-dev-containers/tree/v0.194.3/containers/azure-functions-dotnetcore-3.1
 {
        "name": "My Function App",
-       "dockerFile": "Dockerfile",
+       "dockerComposeFile": "docker-compose.yml",
+       "workspaceFolder": "/workspace",
+       "service": "function",
        "forwardPorts": [ 7071 ],

        // Set *default* container specific settings.json values on container create.

```
Then generated a quick `docker-compose.yml` file:

```yaml
version: '3'

services:
  function:
    build:
      context: .
      dockerfile: Dockerfile            
    init: true
    volumes:
      - ./:/workspace:cached
    command: sleep infinity
    user: vscode

  azurite:
    image: "mcr.microsoft.com/azure-storage/azurite"
    ports:
      - "10000:10000"
      - "10001:10001"
      - "10002:10002"
```
So what did I do there? I switched the `devcontainer.json` to use the `docker-compose.yml` file which required me to add the `workspaceFolder` of where my code is going and the `service` that I want VS Code to be attached to. 

Next I created a simple `docker-compose.yml` file that pointed to my `Dockerfile` for my devcontainer as well as the public container for the Azurite emulator. Once that was done, I opened up the command palete and `Open in Container` and all was well in the world.

We use a private feed on Azure Artifacts so I needed to setup authentication to that feed via the `dotnet` cli, so I again edit `devcontainer.json` to add a `postCreateCommand` to download the `artifacts-credprovider` and install it.

```diff
@@ -20,5 +20,6 @@
        // "postCreateCommand": "dotnet restore",

        // Comment out connect as root instead. More info: https://aka.ms/vscode-remote/containers/non-root.
-       "remoteUser": "vscode"
+       "remoteUser": "vscode",
+       "postCreateCommand": "curl -fsSL https://aka.ms/install-artifacts-credprovider.sh | bash"
```

Rebuild my containers and check the logs

![Container successfully installed Credential Provider](/posts/credprovider-installed.png)

So, again, all is good so far but when I attempt to login via `dotnet restore --interactive` things start to fall apart. It continues to complain about a 401 but never prompts me for my device code. 

![Container would not authenticate](/posts/credprovider-http401.png)

So after much head scratching and google-bingin' I determined it was the networking that was causing the issue. 

Editing the `docker-compose.yml` to tell the containers to use `bridge` networking and use links for  connecting the containers together was all that was needed to get it up and going again. 
```diff
@@ -8,9 +8,10 @@ services:
     init: true
     volumes:
-      - ./:/workspace:cached
+      - ../:/workspace:cached
     command: sleep infinity
     user: vscode
+    network_mode: bridge
+    links:
+      - azurite

   azurite:
     image: "mcr.microsoft.com/azure-storage/azurite"
@@ -18,3 +19,4 @@ services:
       - "10000:10000"
       - "10001:10001"
       - "10002:10002"
+    network_mode: bridge
```

