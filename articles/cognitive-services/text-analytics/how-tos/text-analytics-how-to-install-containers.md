---
title: 'Instalación y ejecución de contenedores: Text Analytics'
titleSuffix: Azure Cognitive Services
description: Cómo descargar, instalar y ejecutar contenedores para Text Analytics en este tutorial paso a paso.
services: cognitive-services
author: aahill
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: conceptual
ms.date: 07/07/2020
ms.author: aahi
ms.openlocfilehash: a83fd67a9ad5e5a36555fcbaed65b1d76936c33d
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90906009"
---
# <a name="install-and-run-text-analytics-containers"></a>Instalación y ejecución de contenedores de Text Analytics

> [!NOTE]
> * El contenedor de Análisis de sentimiento v3 ya está disponible con carácter general. Los contenedores de extracción de frases clave y de detección de idioma están disponibles como versión preliminar pública no validada.
> * La vinculación de entidades y NER no están disponibles actualmente como contenedor.
> * El acceso al Text Analytics para el contenedor de estado requiere un [formulario de solicitud](https://aka.ms/cognitivegate). Actualmente, no se le facturará por su uso.
> * Es posible que las ubicaciones de las imágenes de contenedor hayan cambiado recientemente. Lea este artículo para ver la ubicación actualizada de este contenedor.

Los contenedores permiten ejecutar las API Text Analytics en su propio entorno y son ideales para sus requisitos específicos de seguridad y gobernanza de datos. Los contenedores de Text Analytics proporcionan un procesamiento avanzado de lenguaje natural sobre texto sin formato, e incluye tres funciones principales: análisis de sentimiento, extracción de frases clave y detección de idioma. 

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/) antes de empezar.

