---
title: Instalación y ejecución de contenedores para usar Anomaly Detector API
titleSuffix: Azure Cognitive Services
description: Aprenda a instalar y utilizar el contenedor para emplear los algoritmos avanzados de la API de Anomaly Detector.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: conceptual
ms.date: 09/10/2020
ms.author: aahi
ms.openlocfilehash: 2a4ff7da16524e0706601e43dff39325952990ff
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90903553"
---
# <a name="install-and-run-anomaly-detector-containers"></a>Instalación y ejecución de contenedores de Anomaly Detector 

[!INCLUDE [container image location note](../containers/includes/image-location-note.md)]

Anomaly Detector tiene la siguiente funcionalidad de la característica de contenedor:

| Función | Características |
|--|--|
| Anomaly Detector | <li> Detecta las anomalías que se producen en tiempo real. <li> Detecta las anomalías en todo el conjunto de datos como un lote. <li> Detecta puntos de cambio de tendencia en el conjunto de datos como lote.<li> Deduce el intervalo normal esperado de los datos. <li> Permite el ajuste de la sensibilidad de la detección de anomalías para adaptarse mejor a los datos. |

Para más información sobre las API, consulte:
* [Más información sobre el servicio de API Anomaly Detector](https://go.microsoft.com/fwlink/?linkid=2080698&clcid=0x409)

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Debe cumplir los siguientes requisitos previos para poder usar los contenedores de Anomaly Detector:

|Obligatorio|Propósito|
|--|--|
|Motor de Docker| Necesita que el motor de Docker esté instalado en un [equipo host](#the-host-computer). Docker dispone de paquetes que configuran el entorno de Docker en [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) y [Linux](https://docs.docker.com/engine/installation/#supported-platforms). Para conocer los principios básicos de Docker y de los contenedores, consulte [Introducción a Docker](https://docs.docker.com/engine/docker-overview/).<br><br> Docker debe configurarse para permitir que los contenedores se conecten con Azure y envíen datos de facturación a dicho servicio. <br><br> **En Windows**, Docker también debe estar configurado de forma que admita los contenedores de Linux.<br><br>|
|Conocimientos sobre Docker | Debe tener conocimientos básicos sobre los conceptos de Docker, como los registros, los repositorios, los contenedores y las imágenes de contenedor, así como conocer los comandos `docker` básicos.|
|Recurso de Anomaly Detector |Para usar estos contenedores, debe tener:<br><br>Un recurso de _Anomaly Detector_ de Azure para obtener la clave de API y el URI de punto de conexión asociados. Ambos valores están disponibles en las páginas de claves y de información general de **Anomaly Detector** en Azure Portal y son necesarios para iniciar el contenedor.<br><br>**{API_KEY}** : una de las dos claves de recurso disponibles en la página **Claves**<br><br>**{ENDPOINT_URI}** : el punto de conexión tal como se proporciona en la página de **Información general**.|

[!INCLUDE [Gathering required container parameters](../containers/includes/container-gathering-required-parameters.md)]

## <a name="the-host-computer"></a>El equipo host

[!INCLUDE [Host Computer requirements](../../../includes/cognitive-services-containers-host-computer.md)]

<!--* [Azure IoT Edge](https://docs.microsoft.com/azure/iot-edge/). For instructions of deploying Anomaly Detector module in IoT Edge, see [How to deploy Anomaly Detector module in IoT Edge](how-to-deploy-anomaly-detector-module-in-iot-edge.md).-->

### <a name="container-requirements-and-recommendations"></a>Recomendaciones y requisitos del contenedor

En la tabla siguiente se describen los núcleos de CPU y los valores de memoria mínimos y recomendados para asignar a cada contenedor de Anomaly Detector.

| QPS (Consultas por segundo) | Mínima | Recomendado |
|-----------|---------|-------------|
| 10 QPS | 4 núcleos, 1 GB de memoria | 8 núcleos, 2 GB de memoria |
| 20 QPS | 8 núcleos, 2 GB de memoria | 16 núcleos, 4 GB de memoria |

Cada núcleo debe ser de 2,6 gigahercios (GHz) como mínimo.

El núcleo y la memoria se corresponden con los valores de `--cpus` y `--memory` que se usan como parte del comando `docker run`.

## <a name="get-the-container-image-with-docker-pull"></a>Obtención de la imagen del contenedor con `docker pull`

Use el comando [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull/) para descargar una imagen de contenedor.

| Contenedor | Repositorio |
|-----------|------------|
| cognitive-services-anomaly-detector | `mcr.microsoft.com/azure-cognitive-services/decision/anomaly-detector:latest` |

<!--
For a full description of available tags, such as `latest` used in the preceding command, see [anomaly-detector](https://go.microsoft.com/fwlink/?linkid=2083827&clcid=0x409) on Docker Hub.
-->
[!INCLUDE [Tip for using docker list](../../../includes/cognitive-services-containers-docker-list-tip.md)]

### <a name="docker-pull-for-the-anomaly-detector-container"></a>docker pull del contenedor de Anomaly Detector

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/anomaly-detector:latest
```

## <a name="how-to-use-the-container"></a>Uso del contenedor

Una vez que el contenedor esté en el [equipo host](#the-host-computer), utilice el siguiente proceso para trabajar con el contenedor.

1. [Ejecute el contenedor](#run-the-container-with-docker-run) con la configuración de facturación requerida. Hay más [ejemplos](anomaly-detector-container-configuration.md#example-docker-run-commands) del comando `docker run` disponibles.
1. [Consulta del punto de conexión de predicción del contenedor](#query-the-containers-prediction-endpoint).

## <a name="run-the-container-with-docker-run"></a>Ejecute el contenedor con `docker run`.

Utilice el comando [docker run](https://docs.docker.com/engine/reference/commandline/run/) para ejecutar el contenedor. Consulte [Recopilación de los parámetros obligatorios](#gathering-required-parameters) para más información sobre cómo obtener los valores de `{ENDPOINT_URI}` y `{API_KEY}`.

Hay disponibles [ejemplos](anomaly-detector-container-configuration.md#example-docker-run-commands) del comando `docker run`.

```bash
docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 \
mcr.microsoft.com/azure-cognitive-services/decision/anomaly-detector:latest \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Este comando:

* Ejecuta un contenedor de Anomaly Detector desde la imagen de contenedor.
* Asigna un núcleo de CPU y 4 gigabytes (GB) de memoria.
* Expone el puerto TCP 5000 y asigna un seudo-TTY para el contenedor.
* Una vez que se produce la salida, quita automáticamente el contenedor. La imagen del contenedor sigue estando disponible en el equipo host.

> [!IMPORTANT]
> Para poder ejecutar el contenedor, las opciones `Eula`, `Billing` y `ApiKey` deben estar especificadas; de lo contrario, el contenedor no se iniciará.  Para obtener más información, vea [Facturación](#billing).

### <a name="running-multiple-containers-on-the-same-host"></a>Ejecución de varios contenedores en el mismo host

Si tiene pensado ejecutar varios contenedores con puertos expuestos, asegúrese de ejecutar cada contenedor con un puerto diferente. Por ejemplo, ejecute el primer contenedor en el puerto 5000 y el segundo en el puerto 5001.

Reemplace `<container-registry>` y `<container-name>` por los valores de los contenedores que utilice. Estos contenedores pueden ser diferentes. Puede hacer que el contenedor de Anomaly Detector y el contenedor de LUIS se ejecuten juntos en el HOST, o puede hacer que se ejecuten varios contenedores de Anomaly Detector.

Ejecute el primer contenedor en el puerto 5000.

```bash
docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 \
<container-registry>/microsoft/<container-name> \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Ejecute el segundo contenedor en el puerto 5001.


```bash
docker run --rm -it -p 5000:5001 --memory 4g --cpus 1 \
<container-registry>/microsoft/<container-name> \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Cada contenedor sucesivo debe estar en un puerto diferente.

## <a name="query-the-containers-prediction-endpoint"></a>Consulta del punto de conexión de predicción del contenedor

El contenedor proporciona varias API de puntos de conexión de predicción de consultas basadas en REST.

Utilice el host, http://localhost:5000, con las API de contenedor.

<!--  ## Validate container is running -->

[!INCLUDE [Container's API documentation](../../../includes/cognitive-services-containers-api-documentation.md)]

## <a name="stop-the-container"></a>Detención del contenedor

[!INCLUDE [How to stop the container](../../../includes/cognitive-services-containers-stop.md)]

## <a name="troubleshooting"></a>Solución de problemas

Si ejecuta el contenedor con un [montaje](anomaly-detector-container-configuration.md#mount-settings) de salida y el registro habilitados, el contenedor genera archivos de registro que resultan útiles para solucionar problemas que se producen al iniciar o ejecutar el contenedor.

[!INCLUDE [Cognitive Services FAQ note](../containers/includes/cognitive-services-faq-note.md)]

## <a name="billing"></a>Facturación

Los contenedores de Anomaly Detector envían información de facturación a Azure mediante un recurso de _Anomaly Detector_ de la cuenta de Azure.

[!INCLUDE [Container's Billing Settings](../../../includes/cognitive-services-containers-how-to-billing-info.md)]

Para obtener más información acerca de estas opciones, consulte [Configure containers](anomaly-detector-container-configuration.md) (Configuración de contenedores).

<!--blogs/samples/video coures -->

[!INCLUDE [Discoverability of more container information](../../../includes/cognitive-services-containers-discoverability.md)]

## <a name="summary"></a>Resumen

En este artículo, ha aprendido los conceptos y el flujo de trabajo de la descarga, la instalación y la ejecución de contenedores de Anomaly Detector. En resumen:

* Anomaly Detector proporciona un contenedor Linux para Docker, que encapsula la detección de anomalías con lotes frente a transmisión por secuencias, inferencia de intervalo esperado y ajuste de sensibilidad.
* Las imágenes de contenedor se descargan de una instancia privada de Azure Container Registry dedicada para contenedores.
* Las imágenes de contenedor se ejecutan en Docker.
* Puede usar la API REST o el SDK para llamar a operaciones de contenedores de Anomaly Detector mediante la especificación del URI del host del contenedor.
* Debe especificar la información de facturación al crear una instancia de un contenedor.

> [!IMPORTANT]
> Los contenedores de Cognitive Services no tienen licencia para ejecutarse sin estar conectados a Azure para realizar mediciones. Los clientes tienen que habilitar los contenedores para comunicar la información de facturación con el servicio de medición en todo momento. Los contenedores de Cognitive Services no envían datos de los clientes (por ejemplo, los datos de serie temporal que se analizan) a Microsoft.

## <a name="next-steps"></a>Pasos siguientes

* Revise [Configure containers](anomaly-detector-container-configuration.md) (Configuración de contenedores) para ver las opciones de configuración.
* [Implementación de un contenedor Anomaly Detector en Azure Container Instances](how-to/deploy-anomaly-detection-on-container-instances.md)
* [Más información sobre el servicio de API Anomaly Detector](https://go.microsoft.com/fwlink/?linkid=2080698&clcid=0x409)
