---
title: Instalación y ejecución de un contenedor para Form Recognizer
titleSuffix: Azure Cognitive Services
description: En este artículo se explica cómo usar el contenedor de Azure Cognitive Services Form Recognizer para analizar datos de formularios y tablas.
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: conceptual
ms.date: 07/14/2020
ms.author: aahi
ms.custom: devx-track-csharp
ms.openlocfilehash: e4e85de2fad5c08f296d8089f48fa8614f7f1739
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88925188"
---
# <a name="install-and-run-form-recognizer-containers-preview"></a>Instalación y ejecución de contenedores de Form Recognizer (versión preliminar)

[!INCLUDE [Form Recognizer containers limit](includes/container-limit.md)]

Azure Form Recognizer aplica tecnología de aprendizaje automático para identificar y extraer pares clave-valor de los formularios. Asocia valores y entradas de tabla con los pares clave-valor y, después, genera datos estructurados que incluyen las relaciones en el fichero original. 

Para reducir la complejidad e integrar fácilmente un modelo de Form Recognizer personalizado en su proceso de automatización del flujo de trabajo u otra aplicación, puede llamar al modelo personalizado de Form Recognizer mediante una simple API REST. Solo se necesitan cinco documentos de formulario (o un formulario vacío y dos formularios completados), por lo que puede obtener resultados rápidamente, con precisión y adaptados a su contenido específico. No son necesarias las intervenciones manuales complicadas ni un amplio conocimiento sobre ciencia de datos. No requiere etiquetas de datos ni anotaciones de datos.

| Función | Características |
|----------|----------|
| Form Recognizer | <li>Procesa archivos PDF, PNG y JPG.<li>Entrena modelos personalizados con un mínimo de cinco formularios del mismo diseño. <li>Extrae información de la tabla y de pares clave-valor. <li>Utiliza la característica Reconocer texto de la API Computer Vision de Azure Cognitive Services con el fin de detectar y extraer texto impreso de las imágenes dentro de formularios.<li>No requiere anotaciones ni etiquetas. |

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Antes de utilizar los contenedores de Form Recognizer, debe cumplir los siguientes requisitos previos:

