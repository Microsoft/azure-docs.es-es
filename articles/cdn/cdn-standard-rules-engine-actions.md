---
title: Acciones en el motor de reglas estándar de Azure CDN | Microsoft Docs
description: Documentación de referencia para las acciones del motor de reglas estándar de Azure Content Delivery Network (Azure CDN).
services: cdn
author: asudbring
ms.service: azure-cdn
ms.topic: article
ms.date: 08/04/2020
ms.author: allensu
ms.openlocfilehash: 051737a9f5e0d4092cda26a3f7ce3df1d7f535ef
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87760131"
---
# <a name="actions-in-the-standard-rules-engine-for-azure-cdn"></a>Acciones en el motor de reglas estándar de Azure CDN

En el [motor de reglas estándar](cdn-standard-rules-engine.md) de Azure Content Delivery Network (Azure CDN), una regla consta de una o varias condiciones de coincidencia y una acción. En este artículo se muestran descripciones detalladas de las acciones que puede usar en el motor de reglas estándar de Azure CDN.

La segunda parte de una regla es una acción. Una acción define el comportamiento que se aplica al tipo de solicitud que identifica una condición de coincidencia o un conjunto de condiciones de coincidencia.

## <a name="actions"></a>Acciones

Las siguientes acciones están disponibles para su uso en el motor de reglas estándar de Azure CDN. 

### <a name="cache-expiration"></a>Expiración de la caché

Use esta acción para sobrescribir el valor de período de vida (TTL) del punto de conexión para las solicitudes que especifican las condiciones de coincidencia de reglas.

#### <a name="required-fields"></a>Campos obligatorios

Comportamiento de la caché |  Descripción              
---------------|----------------
Omitir caché | Cuando esta opción está seleccionada y la regla coincide, el contenido no se almacena en caché.
Invalidar | Cuando esta opción está seleccionada y la regla coincide, el valor de TTL devuelto desde el origen se sobrescribe con el valor especificado en la acción. Este comportamiento solo se aplicará si la respuesta se puede almacenar en caché. Para el encabezado de respuesta Cache-Control con los valores "no-cache", "private", "no-store", la acción no será aplicable.
Establecer si falta | Cuando esta opción está seleccionada y la regla coincide, si no se ha devuelto ningún valor de TTL desde el origen, la regla establece el TTL en el valor especificado en la acción. Este comportamiento solo se aplicará si la respuesta se puede almacenar en caché. Para el encabezado de respuesta Cache-Control con los valores "no-cache", "private", "no-store", la acción no será aplicable.

#### <a name="additional-fields"></a>Campos adicionales

Días | Horas | Minutos | Segundos
-----|-------|---------|--------
Int | Int | Int | Int 

### <a name="cache-key-query-string"></a>Cadena de consulta de clave de caché

Use esta acción para modificar la clave de caché en función de las cadenas de consulta.

#### <a name="required-fields"></a>Campos obligatorios

Comportamiento | Descripción
---------|------------
Incluir | Cuando esta opción está seleccionada y la regla coincide, las cadenas de consulta especificadas en los parámetros se incluyen al generar la clave de caché. 
Almacenar en caché todas las URL únicas | Cuando esta opción está seleccionada y la regla coincide, cada dirección URL única tiene su propia clave de caché. 
Exclude | Cuando esta opción está seleccionada y la regla coincide, las cadenas de consulta especificadas en los parámetros se excluyen al generar la clave de caché.
Pasar por alto las cadenas de consulta | Cuando esta opción está seleccionada y la regla coincide, las cadenas de consulta no se tienen en cuenta al generar la clave de caché. 

### <a name="modify-request-header"></a>Modificación del encabezado de solicitud

Use esta acción para modificar los encabezados presentes en las solicitudes enviadas a su origen.

#### <a name="required-fields"></a>Campos obligatorios

