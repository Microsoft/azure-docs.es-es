---
title: Configuración de un contenedor para Anomaly Detector API
titleSuffix: Azure Cognitive Services
description: El entorno de tiempo de ejecución de los contenedores de Anomaly Detector API se configura mediante los argumentos del comando `docker run`. Este contenedor tiene varias opciones de configuración necesarias, así como otras opcionales.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: conceptual
ms.date: 05/07/2020
ms.author: aahi
ms.openlocfilehash: c0bf08ae0b2d26b2f4992181d2e300e9dbeed818
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90903536"
---
# <a name="configure-anomaly-detector-containers"></a>Configuración de los contenedores de Anomaly Detector

El entorno de tiempo de ejecución de los contenedores de **Anomaly Detector** se configura mediante los argumentos del comando `docker run`. Este contenedor tiene varias opciones de configuración necesarias, así como otras opcionales. Hay disponibles varios [ejemplos](#example-docker-run-commands) del comando. La configuración específica del contenedor es la configuración de facturación. 

## <a name="configuration-settings"></a>Parámetros de configuración

Este contenedor tiene las siguientes opciones de configuración:

|Obligatorio|Configuración|Propósito|
|--|--|--|
|Sí|[ApiKey](#apikey-configuration-setting)|Se usa para realizar un seguimiento de la información de facturación.|
|No|[Application Insights](#applicationinsights-setting)|Le permite agregar compatibilidad con los datos de telemetría de [Azure Application Insights](https://docs.microsoft.com/azure/application-insights) al contenedor.|
|Sí|[Facturación](#billing-configuration-setting)|Especifica el URI del punto de conexión del recurso de servicio en Azure.|
|Sí|[Eula](#eula-setting)| Indica que ha aceptado la licencia del contenedor.|
|No|[Fluentd](#fluentd-settings)|Escribe el registro y, opcionalmente, los datos de métricas en un servidor de Fluentd.|
|No|[Proxy Http](#http-proxy-credentials-settings)|Configure un proxy HTTP para hacer solicitudes salientes.|
|No|[Logging](#logging-settings)|Proporciona compatibilidad con el registro de ASP.NET Core al contenedor. |
|No|[Mounts](#mount-settings)|Leer y escribir datos desde el equipo host al contenedor y del contenedor al equipo host.|

> [!IMPORTANT]
> Las opciones [`ApiKey`](#apikey-configuration-setting), [`Billing`](#billing-configuration-setting) y [`Eula`](#eula-setting) se usan en conjunto y debe proporcionar valores válidos para las tres; en caso contrario, no se inicia el contenedor. Para obtener más información sobre el uso de estas opciones de configuración para crear instancias de un contenedor, consulte [Facturación](anomaly-detector-container-howto.md#billing).

## <a name="apikey-configuration-setting"></a>Opción de configuración ApiKey

La opción de configuración `ApiKey` especifica la clave de recurso de Azure usada para realizar un seguimiento de la información de facturación del contenedor. Debe especificar un valor para ApiKey, que debe ser una clave válida para el recurso de _Anomaly Detector_ especificado para la opción de configuración [`Billing`](#billing-configuration-setting).

Este valor se puede encontrar en el siguiente lugar:

* Azure Portal: Administrador de recursos de **Anomaly Detector**, en **Claves**

## <a name="applicationinsights-setting"></a>Opción de configuración ApplicationInsights

[!INCLUDE [Container shared configuration ApplicationInsights settings](../../../includes/cognitive-services-containers-configuration-shared-settings-application-insights.md)]

## <a name="billing-configuration-setting"></a>Opción de configuración Billing

El valor de configuración `Billing` especifica el identificador URI del punto de conexión del recurso de _Anomaly Detector_ en Azure que se usa para medir la información de facturación del contenedor. Debe especificar un valor para esta opción de configuración, y el valor debe ser un URI de punto de conexión válido para un recurso de _Anomaly Detector_ en Azure.

Este valor se puede encontrar en el siguiente lugar:

* Azure Portal: Introducción a **Anomaly Detector**, etiquetado `Endpoint`

|Obligatorio| Nombre | Tipo de datos | Descripción |
|--|------|-----------|-------------|
|Sí| `Billing` | String | Identificador URI del punto de conexión de facturación. Para más información sobre cómo obtener el URI de facturación, consulte la [recopilación de los parámetros necesarios](anomaly-detector-container-howto.md#gathering-required-parameters). Para más información y para obtener una lista completa de los puntos de conexión regionales, consulte [Nombres de subdominios personalizados para Cognitive Services.](../cognitive-services-custom-subdomains.md) |

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

Los contenedores de Anomaly Detector no usan montajes de entrada o salida para almacenar datos de entrenamiento o servicio. 

La sintaxis exacta de la ubicación de montaje del host varía según el sistema operativo del host. Además, la ubicación de montaje del [equipo host](anomaly-detector-container-howto.md#the-host-computer) puede no ser accesible debido a un conflicto entre los permisos que usa la cuenta de servicio de Docker y los permisos de la ubicación de montaje del host. 

|Opcional| Nombre | Tipo de datos | Descripción |
|-------|------|-----------|-------------|
|No permitida| `Input` | String | Los contenedores de Anomaly Detector no usan esto.|
|Opcional| `Output` | String | Destino del montaje de salida. El valor predeterminado es `/output`. Esta es la ubicación de los registros. Esto incluye los registros de contenedor. <br><br>Ejemplo:<br>`--mount type=bind,src=c:\output,target=/output`|

## <a name="example-docker-run-commands"></a>Comandos de ejemplo de docker run 

Los ejemplos siguientes usan las opciones de configuración para ilustrar cómo escribir y usar comandos `docker run`.  Una vez que se está ejecutando, el contenedor continúa ejecutándose hasta que lo [detenga](anomaly-detector-container-howto.md#stop-the-container).

* **Carácter de continuación de línea**: Los comandos de Docker de las secciones siguientes usan la barra diagonal inversa, `\`, como carácter de continuación de línea para un shell de Bash. Puede quitarla o reemplazarla en función de los requisitos del sistema operativo del host. Por ejemplo, el carácter de continuación de línea para Windows es un símbolo de intercalación, `^`. Reemplace la barra diagonal inversa por el símbolo de intercalación. 
* **Orden de los argumentos**: No cambie el orden de los argumentos a menos que esté muy familiarizado con los contenedores de Docker.

Reemplace el valor entre corchetes, `{}`, por sus propios valores:

| Marcador de posición | Value | Formato o ejemplo |
|-------------|-------|---|
| **{CLAVE_API}** | La clave del punto de conexión del recurso `Anomaly Detector` en la página Claves de `Anomaly Detector` de Azure. | `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx` |
| **{URI_PUNTODECONEXIÓN}** | El valor del punto de conexión de facturación está disponible en la página Información general de Azure `Anomaly Detector`.| Consulte el apartado de [recopilación de los parámetros necesarios](anomaly-detector-container-howto.md#gathering-required-parameters) para ejemplos explícitos. |

[!INCLUDE [subdomains-note](../../../includes/cognitive-services-custom-subdomains-note.md)]

> [!IMPORTANT]
> Para poder ejecutar el contenedor, las opciones `Eula`, `Billing` y `ApiKey` deben estar especificadas; de lo contrario, el contenedor no se iniciará.  Para obtener más información, vea [Facturación](anomaly-detector-container-howto.md#billing).
> El valor de ApiKey es la **clave** de la página de claves de recursos de Azure Anomaly Detector. 

## <a name="anomaly-detector-container-docker-examples"></a>Ejemplos de Docker de contenedor de Anomaly Detector

Los siguientes ejemplos de Docker son para el contenedor de Anomaly Detector. 

### <a name="basic-example"></a>Ejemplo básico 

  ```Docker
  docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 \
  mcr.microsoft.com/azure-cognitive-services/decision/anomaly-detector \
  Eula=accept \
  Billing={ENDPOINT_URI} \
  ApiKey={API_KEY} 
  ```

### <a name="logging-example-with-command-line-arguments"></a>Ejemplo de registro con argumentos de la línea de comandos

  ```Docker
  docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 \
  mcr.microsoft.com/azure-cognitive-services/decision/anomaly-detector \
  Eula=accept \
  Billing={ENDPOINT_URI} ApiKey={API_KEY} \
  Logging:Console:LogLevel:Default=Information
  ```

## <a name="next-steps"></a>Pasos siguientes

* [Implementación de un contenedor Anomaly Detector en Azure Container Instances](how-to/deploy-anomaly-detection-on-container-instances.md)
* [Más información sobre el servicio de API Anomaly Detector](https://go.microsoft.com/fwlink/?linkid=2080698&clcid=0x409)
