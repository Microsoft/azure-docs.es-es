---
title: Configurar los contenedores de voz
titleSuffix: Azure Cognitive Services
description: El contenedor de voz
services: cognitive-services
author: diberry
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/15/2019
ms.author: diberry
ms.openlocfilehash: e2ed29bb61f553f68b9f9802884169361d5d983f
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65797925"
---
# <a name="configure-speech-service-containers"></a>Configurar los contenedores de servicio de voz

Contenedores de voz permiten que los clientes creen una arquitectura de aplicaciones de voz que se optimiza para aprovechar las capacidades de nube sólidas y localidad de edge. Los contenedores de dos voz ahora admitimos son **texto a voz** y **texto a voz**. 

El **voz** entorno en tiempo de ejecución del contenedor se configura mediante el `docker run` argumentos de comandos. Este contenedor tiene varias opciones de configuración necesarias, así como otras opcionales. Hay disponibles varios [ejemplos](#example-docker-run-commands) del comando. La configuración específica del contenedor es la configuración de facturación. 

# <a name="configuration-settings"></a>Opciones de configuración

[!INCLUDE [Container shared configuration settings table](../../../includes/cognitive-services-containers-configuration-shared-settings-table.md)]

> [!IMPORTANT]
> Las opciones [`ApiKey`](#apikey-configuration-setting), [`Billing`](#billing-configuration-setting) y [`Eula`](#eula-setting) se usan en conjunto y debe proporcionar valores válidos para las tres; en caso contrario, no se inicia el contenedor. Para obtener más información sobre el uso de estas opciones de configuración para crear instancias de un contenedor, consulte [Facturación](speech-container-howto.md#billing).

## <a name="apikey-configuration-setting"></a>Opción de configuración ApiKey

La opción de configuración `ApiKey` especifica la clave de recurso de Azure usada para realizar un seguimiento de la información de facturación del contenedor. Debe especificar un valor para la ApiKey y el valor debe ser una clave válida para el _voz_ recurso especificado para el [ `Billing` ](#billing-configuration-setting) opción de configuración.

Este valor se puede encontrar en el siguiente lugar:

* Azure Portal: **De voz** administración de recursos, en **claves**

## <a name="applicationinsights-setting"></a>Opción de configuración ApplicationInsights

[!INCLUDE [Container shared configuration ApplicationInsights settings](../../../includes/cognitive-services-containers-configuration-shared-settings-application-insights.md)]

## <a name="billing-configuration-setting"></a>Opción de configuración Billing

El `Billing` configuración especifica el URI del extremo de la _voz_ recursos en Azure se usan para medir la información de facturación para el contenedor. Debe especificar un valor para esta opción de configuración y el valor debe ser un URI de extremo válido para un _voz_ recursos en Azure. El contenedor informa sobre el uso cada 10 a 15 minutos.

Este valor se puede encontrar en el siguiente lugar:

* Azure Portal: **De voz** información general, con la etiqueta `Endpoint`

|Obligatorio| NOMBRE | Tipo de datos | DESCRIPCIÓN |
|--|------|-----------|-------------|
|Sí| `Billing` | String | Identificador URI del punto de conexión de facturación<br><br>Ejemplo:<br>`Billing=https://westus.api.cognitive.microsoft.com/sts/v1.0` |

## <a name="eula-setting"></a>Opción de configuración Eula

[!INCLUDE [Container shared configuration eula settings](../../../includes/cognitive-services-containers-configuration-shared-settings-eula.md)]

## <a name="fluentd-settings"></a>Opción de configuración Fluentd

[!INCLUDE [Container shared configuration fluentd settings](../../../includes/cognitive-services-containers-configuration-shared-settings-fluentd.md)]

## <a name="http-proxy-credentials-settings"></a>Configuración de las credenciales del proxy HTTP

[!INCLUDE [Container shared configuration fluentd settings](../../../includes/cognitive-services-containers-configuration-shared-settings-http-proxy.md)]

## <a name="logging-settings"></a>Opción de configuración Logging
 
[!INCLUDE [Container shared configuration logging settings](../../../includes/cognitive-services-containers-configuration-shared-settings-logging.md)]

## <a name="mount-settings"></a>Configuración de montaje

Utilice montajes de enlace para leer y escribir datos hacia y desde el contenedor. Puede especificar un montaje de entrada o un montaje de salida mediante la opción `--mount` del comando [docker run](https://docs.docker.com/engine/reference/commandline/run/).

Los contenedores de voz no utiliza la entrada o salida monta para almacenar el entrenamiento o servicio de datos. 

La sintaxis exacta de la ubicación de montaje del host varía según el sistema operativo del host. Además, la ubicación de montaje del [equipo host](speech-container-howto.md#the-host-computer) puede no estar accesible debido a un conflicto entre los permisos que utiliza la cuenta de servicio de Docker y los permisos de la ubicación de montaje del host. 

|Opcional| NOMBRE | Tipo de datos | DESCRIPCIÓN |
|-------|------|-----------|-------------|
|No permitida| `Input` | string | Contenedores de voz no usan.|
|Opcional| `Output` | String | Destino del montaje de salida. El valor predeterminado es `/output`. Esta es la ubicación de los registros. Esto incluye los registros de contenedor. <br><br>Ejemplo:<br>`--mount type=bind,src=c:\output,target=/output`|

## <a name="example-docker-run-commands"></a>Comandos de ejemplo de docker run 

Los ejemplos siguientes usan las opciones de configuración para ilustrar cómo escribir y usar comandos `docker run`.  Una vez que se está ejecutando, el contenedor continúa ejecutándose hasta que lo [detenga](speech-container-howto.md#stop-the-container).

* **Carácter de continuación de línea**: Los comandos de Docker de las secciones siguientes usan la barra diagonal inversa (`\`) como un carácter de continuación de línea. Puede quitarla o reemplazarla en función de los requisitos del sistema operativo del host. 
* **Orden de los argumentos**: No cambie el orden de los argumentos a menos que esté muy familiarizado con los contenedores de Docker.

Reemplace {_argument_name_} por sus propios valores:

| Marcador de posición | Valor | Formato o ejemplo |
|-------------|-------|---|
|{BILLING_KEY} | La clave de punto de conexión del recurso de voz. |xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx|
|{BILLING_ENDPOINT_URI} | El valor del punto de conexión de facturación, incluida la región.|`https://westus.api.cognitive.microsoft.com/sts/v1.0`|

> [!IMPORTANT]
> Para poder ejecutar el contenedor, las opciones `Eula`, `Billing` y `ApiKey` deben estar especificadas; de lo contrario, el contenedor no se iniciará.  Para obtener más información, vea [Facturación](#billing-configuration-setting).
> El valor de ApiKey es el **clave** en la página de claves de recurso de voz de Azure. 

## <a name="speech-container-docker-examples"></a>Ejemplos de voz contenedor Docker

Los siguientes ejemplos de Docker son para el contenedor de voz. 

### <a name="basic-example-for-speech-to-text"></a>Ejemplo básico de voz a texto

```Docker
docker run --rm -it -p 5000:5000 --memory 4g --cpus 2 \
containerpreview.azurecr.io/microsoft/cognitive-services-speech-to-text \
Eula=accept \
Billing={BILLING_ENDPOINT_URI} \
ApiKey={BILLING_KEY}   
```

### <a name="basic-example-for-text-to-speech"></a>Ejemplo básico de texto a voz

```Docker
docker run --rm -it -p 5000:5000 --memory 4g --cpus 2 \
containerpreview.azurecr.io/microsoft/cognitive-services-text-to-speech \
Eula=accept \
Billing={BILLING_ENDPOINT_URI} \
ApiKey={BILLING_KEY}  
```

### <a name="logging-example-for-speech-to-text"></a>Ejemplo de registro de voz a texto

```Docker
docker run --rm -it -p 5000:5000 --memory 4g --cpus 2 \
containerpreview.azurecr.io/microsoft/cognitive-services-speech-to-text \
Eula=accept \
Billing={BILLING_ENDPOINT_URI} \
ApiKey={BILLING_KEY}   
  Logging:Console:LogLevel:Default=Information
```

### <a name="logging-example-for-text-to-speech"></a>Ejemplo de registro de texto a voz

```Docker
docker run --rm -it -p 5000:5000 --memory 4g --cpus 2 \
containerpreview.azurecr.io/microsoft/cognitive-services-text-to-speech \
Eula=accept \
Billing={BILLING_ENDPOINT_URI} \
ApiKey={BILLING_KEY}  
  Logging:Console:LogLevel:Default=Information
```

## <a name="next-steps"></a>Pasos siguientes

* Consulte [Instalación y ejecución de contenedores](speech-container-howto.md)