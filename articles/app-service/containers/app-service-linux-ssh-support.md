---
title: 'Compatibilidad con SSH para App Service en Linux: Azure | Microsoft Docs'
description: Aprenda a usar SSH con Azure App Service en Linux.
keywords: azure app service, aplicación web, linux, oss
services: app-service
documentationcenter: ''
author: msangapu
manager: jeconnoc
editor: ''
ms.assetid: 66f9988f-8ffa-414a-9137-3a9b15a5573c
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/25/2019
ms.author: msangapu
ms.custom: seodec18
ms.openlocfilehash: 18d10afc9132c81c2dcfbb1aa17ded81a21336ca
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65780047"
---
# <a name="ssh-support-for-azure-app-service-on-linux"></a>Compatibilidad con SSH para Azure App Service en Linux

[Secure Shell (SSH)](https://wikipedia.org/wiki/Secure_Shell) se suele utilizar para ejecutar comandos administrativos de forma remota desde un terminal de línea de comandos. App Service en Linux proporciona compatibilidad con SSH en el contenedor de la aplicación para todas las imágenes de Docker integradas que se usan para la pila en tiempo de ejecución de nuevas aplicaciones web. 

![Pilas en tiempo de ejecución](./media/app-service-linux-ssh-support/app-service-linux-runtime-stack.png)

Para imágenes personalizadas de Docker, mediante la configuración del servidor SSH de la imagen personalizada.

También puede conectarse al contenedor directamente desde la máquina de desarrollo local mediante SSH y SFTP.

## <a name="open-ssh-session-in-browser"></a>Abrir sesión SSH en el explorador

[!INCLUDE [Open SSH session in browser](../../../includes/app-service-web-ssh-connect-no-h.md)]

## <a name="use-ssh-support-with-custom-docker-images"></a>Uso de la compatibilidad con SSH para imágenes personalizadas de Docker

Consulte [configurar SSH en un contenedor personalizado](configure-custom-container.md#enable-ssh).

## <a name="open-ssh-session-from-remote-shell"></a>Abrir sesión SSH desde un shell remoto

> [!NOTE]
> Esta característica está actualmente en versión preliminar.
>

Mediante la tunelización TCP puede crear una conexión de red entre la máquina de desarrollo y Web App for Containers a través de una conexión de WebSocket autenticada. Permite abrir una sesión SSH con el contenedor que se ejecuta en App Service desde el cliente de su elección.

Para empezar, es preciso instalar la [CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest). Para ver cómo funciona sin instalar la CLI de Azure, abra [Azure Cloud Shell](../../cloud-shell/overview.md). 

Abra una conexión remota a la aplicación mediante el comando [az webapp remote-connection create](/cli/azure/ext/webapp/webapp/remote-connection?view=azure-cli-latest#ext-webapp-az-webapp-remote-connection-create). Especificar  _\<id-suscripción >_,  _\<nombre del grupo >_ y \__ < Nombre_de_aplicación > para la aplicación.

```azurecli-interactive
az webapp create-remote-connection --subscription <subscription-id> --resource-group <resource-group-name> -n <app-name> &
```

> [!TIP]
> El `&` al final del comando es solo para su comodidad si usa Cloud Shell. El proceso se ejecuta en segundo plano, por lo que puede ejecutar el siguiente comando en el mismo shell.

La salida del comando le ofrece la información necesaria para abrir una sesión SSH.

```
Port 21382 is open
SSH is available { username: root, password: Docker! }
Start your favorite client and connect to port 21382
```

Abra una sesión SSH con el contenedor con el cliente de su elección mediante el puerto local. En el ejemplo siguiente se utiliza el comando [ssh](https://ss64.com/bash/ssh.html) predeterminado:

```azurecli-interactive
ssh root@127.0.0.1 -p <port>
```

Cuando se solicite, escriba `yes` para continuar con la conexión. Se le pedirá la contraseña. Use `Docker!`, que ya apareció anteriormente.

```
Warning: Permanently added '[127.0.0.1]:21382' (ECDSA) to the list of known hosts.
root@127.0.0.1's password:
```

Una vez autenticado, debería ver la pantalla de inicio de sesión.

```
  _____
  /  _  \ __________ _________   ____
 /  /_\  \___   /  |  \_  __ \_/ __ \
/    |    \/    /|  |  /|  | \/\  ___/
\____|__  /_____ \____/ |__|    \___  >
        \/      \/                  \/
A P P   S E R V I C E   O N   L I N U X

0e690efa93e2:~#
```

Ahora está conectado a su conector.  

Intente ejecutar el comando[top](https://ss64.com/bash/top.html). Debe poder ver el proceso de la aplicación en la lista de procesos. En la salida del ejemplo siguiente, es el marcado con `PID 263`.

```
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
```

## <a name="next-steps"></a>Pasos siguientes

Puede publicar preguntas y problemas en el [foro de Azure](https://social.msdn.microsoft.com/forums/azure/home?forum=windowsazurewebsitespreview).

Para obtener más información sobre Web App for Containers, vea:

* [Introducing remote debugging of Node.js apps on Azure App Service from VS Code](https://medium.com/@auchenberg/introducing-remote-debugging-of-node-js-apps-on-azure-app-service-from-vs-code-in-public-preview-9b8d83a6e1f0) (Introducción a la depuración remota de aplicaciones de Node.js en Azure App Service desde VS Code)
* [Uso de una imagen personalizada de Docker para Web App for Containers](quickstart-docker-go.md)
* [Uso de .NET Core en Azure App Service en Linux](quickstart-dotnetcore.md)
* [Uso de Ruby en Azure App Service en Linux](quickstart-ruby.md)
* [Preguntas más frecuentes sobre Web App for Containers de Azure App Service ](app-service-linux-faq.md)
