---
title: 'Modelo de datos para la telemetría de solicitudes: Azure Application Insights'
description: Modelo de datos de Application Insights para la telemetría de solicitudes
ms.topic: conceptual
ms.date: 01/07/2019
ms.reviewer: sergkanz
ms.openlocfilehash: 7a352f4ce3528d395599a91b53031c74b0873152
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87320568"
---
# <a name="request-telemetry-application-insights-data-model"></a>Telemetría de solicitudes: Modelo de datos de Application Insights

Un elemento de telemetría de solicitud (en [Application Insights](./app-insights-overview.md)) representa la secuencia lógica de ejecución desencadenada por una solicitud externa a la aplicación. Todas las ejecuciones de solicitud se identifican mediante un `ID` y una `url` únicos que contienen todos los parámetros de ejecución. Puede agrupar las solicitudes por `name` lógico y definir el `source` de esta solicitud. La ejecución de código puede ser `success` o `fail`, y tiene una `duration` determinada. Tanto las ejecuciones correctas como las erróneas se pueden seguir agrupando por `resultCode`. La hora de inicio de la telemetría de solicitudes se define en el nivel de sobre.

La telemetría de solicitudes admite el modelo de extensibilidad estándar mediante el uso de `properties` y `measurements` personalizadas.

## <a name="name"></a>Nombre

El nombre de la solicitud representa la ruta de código seguida para procesar la solicitud. Un valor de cardinalidad bajo permite una mejor agrupación de las solicitudes. En las solicitudes de HTTP, representa el método HTTP y la plantilla de la ruta URL como `GET /values/{id}` sin el valor de `id` real.

SDK web de Application Insights envía el nombre de la solicitud tal cual en cuanto a mayúsculas y minúsculas. El agrupamiento de la IU distingue entre mayúsculas y minúsculas, de modo que `GET /Home/Index` se cuenta por separado de `GET /home/INDEX`, incluso aunque a menudo producen la misma ejecución de controlador y acción. El motivo es que las URL suelen [distinguir entre mayúsculas y minúsculas](https://www.w3.org/TR/WD-html40-970708/htmlweb.html). Puede comprobar si todos los errores `404` se produjeron con las URL escritas en mayúsculas. En la [entrada de blog ](https://apmtips.com/posts/2015-02-23-request-name-and-url/) encontrará más información sobre la colección de nombres de solicitud del SDK web de ASP.NET.

Longitud máxima: 1024 caracteres

## <a name="id"></a>id

Identificador de una instancia de llamada de solicitud. Se utiliza para la correlación entre la solicitud y otros elementos de telemetría. El identificador debe ser único en todo el mundo. Para obtener más información, vea la página de [correlación](./correlation.md).

Longitud máxima: 128 caracteres

## <a name="url"></a>Url

URL de solicitud con todos los parámetros de la cadena de consulta.

Longitud máxima: 2048 caracteres

## <a name="source"></a>Source

Origen de la solicitud. Algunos ejemplos son la clave de instrumentación del autor de la llamada o la dirección IP del autor de la llamada. Para obtener más información, vea la página de [correlación](./correlation.md).

Longitud máxima: 1024 caracteres

## <a name="duration"></a>Duration

Duración de la solicitud en formato: `DD.HH:MM:SS.MMMMMM`. Debe ser positiva y tener menos de `1000` días. Este campo es obligatorio, ya que la telemetría de solicitudes representa la operación con el principio y el final.

## <a name="response-code"></a>Response code

Resultado de una ejecución de solicitudes. Código de estado de HTTP para solicitudes HTTP. Puede ser un valor `HRESULT` o un tipo de excepción para otros tipos de solicitudes.

Longitud máxima: 1024 caracteres

## <a name="success"></a>Correcto

Indicación de si la llamada es correcta o no. Este campo es obligatorio. Si no se establece explícitamente en `false`, la solicitud se considera correcta. Establezca este valor en `false` si la operación se ha interrumpido por una excepción o ha devuelto un código de resultado de error.

Para las aplicaciones web, Application Insights define una solicitud como correcta cuando el código de respuesta es menor que `400` o igual a `401`. Sin embargo, hay casos en los que esta asignación predeterminada no coincide con la semántica de la aplicación. El código de respuesta `404` puede indicar que "no hay registros", lo cual puede formar parte de un flujo regular. También puede indicar que un vínculo está roto. Para los vínculos rotos, puede incluso implementar lógica más avanzada. Puede marcar vínculos rotos como errores solo cuando estos se encuentren en el mismo sitio mediante el análisis de origen de referencia de URL. También puede marcarlos como errores accediendo desde la aplicación para dispositivos móviles de la empresa. De modo similar, `301` y `302` indican un error al acceder desde el cliente que no admite redireccionamiento.

El contenido aceptado parcialmente `206` puede indicar un error de una solicitud general. Por ejemplo, el punto de conexión de Application Insights recibe un lote de elementos de telemetría como una solicitud única. Devuelve el error `206` cuando algunos elementos del lote no se han procesado correctamente. Un aumento de la frecuencia del error `206` indica la presencia de un problema que es necesario investigar. Se aplica una lógica similar al error con varios estados de `207`, en el cual el estado correcto puede ser el peor de los códigos de respuesta separados.

Puede leer más sobre el código de resultados de solicitudes y el código de estado en la [publicación del blog](https://apmtips.com/posts/2016-12-03-request-success-and-response-code/).

## <a name="custom-properties"></a>Propiedades personalizadas

[!INCLUDE [application-insights-data-model-properties](../../../includes/application-insights-data-model-properties.md)]

## <a name="custom-measurements"></a>Medidas personalizadas

[!INCLUDE [application-insights-data-model-measurements](../../../includes/application-insights-data-model-measurements.md)]

## <a name="next-steps"></a>Pasos siguientes

- [Escritura de telemetría de solicitud personalizada](./api-custom-events-metrics.md#trackrequest)
- Consulte [modelo de datos](data-model.md) para los tipos y el modelo de datos de Application Insights.
- Aprenda a [configurar la aplicación ASP.NET Core](./asp-net.md) con Application Insights.
- Consulte las [plataformas](./platforms.md) compatibles con Application Insights.

