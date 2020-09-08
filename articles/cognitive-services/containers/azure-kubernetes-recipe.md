---
title: Ejecución de un contenedor de Detección de idioma en Azure Kubernetes Service
titleSuffix: Text Analytics -  Azure Cognitive Services
description: Implemente el contenedor de detección de idioma con un ejemplo en ejecución en Azure Kubernetes Service y pruébelo en un explorador web.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: conceptual
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: b53476bcb05d6e91b157c24795c963c04e6f4bb4
ms.sourcegitcommit: c293217e2d829b752771dab52b96529a5442a190
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/15/2020
ms.locfileid: "88244500"
---
# <a name="deploy-the-text-analytics-language-detection-container-to-azure-kubernetes-service"></a>Implementación del contenedor de detección de idioma Text Analytics en Azure Kubernetes Service

Aprenda a implementar el contenedor de detección de idioma. Este procedimiento muestra cómo crear los contenedores de Docker locales, insertar los contenedores en su propio registro de contenedor privado, ejecutar el contenedor en un clúster de Kubernetes y probarlo en un explorador web.

## <a name="prerequisites"></a>Requisitos previos

Este procedimiento requiere varias herramientas que se deben instalar y ejecutar localmente. No use Azure Cloud Shell.

* Use una suscripción de Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/cognitive-services) antes de empezar.
* [Git](https://git-scm.com/downloads) para su sistema operativo, por lo que se puede clonar el [ejemplo](https://github.com/Azure-Samples/cognitive-services-containers-samples) usado en este procedimiento.
* [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest).
* [Motor de docker](https://www.docker.com/products/docker-engine) y se asegura de que la CLI de Docker funciona en una ventana de consola.
* [kubectl](https://storage.googleapis.com/kubernetes-release/release/v1.13.1/bin/windows/amd64/kubectl.exe).
* Un recurso de Azure con el plan de tarifa correcto. No todos los planes de tarifa funcionan con este contenedor:
  * El recurso **Text Analytics** solo con los planes de tarifas estándar o F0.
  * El recurso **Cognitive Services** con el plan de tarifas S0.

## <a name="running-the-sample"></a>Ejecución del ejemplo

Este procedimiento carga y ejecuta el ejemplo de contenedor de Cognitive Services para la detección de idioma. El ejemplo tiene dos contenedores, uno para la aplicación cliente y otro para el contenedor de Cognitive Services. Insertaremos ambas imágenes en la instancia de Azure Container Registry. Una vez que estén allí, cree una instancia de Azure Kubernetes Service para acceder a estas imágenes y ejecute los contenedores. Cuando se ejecuten los contenedores, use la CLI **kubectl** para ver su rendimiento. Acceda a la aplicación cliente con una solicitud HTTP y vea los resultados.

![Idea conceptual de la ejecución de contenedores de ejemplo](../text-analytics/media/how-tos/container-instance-sample/containers.png)

## <a name="the-sample-containers"></a>Los contenedores de ejemplo

El ejemplo tiene dos imágenes de contenedor, una para el sitio web de front-end. La segunda imagen es el contenedor de detección de idioma que devuelve el idioma detectado (referencia cultural) del texto. Ambos contenedores serán accesibles desde una dirección IP externa cuando haya terminado.

### <a name="the-language-frontend-container"></a>El contenedor language-front-end

Este sitio web es equivalente a su propia aplicación del lado cliente que realiza las solicitudes del punto de conexión de detección de idioma. Cuando finaliza el procedimiento, obtiene el idioma detectado de una cadena de caracteres al acceder al contenedor de sitio web en un explorador con `http://<external-IP>/<text-to-analyze>`. Un ejemplo de esta dirección URL es `http://132.12.23.255/helloworld!`. El resultado en el explorador es `English`.

### <a name="the-language-container"></a>El contenedor de idioma

El contenedor de detección de idioma, en este procedimiento específico, es accesible para cualquier solicitud externa. El contenedor no se ha modificado de ninguna manera para que esté disponible la API de detección de idioma específica del contenedor de Cognitive Services estándar.

Para este contenedor, esa API es una solicitud POST para la detección de idioma. Al igual que con todos los contenedores de Cognitive Services, puede obtener más información sobre el contenedor con la información hospedada de Swagger, `http://<external-IP>:5000/swagger/index.html`.

El puerto 5000 es el predeterminado que se usa con los contenedores de Cognitive Services.

## <a name="create-azure-container-registry-service"></a>Creación de un servicio Azure Container Registry

Para implementar el contenedor en Azure Kubernetes Service, las imágenes de contenedor deben ser accesibles. Cree su propio servicio Azure Container Registry para hospedar las imágenes.

1. Inicio de sesión en la CLI de Azure

    ```azurecli-interactive
    az login
    ```

1. Cree un grupo de recursos denominado `cogserv-container-rg` para contener todos los recursos creados en este procedimiento.

    ```azurecli-interactive
    az group create --name cogserv-container-rg --location westus
    ```

1. Cree su propia instancia de Azure Container Registry con su nombre y después `registry`, como en `pattyregistry`. No utilice guiones ni caracteres de subrayado en el nombre.

    ```azurecli-interactive
    az acr create --resource-group cogserv-container-rg --name pattyregistry --sku Basic
    ```

    Guarde los resultados para obtener la propiedad **loginServer**. Formará parte de la dirección del contenedor hospedado, que luego se usa en el archivo `language.yml`.

    ```azurecli-interactive
    az acr create --resource-group cogserv-container-rg --name pattyregistry --sku Basic
    ```

    ```output
    {
        "adminUserEnabled": false,
        "creationDate": "2019-01-02T23:49:53.783549+00:00",
        "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/cogserv-container-rg/providers/Microsoft.ContainerRegistry/registries/pattyregistry",
        "location": "westus",
        "loginServer": "pattyregistry.azurecr.io",
        "name": "pattyregistry",
        "provisioningState": "Succeeded",
        "resourceGroup": "cogserv-container-rg",
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

1. Inicie sesión en el registro de contenedor. Debe iniciar sesión para poder insertar imágenes en el Registro.

    ```azurecli-interactive
    az acr login --name pattyregistry
    ```

## <a name="get-website-docker-image"></a>Obtención de la imagen de Docker del sitio web

1. El ejemplo de código usado en este procedimiento está en el repositorio de ejemplos de contenedores de Cognitive Services. Clone el repositorio para tener una copia local del ejemplo.

    ```console
    git clone https://github.com/Azure-Samples/cognitive-services-containers-samples
    ```

    Una vez que el repositorio esté en el equipo local, busque el sitio web en el directorio [\dotnet\Language\FrontendService](https://github.com/Azure-Samples/cognitive-services-containers-samples/tree/master/dotnet/Language/FrontendService). Este sitio web actúa como la aplicación cliente al llamar a la API de detección de idioma hospedada en el contenedor de detección de idioma.  

1. Genere la imagen de Docker para este sitio web. Asegúrese de que la consola está en el directorio [\FrontendService](https://github.com/Azure-Samples/cognitive-services-containers-samples/tree/master/dotnet/Language/FrontendService) donde se encuentra el archivo Dockerfile al ejecutar el comando siguiente:

    ```console
    docker build -t language-frontend -t pattiyregistry.azurecr.io/language-frontend:v1 .
    ```

    Para realizar un seguimiento de la versión en el Registro de contenedor, agregue la etiqueta con un formato de versión, como `v1`.

1. Inserte la imagen en el registro de contenedor. Esta operación puede tardar unos minutos.

    ```console
    docker push pattyregistry.azurecr.io/language-frontend:v1
    ```

    Si se produce un error `unauthorized: authentication required`, inicie sesión con el comando `az acr login --name <your-container-registry-name>`. 

    Cuando el proceso concluya, los resultados deben ser similares a:

    ```output
    The push refers to repository [pattyregistry.azurecr.io/language-frontend]
    82ff52ee6c73: Pushed
    07599c047227: Pushed
    816caf41a9a1: Pushed
    2924be3aed17: Pushed
    45b83a23806f: Pushed
    ef68f6734aa4: Pushed
    v1: digest: sha256:31930445deee181605c0cde53dab5a104528dc1ff57e5b3b34324f0d8a0eb286 size: 1580
    ```

## <a name="get-language-detection-docker-image"></a>Obtención de la imagen de Docker de detección de idioma

1. Extraiga la versión más reciente de la imagen de Docker en el equipo local. Esta operación puede tardar unos minutos. Si hay una versión más reciente de este contenedor, cambie el valor de `1.1.006770001-amd64-preview` por la versión más reciente.

    ```console
    docker pull mcr.microsoft.com/azure-cognitive-services/language:1.1.006770001-amd64-preview
    ```

1. Etiquete la imagen con el Registro de contenedor. Busque la versión más reciente y reemplace la versión `1.1.006770001-amd64-preview` si tiene una más reciente. 

    ```console
    docker tag mcr.microsoft.com/azure-cognitive-services/language pattiyregistry.azurecr.io/language:1.1.006770001-amd64-preview
    ```

1. Inserte la imagen en el registro de contenedor. Esta operación puede tardar unos minutos.

    ```console
    docker push pattyregistry.azurecr.io/language:1.1.006770001-amd64-preview
    ```

## <a name="get-container-registry-credentials"></a>Obtención de las credenciales de Container Registry

Los pasos siguientes son necesarios para obtener la información necesaria para conectar el Registro de contenedor con Azure Kubernetes Service que creará posteriormente en este procedimiento.

1. Cree una entidad de servicio.

    ```azurecli-interactive
    az ad sp create-for-rbac --skip-assignment
    ```

    Guarde el valor de `appId` de los resultados para el parámetro de usuario asignado en el paso 3, `<appId>`. Guare la `password` para el parámetro del secreto de cliente `<client-secret>` de la sección siguiente.

    ```output
    {
      "appId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "displayName": "azure-cli-2018-12-31-18-39-32",
      "name": "http://azure-cli-2018-12-31-18-39-32",
      "password": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "tenant": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
    ```

1. Obtenga el identificador del Registro de contenedor.

    ```azurecli-interactive
    az acr show --resource-group cogserv-container-rg --name pattyregistry --query "id" --o table
    ```

    Guarde la salida para el valor del parámetro de ámbito, `<acrId>`, en el paso siguiente. Su aspecto es similar a:

    ```output
    /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/cogserv-container-rg/providers/Microsoft.ContainerRegistry/registries/pattyregistry
    ```

    Guarde el valor completo para el paso 3 de esta sección.

1. Para conceder el acceso correcto al clúster de AKS para usar las imágenes almacenadas en el Registro de contenedor, cree una asignación de roles. Reemplace `<appId>` y `<acrId>` por los valores recopilados en los dos pasos anteriores.

    ```azurecli-interactive
    az role assignment create --assignee <appId> --scope <acrId> --role Reader
    ```

## <a name="create-azure-kubernetes-service"></a>Creación de Azure Kubernetes Service

1. Cree un clúster de Kubernetes. Todos los valores de parámetro son los de las secciones anteriores, excepto el parámetro name. Elija un nombre que indique quién lo creó y su propósito, como `patty-kube`.

    ```azurecli-interactive
    az aks create --resource-group cogserv-container-rg --name patty-kube --node-count 2  --service-principal <appId>  --client-secret <client-secret>  --generate-ssh-keys
    ```

    Esta operación puede tardar unos minutos. El resultado es el siguiente:

    ```output
    {
      "aadProfile": null,
      "addonProfiles": null,
      "agentPoolProfiles": [
        {
          "count": 2,
          "dnsPrefix": null,
          "fqdn": null,
          "maxPods": 110,
          "name": "nodepool1",
          "osDiskSizeGb": 30,
          "osType": "Linux",
          "ports": null,
          "storageProfile": "ManagedDisks",
          "vmSize": "Standard_DS1_v2",
          "vnetSubnetId": null
        }
      ],
      "dnsPrefix": "patty-kube--65a101",
      "enableRbac": true,
      "fqdn": "patty-kube--65a101-341f1f54.hcp.westus.azmk8s.io",
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/cogserv-container-rg/providers/Microsoft.ContainerService/managedClusters/patty-kube",
      "kubernetesVersion": "1.9.11",
      "linuxProfile": {
        "adminUsername": "azureuser",
        "ssh": {
          "publicKeys": [
            {
              "keyData": "ssh-rsa AAAAB3NzaC...ohR2d81mFC
            }
          ]
        }
      },
      "location": "westus",
      "name": "patty-kube",
      "networkProfile": {
        "dnsServiceIp": "10.0.0.10",
        "dockerBridgeCidr": "172.17.0.1/16",
        "networkPlugin": "kubenet",
        "networkPolicy": null,
        "podCidr": "10.244.0.0/16",
        "serviceCidr": "10.0.0.0/16"
      },
      "nodeResourceGroup": "MC_patty_westus",
      "provisioningState": "Succeeded",
      "resourceGroup": "cogserv-container-rg",
      "servicePrincipalProfile": {
        "clientId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "keyVaultSecretRef": null,
        "secret": null
      },
      "tags": null,
      "type": "Microsoft.ContainerService/ManagedClusters"
    }
    ```

    Se crea el servicio, pero aún no tiene el contenedor de sitio web o un contenedor de detección de idioma.  

1. Obtenga las credenciales del clúster de Kubernetes.

    ```azurecli-interactive
    az aks get-credentials --resource-group cogserv-container-rg --name patty-kube
    ```

## <a name="load-the-orchestration-definition-into-your-kubernetes-service"></a>Carga de la definición de la orquestación en el servicio de Kubernetes

Esta sección usa la CLI **kubectl** para comunicarse con Azure Kubernetes Service.

1. Antes de cargar la definición de la orquestación, compruebe que **kubectl** tenga acceso a los nodos.

    ```console
    kubectl get nodes
    ```

    La respuesta tiene el siguiente aspecto:

    ```output
    NAME                       STATUS    ROLES     AGE       VERSION
    aks-nodepool1-13756812-0   Ready     agent     6m        v1.9.11
    aks-nodepool1-13756812-1   Ready     agent     6m        v1.9.11
    ```

1. Copie el archivo siguiente y asígnele el nombre `language.yml`. El archivo tiene una sección `service` y otra sección `deployment` para cada uno de los dos tipos de contenedor, el contenedor del sitio web `language-frontend` y el contenedor de detección de `language`.

    [!code-yml[Kubernetes orchestration file for the Cognitive Services containers sample](~/samples-cogserv-containers/Kubernetes/language/language.yml "Kubernetes orchestration file for the Cognitive Services containers sample")]

1. Cambie las líneas de la implementación de language-front-end de `language.yml` según la tabla siguiente para agregar su propia configuración del análisis de texto, el secreto del cliente y los nombres de imagen del Registro de contenedor.

    Configuración de implementación de Language-front-end|Propósito|
    |--|--|
    |Línea 32<br> Propiedad `image`|Ubicación de la imagen para la imagen de front-end en Container Registry<br>`<container-registry-name>.azurecr.io/language-frontend:v1`|
    |Linea 44<br> Propiedad `name`|Secreto de Container Registry para la imagen, conocido como `<client-secret>` en una sección anterior.|

1. Cambie las líneas de la implementación de idioma de `language.yml` según la tabla siguiente para agregar su propia configuración del análisis de texto, el secreto del cliente y los nombres de imagen del Registro de contenedor.

    |Configuración de la implementación de idioma|Propósito|
    |--|--|
    |Línea 78<br> Propiedad `image`|Ubicación de la imagen para la imagen de idioma de Container Registry<br>`<container-registry-name>.azurecr.io/language:1.1.006770001-amd64-preview`|
    |Línea 95<br> Propiedad `name`|Secreto de Container Registry para la imagen, conocido como `<client-secret>` en una sección anterior.|
    |Línea 91<br> Propiedad `apiKey`|Su clave del recurso de análisis de texto|
    |Línea 92<br> Propiedad `billing`|El punto de conexión de facturación para el recurso de análisis de texto.<br>`https://westus.api.cognitive.microsoft.com/text/analytics/v2.1`|

    Dado que los valores de **apiKey** y **punto de conexión de facturación** se establecen como parte de la definición de la orquestación de Kubernetes, el contenedor de sitio web no necesita saber acerca de ellos ni pasarlos como parte de la solicitud. El contenedor de sitio web hace referencia al contenedor de detección de idioma por su nombre de orquestador `language`.

1. Cargue el archivo de definición de la orquestación de este ejemplo desde la carpeta donde ha creado y guardado el `language.yml`.

    ```console
    kubectl apply -f language.yml
    ```

    La respuesta es:

    ```output
    service "language-frontend" created
    deployment.apps "language-frontend" created
    service "language" created
    deployment.apps "language" created
    ```

## <a name="get-external-ips-of-containers"></a>Obtención de las direcciones IP externas de los contenedores

Para los dos contenedores, compruebe que los servicios `language-frontend` y `language` se ejecutan y obtenga la dirección IP externa.

```console
kubectl get all
```

```output
NAME                                     READY     STATUS    RESTARTS   AGE
pod/language-586849d8dc-7zvz5            1/1       Running   0          13h
pod/language-frontend-68b9969969-bz9bg   1/1       Running   1          13h

NAME                        TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)          AGE
service/kubernetes          ClusterIP      10.0.0.1      <none>          443/TCP          14h
service/language            LoadBalancer   10.0.39.169   104.42.172.68   5000:30161/TCP   13h
service/language-frontend   LoadBalancer   10.0.42.136   104.42.37.219   80:30943/TCP     13h

NAME                                      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/language            1         1         1            1           13h
deployment.extensions/language-frontend   1         1         1            1           13h

NAME                                                 DESIRED   CURRENT   READY     AGE
replicaset.extensions/language-586849d8dc            1         1         1         13h
replicaset.extensions/language-frontend-68b9969969   1         1         1         13h

NAME                                DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/language            1         1         1            1           13h
deployment.apps/language-frontend   1         1         1            1           13h

NAME                                           DESIRED   CURRENT   READY     AGE
replicaset.apps/language-586849d8dc            1         1         1         13h
replicaset.apps/language-frontend-68b9969969   1         1         1         13h
```

Si la dirección `EXTERNAL-IP` para el servicio se muestra como pendiente, vuelva a ejecutar el comando hasta que se muestre la dirección IP antes de continuar con el paso siguiente.

## <a name="test-the-language-detection-container"></a>Prueba del contenedor de detección de idioma

Abra un explorador y vaya a la dirección IP externa del contenedor `language` de la sección anterior: `http://<external-ip>:5000/swagger/index.html`. Puede usar la característica `Try it` de la API para probar el punto de conexión de detección de idioma.

![Visualización de la documentación de Swagger del contenedor](../text-analytics/media/how-tos/container-instance-sample/language-detection-container-swagger-documentation.png)

## <a name="test-the-client-application-container"></a>Prueba del contenedor de la aplicación cliente

Cambie la dirección URL en el explorador por la dirección IP externa del contenedor `language-frontend` con el siguiente formato: `http://<external-ip>/helloworld`. El texto de la referencia cultural de inglés `helloworld` se predice como `English`.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado con el clúster, elimine el grupo de recursos de Azure.

```azurecli-interactive
az group delete --name cogserv-container-rg
```

## <a name="related-information"></a>Información relacionada

* [kubectl para los usuarios de Docker](https://kubernetes.io/docs/reference/kubectl/docker-cli-to-kubectl/)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Contenedores de Cognitive Services](../cognitive-services-container-support.md)
