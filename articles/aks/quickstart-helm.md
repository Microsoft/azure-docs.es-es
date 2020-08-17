---
title: Desarrollo en Azure Kubernetes Service (AKS) con Helm
description: Use Helm con AKS y Azure Container Registry para empaquetar y ejecutar contenedores de aplicaciones en un clúster.
services: container-service
author: zr-msft
ms.topic: article
ms.date: 07/28/2020
ms.author: zarhoads
ms.openlocfilehash: 0ca2d7ccc863e2208db1212ef3d3f10fa709d069
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87407122"
---
# <a name="quickstart-develop-on-azure-kubernetes-service-aks-with-helm"></a>Inicio rápido: Desarrollo en Azure Kubernetes Service (AKS) con Helm

[Helm][helm] es una herramienta de empaquetado de código abierto que ayuda a instalar y administrar el ciclo de vida de las aplicaciones de Kubernetes. Helm, que funciona de forma similar a los administradores de paquetes de Linux como *APT* y *Yum*, se utiliza para administrar los gráficos de Kubernetes, que son paquetes de recursos de Kubernetes preconfigurados.

En este artículo se muestra cómo usar Helm para empaquetar y ejecutar una aplicación en AKS. Para más información sobre cómo instalar una aplicación existente con Helm, consulte [Instalación de aplicaciones existentes con Helm en AKS][helm-existing].

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción a Azure, puede crear una [cuenta gratuita](https://azure.microsoft.com/free).
* [La CLI de Azure instalada](/cli/azure/install-azure-cli?view=azure-cli-latest).
* [Helm v3 instalado][helm-install].

## <a name="create-an-azure-container-registry"></a>Creación de una instancia de Azure Container Registry
Para usar Helm para ejecutar la aplicación en el clúster de AKS, necesita Azure Container Registry para almacenar las imágenes de contenedor. En el ejemplo siguiente se usa [az acr create][az-acr-create] para crear una instancia de ACR llamada *MyHelmACR* en el grupo de recursos *MyResourceGroup* con la SKU *Básica*. Debería proporcionar su propio nombre único de registro. El nombre del registro debe ser único dentro de Azure y contener entre 5 y 50 caracteres alfanuméricos. La SKU *básica* es un punto de entrada optimizado para costo con fines de desarrollo que proporciona un equilibrio entre almacenamiento y rendimiento.

```azurecli
az group create --name MyResourceGroup --location eastus
az acr create --resource-group MyResourceGroup --name MyHelmACR --sku Basic
```

La salida será similar al del ejemplo siguiente: Tome nota del valor de *loginServer* el ACR, puesto que se usará en un paso posterior. En el ejemplo siguiente, *myhelmacr.azurecr.io* es el valor de *loginServer* para *MyHelmACR*.

```console
{
  "adminUserEnabled": false,
  "creationDate": "2019-06-11T13:35:17.998425+00:00",
  "id": "/subscriptions/<ID>/resourceGroups/MyResourceGroup/providers/Microsoft.ContainerRegistry/registries/MyHelmACR",
  "location": "eastus",
  "loginServer": "myhelmacr.azurecr.io",
  "name": "MyHelmACR",
  "networkRuleSet": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "MyResourceGroup",
  "sku": {
    "name": "Basic",
    "tier": "Basic"
  },
  "status": null,
  "storageAccount": null,
  "tags": {},
  "type": "Microsoft.ContainerRegistry/registries"
}
```

## <a name="create-an-azure-kubernetes-service-cluster"></a>Creación de un clúster de Azure Kubernetes Service

Cree un clúster de AKS. El comando siguiente crea un clúster de AKS llamado MyAKS y adjunta MyHelmACR.

```azurecli
az aks create -g MyResourceGroup -n MyAKS --location eastus  --attach-acr MyHelmACR --generate-ssh-keys
```

El clúster de AKS necesita acceder a la instancia de ACR para extraer las imágenes de contenedor y ejecutarlas. El comando anterior también concede al clúster *MyAKS* acceso a la instancia de ACR *MyHelmACR*.

## <a name="connect-to-your-aks-cluster"></a>Conectarse al clúster AKS

Para conectarse al clúster de Kubernetes desde su equipo local, use [kubectl][kubectl], el cliente de la línea de comandos de Kubernetes.

Si usa Azure Cloud Shell, `kubectl` ya está instalado. También lo puede instalar localmente. Para ello debe usar el comando [az aks install-cli][]:

```azurecli
az aks install-cli
```

Para configurar `kubectl` para conectarse a su clúster de Kubernetes, use el comando [az aks get-credentials][]. En el ejemplo siguiente se obtienen las credenciales del clúster de AKS llamado *MyAKS* en *MyResourceGroup*:

```azurecli
az aks get-credentials --resource-group MyResourceGroup --name MyAKS
```

## <a name="download-the-sample-application"></a>Descarga de la aplicación de ejemplo

En esta guía de inicio rápido se usa [una aplicación de Node.js de ejemplo del repositorio de ejemplo de Azure Dev Spaces][example-nodejs]. Clone la aplicación desde GitHub y vaya al directorio `dev-spaces/samples/nodejs/getting-started/webfrontend`.

```console
git clone https://github.com/Azure/dev-spaces
cd dev-spaces/samples/nodejs/getting-started/webfrontend
```

## <a name="create-a-dockerfile"></a>Creación de un archivo Dockerfile

Cree un nuevo archivo *Dockerfile* mediante los siguientes pasos:

```dockerfile
FROM node:latest

WORKDIR /webfrontend

COPY package.json ./

RUN npm install

COPY . .

EXPOSE 80
CMD ["node","server.js"]
```

## <a name="build-and-push-the-sample-application-to-the-acr"></a>Compilación e inserción de la aplicación de ejemplo en la instancia de ACR

Use el comando [az acr build][az-acr-build] para compilar e insertar una imagen en el registro con el Dockerfile anterior. El signo `.` al final del comando establece la ubicación del Dockerfile, en este caso el directorio actual.

```azurecli
az acr build --image webfrontend:v1 \
  --registry MyHelmACR \
  --file Dockerfile .
```

## <a name="create-your-helm-chart"></a>Creación del gráfico de Helm

Genere el gráfico de Helm con el comando `helm create`.

```console
helm create webfrontend
```

Realice las siguientes actualizaciones en *webfrontend/values.yaml*. Sustituya el loginServer del registro que anotó en un paso anterior, como *myhelmacr.azurecr.io*:

* Cambie `image.repository` a `<loginServer>/webfrontend`.
* Cambie `service.type` a `LoadBalancer`.

Por ejemplo:

```yml
# Default values for webfrontend.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: *myhelmacr.azurecr.io*/webfrontend
  pullPolicy: IfNotPresent
...
service:
  type: LoadBalancer
  port: 80
...
```

Actualice `appVersion` a `v1` en *webfrontend/Chart.yaml*. Por ejemplo

```yml
apiVersion: v2
name: webfrontend
...
# This is the version number of the application being deployed. This version number should be
# incremented each time you make changes to the application.
appVersion: v1
```

## <a name="run-your-helm-chart"></a>Ejecución del gráfico de Helm

Use el comando `helm install` para instalar la aplicación con el gráfico de Helm.

```console
helm install webfrontend webfrontend/
```

El servicio puede tardar unos minutos en devolver una dirección IP pública. Para supervisar el progreso, utilice el comando `kubectl get service` con el parámetro *watch*:

```console
$ kubectl get service --watch

NAME                TYPE          CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
webfrontend         LoadBalancer  10.0.141.72   <pending>     80:32150/TCP   2m
...
webfrontend         LoadBalancer  10.0.141.72   <EXTERNAL-IP> 80:32150/TCP   7m
```

Vaya al equilibrador de carga de la aplicación en un explorador mediante el uso del valor de `<EXTERNAL-IP>` para ver la aplicación de ejemplo.

## <a name="delete-the-cluster"></a>Eliminación del clúster

Cuando ya no se necesite el clúster, use el comando [az group delete][az-group-delete] para quitar el grupo de recursos, el clúster de AKS, el registro de contenedores, las imágenes de contenedor almacenadas allí y todos los recursos relacionados.

```azurecli-interactive
az group delete --name MyResourceGroup --yes --no-wait
```

> [!NOTE]
> Cuando elimina el clúster, la entidad de servicio Azure Active Directory que utiliza el clúster de AKS no se quita. Para conocer los pasos que hay que realizar para quitar la entidad de servicio, consulte [Consideraciones principales y eliminación de AKS][sp-delete]. Si usó una identidad administrada, esta está administrada por la plataforma y no requiere que la quite.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre el uso de Helm, consulte la documentación de Helm.

> [!div class="nextstepaction"]
> [Documentación de Helm][helm-documentation]

[az-acr-create]: /cli/azure/acr#az-acr-create
[az-acr-build]: /cli/azure/acr#az-acr-build
[az-group-delete]: /cli/azure/group#az-group-delete
[az aks get-credentials]: /cli/azure/aks#az-aks-get-credentials
[az aks install-cli]: /cli/azure/aks#az-aks-install-cli
[example-nodejs]: https://github.com/Azure/dev-spaces/tree/master/samples/nodejs/getting-started/webfrontend
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[helm]: https://helm.sh/
[helm-documentation]: https://helm.sh/docs/
[helm-existing]: kubernetes-helm.md
[helm-install]: https://helm.sh/docs/intro/install/
[sp-delete]: kubernetes-service-principal.md#additional-considerations
