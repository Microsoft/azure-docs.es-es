---
title: 'Tutorial: implementación de un grupo de varios contenedores - YAML'
description: En este tutorial, aprenderá a implementar un grupo de contenedores con varios contenedores en Azure Container Instances mediante un archivo YAML con la CLI de Azure.
ms.topic: article
ms.date: 07/01/2020
ms.openlocfilehash: f6ba9152003c68192ddc0fdffa5652bc0b6daa86
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88750906"
---
# <a name="tutorial-deploy-a-multi-container-group-using-a-yaml-file"></a>Tutorial: Implementación de un grupo de varios contenedores con un archivo YAML

> [!div class="op_single_selector"]
> * [YAML](container-instances-multi-container-yaml.md)
> * [Resource Manager](container-instances-multi-container-group.md)
>

Azure Container Instances admite la implementación de varios contenedores en un solo host mediante un [grupo de contenedores](container-instances-container-groups.md). Un grupo de contenedores es útil cuando se crea un sidecar de aplicación para el registro, la supervisión o cualquier otra configuración donde un servicio necesite un segundo proceso asociado.

En este tutorial seguirá los pasos para ejecutar una configuración de sidecar sencilla de dos contenedores mediante la implementación de un [archivo YAML](container-instances-reference-yaml.md) con la CLI de Azure. Un archivo YAML proporciona un formato conciso para especificar la configuración de instancias. Aprenderá a:

> [!div class="checklist"]
> * Configurar un archivo YAML
> * Implementación del grupo de contenedores
> * Ver los registros de los contenedores

> [!NOTE]
> Los grupos de varios contenedores están restringidos actualmente a los contenedores Linux.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="configure-a-yaml-file"></a>Configurar un archivo YAML

Para implementar un grupo de varios contenedores con el comando [az container create][az-container-create] en la CLI de Azure, debe especificar la configuración del grupo de contenedores en un archivo YAML. A continuación, pase el archivo YAML como parámetro al comando.

Para empezar, copie el siguiente archivo YAML en un nuevo archivo denominado **deploy-aci.yaml**. En Azure Cloud Shell, puede usar Visual Studio Code para crear el archivo en el directorio de trabajo:

```
code deploy-aci.yaml
```

En este archivo YAML, se define un grupo de contenedores denominado "myContainerGroup" con dos contenedores, una dirección IP pública y dos puertos expuestos. Los contenedores se implementan desde imágenes públicas de Microsoft. El primer contenedor del grupo ejecuta una aplicación web accesible desde Internet. El segundo contenedor, el sidecar, realiza solicitudes HTTP periódicamente a la aplicación web que se ejecuta en el primer contenedor a través de la red local del grupo de contenedores.

```YAML
apiVersion: 2019-12-01
location: eastus
name: myContainerGroup
properties:
  containers:
  - name: aci-tutorial-app
    properties:
      image: mcr.microsoft.com/azuredocs/aci-helloworld:latest
      resources:
        requests:
          cpu: 1
          memoryInGb: 1.5
      ports:
      - port: 80
      - port: 8080
  - name: aci-tutorial-sidecar
    properties:
      image: mcr.microsoft.com/azuredocs/aci-tutorial-sidecar
      resources:
        requests:
          cpu: 1
          memoryInGb: 1.5
  osType: Linux
  ipAddress:
    type: Public
    ports:
    - protocol: tcp
      port: 80
    - protocol: tcp
      port: 8080
tags: {exampleTag: tutorial}
type: Microsoft.ContainerInstance/containerGroups
```

Para usar un registro de imágenes de contenedor privado, agregue la propiedad `imageRegistryCredentials` al grupo de contenedores, con los valores modificados para su entorno:

```YAML
  imageRegistryCredentials:
  - server: imageRegistryLoginServer
    username: imageRegistryUsername
    password: imageRegistryPassword
```

## <a name="deploy-the-container-group"></a>Implementación del grupo de contenedores

Cree un grupo de recursos con el comando [az group create][az-group-create]:

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Implemente el grupo de contenedores con el comando [az container create][az-container-create] y pase el archivo YAML como un argumento:

```azurecli-interactive
az container create --resource-group myResourceGroup --file deploy-aci.yaml
```

