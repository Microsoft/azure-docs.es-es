---
title: 'Instalación de contenedores de voz: servicio de voz'
titleSuffix: Azure Cognitive Services
description: Instale y ejecute contenedores de voz. Voz a texto transcribe secuencias de audio a texto en tiempo real que sus aplicaciones, herramientas o dispositivos pueden usar o mostrar. Texto a voz convierte el texto de entrada en voz sintetizada similar a la humana.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 09/02/2020
ms.author: aahi
ms.openlocfilehash: b51319716035cc4f59d50922846b067f4eda31d3
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90900469"
---
# <a name="install-and-run-speech-service-containers"></a>Instalación y ejecución de contenedores de servicio de Voz 

Los contenedores le permiten ejecutar algunas de las API del servicio de voz en su propio entorno. Los contenedores son excelentes para requisitos específicos de control de datos y seguridad. En este artículo, aprenderá a descargar, instalar y ejecutar un contenedor de Voz.

Los contenedores de Voz permiten a los clientes compilar una arquitectura de aplicación de voz optimizada para las sólidas funcionalidades de la nube y la localidad del perímetro. Hay varios contenedores disponibles, que usan los mismos [precios](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/) que los servicios de Voz de Azure basados en la nube.


> [!IMPORTANT]
> Los siguientes contenedores de Voz están ahora disponibles con carácter general:
> * Conversión de voz en texto estándar
> * Conversión de voz a texto personalizada
> * Conversión de texto a voz estándar
> 
> Los siguientes contenedores de voz están en versión preliminar controlada.
> * Conversión de texto a voz personalizada
> * Detección de idioma de Voz 
> * Texto a voz neuronal
>
> Para usar los contenedores de voz, debe enviar una solicitud en línea y esperar a que se apruebe. Para obtener más información, consulte la sección **Solicitud de aprobación para ejecutar el contenedor** más adelante.

