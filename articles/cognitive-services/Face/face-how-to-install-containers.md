---
title: 'Instalación y ejecución de contenedores: Face'
titleSuffix: Azure Cognitive Services
description: En este artículo se muestra cómo descargar, instalar y ejecutar contenedores para Face en un tutorial paso a paso.
services: cognitive-services
author: aahill
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: conceptual
ms.date: 07/16/2020
ms.author: aahi
ms.openlocfilehash: 766af570c959ff2c49256058dfbfffdd1021295f
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88548470"
---
# <a name="install-and-run-face-containers-preview"></a>Instalación y ejecución de contenedores de Face (versión preliminar)

> [!IMPORTANT]
> Se ha alcanzado el límite de usuarios del contenedor de Face. Actualmente no aceptamos nuevas aplicaciones para el contenedor de Face.

Azure Cognitive Services Face proporciona un contenedor de Linux estandarizado para Docker que detecta rostros humanos en imágenes. También identifica atributos, incluidos faciales (como narices y ojos), sexo, edad y otras características faciales previstas por la máquina. Además de la detección, Face puede comprobar si dos rostros en la misma o en imágenes diferentes coinciden mediante una puntuación de confianza. Face también puede comparar rostros en una base de datos para ver si ya existe algún otro rostro similar o idéntico. También puede organizar rostros similares en grupos mediante rasgos visuales compartidos.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Debe cumplir los siguientes requisitos previos para poder usar los contenedores del servicio Face.