Al cabo de unos segundos, debe recibir una respuesta inicial de Azure.

## <a name="view-deployment-state"></a>Visualización del estado de la implementación

Para ver el estado de la implementación, use el siguiente comando [az container show][az-container-show]:

```azurecli-interactive
az container show --resource-group myResourceGroup --name myContainerGroup --output table
```

Si desea ver la aplicación en ejecución, vaya a su dirección IP en el explorador. Por ejemplo, la dirección IP es `52.168.26.124` en esta salida de ejemplo:

```bash
Name              ResourceGroup    Status    Image                                                                                               IP:ports              Network    CPU/Memory       OsType    Location
----------------  ---------------  --------  --------------------------------------------------------------------------------------------------  --------------------  ---------  ---------------  --------  ----------
myContainerGroup  danlep0318r      Running   mcr.microsoft.com/azuredocs/aci-tutorial-sidecar,mcr.microsoft.com/azuredocs/aci-helloworld:latest  20.42.26.114:80,8080  Public     1.0 core/1.5 gb  Linux     eastus
```

## <a name="view-container-logs"></a>Visualización de registros de contenedores

Visualice la salida del registro de un contenedor con el comando [az container logs][az-container-logs]. El argumento `--container-name` especifica el contenedor del que se van a extraer registros. En este ejemplo, se especifica el contenedor `aci-tutorial-app`.

```azurecli-interactive
az container logs --resource-group myResourceGroup --name myContainerGroup --container-name aci-tutorial-app
```

Salida:

```console
listening on port 80
::1 - - [02/Jul/2020:23:17:48 +0000] "HEAD / HTTP/1.1" 200 1663 "-" "curl/7.54.0"
::1 - - [02/Jul/2020:23:17:51 +0000] "HEAD / HTTP/1.1" 200 1663 "-" "curl/7.54.0"
::1 - - [02/Jul/2020:23:17:54 +0000] "HEAD / HTTP/1.1" 200 1663 "-" "curl/7.54.0"
```

Para ver los registros para el contenedor sidecar, ejecute un comando similar en el que se especifique el contenedor `aci-tutorial-sidecar`.

```azurecli-interactive
az container logs --resource-group myResourceGroup --name myContainerGroup --container-name aci-tutorial-sidecar
```

Salida:

```console
Every 3s: curl -I http://localhost                          2020-07-02 20:36:41

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0  1663    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
HTTP/1.1 200 OK
X-Powered-By: Express
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Wed, 29 Nov 2017 06:40:40 GMT
ETag: W/"67f-16006818640"
Content-Type: text/html; charset=UTF-8
Content-Length: 1663
Date: Thu, 02 Jul 2020 20:36:41 GMT
Connection: keep-alive
```

Como puede ver, el sidecar realiza periódicamente una solicitud HTTP a la aplicación web principal a través de la red local del grupo para asegurarse de que se está ejecutando. Este ejemplo de sidecar se podría ampliar para desencadenar una alerta si recibe un código de respuesta HTTP distinto de `200 OK`.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha usado un archivo YAML para implementar un grupo de varios contenedores en Azure Container Instances. Ha aprendido a:

> [!div class="checklist"]
> * Configurar un archivo YAML para un grupo de varios contenedores
> * Implementación del grupo de contenedores
> * Ver los registros de los contenedores

También puede especificar un grupo de varios contenedores con una [plantilla de Resource Manager](container-instances-multi-container-group.md). Una plantilla de Resource Manager se puede adaptar fácilmente a escenarios donde necesite implementar recursos de servicio de Azure adicionales con el grupo de contenedores.

<!-- LINKS - External -->


<!-- LINKS - Internal -->
[aci-tutorial]: ./container-instances-tutorial-prepare-app.md
[az-container-create]: /cli/azure/container#az-container-create
[az-container-logs]: /cli/azure/container#az-container-logs
[az-container-show]: /cli/azure/container#az-container-show
[az-group-create]: /cli/azure/group#az-group-create
[az-deployment-group-create]: /cli/azure/deployment/group#az-deployment-group-create
[template-reference]: /azure/templates/microsoft.containerinstance/containergroups
