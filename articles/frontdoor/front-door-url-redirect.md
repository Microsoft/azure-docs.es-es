---
title: 'Azure Front Door: Redirección de direcciones URL | Microsoft Docs'
description: Este artículo le ayudará a comprender cómo Azure Front Door admite el redireccionamiento de direcciones URL para las rutas, si se ha configurado.
services: front-door
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/08/2019
ms.author: duau
ms.openlocfilehash: 41cb2343cb86d2ec756bb0a2fb690b7df886024f
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89399045"
---
# <a name="url-redirect"></a>Redirección de direcciones URL
Puede usar Azure Front Door para redirigir el tráfico. Puede redirigir el tráfico en varios niveles (protocolo, nombre de host, ruta de acceso, cadena de consulta), y toda la funcionalidad se puede configurar para microservicios individuales, ya que la redirección se basa en la ruta de acceso. Esto simplifica la configuración de la aplicación, optimiza el uso de recursos y admite nuevos escenarios de redirección incluida la global y la basada en la ruta de acceso.
</br>

![Redirección de direcciones URL de Azure Front Door Service][1]

## <a name="redirection-types"></a>Tipo de redireccionamiento
Un tipo de redirección establece el código de estado de respuesta para que los clientes comprendan el propósito de la redirección. Se admiten los siguientes tipos de redireccionamiento:

- **301 (movido permanentemente)** : Indica que al recurso de destino se ha asignado un nuevo URI permanente y cualquier referencia futura a este recurso debería usar uno de los URI incluidos. Use el código de estado 301 para el redireccionamiento de HTTP a HTTPS. 
- **302 (encontrado)** : Indica que el recurso de destino reside temporalmente en un URI diferente. Puesto que es posible que el redireccionamiento se modifique en ocasiones, el cliente debería seguir usando el URI de solicitud efectivo para las solicitudes futuras.
- **307 (redirección temporal)** : Indica que el recurso de destino reside temporalmente en otro URI, y el agente de usuario NO DEBE cambiar el método de solicitud si realiza un redireccionamiento automático a ese URI. Puesto que el redireccionamiento puede cambiar con el tiempo, el cliente debería seguir usando el URI de solicitud efectivo original para las solicitudes futuras.
- **308 (redirección permanente)** : Indica que al recurso de destino se ha asignado un nuevo URI permanente y cualquier referencia futura a este recurso debería usar uno de los URI incluidos. Los clientes funcionalidades de edición de vínculos deberían a volver a vincular automáticamente las referencias al URI de solicitud efectivo a una o varias de las nuevas referencias enviadas por el servidor, siempre que sea posible.

## <a name="redirection-protocol"></a>Protocolo de redireccionamiento
Puede establecer el protocolo que se usará para el redireccionamiento. Esto permite uno de los casos de uso más comunes de la característica de redireccionamiento, que consiste en establecer el redireccionamiento de HTTP a HTTPS.

- **Solo HTTPS**: Establezca el protocolo en solo HTTPS, si busca redirigir el tráfico de HTTP a HTTPS. Azure Front Door recomienda que siempre establezca el redireccionamiento en solo HTTPS.
- **Solo HTTP**: Esto redirige la solicitud entrante a HTTP. Use este valor solo si busca mantener el tráfico HTTP tal cual, sin cifrar.
- **Confrontar solicitud**: Esta opción conserva el protocolo usado por la solicitud entrante. Por lo tanto, una solicitud HTTP permanece HTTP, y una solicitud HTTPS permanece HTTPS después del redireccionamiento.

## <a name="destination-host"></a>Host de destino
Como parte de la configuración de una ruta de redireccionamiento, también puede cambiar el nombre de host o el dominio para la solicitud de redireccionamiento. Puede establecer este campo para cambiar el nombre de host en la dirección URL para el redireccionamiento o, de lo contrario, conservar el nombre de host de la solicitud entrante. Por lo tanto, usar este campo le permite redirigir todas las solicitudes enviadas en `https://www.contoso.com/*` hacia `https://www.fabrikam.com/*`.

## <a name="destination-path"></a>Ruta de acceso de destino
Para los casos en los que quiera reemplazar el segmento de ruta de acceso de una dirección URL como parte del redireccionamiento, puede establecer este campo en el nuevo valor de la ruta de acceso. En caso contrario, puede elegir conservar el valor de ruta de acceso como parte del redireccionamiento. Por lo tanto, usar este campo le permite redirigir todas las solicitudes enviadas a `https://www.contoso.com/\*` hacia `https://www.contoso.com/redirected-site`.

## <a name="query-string-parameters"></a>Parámetros de cadena de consulta
También puede reemplazar los parámetros de la cadena de consulta en la dirección URL redirigida. Para reemplazar cualquier cadena de consulta existente desde la dirección URL de solicitud entrante, establezca este campo en "Reemplazar" y, luego, establezca el valor adecuado. En caso contrario, puede conservar el conjunto original de las cadenas de consulta estableciendo el campo en "Conservar". Como ejemplo, usar este campo le permite redirigir todo el tráfico enviado a `https://www.contoso.com/foo/bar` hacia `https://www.contoso.com/foo/bar?&utm_referrer=https%3A%2F%2Fwww.bing.com%2F`. 

## <a name="destination-fragment"></a>Fragmento de destino
El fragmento de destino es la parte de la dirección URL tras "#", normalmente los exploradores lo usan para aterrizar en una sección en concreto de una página. Puede establecer este campo para agregar un fragmento a la dirección URL de redireccionamiento.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [crear una instancia de Front Door](quickstart-create-front-door.md).
- Más información acerca de cómo [funciona Front Door](front-door-routing-architecture.md).

<!--Image references-->
[1]: ./media/front-door-url-redirect/front-door-url-redirect.png