| Función | Características | Más reciente |
|--|--|--|
| Voz a texto | Analice opiniones y transcriba grabaciones continuas de audio por lotes o de voz en tiempo real con resultados intermedios.  | 2.3.1 |
| Conversión de voz a texto personalizada | Con un modelo personalizado del [portal de Habla personalizada](https://speech.microsoft.com/customspeech), transcribe las grabaciones continuas de voz en tiempo real o de audio por lotes a texto con resultados inmediatos. | 2.3.1 |
| Texto a voz | Convierte texto a voz de sonido natural con entrada de texto sin formato o Lenguaje de marcado de síntesis de voz (SSML). | 1.5.0 |
| Conversión de texto a voz personalizada | Con un modelo personalizado del [portal de Voz personalizada](https://aka.ms/custom-voice-portal), convierte texto a voz de sonido natural con entrada de texto sin formato o Lenguaje de marcado de síntesis de voz (SSML). | 1.5.0 |
| Detección de idioma de Voz | Detecte el idioma que se habla en los archivos de audio. | 1,0 |
| Texto a voz neuronal | Convierte texto en voz con un sonido natural utilizando una tecnología de red neuronal profunda, lo que permite obtener una voz sintetizada más natural. | 1.1.0 |

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Requisitos previos para poder usar los contenedores de Voz:

| Obligatorio | Propósito |
|--|--|
| Motor de Docker | Necesita que el motor de Docker esté instalado en un [equipo host](#the-host-computer). Docker dispone de paquetes que configuran el entorno de Docker en [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) y [Linux](https://docs.docker.com/engine/installation/#supported-platforms). Para conocer los principios básicos de Docker y de los contenedores, consulte [Introducción a Docker](https://docs.docker.com/engine/docker-overview/).<br><br> Docker debe configurarse para permitir que los contenedores se conecten con Azure y envíen datos de facturación a dicho servicio. <br><br> **En Windows**, Docker también debe estar configurado de forma que admita los contenedores de Linux.<br><br> |
| Conocimientos sobre Docker | Debe tener conocimientos básicos sobre los conceptos de Docker, como los registros, los repositorios, los contenedores y las imágenes de contenedor, así como conocer los comandos `docker` básicos. |
| Recurso de Voz | Para usar estos contenedores, debe tener:<br><br>Recurso de _Voz_ de Azure para obtener la clave de API y el URI de punto de conexión asociados. Ambos valores están disponibles en las páginas Introducción y Claves de **Voz** de Azure Portal. Los dos son necesarios para iniciar el contenedor.<br><br>**{API_KEY}** : una de las dos claves de recurso disponibles en la página **Claves**<br><br>**{ENDPOINT_URI}** : el punto de conexión tal como se proporciona en la página de **Información general**. |

[!INCLUDE [Gathering required parameters](../containers/includes/container-gathering-required-parameters.md)]

## <a name="the-host-computer"></a>El equipo host

[!INCLUDE [Host Computer requirements](../../../includes/cognitive-services-containers-host-computer.md)]

### <a name="advanced-vector-extension-support"></a>Compatibilidad con la extensión avanzada de vector

El **host** es el equipo que ejecuta el contenedor de Docker. El host *debe ser compatible* con las [extensiones avanzadas de vector](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#CPUs_with_AVX2) (AVX2). Puede comprobar la compatibilidad con AVX2 en los hosts de Linux con el comando siguiente:

```console
grep -q avx2 /proc/cpuinfo && echo AVX2 supported || echo No AVX2 support detected
```
> [!WARNING]
> El equipo host es *necesario* para admitir AVX2. El contenedor *no* funcionará correctamente sin compatibilidad con AVX2.

### <a name="container-requirements-and-recommendations"></a>Recomendaciones y requisitos del contenedor

En la tabla siguiente se describe la asignación mínima y recomendada de recursos para cada contenedor de Voz.

| Contenedor | Mínima | Recomendado |
|-----------|---------|-------------|
| Voz a texto | 2 núcleos, 2 GB de memoria | 4 núcleos, 4 GB de memoria |
| Conversión de voz a texto personalizada | 2 núcleos, 2 GB de memoria | 4 núcleos, 4 GB de memoria |
| Texto a voz | 1 núcleo, 2 GB de memoria | 2 núcleo, 3 GB de memoria |
| Conversión de texto a voz personalizada | 1 núcleo, 2 GB de memoria | 2 núcleo, 3 GB de memoria |
| Detección de idioma de Voz | 1 núcleo, 1 GB de memoria | 1 núcleo, 1 GB de memoria |
| Texto a voz neuronal | 6 núcleos, 12 GB de memoria | 8 núcleos, 16 GB de memoria |

* Cada núcleo debe ser de 2,6 gigahercios (GHz) como mínimo.

El núcleo y la memoria se corresponden con los valores de `--cpus` y `--memory` que se usan como parte del comando `docker run`.

> [!NOTE]
> Los valores mínimos y recomendados se basan en los límites de Docker y *no* en los recursos de la máquina host. Por ejemplo, partes de la asignación de memoria de contenedores de voz a texto de un modelo grande de lenguaje, y se *recomienda* que todo el archivo se ajuste en memoria, que es de 4 a 6 GB adicionales. Además, la primera ejecución de cualquier contenedor puede tardar más, dado que los modelos se van a paginar en la memoria.

## <a name="request-approval-to-the-run-the-container"></a>Solicitud de aprobación para ejecutar el contenedor

Rellene y envíe el [formulario de solicitud](https://aka.ms/cognitivegate) para solicitar acceso al contenedor. 

[!INCLUDE [Request access to public preview](../../../includes/cognitive-services-containers-request-access.md)]


## <a name="get-the-container-image-with-docker-pull"></a>Obtención de la imagen del contenedor con `docker pull`

Las imágenes de contenedor para Voz están disponibles en la instancia de Container Registry siguiente.

# <a name="speech-to-text"></a>[Voz a texto](#tab/stt)

| Contenedor | Repositorio |
|-----------|------------|
| Voz a texto | `mcr.microsoft.com/azure-cognitive-services/speechservices/speech-to-text:latest` |

# <a name="custom-speech-to-text"></a>[Conversión de voz a texto personalizada](#tab/cstt)

| Contenedor | Repositorio |
|-----------|------------|
| Conversión de voz a texto personalizada | `mcr.microsoft.com/azure-cognitive-services/speechservices/custom-speech-to-text:latest` |

# <a name="text-to-speech"></a>[Texto a voz](#tab/tts)

| Contenedor | Repositorio |
|-----------|------------|
| Texto a voz | `mcr.microsoft.com/azure-cognitive-services/speechservices/text-to-speech:latest` |

# <a name="neural-text-to-speech"></a>[Texto a voz neuronal](#tab/ntts)

| Contenedor | Repositorio |
|-----------|------------|
| Texto a voz neuronal | `mcr.microsoft.com/azure-cognitive-services/speechservices/neural-text-to-speech:latest` |

# <a name="custom-text-to-speech"></a>[Conversión de texto a voz personalizada](#tab/ctts)

| Contenedor | Repositorio |
|-----------|------------|
| Conversión de texto a voz personalizada | `mcr.microsoft.com/azure-cognitive-services/speechservices/custom-text-to-speech:latest` |

# <a name="speech-language-detection"></a>[Detección de idioma de Voz](#tab/lid)

| Contenedor | Repositorio |
|-----------|------------|
| Detección de idioma de Voz | `mcr.microsoft.com/azure-cognitive-services/speechservices/language-detection:latest` |

***

[!INCLUDE [Tip for using docker list](../../../includes/cognitive-services-containers-docker-list-tip.md)]

### <a name="docker-pull-for-the-speech-containers"></a>Docker pull para los contenedores de Voz

# <a name="speech-to-text"></a>[Voz a texto](#tab/stt)

#### <a name="docker-pull-for-the-speech-to-text-container"></a>Docker pull para el contenedor de conversión de voz a texto

Use el comando [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) para descargar una imagen de contenedor desde la versión preliminar del registro de contenedor.

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/speechservices/speech-to-text:latest
```

> [!IMPORTANT]
> La etiqueta `latest` extrae la configuración regional `en-US`. Para otras configuraciones regionales, consulte [Configuración regional de conversión de voz a texto](#speech-to-text-locales).

#### <a name="speech-to-text-locales"></a>Configuraciones regionales de voz a texto

Todas las etiquetas, a excepción de `latest` tienen el formato siguiente y distinguen mayúsculas de minúsculas:

```
<major>.<minor>.<patch>-<platform>-<locale>-<prerelease>
```

La etiqueta siguiente es un ejemplo del formato:

```
2.4.0-amd64-en-us-preview
```

Para ver todas las configuraciones regionales admitidas del contenedor de **conversión de voz a texto**, consulte las [etiquetas de imágenes de la conversión de voz a texto](../containers/container-image-tags.md#speech-to-text).

# <a name="custom-speech-to-text"></a>[Conversión de voz a texto personalizada](#tab/cstt)

#### <a name="docker-pull-for-the-custom-speech-to-text-container"></a>Docker pull para el contenedor de conversión de voz a texto personalizada

Use el comando [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) para descargar una imagen de contenedor desde la versión preliminar del registro de contenedor.

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/speechservices/custom-speech-to-text:latest
```

> [!NOTE]
> Los valores de `locale` y `voice` de los contenedores de voz personalizados los determina el modelo personalizado que ingiere el contenedor.

# <a name="text-to-speech"></a>[Texto a voz](#tab/tts)

#### <a name="docker-pull-for-the-text-to-speech-container"></a>Docker pull para el contenedor de conversión de texto a voz

Use el comando [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) para descargar una imagen de contenedor desde la versión preliminar del registro de contenedor.

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/speechservices/text-to-speech:latest
```

> [!IMPORTANT]
> La etiqueta `latest` extrae la configuración regional `en-US` y la voz `ariarus`. Para otras configuraciones, consulte [Configuración regional de conversión de texto a voz](#text-to-speech-locales).

#### <a name="text-to-speech-locales"></a>Configuración regional de conversión de texto a voz

Todas las etiquetas, a excepción de `latest` tienen el formato siguiente y distinguen mayúsculas de minúsculas:

```
<major>.<minor>.<patch>-<platform>-<locale>-<voice>-<prerelease>
```

La etiqueta siguiente es un ejemplo del formato:

```
1.6.0-amd64-en-us-ariarus-preview
```

Para ver todas las configuraciones regionales admitidas y las voces correspondientes del contenedor de **conversión de texto a voz**, consulte las [etiquetas de imágenes de conversión de texto a voz](../containers/container-image-tags.md#text-to-speech).

> [!IMPORTANT]
> Cuando se crea una solicitud HTTP POST de *texto a voz*, el mensaje del [Lenguaje de marcado de síntesis de voz (SSML)](speech-synthesis-markup.md) necesita un elemento `voice` con un atributo `name`. El valor es la configuración regional del contenedor y la voz correspondiente, que también se conoce como ["nombre corto"](language-support.md#standard-voices). Por ejemplo, la etiqueta `latest` tendría un nombre de voz de `en-US-AriaRUS`.

# <a name="neural-text-to-speech"></a>[Texto a voz neuronal](#tab/ntts)

#### <a name="docker-pull-for-the-neural-text-to-speech-container"></a>Comando docker pull para el contenedor de texto a voz neuronal

Use el comando [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) para descargar una imagen de contenedor desde la versión preliminar del registro de contenedor.

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/speechservices/neural-text-to-speech:latest
```

> [!IMPORTANT]
> La etiqueta `latest` extrae la configuración regional `en-US` y la voz `arianeural`. Para otras configuraciones, consulte [Configuraciones regionales de texto a voz neuronal](#neural-text-to-speech-locales).

#### <a name="neural-text-to-speech-locales"></a>Configuraciones regionales de texto a voz neuronal

Todas las etiquetas, a excepción de `latest` tienen el formato siguiente y distinguen mayúsculas de minúsculas:

```
<major>.<minor>.<patch>-<platform>-<locale>-<voice>-<prerelease>
```

La etiqueta siguiente es un ejemplo del formato:

```
1.1.0-amd64-en-us-arianeural-preview
```

Para ver todas las configuraciones regionales admitidas y las voces correspondientes del contenedor de **texto a voz neuronal**, consulte las [etiquetas de imágenes de texto a voz neuronal](../containers/container-image-tags.md#neural-text-to-speech).

> [!IMPORTANT]
> Cuando se crea una solicitud HTTP POST de *texto a voz neuronal*, el mensaje del [Lenguaje de marcado de síntesis de voz (SSML)](speech-synthesis-markup.md) necesita un elemento `voice` con un atributo `name`. El valor es la configuración regional del contenedor y la voz correspondiente, que también se conoce como ["nombre corto"](language-support.md#neural-voices). Por ejemplo, la etiqueta `latest` tendría un nombre de voz de `en-US-AriaNeural`.

# <a name="custom-text-to-speech"></a>[Conversión de texto a voz personalizada](#tab/ctts)

#### <a name="docker-pull-for-the-custom-text-to-speech-container"></a>Docker pull para el contenedor de conversión de texto a voz personalizada

Use el comando [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) para descargar una imagen de contenedor desde la versión preliminar del registro de contenedor.

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/speechservices/custom-text-to-speech:latest
```

> [!NOTE]
> Los valores de `locale` y `voice` de los contenedores de voz personalizados los determina el modelo personalizado que ingiere el contenedor.

# <a name="speech-language-detection"></a>[Detección de idioma de Voz](#tab/lid)

#### <a name="docker-pull-for-the-speech-language-detection-container"></a>Docker pull para el contenedor de detección de idioma de Voz

Use el comando [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) para descargar una imagen de contenedor desde la versión preliminar del registro de contenedor.

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/speechservices/language-detection:latest
```

***

## <a name="how-to-use-the-container"></a>Uso del contenedor

Una vez que el contenedor esté en el [equipo host](#the-host-computer), utilice el siguiente proceso para trabajar con el contenedor.

1. [Ejecute el contenedor](#run-the-container-with-docker-run) con la configuración de facturación requerida. Hay más [ejemplos](speech-container-configuration.md#example-docker-run-commands) del comando `docker run` disponibles.
1. [Consulta del punto de conexión de predicción del contenedor](#query-the-containers-prediction-endpoint).

## <a name="run-the-container-with-docker-run"></a>Ejecute el contenedor con `docker run`.

Utilice el comando [docker run](https://docs.docker.com/engine/reference/commandline/run/) para ejecutar el contenedor. Consulte [Recopilación de los parámetros obligatorios](#gathering-required-parameters) para más información sobre cómo obtener los valores de `{Endpoint_URI}` y `{API_Key}`. También hay disponibles otros [ejemplos](speech-container-configuration.md#example-docker-run-commands) del comando `docker run`.

# <a name="speech-to-text"></a>[Voz a texto](#tab/stt)

Para ejecutar el contenedor de *conversión de voz en texto* estándar, utilice el siguiente comando `docker run`.

```bash
docker run --rm -it -p 5000:5000 --memory 4g --cpus 4 \
mcr.microsoft.com/azure-cognitive-services/speechservices/speech-to-text \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Este comando:

* Ejecuta un contenedor *Conversión de voz a texto* desde la imagen de contenedor.
* Asigna 4 núcleos de CPU y 4 gigabytes (GB) de memoria.
* Expone el puerto TCP 5000 y asigna un seudo-TTY para el contenedor.
* Una vez que se produce la salida, quita automáticamente el contenedor. La imagen del contenedor sigue estando disponible en el equipo host.


#### <a name="analyze-sentiment-on-the-speech-to-text-output"></a>Análisis de la opinión sobre la salida de voz a texto 

A partir de la versión 2.2.0 del contenedor de voz a texto, puede llamar a la [API de análisis de opiniones v3](../text-analytics/how-tos/text-analytics-how-to-sentiment-analysis.md). Para llamar al análisis de opiniones, necesitará un punto de conexión de recurso de Text Analytics API. Por ejemplo: 
* `https://westus2.api.cognitive.microsoft.com/text/analytics/v3.0-preview.1/sentiment`
* `https://localhost:5000/text/analytics/v3.0-preview.1/sentiment`

Si tiene acceso a un punto de conexión de análisis de texto en la nube, necesitará una clave. Si ejecuta Text Analytics localmente, es posible que no tenga que proporcionarla.

La clave y el punto de conexión se pasan al contenedor de Voz en forma de argumentos, como en el ejemplo siguiente.

```bash
docker run -it --rm -p 5000:5000 \
mcr.microsoft.com/azure-cognitive-services/speechservices/speech-to-text:latest \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
CloudAI:SentimentAnalysisSettings:TextAnalyticsHost={TEXT_ANALYTICS_HOST} \
CloudAI:SentimentAnalysisSettings:SentimentAnalysisApiKey={SENTIMENT_APIKEY}
```

Este comando:

* Realiza los mismos pasos que el comando anterior.
* Almacena un punto de conexión y una clave de Text Analytics API para el envío de solicitudes de análisis de opiniones. 


# <a name="custom-speech-to-text"></a>[Conversión de voz a texto personalizada](#tab/cstt)

El contenedor *Conversión de voz a texto personalizada* se basa en un modelo de voz personalizado. El modelo personalizado se debe [entrenar](how-to-custom-speech-train-model.md) con el [portal de Habla personalizada](https://speech.microsoft.com/customspeech).

> [!IMPORTANT]
> El modelo de Voz personalizada debe entrenarse desde una de las siguientes versiones del modelo:
> * **20181201 (v3.3 unificada)**
> * **20190520 (v4.14 unificada)**
> * **20190701 (v4.17 unificada)**<br>
> ![Modelo de contenedor de entrenamiento de Voz personalizada](media/custom-speech/custom-speech-train-model-container-scoped.png)

El **identificador de modelo** de Habla personalizada es necesario para ejecutar el contenedor. Se puede encontrar en la página de **entrenamiento** del portal de Habla personalizada. En el portal de Habla personalizada, vaya a la página de **entrenamiento** y seleccione el modelo.
<br>

![Página de entrenamiento de voz personalizada](media/custom-speech/custom-speech-model-training.png)

Obtenga el **identificador de modelo** que se va a usar como argumento para el parámetro `ModelId` del comando `docker run`.
<br>

![Detalles del modelo de voz personalizado](media/custom-speech/custom-speech-model-details.png)

En la tabla siguiente se representan los diversos parámetros de `docker run` y las descripciones correspondientes:

| Parámetro | Descripción |
|---------|---------|
| `{VOLUME_MOUNT}` | El [montaje de volumen](https://docs.docker.com/storage/volumes/) del equipo host, que docker usa para conservar el modelo personalizado. Por ejemplo, *C:\CustomSpeech* donde la *unidad C* está en la máquina host. |
| `{MODEL_ID}` | El **identificador de modelo** de Habla personalizada de la página de **entrenamiento** del portal de Habla personalizada. |
| `{ENDPOINT_URI}` | El punto de conexión es necesario para la medición y la facturación. Para más información, consulte cómo [recopilar los parámetros necesarios](#gathering-required-parameters). |
| `{API_KEY}` | Se necesita la clave de API. Para más información, consulte cómo [recopilar los parámetros necesarios](#gathering-required-parameters). |

Para ejecutar el contenedor *Conversión de voz a texto personalizada*, ejecute el comando `docker run` siguiente:

```bash
docker run --rm -it -p 5000:5000 --memory 4g --cpus 4 \
-v {VOLUME_MOUNT}:/usr/local/models \
mcr.microsoft.com/azure-cognitive-services/speechservices/custom-speech-to-text \
ModelId={MODEL_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Este comando:

* Ejecuta un contenedor *Conversión de voz a texto personalizada* desde la imagen de contenedor.
* Asigna 4 núcleos de CPU y 4 gigabytes (GB) de memoria.
* Carga el modelo de *Conversión de voz a texto personalizada* desde el montaje de entrada de volumen, por ejemplo, *C:\CustomSpeech*.
* Expone el puerto TCP 5000 y asigna un seudo-TTY para el contenedor.
* Descarga el modelo dado el `ModelId` (si no se encuentra en el montaje de volumen).
* Si el modelo personalizado se descargó anteriormente, se omite el `ModelId`.
* Una vez que se produce la salida, quita automáticamente el contenedor. La imagen del contenedor sigue estando disponible en el equipo host.

# <a name="text-to-speech"></a>[Texto a voz](#tab/tts)

Para ejecutar el contenedor de *texto a voz* estándar, ejecute el siguiente comando `docker run`.

```bash
docker run --rm -it -p 5000:5000 --memory 2g --cpus 1 \
mcr.microsoft.com/azure-cognitive-services/speechservices/text-to-speech \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Este comando:

* Ejecuta un contenedor de *texto a voz* estándar desde la imagen de contenedor.
* Asigna un núcleo de 1 CPU y 2 gigabytes (GB) de memoria.
* Expone el puerto TCP 5000 y asigna un seudo-TTY para el contenedor.
* Una vez que se produce la salida, quita automáticamente el contenedor. La imagen del contenedor sigue estando disponible en el equipo host.

# <a name="neural-text-to-speech"></a>[Texto a voz neuronal](#tab/ntts)

Para ejecutar el contenedor de *texto a voz neuronal*, utilice el siguiente comando `docker run`.

```bash
docker run --rm -it -p 5000:5000 --memory 12g --cpus 6 \
mcr.microsoft.com/azure-cognitive-services/speechservices/neural-text-to-speech \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Este comando:

* Ejecuta un contenedor de *texto a voz neuronal* desde la imagen de contenedor.
* Asigna 6 núcleos de CPU y 12 gigabytes (GB) de memoria.
* Expone el puerto TCP 5000 y asigna un seudo-TTY para el contenedor.
* Una vez que se produce la salida, quita automáticamente el contenedor. La imagen del contenedor sigue estando disponible en el equipo host.

# <a name="custom-text-to-speech"></a>[Conversión de texto a voz personalizada](#tab/ctts)

El contenedor *Conversión de texto a voz personalizada* se basa en un modelo de voz personalizado. El modelo personalizado se debe [entrenar](how-to-custom-voice-create-voice.md) con el [portal de Voz personalizada](https://aka.ms/custom-voice-portal). El **identificador de modelo** de voz personalizada es necesario para ejecutar el contenedor. Se puede encontrar en la página de **entrenamiento** del portal de Voz personalizada. En el portal de Voz personalizada, vaya a la página de **entrenamiento** y seleccione el modelo.
<br>

![Página de entrenamiento de Voz personalizada](media/custom-voice/custom-voice-model-training.png)

Obtenga el **identificador de modelo** que se va a usar como argumento para el parámetro `ModelId` del comando docker run.
<br>

![Detalles del modelo de voz personalizado](media/custom-voice/custom-voice-model-details.png)

En la tabla siguiente se representan los diversos parámetros de `docker run` y las descripciones correspondientes:

| Parámetro | Descripción |
|---------|---------|
| `{VOLUME_MOUNT}` | El [montaje de volumen](https://docs.docker.com/storage/volumes/) del equipo host, que docker usa para conservar el modelo personalizado. Por ejemplo, *C:\CustomSpeech* donde la *unidad C* está en la máquina host. |
| `{MODEL_ID}` | El **identificador de modelo** de Habla personalizada de la página de **entrenamiento** del portal de Habla personalizada. |
| `{ENDPOINT_URI}` | El punto de conexión es necesario para la medición y la facturación. Para más información, consulte cómo [recopilar los parámetros necesarios](#gathering-required-parameters). |
| `{API_KEY}` | Se necesita la clave de API. Para más información, consulte cómo [recopilar los parámetros necesarios](#gathering-required-parameters). |

Para ejecutar el contenedor *Conversión de texto a voz personalizada*, ejecute el comando `docker run` siguiente:

```bash
docker run --rm -it -p 5000:5000 --memory 2g --cpus 1 \
-v {VOLUME_MOUNT}:/usr/local/models \
mcr.microsoft.com/azure-cognitive-services/speechservices/custom-text-to-speech \
ModelId={MODEL_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Este comando:

* Ejecuta un contenedor *Conversión de texto a voz personalizada* desde la imagen de contenedor.
* Asigna un núcleo de 1 CPU y 2 gigabytes (GB) de memoria.
* Carga el modelo de *Conversión de texto a voz personalizada* desde el montaje de entrada de volumen, por ejemplo, *C:\CustomVoice*.
* Expone el puerto TCP 5000 y asigna un seudo-TTY para el contenedor.
* Descarga el modelo dado el `ModelId` (si no se encuentra en el montaje de volumen).
* Si el modelo personalizado se descargó anteriormente, se omite el `ModelId`.
* Una vez que se produce la salida, quita automáticamente el contenedor. La imagen del contenedor sigue estando disponible en el equipo host.

# <a name="language-detection"></a>[Detección de idioma](#tab/lid)

Para ejecutar el contenedor *Detección de idioma de Voz*, ejecute el siguiente comando `docker run`.

```bash
docker run --rm -it -p 5003:5003 --memory 1g --cpus 1 \
mcr.microsoft.com/azure-cognitive-services/speechservices/language-detection \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Este comando: 

* Ejecuta un contenedor de detección de idioma de Voz desde la imagen de contenedor.
* Asigna 1 núcleo de CPU y 1 gigabyte (GB) de memoria.
* Expone el puerto TCP 5003 y asigna un seudo-TTY para el contenedor.
* Una vez que se produce la salida, quita automáticamente el contenedor. La imagen del contenedor sigue estando disponible en el equipo host.

Si desea ejecutar este contenedor con el contenedor de conversión de voz en texto, puede usar esta [imagen de Docker](https://hub.docker.com/r/antsu/on-prem-client). Una vez iniciados los dos contenedores, use este comando de Docker Run para ejecutar `speech-to-text-with-languagedetection-client`.

```Docker
docker run --rm -v ${HOME}:/root -ti antsu/on-prem-client:latest ./speech-to-text-with-languagedetection-client ./audio/LanguageDetection_en-us.wav --host localhost --lport 5003 --sport 5000
```

> [!NOTE]
> Aumentar el número de llamadas simultáneas puede afectar a la confiabilidad y la latencia. Para la detección de idioma, se recomienda un máximo de 4 llamadas simultáneas mediante 1 CPU con 1 GB de memoria. En el caso de hosts con 2 CPU y 2 GB de memoria, se recomienda un máximo de 6 llamadas simultáneas.

***

> [!IMPORTANT]
> Para poder ejecutar el contenedor, las opciones `Eula`, `Billing` y `ApiKey` deben estar especificadas; de lo contrario, el contenedor no se iniciará.  Para obtener más información, vea [Facturación](#billing).

## <a name="query-the-containers-prediction-endpoint"></a>Consulta del punto de conexión de predicción del contenedor

> [!NOTE]
> Use un número de puerto único si ejecuta varios contenedores.

| Contenedores | Dirección URL del host del SDK | Protocolo |
|--|--|--|
| Conversión de voz en texto estándar y personalizada | `ws://localhost:5000` | WS |
| Conversión de texto a voz (incluida estándar, personalizada y neuronal), detección de idioma | `http://localhost:5000` | HTTP |

Para más información sobre cómo usar los protocolos WSS y HTTPS, consulte la [seguridad del contenedor](../cognitive-services-container-support.md#azure-cognitive-services-container-security).

### <a name="speech-to-text-standard-and-custom"></a>Conversión de voz en texto (estándar y personalizada)

[!INCLUDE [Query Speech-to-text container endpoint](includes/speech-to-text-container-query-endpoint.md)]

#### <a name="analyze-sentiment"></a>Análisis de opinión

Si proporcionó sus credenciales de Text Analytics API [al contenedor](#analyze-sentiment-on-the-speech-to-text-output), puede usar el SDK de Voz para enviar solicitudes de reconocimiento de voz con análisis de opiniones. Puede configurar las respuestas de la API para usar un formato *simple* o *detallado*.
> [!NOTE]
> Se ha detectado un problema con el análisis de opiniones en la versión 1.13 del SDK de Python para el servicio Voz. Utilice la versión 1.12.x u otra anterior si desea usar el análisis de opiniones en el SDK de Python para el servicio Voz.

# <a name="simple-format"></a>[Formato simple](#tab/simple-format)

Para configurar el cliente de Voz de forma que use un formato simple, agregue `"Sentiment"` como valor para `Simple.Extensions`. Si quiere elegir una versión específica del modelo de Text Analytics, reemplace `'latest'` en la configuración de la propiedad `speechcontext-phraseDetection.sentimentAnalysis.modelversion`.

```python
speech_config.set_service_property(
    name='speechcontext-PhraseOutput.Simple.Extensions',
    value='["Sentiment"]',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
speech_config.set_service_property(
    name='speechcontext-phraseDetection.sentimentAnalysis.modelversion',
    value='latest',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
```

`Simple.Extensions` devolverá el resultado de la opinión en la capa raíz de la respuesta.

```json
{
   "DisplayText":"What's the weather like?",
   "Duration":13000000,
   "Id":"6098574b79434bd4849fee7e0a50f22e",
   "Offset":4700000,
   "RecognitionStatus":"Success",
   "Sentiment":{
      "Negative":0.03,
      "Neutral":0.79,
      "Positive":0.18
   }
}
```

# <a name="detailed-format"></a>[Formato detallado](#tab/detailed-format)

Para configurar el cliente de Voz de forma que use un formato detallado, agregue `"Sentiment"` como valor para `Detailed.Extensions`, `Detailed.Options` o ambos. Si quiere elegir una versión específica del modelo de Text Analytics, reemplace `'latest'` en la configuración de la propiedad `speechcontext-phraseDetection.sentimentAnalysis.modelversion`.

```python
speech_config.set_service_property(
    name='speechcontext-PhraseOutput.Detailed.Options',
    value='["Sentiment"]',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
speech_config.set_service_property(
    name='speechcontext-PhraseOutput.Detailed.Extensions',
    value='["Sentiment"]',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
speech_config.set_service_property(
    name='speechcontext-phraseDetection.sentimentAnalysis.modelversion',
    value='latest',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
```

`Detailed.Extensions` proporciona el resultado de la opinión en la capa raíz de la respuesta. `Detailed.Options` proporciona el resultado en la capa `NBest` de la respuesta. Se pueden usar por separado o juntos.

```json
{
   "DisplayText":"What's the weather like?",
   "Duration":13000000,
   "Id":"6a2aac009b9743d8a47794f3e81f7963",
   "NBest":[
      {
         "Confidence":0.973695,
         "Display":"What's the weather like?",
         "ITN":"what's the weather like",
         "Lexical":"what's the weather like",
         "MaskedITN":"What's the weather like",
         "Sentiment":{
            "Negative":0.03,
            "Neutral":0.79,
            "Positive":0.18
         }
      },
      {
         "Confidence":0.9164971,
         "Display":"What is the weather like?",
         "ITN":"what is the weather like",
         "Lexical":"what is the weather like",
         "MaskedITN":"What is the weather like",
         "Sentiment":{
            "Negative":0.02,
            "Neutral":0.88,
            "Positive":0.1
         }
      }
   ],
   "Offset":4700000,
   "RecognitionStatus":"Success",
   "Sentiment":{
      "Negative":0.03,
      "Neutral":0.79,
      "Positive":0.18
   }
}
```

---

Si quiere deshabilitar por completo el análisis de opiniones, agregue un valor `false` a `sentimentanalysis.enabled`.

```python
speech_config.set_service_property(
    name='speechcontext-phraseDetection.sentimentanalysis.enabled',
    value='false',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
```

### <a name="text-to-speech-standard-neural-and-custom"></a>Texto a voz (estándar, neuronal y personalizada)

[!INCLUDE [Query Text-to-speech container endpoint](includes/text-to-speech-container-query-endpoint.md)]

### <a name="run-multiple-containers-on-the-same-host"></a>Ejecución de varios contenedores en el mismo host

Si tiene pensado ejecutar varios contenedores con puertos expuestos, asegúrese de que ejecuta cada contenedor con un puerto expuesto diferente. Por ejemplo, ejecute el primer contenedor en el puerto 5000 y el segundo en el puerto 5001.

Puede tener este contenedor y un contenedor de Azure Cognitive Services diferente en ejecución simultáneamente en el HOST. También puede tener varios contenedores del mismo contenedor de Cognitive Services en ejecución.

[!INCLUDE [Validate container is running - Container's API documentation](../../../includes/cognitive-services-containers-api-documentation.md)]

## <a name="stop-the-container"></a>Detención del contenedor

[!INCLUDE [How to stop the container](../../../includes/cognitive-services-containers-stop.md)]

## <a name="troubleshooting"></a>Solución de problemas

Puede encontrar algunos problemas al iniciar o ejecutar el contenedor. Use un [montaje](speech-container-configuration.md#mount-settings) de salida y habilite el registro. Si lo hace, permitirá que el contenedor genere archivos de registro que son útiles para solucionar problemas.

[!INCLUDE [Cognitive Services FAQ note](../containers/includes/cognitive-services-faq-note.md)]

## <a name="billing"></a>Facturación

Los contenedores de Voz envían información de facturación a Azure mediante un recurso de *Voz* en la cuenta de Azure.

[!INCLUDE [Container's Billing Settings](../../../includes/cognitive-services-containers-how-to-billing-info.md)]

Para obtener más información acerca de estas opciones, consulte [Configure containers](speech-container-configuration.md) (Configuración de contenedores).

<!--blogs/samples/video courses -->

[!INCLUDE [Discoverability of more container information](../../../includes/cognitive-services-containers-discoverability.md)]

## <a name="summary"></a>Resumen

En este artículo, ha aprendido los conceptos y el flujo de trabajo para la descarga, instalación y ejecución de contenedores de Voz. En resumen:

* Voz proporciona cuatro contenedores Linux para Docker, que encapsulan varias funcionalidades:
  * *Voz a texto*
  * *Conversión de voz a texto personalizada*
  * *Texto a voz*
  * *Conversión de texto a voz personalizada*
  * *Texto a voz neuronal*
  * *Detección de idioma de Voz*
* Las imágenes de contenedor se descargan desde el registro de contenedor de Azure.
* Las imágenes de contenedor se ejecutan en Docker.
* Si usa la API REST (solo conversión de texto a voz) o el SDK (conversión de voz a texto o de texto a voz), especifique el URI del host del contenedor. 
* Debe proporcionar la información de facturación al crear una instancia de un contenedor.

> [!IMPORTANT]
>  Los contenedores de Cognitive Services no tienen licencia para ejecutarse sin estar conectados a Azure para realizar mediciones. Los clientes tienen que habilitar los contenedores para comunicar la información de facturación con el servicio de medición en todo momento. Los contenedores de Cognitive Services no envían datos de los clientes (por ejemplo, la imagen o el texto que se está analizando) a Microsoft.

## <a name="next-steps"></a>Pasos siguientes

* Revise [Configuración de contenedores](speech-container-configuration.md) para ver las opciones de configuración.
* Aprenda a [usar contenedores del servicio de voz con Kubernetes y Helm](speech-container-howto-on-premises.md).
* Uso de otros [contenedores de Cognitive Services](../cognitive-services-container-support.md)
