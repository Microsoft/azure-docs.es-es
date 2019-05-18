---
title: Informes principales de Verizon | Microsoft Docs
description: 'Puede ver los patrones de uso de la red CDN mediante los siguientes informes: Ancho de banda, datos transferidos, aciertos, Estados de la memoria caché, datos de IPV4/IPV6 transferidos proporción de aciertos de caché.'
services: cdn
documentationcenter: ''
author: zhangmanling
manager: erikre
editor: ''
ms.assetid: 5a0d9018-8bdb-48ff-84df-23648ebcf763
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/23/2017
ms.author: mazha
ms.openlocfilehash: 88cbd942413757388278d69d728d407271e4c4a3
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65606368"
---
# <a name="core-reports-from-verizon"></a>Informes principales de Verizon

[!INCLUDE [cdn-verizon-only](../../includes/cdn-verizon-only.md)]

Mediante el uso de los informes principales de Verizon a través del portal de administración para perfiles de Verizon, puede ver los patrones de uso de la red CDN con los siguientes informes:

* Ancho de banda
* Datos transferidos
* Accesos
* Estados de la memoria caché
* Frecuencia de aciertos de caché
* Datos de IPV4/IPV6 transferidos

## <a name="accessing-verizon-core-reports"></a>Acceso a los informes principales de Verizon
1. En la hoja de perfil de CDN, haga clic en el botón **Administrar** .
   
    ![Botón de administración del perfil de la red CDN](./media/cdn-reports/cdn-manage-btn.png)
   
    Se abre el Portal de administración de CDN.
2. Mueva el puntero sobre la pestaña **Análisis** y después sobre el control flotante **Core Reports** (Informes básicos). Haga clic en un informe en el menú.
   
    ![Portal de administración de la red CDN - menú Core Reports (Informes básicos)](./media/cdn-reports/cdn-core-reports.png)

3. Para cada informe, seleccione un intervalo de fechas desde la lista **Intervalo de fechas**. Puede seleccionar un intervalo de fechas predefinido, como **Hoy** o **Esta semana** o puede seleccionar **Personalizado** y escribir manualmente un intervalo de fechas haciendo clic en los iconos del calendario. 

4. Después de haber seleccionado un intervalo de fechas, haga clic en **Ir** para generar el informe. 

4. Si desea exportar los datos en formato Excel, haga clic en el icono de Excel sobre el botón **Ir**.

## <a name="bandwidth"></a>Ancho de banda
El informe de ancho de banda consta de una tabla de datos y un gráfico que indican el uso de ancho de banda de CDN para HTTP y HTTPS en un período de tiempo determinado, en Mbps. Puede ver el uso del ancho de banda en un POP determinado o todos los POP. Este informe permite ver los picos y la distribución del tráfico de los servidores POP.

Desde la lista **Nodos perimetrales**, seleccione **Todos los nodos perimetrales** para ver el tráfico de todos los nodos o seleccione una región específica.

El informe se actualiza cada cinco minutos.

![Informe Ancho de banda](./media/cdn-reports/cdn-bandwidth.png)

## <a name="data-transferred"></a>Datos transferidos
Este informe consta de una tabla de datos y un gráfico que indican el uso del tráfico de CDN para HTTP y HTTPS en un período de tiempo determinado, en GB. Puede ver el uso del tráfico en todos los POP o un POP determinado. Este informe permite ver los picos y la distribución del tráfico a través de los POP.

Desde la lista **Nodos perimetrales**, seleccione **Todos los nodos perimetrales** para ver el tráfico de todos los nodos o seleccione una región específica.

El informe se actualiza cada cinco minutos.

![Informe Datos transferidos](./media/cdn-reports/cdn-data-transferred.png)

## <a name="hits-status-codes"></a>Aciertos (códigos de estado)
Este informe describe la distribución de códigos de estado de la solicitud para su contenido. Todas las solicitudes de contenido generan un código de estado HTTP. El código de estado describe cómo los POP perimetrales procesaron la solicitud. Por ejemplo, los códigos de estado 2xx indican que la solicitud se ha servido correctamente a un cliente, mientras que un código de estado 4xx indica que se produjo un error. Para más información sobre los códigos de estado HTTP, consulte la [Lista de códigos de estado HTTP](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes).

![Informe Aciertos](./media/cdn-reports/cdn-hits.png)

## <a name="cache-statuses"></a>Estados de la memoria caché
Este informe describe la distribución de aciertos de caché y errores de caché de las solicitudes de cliente. Puesto que el rendimiento más rápido procede de los aciertos de caché, para optimizar velocidades de entrega de datos puede minimizar los errores de caché y aciertos de caché expirados. 

Para reducir los errores de caché, configure el servidor de origen para minimizar el uso de las siguientes acciones: 
 * Encabezados de respuesta `no-cache`
 * Almacenamiento en caché de la cadena de consulta, a menos que sea necesario  
 * Códigos de respuesta no almacenables en caché

Para reducir los aciertos de caché expirados, establezca el valor de `max-age` de un recurso en un período largo para minimizar el número de solicitudes al servidor de origen.

![Informe Estados de la memoria caché](./media/cdn-reports/cdn-cache-statuses.png)