> [!IMPORTANT]
> La cuenta gratuita se limita a 5.000 transacciones al mes y solo las **gratis** y **estándar** <a href="https://azure.microsoft.com/pricing/details/cognitive-services/text-analytics" target="_blank">los planes de tarifa <span class="docon docon-navigate-external x-hidden-focus"></span></a> son válidas para los contenedores. Para más información sobre las tasas de solicitudes de las transacciones, consulte [Límites de datos](https://docs.microsoft.com/azure/cognitive-services/text-analytics/overview#data-limits).

## <a name="prerequisites"></a>Requisitos previos

Para ejecutar cualquiera de los contenedores de Text Analytics, debe tener un equipo host y los entornos de contenedores.

## <a name="preparation"></a>Preparación

Debe cumplir los siguientes requisitos previos para poder usar contenedores de Text Analytics:

|Obligatorio|Propósito|
|--|--|
|Motor de Docker| Necesita que el motor de Docker esté instalado en un [equipo host](#the-host-computer). Docker dispone de paquetes que configuran el entorno de Docker en [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) y [Linux](https://docs.docker.com/engine/installation/#supported-platforms). Para conocer los principios básicos de Docker y de los contenedores, consulte [Introducción a Docker](https://docs.docker.com/engine/docker-overview/).<br><br> Docker debe configurarse para permitir que los contenedores se conecten con Azure y envíen datos de facturación a dicho servicio. <br><br> **En Windows**, Docker también debe estar configurado de forma que admita los contenedores de Linux.<br><br>|
|Conocimientos sobre Docker | Debe tener conocimientos básicos sobre los conceptos de Docker, como los registros, los repositorios, los contenedores y las imágenes de contenedor, así como conocer los comandos `docker` básicos.| 
|Recurso de Text Analytics |Para poder usar el contenedor, debe tener:<br><br>Un [recurso de Text Analytics](../../cognitive-services-apis-create-account.md) de Azure para obtener la clave de API y el URI de punto de conexión asociados. Ambos valores están disponibles en las páginas de claves y de información general de Text Analytics en Azure Portal y son necesarios para iniciar el contenedor.<br><br>**{API_KEY}** : una de las dos claves de recurso disponibles en la página **Claves**<br><br>**{ENDPOINT_URI}** : el punto de conexión tal como se proporciona en la página de **Información general**.|

[!INCLUDE [Gathering required parameters](../../containers/includes/container-gathering-required-parameters.md)]

## <a name="the-host-computer"></a>El equipo host

[!INCLUDE [Host Computer requirements](../../../../includes/cognitive-services-containers-host-computer.md)]

### <a name="container-requirements-and-recommendations"></a>Recomendaciones y requisitos del contenedor

En la tabla siguiente se describen las especificaciones mínima y recomendada para los contenedores de Text Analytics. Se necesitan al menos 2 gigabytes (GB) de memoria y cada núcleo de CPU debe ser de 2,6 gigahercios (GHz) como mínimo o más rápido. También se enumeran las transacciones permitidas por sección (TPS).

|  | Especificaciones de host mínimas | Especificaciones de host recomendadas | TPS mínimas | TPS máximas|
|---|---------|-------------|--|--|
| **Detección de idioma, extracción de frases clave**   | 1 núcleo, 2 GB de memoria | 1 núcleo, 4 GB de memoria |15 | 30|
| **Análisis de sentimiento, versión 3**   | 1 núcleo, 2 GB de memoria | 4 núcleos, 8 GB de memoria |15 | 30|
| **Text Analytics para el estado: 1 documento/solicitud**   |  4 núcleos, 10 GB de memoria | 6 núcleos, 12 GB de memoria |15 | 30|
| **Text Analytics para el estado: 10 documento/solicitud**   |  6 núcleos, 16 GB de memoria | 8 núcleos, 20 GB de memoria |15 | 30|

El núcleo de CPU y la memoria corresponden a los valores de `--cpus` y `--memory` que se usan como parte del comando `docker run`.

## <a name="get-the-container-image-with-docker-pull"></a>Obtención de la imagen del contenedor con `docker pull`

[!INCLUDE [Tip for using docker list](../../../../includes/cognitive-services-containers-docker-list-tip.md)]

Las imágenes de contenedor de Text Analytics están disponibles en Microsoft Container Registry.

# <a name="sentiment-analysis-v3"></a>[Análisis de sentimiento, versión 3](#tab/sentiment)

[!INCLUDE [docker-pull-sentiment-analysis-container](../includes/docker-pull-sentiment-analysis-container.md)]

# <a name="key-phrase-extraction-preview"></a>[Extracción de frases clave (versión preliminar)](#tab/keyphrase)

[!INCLUDE [docker-pull-key-phrase-extraction-container](../includes/docker-pull-key-phrase-extraction-container.md)]

# <a name="language-detection-preview"></a>[Detección de idioma (versión preliminar)](#tab/language)

[!INCLUDE [docker-pull-language-detection-container](../includes/docker-pull-language-detection-container.md)]

# <a name="text-analytics-for-health-preview"></a>[Text Analytics para el estado (versión preliminar)](#tab/healthcare)

[!INCLUDE [docker-pull-health-container](../includes/docker-pull-health-container.md)]

***

## <a name="how-to-use-the-container"></a>Uso del contenedor

Una vez que el contenedor esté en el [equipo host](#the-host-computer), utilice el siguiente proceso para trabajar con el contenedor.

1. [Ejecute el contenedor](#run-the-container-with-docker-run) con la configuración de facturación requerida.
1. [Consulta del punto de conexión de predicción del contenedor](#query-the-containers-prediction-endpoint).

## <a name="run-the-container-with-docker-run"></a>Ejecute el contenedor con `docker run`.

Utilice el comando [docker run](https://docs.docker.com/engine/reference/commandline/run/) para ejecutar los contenedores. El contenedor continuará ejecutándose hasta que lo detenga.

> [!IMPORTANT]
> * Los comandos de Docker de las secciones siguientes usan la barra diagonal inversa, `\`, como un carácter de continuación de línea. Puede quitarla o reemplazarla en función de los requisitos del sistema operativo del host. 
> * Para poder ejecutar el contenedor, las opciones `Eula`, `Billing` y `ApiKey` deben estar especificadas; de lo contrario, el contenedor no se iniciará.  Para obtener más información, vea [Facturación](#billing).
> * El contenedor de Análisis de sentimiento v3 ya está disponible con carácter general y devuelve [etiquetas de sentimiento](../how-tos/text-analytics-how-to-sentiment-analysis.md#sentiment-analysis-versions-and-features) en la respuesta. Los contenedores de extracción de frases clave y de detección de idioma usan la versión 2 de la API y se encuentran en versión preliminar.

# <a name="sentiment-analysis-v3"></a>[Análisis de sentimiento, versión 3](#tab/sentiment)

[!INCLUDE [docker-run-sentiment-analysis-container](../includes/docker-run-sentiment-analysis-container.md)]

# <a name="key-phrase-extraction-preview"></a>[Extracción de frases clave (versión preliminar)](#tab/keyphrase)

[!INCLUDE [docker-run-key-phrase-extraction-container](../includes/docker-run-key-phrase-extraction-container.md)]

# <a name="language-detection-preview"></a>[Detección de idioma (versión preliminar)](#tab/language)

[!INCLUDE [docker-run-language-detection-container](../includes/docker-run-language-detection-container.md)]

# <a name="text-analytics-for-health-preview"></a>[Text Analytics para el estado (versión preliminar)](#tab/healthcare)

[!INCLUDE [docker-run-health-container](../includes/docker-run-health-container.md)]

***

[!INCLUDE [Running multiple containers on the same host](../../../../includes/cognitive-services-containers-run-multiple-same-host.md)]

## <a name="query-the-containers-prediction-endpoint"></a>Consulta del punto de conexión de predicción del contenedor

El contenedor proporciona varias API de puntos de conexión de predicción de consultas basadas en REST.

Utilice el host, `http://localhost:5000`, con las API de contenedor.

<!--  ## Validate container is running -->

[!INCLUDE [Container's API documentation](../../../../includes/cognitive-services-containers-api-documentation.md)]

## <a name="stop-the-container"></a>Detención del contenedor

[!INCLUDE [How to stop the container](../../../../includes/cognitive-services-containers-stop.md)]

## <a name="troubleshooting"></a>Solución de problemas

Si ejecuta el contenedor con un [montaje](../text-analytics-resource-container-config.md#mount-settings) de salida y el registro habilitados, el contenedor genera archivos de registro que resultan útiles para solucionar problemas que se producen al iniciar o ejecutar el contenedor.

[!INCLUDE [Cognitive Services FAQ note](../../containers/includes/cognitive-services-faq-note.md)]

## <a name="billing"></a>Facturación

Los contenedores de Text Analytics envían información de facturación a Azure mediante un recurso de _Text Analytics_ en la cuenta de Azure. 

[!INCLUDE [Container's Billing Settings](../../../../includes/cognitive-services-containers-how-to-billing-info.md)]

Para obtener más información acerca de estas opciones, consulte [Configure containers](../text-analytics-resource-container-config.md) (Configuración de contenedores).

<!--blogs/samples/video course -->

[!INCLUDE [Discoverability of more container information](../../../../includes/cognitive-services-containers-discoverability.md)]

## <a name="summary"></a>Resumen

En este artículo, ha aprendido los conceptos y el flujo de trabajo para la descarga, instalación y ejecución de contenedores de Text Analytics. En resumen:

* Text Analytics proporciona tres contenedores Linux para Docker, que encapsulan varias funcionalidades:
   * *Análisis de sentimiento*
   * *Extracción de frases clave (versión preliminar)* 
   * *Detección de idioma (versión preliminar)*
   * *Text Analytics para el estado (versión preliminar)*
* Las imágenes del contenedor se descargan desde Microsoft Container Registry (MCR) o del repositorio del contenedor de versión preliminar.
* Las imágenes de contenedor se ejecutan en Docker.
* Puede usar la API REST o el SDK para llamar a operaciones en contenedores de Text Analytics mediante la especificación del URI del host del contenedor.
* Debe especificar la información de facturación al crear una instancia de un contenedor.

> [!IMPORTANT]
> Los contenedores de Cognitive Services no tienen licencia para ejecutarse sin estar conectados a Azure para realizar mediciones. Los clientes tienen que habilitar los contenedores para comunicar la información de facturación con el servicio de medición en todo momento. Los contenedores de Cognitive Services no envían datos de los clientes (por ejemplo, el texto que se está analizando) a Microsoft.

## <a name="next-steps"></a>Pasos siguientes

* Revise [Configure containers](../text-analytics-resource-container-config.md) (Configuración de contenedores) para ver las opciones de configuración.
* Consulte [Preguntas más frecuentes (P+F)](../text-analytics-resource-faq.md) para resolver problemas relacionados con la funcionalidad.
