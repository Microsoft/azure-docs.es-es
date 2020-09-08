---
title: Acceso SSH para contenedores Linux
description: Puede abrir una sesión de SSH en un contenedor Linux en Azure App Service. Los contenedores Linux personalizados se admiten con algunas modificaciones en la imagen personalizada.
keywords: azure app service, aplicación web, linux, oss
author: msangapu-msft
ms.assetid: 66f9988f-8ffa-414a-9137-3a9b15a5573c
ms.topic: article
ms.date: 02/25/2019
ms.author: msangapu
ms.custom: seodec18
ms.openlocfilehash: d46aacc441b412a117d906e0201a9bed6046a338
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88961612"
---
# <a name="open-an-ssh-session-to-a-linux-container-in-azure-app-service"></a>Apertura de una sesión de SSH en un contenedor Linux en Azure App Service

[Secure Shell (SSH)](https://wikipedia.org/wiki/Secure_Shell) se suele utilizar para ejecutar comandos administrativos de forma remota desde un terminal de línea de comandos. App Service en Linux proporciona compatibilidad con SSH en el contenedor de aplicación. 

![SSH de App Service para Linux](./media/configure-linux-open-ssh-session/app-service-linux-ssh.png)

También puede conectarse al contenedor directamente desde la máquina de desarrollo local mediante SSH y SFTP.

## <a name="open-ssh-session-in-browser"></a>Abrir sesión SSH en el explorador

[!INCLUDE [Open SSH session in browser](../../includes/app-service-web-ssh-connect-no-h.md)]

## <a name="use-ssh-support-with-custom-docker-images"></a>Uso de la compatibilidad con SSH para imágenes personalizadas de Docker

Vea [Configuración de SSH en un contenedor personalizado](configure-custom-container.md#enable-ssh).

## <a name="open-ssh-session-from-remote-shell"></a>Abrir sesión SSH desde un shell remoto

> [!NOTE]
> Esta característica está actualmente en versión preliminar.
>

Mediante la tunelización TCP puede crear una conexión de red entre la máquina de desarrollo y Web App for Containers a través de una conexión de WebSocket autenticada. Permite abrir una sesión SSH con el contenedor que se ejecuta en App Service desde el cliente de su elección.

Para empezar, es preciso instalar la [CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest). Para ver cómo funciona sin instalar la CLI de Azure, abra [Azure Cloud Shell](../cloud-shell/overview.md). 

Abra una conexión remota a la aplicación mediante el comando [az webapp remote-connection create](/cli/azure/ext/webapp/webapp/remote-connection?view=azure-cli-latest#ext-webapp-az-webapp-remote-connection-create). Especifique _\<subscription-id>_ , _\<group-name>_ y \_\<app-name>_ de la aplicación.

```azurecli-interactive
az webapp create-remote-connection --subscription <subscription-id> --resource-group <resource-group-name> -n <app-name> &
```

> [!TIP]
> El `&` al final del comando es solo para su comodidad si usa Cloud Shell. El proceso se ejecuta en segundo plano, por lo que puede ejecutar el siguiente comando en el mismo shell.

La salida del comando le ofrece la información necesaria para abrir una sesión SSH.

```output
Port 21382 is open
SSH is available { username: root, password: Docker! }
Start your favorite client and connect to port 21382
```

Abra una sesión SSH con el contenedor con el cliente de su elección mediante el puerto local. En el ejemplo siguiente se utiliza el comando [ssh](https://ss64.com/bash/ssh.html) predeterminado:

```bash
ssh root@127.0.0.1 -p <port>
```

Cuando se solicite, escriba `yes` para continuar con la conexión. Se le pedirá la contraseña. Use `Docker!`, que ya apareció anteriormente.

<pre>
Warning: Permanently added '[127.0.0.1]:21382' (ECDSA) to the list of known hosts.
root@127.0.0.1's password:
</pre>

Una vez autenticado, debería ver la pantalla de inicio de sesión.

<pre>
  _____
  /  _  \ __________ _________   ____
 /  /_\  \___   /  |  \_  __ \_/ __ \
/    |    \/    /|  |  /|  | \/\  ___/
\____|__  /_____ \____/ |__|    \___  &gt;
        \/      \/                  \/
A P P   S E R V I C E   O N   L I N U X

0e690efa93e2:~#
</pre>

Ahora está conectado a su conector.  

Intente ejecutar el comando[top](https://ss64.com/bash/top.html). Debe poder ver el proceso de la aplicación en la lista de procesos. En la salida del ejemplo siguiente, es el marcado con `PID 263`.

<pre>
Mem: 1578756K used, 127032K free, 8744K shrd, 201592K buff, 341348K cached
CPU:   3% usr   3% sys   0% nic  92% idle   0% io   0% irq   0% sirq
Load average: 0.07 0.04 0.08 4/765 45738
  PID  PPID USER     STAT   VSZ %VSZ CPU %CPU COMMAND
    1     0 root     S     1528   0%   0   0% /sbin/init
  235     1 root     S     632m  38%   0   0% PM2 v2.10.3: God Daemon (/root/.pm2)
  263   235 root     S     630m  38%   0   0% node /home/site/wwwroot/app.js
  482   291 root     S     7368   0%   0   0% sshd: root@pts/0
45513   291 root     S     7356   0%   0   0% sshd: root@pts/1
  291     1 root     S     7324   0%   0   0% /usr/sbin/sshd
  490   482 root     S     1540   0%   0   0% -ash
45539 45513 root     S     1540   0%   0   0% -ash
45678 45539 root     R     1536   0%   0   0% top
45733     1 root     Z        0   0%   0   0% [init]
45734     1 root     Z        0   0%   0   0% [init]
45735     1 root     Z        0   0%   0   0% [init]
45736     1 root     Z        0   0%   0   0% [init]
45737     1 root     Z        0   0%   0   0% [init]
45738     1 root     Z        0   0%   0   0% [init]
</pre>

## <a name="next-steps"></a>Pasos siguientes

Puede publicar preguntas y problemas en el [foro de Azure](/answers/topics/azure-webapps.html).

Para obtener más información sobre Web App for Containers, vea:

* [Introducing remote debugging of Node.js apps on Azure App Service from VS Code](https://medium.com/@auchenberg/introducing-remote-debugging-of-node-js-apps-on-azure-app-service-from-vs-code-in-public-preview-9b8d83a6e1f0) (Introducción a la depuración remota de aplicaciones de Node.js en Azure App Service desde VS Code)
* [Inicio rápido: Ejecución de un contenedor personalizado en App Service](quickstart-custom-container.md?pivots=container-linux)
* [Uso de Ruby en Azure App Service en Linux](quickstart-ruby.md)
* [Preguntas más frecuentes sobre Web App for Containers de Azure App Service ](faq-app-service-linux.md)