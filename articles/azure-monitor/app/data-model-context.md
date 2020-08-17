---
title: 'Modelo de datos de telemetría de Azure Application Insights: contexto de telemetría | Microsoft Docs'
description: Modelo de datos de contexto de telemetría de Application Insights
ms.topic: conceptual
ms.date: 05/15/2017
ms.reviewer: sergkanz
ms.openlocfilehash: 25ff7d92da5ea0a6aba84aad1cfc98e5295e151e
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87322676"
---
# <a name="telemetry-context-application-insights-data-model"></a>Contexto de telemetría: Modelo de datos de Application Insights

Cada elemento de telemetría puede tener campos de contexto fuertemente tipados. Cada campo permite un escenario de supervisión específico. Use la recopilación de propiedades personalizadas para guardar información contextual personalizada o específica de aplicaciones.


## <a name="application-version"></a>Versión de la aplicación

La información de los campos de contexto de la aplicación siempre trata sobre la aplicación que envía la telemetría. La versión de la aplicación se usa para analizar cambios de tendencia en el comportamiento de la aplicación y su correlación con las implementaciones.

Longitud máxima: 1024


## <a name="client-ip-address"></a>Dirección IP de cliente

La dirección IP del dispositivo cliente. Se admiten IPv4 e IPv6. Al enviar telemetría desde un servicio, el contexto de la ubicación trata sobre el usuario que inició la operación en el servicio. Application Insights extrae la información de geolocalización de la IP del cliente y seguidamente la trunca. La IP del cliente no puede usarse por sí misma a modo de información de identificación del usuario final. 

Longitud máxima: 46


## <a name="device-type"></a>Tipo de dispositivo

Originalmente, este campo se usaba para indicar el tipo de dispositivo que usaba el usuario final de la aplicación. Hoy se usa principalmente para distinguir la telemetría de JavaScript con el tipo de dispositivo "Navegador" de la telemetría del lado del servidor con el tipo de dispositivo "PC".

Longitud máxima: 64


## <a name="operation-id"></a>Identificador de operación

Es un identificador único de la operación de raíz. Este identificador permite agrupar telemetría de varios componentes. Consulte [Correlación de telemetría](./correlation.md) para obtener más información. El identificador de la operación se crea, bien mediante una solicitud, bien mediante una vista de página. El resto de telemetría establece este campo en el valor de la solicitud o la vista de página que la contienen. 

Longitud máxima: 128


## <a name="parent-operation-id"></a>Identificador de la operación principal

Es el identificador único del primario inmediato del elemento de telemetría. Consulte [Correlación de telemetría](./correlation.md) para obtener más información.

Longitud máxima: 128


## <a name="operation-name"></a>Nombre de la operación

Este es el nombre (grupo) de la operación. El nombre de la operación se crea, bien mediante una solicitud, bien mediante una vista de página. El resto de los elementos de telemetría establece este campo en el valor de la solicitud o la vista de página que la contienen. El nombre de la operación se usa para localizar todos los elementos de telemetría de un grupo de operaciones (por ejemplo, "GET Home/Index"). Esta propiedad del contexto se usa para contestar a preguntas como "¿cuáles son las excepciones habituales que se producen en esta página?".

Longitud máxima: 1024


## <a name="synthetic-source-of-the-operation"></a>Origen sintético de la operación

Nombre del origen sintético. Cierta telemetría de la aplicación puede representar tráfico sintético. Puede tratarse de un robot de búsqueda que esté indexando el sitio web, de pruebas de disponibilidad del sitio o de seguimientos de bibliotecas de diagnóstico, como el propio SDK de Application Insights.

Longitud máxima: 1024


## <a name="session-id"></a>Identificador de la sesión

Identificador de la sesión: la instancia de la interacción del usuario con la aplicación. La información de los campos de contexto de la sesión siempre trata sobre el usuario final. Al enviar telemetría desde un servicio, el contexto de la sesión trata sobre el usuario que inició la operación en el servicio.

Longitud máxima: 64


## <a name="anonymous-user-id"></a>Identificador del usuario anónimo

Este es el identificador del usuario anónimo. Representa al usuario final de la aplicación. Al enviar telemetría desde un servicio, el contexto del usuario trata sobre el usuario que inició la operación en el servicio.

El [muestreo](./sampling.md) es una de las técnicas mediante las cuales se reduce al mínimo la cantidad de telemetría recopilada. El algoritmo de muestreo intenta muestrear toda la telemetría correlacionada para aceptarla o rechazarla. El identificador del usuario anónimo se usa para muestrear la generación de puntuaciones. Por ello, el identificador del usuario anónimo debería ser un valor suficiente aleatorio. 

Usar el identificador del usuario anónimo para almacenar el nombre de usuario es usar el campo de forma incorrecta. Use el identificador del usuario autenticado.

Longitud máxima: 128


## <a name="authenticated-user-id"></a>Identificador del usuario autenticado

Este es el identificador del usuario autenticado. Este campo, el contrario al del identificador del usuario anónimo, representa al usuario con un nombre sencillo. Puesto que se trata de información de identificación personal, la mayoría de SDK no la recopilan de forma predeterminada.

Longitud máxima: 1024


## <a name="account-id"></a>Identificador de la cuenta

En aplicaciones multiinquilino, este es el identificador o el nombre de la cuenta con el que actúa el usuario. Algunos ejemplos son el identificador de la suscripción de Azure Portal o el nombre del blog de la plataforma de creación de blogs.

Longitud máxima: 1024


## <a name="cloud-role"></a>Rol en la nube

Este es el nombre del que la aplicación forma parte. Se asigna directamente al nombre del rol de Azure. También puede usarse para distinguir microservicios que formen parte de una única aplicación.

Longitud máxima: 256


## <a name="cloud-role-instance"></a>Instancia de rol en la nube

Este es el nombre de la instancia en la que se ejecuta la aplicación. El nombre de equipo del nombre de instancia local para Azure.

Longitud máxima: 256


## <a name="internal-sdk-version"></a>Interno: Versión del SDK

Esta es la versión del SDK. Consulte [este artículo](https://github.com/microsoft/ApplicationInsights-Home/blob/master/EndpointSpecs/SDK-VERSIONS.md) para más información.

Longitud máxima: 64


## <a name="internal-node-name"></a>Interno: nombre del nodo

Este campo representa el nombre del nodo que se usa para la facturación. Úselo para invalidar la detección estándar de nodos.

Longitud máxima: 256


## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo [ampliar y filtrar la telemetría](./api-filtering-sampling.md).
- Consulte [modelo de datos](data-model.md) para los tipos y el modelo de datos de Application Insights.
- Compruebe la [configuración](./configuration-with-applicationinsights-config.md#telemetry-initializers-aspnet) de la recopilación de propiedades estándar de contexto.

