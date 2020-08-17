---
title: 'Inicio rápido: implementación de un contenedor de Docker en una instancia de contenedor (CLI de Docker)'
description: En esta guía de inicio rápido, usará la CLI de Docker para implementar rápidamente una aplicación web en contenedores que se ejecute en una instancia de contenedor aislada de Azure
ms.topic: quickstart
ms.date: 07/16/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: 0e4569904ef6aee304518ce012889d10dc2ecbce
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87408091"
---
# <a name="quickstart-deploy-a-container-instance-in-azure-using-the-docker-cli"></a>Inicio rápido: Implementación de una instancia de contenedor en Azure mediante la CLI de Docker

Use Azure Container Instances para ejecutar contenedores de Docker sin servidor en Azure con sencillez y velocidad. Implemente en una instancia de contenedor a petición cuando desarrolle aplicaciones nativas en la nube y quiera cambiar sin problemas entre el desarrollo local y la implementación en la nube.

En esta guía de inicio rápido se usan los comandos nativos de la CLI de Docker para implementar un contenedor de Docker y hacer que la aplicación esté disponible en Azure Container Instances. La [integración entre Docker y Azure](https://docs.docker.com/engine/context/aci-integration/) (versión beta) habilita esta capacidad. Unos pocos segundos después de ejecutar un comando `docker run`, puede ir a la aplicación que se ejecuta en el contenedor:

:::image type="content" source="media/container-instances-quickstart-docker-cli/view-application-running-in-an-azure-container-instance.png" alt-text="Aplicación implementada mediante Azure Container Instances vista en el explorador":::

Si no tiene una suscripción a Azure, cree una [cuenta gratuita][azure-account] antes de empezar.

Para esta guía de inicio rápido, debe instalar Docker Desktop Edge versión 2.3.2.0 o posterior, disponible para [Windows](https://desktop.docker.com/win/edge/Docker%20Desktop%20Installer.exe) o [macOS](https://desktop.docker.com/mac/edge/Docker.dmg). O bien, instale la [CLI de integración de Docker ACI para Linux](https://docs.docker.com/engine/context/aci-integration/#install-the-docker-aci-integration-cli-on-linux) (versión beta). 

> [!IMPORTANT]
> Esta característica se encuentra actualmente en versión preliminar y requiere características en versión beta (versión preliminar) en Docker. Obtenga más información sobre las [versiones estable y perimetral de Docker Desktop](https://docs.docker.com/desktop/#stable-and-edge-versions). No se admiten todas las características de Azure Container Instances. Proporcione comentarios sobre la integración entre Docker y Azure mediante la creación de una incidencia en el repositorio de GitHub [aci-integration-beta](https://github.com/docker/aci-integration-beta).

## <a name="create-azure-context"></a>Creación de un contexto de Azure

Para usar los comandos de Docker para ejecutar contenedores en Azure Container Instances, primero inicie sesión en Azure:

```bash
docker login azure
```

Cuando se le solicite, escriba o seleccione sus credenciales de Azure.


Ejecute `docker context create aci` para crear un contexto de ACI. Este contexto asocia Docker con su suscripción a Azure para que pueda crear instancias de contenedor. Por ejemplo, para crear un contexto llamado *myacicontext*:

```
docker context create aci myacicontext
```

Cuando se le solicite, seleccione el id. de suscripción de Azure y después seleccione un grupo de recursos existente o **cree un nuevo grupo de recursos**. Si elige un nuevo grupo de recursos, se creará con un nombre generado por el sistema. Las instancias de contenedores de Azure, al igual que otros recursos de Azure, se deben implementar en un grupo de recursos. Los grupos de recursos le permiten organizar y administrar los recursos relacionados de Azure.


Ejecute `docker context ls` para confirmar que agregó el contexto de ACI a los contextos de Docker:

```
docker context ls
```

## <a name="create-a-container"></a>Crear un contenedor

Después de crear un contexto de Docker, puede crear un contenedor en Azure. En este inicio rápido, usará la imagen `mcr.microsoft.com/azuredocs/aci-helloworld` pública. Esta imagen empaqueta una pequeña aplicación web escrita en Node.js que sirve una página HTML estática.

En primer lugar, cambie al contexto de ACI. Todos los comandos de Docker subsiguientes se ejecutarán en este contexto.

```
docker context use myacicontext
```

Ejecute el siguiente comando `docker run` para crear la instancia de Azure Container Instances con el puerto 80 expuesto a Internet:

```
docker run -p 80:80 mcr.microsoft.com/azuredocs/aci-helloworld
```

Salida de ejemplo para una implementación correcta:

```
[+] Running 2/2
 ⠿ hungry-kirch            Created                                                                               5.1s
 ⠿ single--container--aci  Done                                                                                 11.3s
hungry-kirch
```

Ejecute `docker ps` para obtener detalles sobre el contenedor en ejecución, incluida la dirección IP pública:

```
docker ps
```


La salida de ejemplo muestra una dirección IP pública, en este caso *52.230.225.232*:

```
CONTAINER ID        IMAGE                                        COMMAND             STATUS              PORTS
hungry-kirch        mcr.microsoft.com/azuredocs/aci-helloworld                       Running             52.230.225.232:80->80/tcp
```

 Ahora, vaya a la dirección IP en el explorador. Si ve una página web parecida a la siguiente, enhorabuena. Ha implementado correctamente una aplicación que se ejecuta en un contenedor de Docker en Azure.

:::image type="content" source="media/container-instances-quickstart-docker-cli/view-application-running-in-an-azure-container-instance.png" alt-text="Aplicación implementada mediante Azure Container Instances vista en el explorador":::

## <a name="pull-the-container-logs"></a>Extracción de los registros del contenedor

Cuando necesite solucionar problemas de un contenedor o de la aplicación que este ejecuta (o simplemente ver la salida), empiece por ver los registros de la instancia de contenedor.

Por ejemplo, ejecute el comando `docker logs` para ver los registros del contenedor *hungry-kirch* en el contexto de ACI:

```azurecli-interactive
docker logs hungry-kirch
```

En la salida se muestran los registros del contenedor y se mostrarán las solicitudes HTTP GET generadas al ver la aplicación en el explorador.

```output
listening on port 80
::ffff:10.240.255.55 - - [07/Jul/2020:17:43:53 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
::ffff:10.240.255.55 - - [07/Jul/2020:17:44:36 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
::ffff:10.240.255.55 - - [07/Jul/2020:17:44:36 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
```


## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado con el contenedor, ejecute `docker rm` para eliminarlo. Este comando detiene y elimina la instancia de Azure Container Instances.

```
docker rm hungry-kirch
```


## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha creado una instancia de contenedor de Azure a partir de una imagen pública al integrar Docker y Azure. Obtenga más información sobre los escenarios de integración en la [documentación de Docker](https://docs.docker.com/engine/context/aci-integration/). 

También puede usar la [extensión de Docker](https://aka.ms/VSCodeDocker) para Visual Studio Code para obtener una experiencia integrada para desarrollar, ejecutar y administrar contenedores, imágenes y contextos.

Si quiere usar las herramientas de Azure para crear y administrar instancias de contenedor, consulte otras guías de inicio rápido para hacerlo mediante la [CLI de Azure](container-instances-quickstart.md), [Azure PowerShell](container-instances-quickstart-powershell.md), [Azure Portal](container-instances-quickstart-portal.md) y una [plantilla de Azure Resource Manager](container-instances-quickstart-template.md).

Si quiere compilar una imagen de contenedor e implementarla desde un registro de contenedor privado de Azure, vaya al tutorial de Azure Container Instances.

> [!div class="nextstepaction"]
> [Tutorial de Azure Container Instances](./container-instances-tutorial-prepare-app.md)

<!-- LINKS - External -->

[azure-account]: https://azure.microsoft.com/free/
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