Acción | Nombre del encabezado HTTP | Value
-------|------------------|------
Append | Cuando esta opción está seleccionada y la regla coincide, el encabezado especificado en **Nombre del encabezado** se agrega a la solicitud con el valor especificado. Si el encabezado ya está presente, este valor se anexa al valor existente. | String
Sobrescribir | Cuando esta opción está seleccionada y la regla coincide, el encabezado especificado en **Nombre del encabezado** se agrega a la solicitud con el valor especificado. Si el encabezado ya está presente, el valor especificado sobrescribe el valor existente. | String
Eliminar | Cuando esta opción está seleccionada, la regla coincide y el encabezado especificado en la regla está presente, el encabezado se elimina de la solicitud. | String

### <a name="modify-response-header"></a>Modificación del encabezado de respuesta

Use esta acción para modificar los encabezados presentes en las respuestas devueltas a los clientes.

#### <a name="required-fields"></a>Campos obligatorios

Acción | Nombre del encabezado HTTP | Value
-------|------------------|------
Append | Cuando esta opción está seleccionada y la regla coincide, el encabezado especificado en **Nombre del encabezado** se agrega a la respuesta con el **Valor** especificado. Si el encabezado ya está presente, el **Valor** se anexa al valor existente. | String
Sobrescribir | Cuando esta opción está seleccionada y la regla coincide, el encabezado especificado en **Nombre del encabezado** se agrega a la respuesta con el **Valor** especificado. Si el encabezado ya está presente, el **Valor** sobrescribe el valor existente. | String
Eliminar | Cuando esta opción está seleccionada, la regla coincide y el encabezado especificado en la regla está presente, el encabezado se elimina de la respuesta. | String

### <a name="url-redirect"></a>Redirección de URL

Use esta acción para redirigir a los clientes a una nueva dirección URL. 

#### <a name="required-fields"></a>Campos obligatorios

Campo | Descripción 
------|------------
Tipo | Seleccione el tipo de respuesta que se devuelva al solicitante: Encontrado (302), Movido (301), Redireccionamiento temporal (307) y Redirección permanente (308).
Protocolo | Confrontar solicitud, HTTP, HTTPS.
Hostname | Seleccione el nombre de host al que desea que se redirija la solicitud. Déjelo en blanco para conservar el host entrante.
Path | Defina la ruta de acceso que se va a usar en la redirección. Déjelo en blanco para conservar la ruta de acceso entrante.  
Cadena de consulta | Defina la cadena de consulta utilizada en la redirección. Déjelo en blanco para conservar la cadena de consulta entrante. 
Fragmento | Defina el fragmento que se va a usar en la redirección. Déjelo en blanco para conservar el fragmento entrante. 

Se recomienda encarecidamente usar una dirección URL absoluta. El uso de direcciones URL relativas podría redirigir las direcciones URL de Azure CDN a rutas de acceso no válidas. 

### <a name="url-rewrite"></a>Reescritura de direcciones URL

Use esta acción para reescribir la ruta de acceso de una solicitud en camino hacia el origen.

#### <a name="required-fields"></a>Campos obligatorios

Campo | Descripción 
------|------------
Patrón de origen | Defina el patrón de origen en la ruta de acceso URL que se va a reemplazar. Actualmente, el patrón de origen usa una coincidencia basada en el prefijo. Para una coincidencia con todas las ruta de acceso de las direcciones URL, use una barra ( **/** ) como valor de patrón de origen.
Destination | Defina la ruta de acceso de destino que se va a usar en la reescritura. La ruta de acceso de destino sobrescribe el patrón de origen.
Conservar la ruta de acceso sin coincidencia | Si se establece en **Sí**, el resto de la ruta de acceso después del patrón de origen se anexa a la nueva ruta de acceso de destino. 

## <a name="next-steps"></a>Pasos siguientes

- [Información general de Azure CDN](cdn-overview.md)
- [Referencia del motor de reglas estándar](cdn-standard-rules-engine-reference.md)
- [Condiciones de coincidencia en el motor de reglas estándar](cdn-standard-rules-engine-match-conditions.md)
- [Aplicación de HTTPS mediante el motor de reglas estándar](cdn-standard-rules-engine.md)