| Obligatorio | Propósito |
|----------|---------|
| Motor de Docker | Necesita que el motor de Docker esté instalado en un [equipo host](#the-host-computer). Docker dispone de paquetes que configuran el entorno de Docker en [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) y [Linux](https://docs.docker.com/engine/installation/#supported-platforms). Para conocer los principios básicos de Docker y de los contenedores, consulte [Introducción a Docker](https://docs.docker.com/engine/docker-overview/).<br><br> Docker debe configurarse para permitir que los contenedores se conecten con Azure y envíen datos de facturación a dicho servicio. <br><br> En Windows, Docker también debe estar configurado de forma que admita los contenedores de Linux.<br><br> |
| Conocimientos sobre Docker | Debe tener conocimientos básicos sobre los conceptos de Docker, como los registros, los repositorios, los contenedores y las imágenes de contenedor, así como conocer los comandos `docker` básicos. |
| La CLI de Azure | Instale la [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli) en el host. |
| Recurso de la API Computer Vision | Para procesar documentos e imágenes digitalizados, necesita un recurso de Computer Vision. Puede acceder a la característica Reconocer texto como un recurso de Azure (API REST o SDK) o como un [contenedor](../Computer-vision/computer-vision-how-to-install-containers.md#get-the-container-image-with-docker-pull) *cognitive-services-recognize-text*. Se aplican las tarifas de facturación habituales. <br><br>Pase tanto la clave de API como los puntos de conexión del recurso de Computer Vision (nube de Azure o contenedor de Cognitive Services). Utilice esta clave de API y el punto de conexión como **{COMPUTER_VISION_API_KEY}** y **{COMPUTER_VISION_ENDPOINT_URI}** .<br><br> Si utiliza el contenedor *cognitive-services-recognize-text*, asegúrese de cumplir con lo siguiente:<br><br>La clave de Computer Vision para el contenedor de Form Recognizer es la clave especificada en el comando `docker run` de Computer Vision para el contenedor *cognitive-services-recognize-text*.<br>El punto de conexión de facturación es el punto de conexión del contenedor (por ejemplo, `http://localhost:5000`). Si se utilizan los contenedores Computer Vision y Form Recognizer juntos en el mismo host, no se pueden iniciar con el puerto predeterminado de *5000*. |
| Recurso de Form Recognizer | Para usar estos contenedores, debe tener lo siguiente:<br><br>Un recurso de **Form Recognizer** de Azure para obtener la clave de API y el identificador URI de punto de conexión asociados. Ambos valores están disponibles en las páginas de claves y de información general de **Form Recognizer** en Azure Portal y ambos son necesarios para iniciar el contenedor.<br><br>**{FORM_RECOGNIZER_API_KEY}** : una de las dos claves de recursos disponibles en la página Claves<br><br>**{FORM_RECOGNIZER_ENDPOINT_URI}** : el punto de conexión tal como se proporciona en la página Información general |

> [!NOTE]
> El nombre del recurso de Computer Vision debe ser una sola palabra, sin un guion `-` ni cualquier otro carácter especial. Esta restricción está implementada para garantizar la compatibilidad de Form Recognizer y de Recognize Text.

## <a name="gathering-required-parameters"></a>Recopilación de los parámetros obligatorios

Hay tres parámetros principales para todos los contenedores de Cognitive Services que son necesarios. El contrato de licencia para el usuario final (CLUF) debe estar presente con un valor de `accept`. Además, se necesitan una dirección URL de punto de conexión y una clave de API.

### <a name="endpoint-uri-computer_vision_endpoint_uri-and-form_recognizer_endpoint_uri"></a>URI de punto de conexión `{COMPUTER_VISION_ENDPOINT_URI}` y `{FORM_RECOGNIZER_ENDPOINT_URI}`

El valor del URI del **punto de conexión** está disponible en la página *Información general* de Azure Portal del recurso de Cognitive Services correspondiente. Vaya a la página *Información general*, mantenga el cursor sobre el punto de conexión y aparecerá un icono `Copy to clipboard` <span class="docon docon-edit-copy x-hidden-focus"></span>. Cópielo y utilícelo cuando sea necesario.

![Recopilación el URI del punto de conexión para su uso posterior](../containers/media/overview-endpoint-uri.png)

### <a name="keys-computer_vision_api_key-and-form_recognizer_api_key"></a>Claves `{COMPUTER_VISION_API_KEY}` y `{FORM_RECOGNIZER_API_KEY}`

Esta clave se usa para iniciar el contenedor y está disponible en la página de claves de Azure Portal del recurso de Cognitive Services correspondiente. Vaya a la página *Claves* y haga clic en el icono `Copy to clipboard` <span class="docon docon-edit-copy x-hidden-focus"></span>.

![Obtención de una de las dos claves para su uso posterior](../containers/media/keys-copy-api-key.png)

> [!IMPORTANT]
> Estas claves de suscripción se usan para tener acceso a la API de Cognitive Services. No comparta las claves. Almacénelas de forma segura, por ejemplo, con Azure Key Vault. También se recomienda regenerar estas claves periódicamente. Solo se necesita una clave para realizar una llamada API. Al volver a generar la primera clave, puede usar la segunda clave para seguir teniendo acceso al servicio.

## <a name="the-host-computer"></a>El equipo host

[!INCLUDE [Host Computer requirements](../../../includes/cognitive-services-containers-host-computer.md)]

### <a name="container-requirements-and-recommendations"></a>Recomendaciones y requisitos del contenedor

Los núcleos de CPU y de memoria mínimos y recomendados para asignar a cada contenedor de Form Recognizer se describen en la siguiente tabla:

| Contenedor | Mínima | Recomendado |
|-----------|---------|-------------|
| Form Recognizer | 2 núcleos, 4 GB de memoria | 4 núcleos, 8 GB de memoria |
| Reconocer texto | 1 núcleo, 8 GB de memoria | 2 núcleos, 8 GB de memoria |

* Cada núcleo debe ser de 2,6 gigahercios (GHz) como mínimo.
* El núcleo y la memoria se corresponden con los valores de `--cpus` y `--memory` que se usan como parte del comando `docker run`.

> [!Note]
> Los valores mínimos y recomendados se basan en los límites de Docker y *no* de los recursos de la máquina host.

## <a name="get-the-container-images-with-the-docker-pull-command"></a>Obtención de las imágenes de contenedor con el comando docker pull

Las imágenes de contenedor para las ofertas **Form Recognizer** y **Reconocer texto** están disponibles en el siguiente registro de contenedor:

| Contenedor | Nombre completo de la imagen |
|-----------|------------|
| Form Recognizer | `containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer:latest` |
| Reconocer texto | `containerpreview.azurecr.io/microsoft/cognitive-services-recognize-text:latest` |

Necesitará ambos contenedores; tenga en cuenta que el contenedor **Reconocer texto** se [detalla fuera de este artículo](../Computer-vision/computer-vision-how-to-install-containers.md#get-the-container-image-with-docker-pull).

[!INCLUDE [Tip for using docker list](../../../includes/cognitive-services-containers-docker-list-tip.md)]

### <a name="docker-pull-for-the-form-recognizer-container"></a>Extracción de Docker para el contenedor de Form Recognizer

#### <a name="form-recognizer"></a>Form Recognizer

Para obtener el contenedor de Form Recognizer, utilice el siguiente comando:

```Docker
docker pull containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer:latest
```
### <a name="docker-pull-for-the-recognize-text-container"></a>Comando Docker pull para el contenedor Reconocer texto

#### <a name="recognize-text"></a>Reconocer texto

Para obtener el contenedor Reconocer texto, use el siguiente comando:

```Docker
docker pull containerpreview.azurecr.io/microsoft/cognitive-services-recognize-text:latest
```

## <a name="how-to-use-the-container"></a>Uso del contenedor

Cuando el contenedor esté en el [equipo host](#the-host-computer), use el siguiente proceso para trabajar con el contenedor.

1. [Ejecute el contenedor](#run-the-container-by-using-the-docker-run-command) con la configuración de facturación requerida. Hay más [ejemplos](form-recognizer-container-configuration.md#example-docker-run-commands) del comando `docker run` disponibles.
1. [Consulta del punto de conexión de predicción del contenedor](#query-the-containers-prediction-endpoint).

## <a name="run-the-container-by-using-the-docker-run-command"></a>Ejecución del contenedor mediante el comando docker run

Utilice el comando [docker run](https://docs.docker.com/engine/reference/commandline/run/) para ejecutar el contenedor. Consulte [Recopilación de los parámetros obligatorios](#gathering-required-parameters) para más información sobre cómo obtener los valores de `{COMPUTER_VISION_ENDPOINT_URI}`, `{COMPUTER_VISION_API_KEY}`, `{FORM_RECOGNIZER_ENDPOINT_URI}` y `{FORM_RECOGNIZER_API_KEY}`.

Hay disponibles [ejemplos](form-recognizer-container-configuration.md#example-docker-run-commands) del comando `docker run`.

### <a name="form-recognizer"></a>Form Recognizer

> [!NOTE]
> Los directorios que se usan con `--mount` en estos ejemplos son rutas de acceso de directorios de Windows. Si usa Linux o macOS, cambie el parámetro de su entorno. 

```bash
docker run --rm -it -p 5000:5000 --memory 8g --cpus 2 \
--mount type=bind,source=c:\input,target=/input  \
--mount type=bind,source=c:\output,target=/output \
containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer \
Eula=accept \
Billing={FORM_RECOGNIZER_ENDPOINT_URI} \
ApiKey={FORM_RECOGNIZER_API_KEY} \
FormRecognizer:ComputerVisionApiKey={COMPUTER_VISION_API_KEY} \
FormRecognizer:ComputerVisionEndpointUri={COMPUTER_VISION_ENDPOINT_URI}
```

Este comando:

* Ejecuta un contenedor de Form Recognizer desde la imagen de contenedor.
* Asigna dos núcleos de CPU y 8 gigabytes (GB) de memoria.
* Expone el puerto TCP 5000 y asigna un seudo-TTY para el contenedor.
* Una vez que se produce la salida, quita automáticamente el contenedor. La imagen del contenedor sigue estando disponible en el equipo host.
* Monta un volumen /input y uno /output para el contenedor.

[!INCLUDE [Running multiple containers on the same host H2](../../../includes/cognitive-services-containers-run-multiple-same-host.md)]

### <a name="run-separate-containers-as-separate-docker-run-commands"></a>Ejecución de contenedores diferentes como comandos de ejecución de Docker independientes

Para la combinación de Form Recognizer y Reconocedor de texto hospedados localmente en el mismo host, use los dos siguientes ejemplos de comandos de la CLI de Docker:

Ejecute el primer contenedor en el puerto 5000. 

```bash 
docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 \
--mount type=bind,source=c:\input,target=/input  \
--mount type=bind,source=c:\output,target=/output \
containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer \
Eula=accept \
Billing={FORM_RECOGNIZER_ENDPOINT_URI} \
ApiKey={FORM_RECOGNIZER_API_KEY}
FormRecognizer:ComputerVisionApiKey={COMPUTER_VISION_API_KEY} \
FormRecognizer:ComputerVisionEndpointUri={COMPUTER_VISION_ENDPOINT_URI}
```

Ejecute el segundo contenedor en el puerto 5001.

```bash 
docker run --rm -it -p 5001:5000 --memory 4g --cpus 1 \
containerpreview.azurecr.io/microsoft/cognitive-services-recognize-text \
Eula=accept \
Billing={COMPUTER_VISION_ENDPOINT_URI} \
ApiKey={COMPUTER_VISION_API_KEY}
```
Cada contenedor sucesivo debe estar en un puerto diferente. 

### <a name="run-separate-containers-with-docker-compose"></a>Ejecución de contenedores diferentes con Docker Compose

Para la combinación de Form Recognizer y Reconocedor de texto hospedados localmente en el mismo host, consulte el siguiente ejemplo de archivo YAML de Docker Compose. El reconocedor de texto `{COMPUTER_VISION_API_KEY}` debe ser el mismo para los contenedores `formrecognizer` y `ocr`. `{COMPUTER_VISION_ENDPOINT_URI}` solo se utiliza en el contenedor `ocr` porque el contenedor `formrecognizer` utiliza el nombre y el puerto `ocr`. 

```docker
version: '3.3'
services:   
  ocr:
    image: "containerpreview.azurecr.io/microsoft/cognitive-services-recognize-text"
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 8g
        reservations:
          cpus: '1'
          memory: 4g
    environment:
      eula: accept
      billing: "{COMPUTER_VISION_ENDPOINT_URI}"
      apikey: "{COMPUTER_VISION_API_KEY}"

  formrecognizer:
    image: "containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer"
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 8g
        reservations:
          cpus: '1'
          memory: 4g
    environment:
      eula: accept
      billing: "{FORM_RECOGNIZER_ENDPOINT_URI}"
      apikey: "{FORM_RECOGNIZER_API_KEY}"
      FormRecognizer__ComputerVisionApiKey: {COMPUTER_VISION_API_KEY}
      FormRecognizer__ComputerVisionEndpointUri: "http://ocr:5000"
      FormRecognizer__SyncProcessTaskCancelLimitInSecs: 75
    links:
      - ocr
    volumes:
      - type: bind
        source: c:\output
        target: /output
      - type: bind
        source: c:\input
        target: /input
    ports:
      - "5000:5000"
```

> [!IMPORTANT]
> Para ejecutar el contenedor, las opciones `Eula`, `Billing` y `ApiKey`, así como las opciones `FormRecognizer:ComputerVisionApiKey` y `FormRecognizer:ComputerVisionEndpointUri`, deben estar especificadas; de lo contrario, el contenedor no se iniciará. Para obtener más información, vea [Facturación](#billing).

## <a name="query-the-containers-prediction-endpoint"></a>Consulta del punto de conexión de predicción del contenedor

|Contenedor|Punto de conexión|
|--|--|
|form-recognizer|http://localhost:5000

### <a name="form-recognizer"></a>Form Recognizer

El contenedor proporciona las API de punto de conexión de consulta basadas en WebSocket, a las que puede acceder mediante la [documentación del SDK de servicios de Form Recognizer](https://docs.microsoft.com/azure/cognitive-services/form-recognizer/).

De forma predeterminada, el SDK de Form Recognizer utiliza los servicios en línea. Para usar el contenedor, deberá cambiar el método de inicialización. Consulte los ejemplos siguientes.

#### <a name="for-c"></a>Para C#

Cambie de usar esta llamada de inicialización en la nube de Azure:

```csharp
var config =
    FormRecognizerConfig.FromSubscription(
        "YourSubscriptionKey",
        "YourServiceRegion");
```
por esta llamada, que usa el punto de conexión de contenedor:

```csharp
var config =
    FormRecognizerConfig.FromEndpoint(
        "ws://localhost:5000/formrecognizer/v1.0-preview/custom",
        "YourSubscriptionKey");
```

#### <a name="for-python"></a>Para Python

Cambie de usar esta llamada de inicialización en la nube de Azure:

```python
formrecognizer_config =
    formrecognizersdk.FormRecognizerConfig(
        subscription=formrecognizer_key, region=service_region)
```

por esta llamada, que usa el punto de conexión de contenedor:

```python
formrecognizer_config = 
    formrecognizersdk.FormRecognizerConfig(
        subscription=formrecognizer_key,
        endpoint="ws://localhost:5000/formrecognizer/v1.0-preview/custom"
```

### <a name="form-recognizer"></a>Form Recognizer

El contenedor proporciona las API de punto de conexión REST, que puede encontrar en la página [API de Form Recognizer](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api/operations/AnalyzeWithCustomModel).


[!INCLUDE [Validate container is running - Container's API documentation](../../../includes/cognitive-services-containers-api-documentation.md)]


## <a name="stop-the-container"></a>Detención del contenedor

[!INCLUDE [How to stop the container](../../../includes/cognitive-services-containers-stop.md)]

## <a name="troubleshooting"></a>Solución de problemas

Si ejecuta el contenedor con un [montaje](form-recognizer-container-configuration.md#mount-settings) de salida y el registro habilitados, el contenedor genera archivos de registro que resultan útiles para solucionar problemas que se producen al iniciar o ejecutar el contenedor.

[!INCLUDE [Cognitive Services FAQ note](../containers/includes/cognitive-services-faq-note.md)]

## <a name="billing"></a>Facturación

Los contenedores de Form Recognizer envían información de facturación a Azure mediante un recurso de _Form Recognizer_ en la cuenta de Azure.

[!INCLUDE [Container's Billing Settings](../../../includes/cognitive-services-containers-how-to-billing-info.md)]

Para obtener más información acerca de estas opciones, consulte [Configure containers](form-recognizer-container-configuration.md) (Configuración de contenedores).

<!--blogs/samples/video courses -->

[!INCLUDE [Discoverability of more container information](../../../includes/cognitive-services-containers-discoverability.md)]

## <a name="summary"></a>Resumen

En este artículo, ha aprendido los conceptos y el flujo de trabajo para la descarga, instalación y ejecución de contenedores de Form Recognizer. En resumen:

* Form Recognizer proporciona un contenedor de Linux para Docker.
* Las imágenes de contenedor se descargan desde el registro de contenedor privado de Azure.
* Las imágenes de contenedor se ejecutan en Docker.
* Puede usar la API REST o el SDK REST para llamar a operaciones en contenedores de Form Recognizer mediante la especificación del URI del host del contenedor.
* Debe especificar la información de facturación al crear una instancia de contenedor.

> [!IMPORTANT]
>  Los contenedores de Cognitive Services no tienen licencia para ejecutarse sin estar conectados a Azure para realizar mediciones. Los clientes tienen que habilitar los contenedores para comunicar la información de facturación con el servicio de medición en todo momento. Los contenedores de Cognitive Services no envían datos de los clientes (por ejemplo, la imagen o el texto que se está analizando) a Microsoft.

## <a name="next-steps"></a>Pasos siguientes

* Revise [Configuración de contenedores](form-recognizer-container-configuration.md) para ver las opciones de configuración.
* Use otros [Contenedores de Cognitive Services](../cognitive-services-container-support.md).