|Obligatorio|Propósito|
|--|--|
|Motor de Docker| El motor de Docker debe estar instalado en un [equipo host](#the-host-computer). Docker dispone de paquetes que configuran el entorno de Docker en [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) y [Linux](https://docs.docker.com/engine/installation/#supported-platforms). Para conocer los principios básicos de Docker y de los contenedores, consulte [Introducción a Docker](https://docs.docker.com/engine/docker-overview/).<br><br> Docker debe configurarse para permitir que los contenedores se conecten con Azure y envíen datos de facturación a dicho servicio. <br><br> En Windows, Docker también debe configurarse para admitir los contenedores de Linux.<br><br>|
|Conocimientos sobre Docker | Debe tener conocimientos básicos sobre los conceptos de Docker, como los registros, los repositorios, los contenedores y las imágenes de contenedor. También necesita conocimientos sobre comandos de `docker` básicos.| 
|Recurso de Face |Para usar el contenedor, debe tener:<br><br>Un recurso de Azure **Face** y la clave de API y el URI de punto de conexión asociados. Ambos valores están disponibles en las páginas de **introducción** y **claves** del recurso. Esto es necesario para iniciar el contenedor.<br><br>**{API_KEY}** : una de las dos claves de recurso disponibles en la página **Claves**<br><br>**{ENDPOINT_URI}** : el punto de conexión tal como se proporciona en la página de **Información general**.

[!INCLUDE [Gathering required container parameters](../containers/includes/container-gathering-required-parameters.md)]

### <a name="the-host-computer"></a>El equipo host

[!INCLUDE [Host Computer requirements](../../../includes/cognitive-services-containers-host-computer.md)]

### <a name="container-requirements-and-recommendations"></a>Recomendaciones y requisitos del contenedor

En la tabla siguiente se describen los núcleos de CPU y memoria mínimos y recomendados para asignar a cada contenedor del servicio Face.

| Contenedor | Mínima | Recomendado | Transacciones por segundo<br>(mínimo, máximo)|
|-----------|---------|-------------|--|
|Caras | 1 núcleo, 2 GB de memoria | 1 núcleos, 4 GB de memoria |10, 20|

* Cada núcleo debe ser de 2,6 GHz como mínimo.
* Transacciones por segundo (TPS).

El núcleo y la memoria se corresponden con los valores de `--cpus` y `--memory` que se usan como parte del comando `docker run`.

## <a name="get-the-container-image-with-docker-pull"></a>Obtención de la imagen de contenedor con el comando docker pull

Las imágenes de contenedor del servicio Face están disponibles. 

| Contenedor | Repositorio |
|-----------|------------|
| Caras | `containerpreview.azurecr.io/microsoft/cognitive-services-face:latest` |

[!INCLUDE [Tip for using docker list](../../../includes/cognitive-services-containers-docker-list-tip.md)]

### <a name="docker-pull-for-the-face-container"></a>Docker pull para el contenedor de Face

```
docker pull containerpreview.azurecr.io/microsoft/cognitive-services-face:latest
```

## <a name="use-the-container"></a>Uso del contenedor

Cuando el contenedor esté en el [equipo host](#the-host-computer), use el siguiente proceso para trabajar con el contenedor.

1. [Ejecute el contenedor](#run-the-container-with-docker-run) con la configuración de facturación requerida. Hay más [ejemplos](./face-resource-container-config.md#example-docker-run-commands) del comando `docker run` disponibles. 
1. [Consulta del punto de conexión de predicción del contenedor](#query-the-containers-prediction-endpoint). 

## <a name="run-the-container-with-docker-run"></a>Ejecute el contenedor con docker run

Utilice el comando [docker run](https://docs.docker.com/engine/reference/commandline/run/) para ejecutar el contenedor. Consulte [Recopilación de los parámetros obligatorios](#gathering-required-parameters) para más información sobre cómo obtener los valores de `{ENDPOINT_URI}` y `{API_KEY}`.

Hay disponibles [ejemplos](face-resource-container-config.md#example-docker-run-commands) del comando `docker run`.

```bash
docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 \
containerpreview.azurecr.io/microsoft/cognitive-services-face \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Este comando:

* Ejecuta un contenedor de Face desde la imagen de contenedor.
* Asigna un núcleo de CPU y 4 GB de memoria.
* Expone el puerto TCP 5000 y asigna un seudo-TTY para el contenedor.
* Una vez que se produce la salida, quita automáticamente el contenedor. La imagen del contenedor sigue estando disponible en el equipo host. 

Hay más [ejemplos](./face-resource-container-config.md#example-docker-run-commands) del comando `docker run` disponibles. 

> [!IMPORTANT]
> Para poder ejecutar el contenedor, las opciones `Eula`, `Billing` y `ApiKey` deben estar especificadas; de lo contrario, el contenedor no se iniciará. Para obtener más información, vea [Facturación](#billing).

[!INCLUDE [Running multiple containers on the same host](../../../includes/cognitive-services-containers-run-multiple-same-host.md)]


## <a name="query-the-containers-prediction-endpoint"></a>Consulta del punto de conexión de predicción del contenedor

El contenedor proporciona varias API de puntos de conexión de predicción de consultas basadas en REST. 

Utilice el host, `http://localhost:5000`, con las API de contenedor.


<!--  ## Validate container is running -->

[!INCLUDE [Container's API documentation](../../../includes/cognitive-services-containers-api-documentation.md)]

## <a name="stop-the-container"></a>Detención del contenedor

[!INCLUDE [How to stop the container](../../../includes/cognitive-services-containers-stop.md)]

## <a name="troubleshooting"></a>Solución de problemas

Si ejecuta el contenedor con un [montaje](./face-resource-container-config.md#mount-settings) de salida y el registro habilitados, el contenedor genera archivos de registro que resultan útiles para solucionar problemas que se producen al iniciar o ejecutar el contenedor.

[!INCLUDE [Cognitive Services FAQ note](../containers/includes/cognitive-services-faq-note.md)]

## <a name="billing"></a>Facturación

Los contenedores del servicio Face envían información de facturación a Azure mediante un recurso de Face en la cuenta de Azure. 

[!INCLUDE [Container's Billing Settings](../../../includes/cognitive-services-containers-how-to-billing-info.md)]

Para obtener más información acerca de estas opciones, consulte [Configure containers](./face-resource-container-config.md) (Configuración de contenedores).

<!--blogs/samples/video coures -->

[!INCLUDE [Discoverability of more container information](../../../includes/cognitive-services-containers-discoverability.md)]

## <a name="summary"></a>Resumen

En este artículo aprendió los conceptos y el flujo de trabajo para la descarga, instalación y ejecución de contenedores del servicio Face. En resumen:

* Las imágenes de contenedor se descargan de Azure Container Registry.
* Las imágenes de contenedor se ejecutan en Docker.
* Puede usar la API REST o el SDK para llamar a operaciones en contenedores del servicio Face mediante la especificación del URI del host del contenedor.
* Debe especificar la información de facturación al crear una instancia de contenedor.

> [!IMPORTANT]
> Los contenedores de Cognitive Services no tienen licencia para ejecutarse sin estar conectados a Azure para realizar mediciones. Los clientes tienen que habilitar los contenedores para comunicar la información de facturación con el servicio de medición en todo momento. Los contenedores de Cognitive Services no envían datos de los clientes (por ejemplo, la imagen o el texto que se está analizando) a Microsoft.

## <a name="next-steps"></a>Pasos siguientes

* Para ver las opciones de configuración, vea [Configuración de contenedores](face-resource-container-config.md).
* Para obtener más información sobre cómo detectar e identificar rostros, vea [Face overview](Overview.md) (Introducción a Face).
* Para obtener más información acerca de los métodos que admite el contenedor, vea [Face API](//westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236).
* Para usar más contenedores de Cognitive Services, ver [Contenedores de Cognitive Services](../cognitive-services-container-support.md).
