---
title: Configuración de contenedores de Voz
titleSuffix: Azure Cognitive Services
description: El servicio de voz proporciona a cada contenedor un marco de configuración común, por lo que puede configurar y administrar fácilmente los valores de almacenamiento, registro, telemetría y seguridad de los contenedores.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: aahi
ms.openlocfilehash: e65bb7c7d8fc04baec6b50a53519e689e748fbe1
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90900281"
---
# <a name="configure-speech-service-containers"></a>Configuración de contenedores del servicio de voz

Los contenedores de Voz permiten a los clientes compilar una arquitectura de aplicación de voz optimizada para aprovechar las sólidas capacidades de la nube y la localidad del perímetro. Los cinco contenedores que se admiten ahora son: **conversión de voz en texto**, **conversión de voz en texto personalizada**, **texto a voz**, **texto a voz neuronal** y **conversión de texto a voz personalizada**.

El entorno en tiempo de ejecución del contenedor de **Speech** se configura mediante los argumentos del comando `docker run`. Este contenedor tiene varias opciones de configuración necesarias, así como otras opcionales. Hay disponibles varios [ejemplos](#example-docker-run-commands) del comando. La configuración específica del contenedor es la configuración de facturación.

## <a name="configuration-settings"></a>Parámetros de configuración

[!INCLUDE [Container shared configuration settings table](../../../includes/cognitive-services-containers-configuration-shared-settings-table.md)]

> [!IMPORTANT]
> Las opciones [`ApiKey`](#apikey-configuration-setting), [`Billing`](#billing-configuration-setting) y [`Eula`](#eula-setting) se usan en conjunto y debe proporcionar valores válidos para las tres; en caso contrario, no se inicia el contenedor. Para obtener más información sobre el uso de estas opciones de configuración para crear instancias de un contenedor, consulte [Facturación](speech-container-howto.md#billing).

## <a name="apikey-configuration-setting"></a>Opción de configuración ApiKey

La opción de configuración `ApiKey` especifica la clave de recurso de Azure usada para realizar un seguimiento de la información de facturación del contenedor. Debe especificar un valor para ApiKey que debe ser una clave válida para el recurso de _Speech_ especificado para la opción de configuración [`Billing`](#billing-configuration-setting).

Este valor se puede encontrar en el siguiente lugar:

- Azure Portal: Administración de recursos de **Speech** en **Claves**

## <a name="applicationinsights-setting"></a>Opción de configuración ApplicationInsights

[!INCLUDE [Container shared configuration ApplicationInsights settings](../../../includes/cognitive-services-containers-configuration-shared-settings-application-insights.md)]

## <a name="billing-configuration-setting"></a>Opción de configuración Billing

La configuración `Billing` especifica el URI de punto de conexión del recurso de _Speech_ en Azure que se usa para medir la información de facturación del contenedor. Debe especificar un valor para esta opción de configuración y este debe ser un URI de punto de conexión válido para un recurso de _Speech_ en Azure. El contenedor informa sobre el uso cada 10 a 15 minutos.

Este valor se puede encontrar en el siguiente lugar:

- Azure Portal: Información general de **Speech**, con la etiqueta `Endpoint`

| Obligatorio | Nombre | Tipo de datos | Descripción |
| -------- | ---- | --------- | ----------- |
| Sí | `Billing` | String | Identificador URI del punto de conexión de facturación. Para más información sobre cómo obtener el URI de facturación, consulte la [recopilación de los parámetros necesarios](speech-container-howto.md#gathering-required-parameters). Para más información y para obtener una lista completa de los puntos de conexión regionales, consulte [Nombres de subdominios personalizados para Cognitive Services.](../cognitive-services-custom-subdomains.md) |

## <a name="eula-setting"></a>Opción de configuración Eula

[!INCLUDE [Container shared configuration eula settings](../../../includes/cognitive-services-containers-configuration-shared-settings-eula.md)]

## <a name="fluentd-settings"></a>Opción de configuración Fluentd

[!INCLUDE [Container shared configuration fluentd settings](../../../includes/cognitive-services-containers-configuration-shared-settings-fluentd.md)]

## <a name="http-proxy-credentials-settings"></a>Configuración de las credenciales del proxy HTTP

[!INCLUDE [Container shared HTTP proxy settings](../../../includes/cognitive-services-containers-configuration-shared-settings-http-proxy.md)]

## <a name="logging-settings"></a>Opción de configuración Logging

[!INCLUDE [Container shared configuration logging settings](../../../includes/cognitive-services-containers-configuration-shared-settings-logging.md)]

## <a name="mount-settings"></a>Configuración de montaje

Utilice montajes de enlace para leer y escribir datos hacia y desde el contenedor. Puede especificar un montaje de entrada o un montaje de salida mediante la opción `--mount` del comando [docker run](https://docs.docker.com/engine/reference/commandline/run/).

Los contenedores de Voz estándar no usan montajes de entrada o salida para almacenar datos de entrenamiento o servicio. Sin embargo, los contenedores de voz personalizados dependen de los montajes de volumen.

La sintaxis exacta de la ubicación de montaje del host varía según el sistema operativo del host. Además, la ubicación de montaje del [equipo host](speech-container-howto.md#the-host-computer) puede no estar accesible debido a un conflicto entre los permisos que utiliza la cuenta de servicio de Docker y los permisos de la ubicación de montaje del host.

| Opcional | Nombre | Tipo de datos | Descripción |
| -------- | ---- | --------- | ----------- |
| No permitida | `Input` | String | Los contenedores de Voz estándar no usan esto. Los contenedores de voz personalizados usan los [montajes de volumen](#volume-mount-settings).                                                                                    |
| Opcional | `Output` | String | Destino del montaje de salida. El valor predeterminado es `/output`. Esta es la ubicación de los registros. Esto incluye los registros de contenedor. <br><br>Ejemplo:<br>`--mount type=bind,src=c:\output,target=/output` |

## <a name="volume-mount-settings"></a>Configuración de montaje de volumen

Los contenedores de voz personalizados usan [montajes de volumen](https://docs.docker.com/storage/volumes/) para conservar los modelos personalizados. Para especificar un montaje de volumen, agregue la opción `-v` (o `--volume`) al comando [docker run](https://docs.docker.com/engine/reference/commandline/run/).

Los modelos personalizados se descargan la primera vez que se ingesta un nuevo modelo como parte del comando docker run del contenedor de Voz personalizado. Las ejecuciones secuenciales del mismo `ModelId` para un contenedor de voz personalizado usarán el modelo descargado anteriormente. Si no se proporciona el montaje de volumen, no se pueden conservar los modelos personalizados.

La configuración de montaje de volumen consta de tres campos `:` de colores separados:

1. El primer campo es el nombre del volumen en la máquina host, por ejemplo, _C:\input_.
2. El segundo campo es el directorio del contenedor, por ejemplo _/usr/local/models_.
3. El tercer campo (opcional) es una lista de opciones separada por comas. Para más información, consulte el [uso de los volúmenes](https://docs.docker.com/storage/volumes/).

### <a name="volume-mount-example"></a>Ejemplo de montaje de volumen

```bash
-v C:\input:/usr/local/models
```

Este comando monta el directorio _C:\input_ de la máquina host en el directorio _/usr/local/models_ de los contenedores.

> [!IMPORTANT]
> La configuración del montaje de volumen solo se aplica a los contenedores **Conversión de voz a texto personalizada** y **Conversión de texto a voz personalizada**. Los contenedores de **conversión de voz en texto**, **texto a voz neuronal** y **texto a voz** no utilizan montajes de volumen.

## <a name="example-docker-run-commands"></a>Comandos de ejemplo de docker run

Los ejemplos siguientes usan las opciones de configuración para ilustrar cómo escribir y usar comandos `docker run`. Una vez que se está ejecutando, el contenedor continúa ejecutándose hasta que lo [detenga](speech-container-howto.md#stop-the-container).

- **Carácter de continuación de línea**: Los comandos de Docker de las secciones siguientes usan la barra diagonal inversa (`\`) como un carácter de continuación de línea. Puede quitarla o reemplazarla en función de los requisitos del sistema operativo del host.
- **Orden de los argumentos**: No cambie el orden de los argumentos a menos que esté familiarizado con los contenedores de Docker.

Reemplace {_argument_name_} por sus propios valores:

| Marcador de posición | Value | Formato o ejemplo |
| ----------- | ----- | ----------------- |
| **{CLAVE_API}** | La clave del punto de conexión del recurso `Speech` en la página Claves de `Speech` de Azure.   | `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`                                                                                  |
| **{URI_PUNTODECONEXIÓN}** | El valor del punto de conexión de facturación está disponible en la página Información general de Azure `Speech`. | Consulte el apartado de [recopilación de los parámetros necesarios](speech-container-howto.md#gathering-required-parameters) para ejemplos explícitos. |

[!INCLUDE [subdomains-note](../../../includes/cognitive-services-custom-subdomains-note.md)]

> [!IMPORTANT]
> Para poder ejecutar el contenedor, las opciones `Eula`, `Billing` y `ApiKey` deben estar especificadas; de lo contrario, el contenedor no se iniciará. Para obtener más información, vea [Facturación](#billing-configuration-setting).
> El valor de ApiKey es la **clave** de la página de claves de recursos de Azure Speech.

## <a name="speech-container-docker-examples"></a>Ejemplos de Docker del contenedor de Speech

En los siguientes ejemplos de Docker encontrará el contenedor de Speech.

## <a name="speech-to-text"></a>[Voz a texto](#tab/stt)

### <a name="basic-example-for-speech-to-text"></a>Ejemplo básico de Conversión de voz a texto

```Docker
docker run --rm -it -p 5000:5000 --memory 4g --cpus 4 \
mcr.microsoft.com/azure-cognitive-services/speechservices/speech-to-text \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

### <a name="logging-example-for-speech-to-text"></a>Ejemplo de registro de Conversión de voz a texto

```Docker
docker run --rm -it -p 5000:5000 --memory 4g --cpus 4 \
mcr.microsoft.com/azure-cognitive-services/speechservices/custom-speech-to-text \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Console:LogLevel:Default=Information
```

## <a name="custom-speech-to-text"></a>[Conversión de voz a texto personalizada](#tab/cstt)

### <a name="basic-example-for-custom-speech-to-text"></a>Ejemplo básico de Conversión de voz a texto personalizada

```Docker
docker run --rm -it -p 5000:5000 --memory 4g --cpus 4 \
-v {VOLUME_MOUNT}:/usr/local/models \
mcr.microsoft.com/azure-cognitive-services/speechservices/custom-speech-to-text \
ModelId={MODEL_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

### <a name="logging-example-for-custom-speech-to-text"></a>Ejemplo de registro de Conversión de voz a texto personalizada

```Docker
docker run --rm -it -p 5000:5000 --memory 4g --cpus 4 \
-v {VOLUME_MOUNT}:/usr/local/models \
mcr.microsoft.com/azure-cognitive-services/speechservices/custom-speech-to-text \
ModelId={MODEL_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Console:LogLevel:Default=Information
```

## <a name="text-to-speech"></a>[Texto a voz](#tab/tss)

### <a name="basic-example-for-text-to-speech"></a>Ejemplo básico de Conversión de texto a voz

```Docker
docker run --rm -it -p 5000:5000 --memory 2g --cpus 1 \
mcr.microsoft.com/azure-cognitive-services/speechservices/text-to-speech \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

### <a name="logging-example-for-text-to-speech"></a>Ejemplo de registro de Conversión de texto a voz

```Docker
docker run --rm -it -p 5000:5000 --memory 2g --cpus 1 \
mcr.microsoft.com/azure-cognitive-services/speechservices/text-to-speech \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Console:LogLevel:Default=Information
```

## <a name="custom-text-to-speech"></a>[Conversión de texto a voz personalizada](#tab/ctts)

### <a name="basic-example-for-custom-text-to-speech"></a>Ejemplo básico de Conversión e texto a voz personalizada

```Docker
docker run --rm -it -p 5000:5000 --memory 2g --cpus 1 \
-v {VOLUME_MOUNT}:/usr/local/models \
mcr.microsoft.com/azure-cognitive-services/speechservices/custom-text-to-speech \
ModelId={MODEL_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

### <a name="logging-example-for-custom-text-to-speech"></a>Ejemplo de registro de Conversión e texto a voz personalizada

```Docker
docker run --rm -it -p 5000:5000 --memory 2g --cpus 1 \
-v {VOLUME_MOUNT}:/usr/local/models \
mcr.microsoft.com/azure-cognitive-services/speechservices/custom-text-to-speech \
ModelId={MODEL_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Console:LogLevel:Default=Information
```

## <a name="neural-text-to-speech"></a>[Texto a voz neuronal](#tab/ntts)

### <a name="basic-example-for-neural-text-to-speech"></a>Ejemplo básico de Texto a voz neuronal

```Docker
docker run --rm -it -p 5000:5000 --memory 12g --cpus 6 \
mcr.microsoft.com/azure-cognitive-services/speechservices/neural-text-to-speech \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

### <a name="logging-example-for-neural-text-to-speech"></a>Ejemplo de registro de Texto a voz neuronal

```Docker
docker run --rm -it -p 5000:5000 --memory 12g --cpus 6 \
mcr.microsoft.com/azure-cognitive-services/speechservices/neural-text-to-speech \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Console:LogLevel:Default=Information
```

## <a name="speech-language-detection"></a>[Detección de idioma de Voz](#tab/lid)

### <a name="basic-example-for-speech-language-detection"></a>Ejemplo básico de detección de idioma de Voz

```Docker
docker run --rm -it -p 5000:5000 --memory 12g --cpus 6 \
mcr.microsoft.com/azure-cognitive-services/speechservices/language-detection \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

### <a name="logging-example-for-speech-language-detection"></a>Ejemplo de registro para detección de idioma de Voz

```Docker
docker run --rm -it -p 5000:5000 --memory 12g --cpus 6 \
mcr.microsoft.com/azure-cognitive-services/speechservices/language-detection \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Console:LogLevel:Default=Information
```

---

## <a name="next-steps"></a>Pasos siguientes

- Consulte [Instalación y ejecución de contenedores](speech-container-howto.md)