### <a name="main-cache-statuses-include"></a>Los principales estados de la memoria caché incluyen:
* TCP_HIT: Servido desde el servidor perimetral. El objeto estaba en la caché y no ha excedido su duración máxima.
* TCP_MISS: Servido desde el servidor de origen. El objeto no estaba en la caché y la respuesta se devolvió al origen.
* TCP_EXPIRED _MISS: Servido desde el servidor de origen después de la revalidación con el origen. El objeto estaba en la caché, pero ha excedido su duración máxima. Como resultado de una revalidación con el origen, el objeto de caché será reemplazado por una nueva respuesta desde el origen.
* TCP_EXPIRED _HIT: Servido desde el extremo después de la revalidación con el origen. El objeto estaba en la memoria caché, pero ha excedido su duración máxima. Como resultado de una revalidación con el servidor de origen, el objeto de caché no se ha modificado.

### <a name="full-list-of-cache-statuses"></a>Lista completa de los estados de la memoria caché
* TCP_HIT: este estado se comunica cuando una solicitud se sirve directamente desde el POP al cliente. Un activo se sirve inmediatamente desde un servidor POP cuando se almacena en caché en el servidor POP más cercano al cliente y tiene un valor de período de vida (TTL) válido. TTL se determina mediante los siguientes encabezados de respuesta:
  
  * Cache-Control: s-maxage
  * Cache-Control: max-age
  * Expira
* TCP_MISS: Este estado indica que no se encontró una versión almacenada en caché del recurso solicitado en el POP más cercano al cliente. El recurso se solicita desde un servidor de origen o un servidor de escudo de origen. Si el servidor de origen o el servidor de escudo de origen devuelve un recurso, se sirve al cliente y se almacena en caché en el cliente y el servidor perimetral. En caso contrario, se devuelve un código de estado no 200 (por ejemplo, 403 Prohibido o 404 No encontrado).
* TCP_EXPIRED_HIT: Este estado se comunica cuando una solicitud destinada a un recurso con TTL expirado se sirve directamente desde el POP al cliente. Por ejemplo, cuando la duración máxima del recurso ha expirado. 
  
   Una solicitud expirada normalmente produce una solicitud de revalidación al servidor de origen. Para que se produzca un estado TCP_EXPIRED_HIT, el servidor de origen debe indicar que no existe una versión más reciente del recurso. Esta situación da lugar normalmente a una actualización de los encabezados Cache-Control y Expires del recurso.
* TCP_EXPIRED_MISS: Este estado se comunica cuando una versión más reciente de un recurso almacenado en caché expirado se sirve desde el servidor POP al cliente. Este estado se produce cuando el TTL de un recurso almacenado en caché ha expirado (por ejemplo, la duración máxima ha expirado) y el servidor de origen devuelve una versión más reciente de dicho recurso. Esta nueva versión del recurso se sirve al cliente en lugar de la versión almacenada en caché. Además, se almacena en caché en el servidor perimetral y el cliente.
* CONFIG_NOCACHE: Este estado indica que una configuración específica del cliente el POP perimetral impidió que el recurso se almacene en caché.
* NONE: este estado indica que no se realizó una comprobación de actualización del contenido de la memoria caché.
* TCP_CLIENT_REFRESH_MISS: Este estado se comunica cuando un cliente HTTP, como un explorador, fuerza un POP perimetral para recuperar una versión nueva de un recurso obsoleto desde el servidor de origen. De forma predeterminada, los servidores impiden que un cliente HTTP fuerce los servidores perimetrales para recuperar una nueva versión del recurso desde el servidor de origen.
* TCP_PARTIAL_HIT: Este estado se comunica cuando una solicitud de intervalo de bytes da como resultado un acierto para un recurso almacenado parcialmente en caché. El intervalo de bytes solicitado se sirve inmediatamente desde el POP al cliente.
* UNCACHEABLE: Este estado se comunica cuando un recurso `Cache-Control` y `Expires` encabezados indican que se debe no se puede almacenar en caché en un servidor POP o por el cliente HTTP. Estos tipos de solicitudes se atienden desde el servidor de origen.

## <a name="cache-hit-ratio"></a>Frecuencia de aciertos de caché
Este informe indica el porcentaje de solicitudes almacenadas en caché que se sirven directamente desde la memoria caché.

El informe proporciona los detalles siguientes:

* El contenido solicitado se almacenó en caché en el POP más cercano al solicitante.
* La solicitud se sirve directamente desde el extremo de la red.
* La solicitud no requería revalidación con el servidor de origen.

El informe no incluye:

* Solicitudes denegadas debido a las opciones de filtrado de país o región.
* Solicitudes de recursos cuyos encabezados indican que no se deben almacenar en caché. Por ejemplo, los encabezados `Cache-Control: private`, `Cache-Control: no-cache` o `Pragma: no-cache` impiden que un recurso se almacene en caché.
* Solicitudes de intervalo de bytes para contenido almacenado parcialmente en caché.

La fórmula es: (TCP_ HIT / (TCP_ HIT + TCP_MISS)) * 100

![Informe Frecuencia de aciertos de caché](./media/cdn-reports/cdn-cache-hit-ratio.png)

## <a name="ipv4ipv6-data-transferred"></a>Datos de IPV4/IPV6 transferidos
Este informe muestra la distribución de uso de tráfico en IPV4 frente a IPV6.

![Datos de IPV4/IPV6 transferidos](./media/cdn-reports/cdn-ipv4-ipv6.png)

## <a name="considerations"></a>Consideraciones
Los informes solo pueden generarse en los últimos 18 meses.

