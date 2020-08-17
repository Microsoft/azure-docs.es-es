---
title: Acciones del motor de reglas de Azure Front Door
description: En este artículo se proporciona una lista de las diversas acciones que puede realizar con el motor de reglas de Azure Front Door.
services: frontdoor
documentationcenter: ''
author: megan-beatty
editor: ''
ms.service: frontdoor
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 4/30/2020
ms.author: mebeatty
ms.openlocfilehash: 74c0a2617a01e8c24cd93a015b667081250657ad
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86521503"
---
# <a name="azure-front-door-rules-engine-actions"></a>Acciones del motor de reglas de Azure Front Door

En el [motor de reglas de Azure Front Door](front-door-rules-engine.md) una regla consta de cero o más condiciones y acciones de coincidencia. En este artículo se muestran descripciones detalladas de las acciones que puede usar en el motor de reglas de Azure Front Door.

Una acción define el comportamiento que se aplica al tipo de solicitud que identifica una condición de coincidencia o un conjunto de condiciones de coincidencia. En el motor de reglas de Azure Front Door, una regla puede contener hasta cinco acciones, solo una de las cuales puede ser una acción de invalidación de la configuración de enrutamiento (reenvío o redireccionamiento).

Las siguientes acciones están disponibles para su uso en el motor de reglas de Azure Front Door.  

## <a name="modify-request-header"></a>Modificación del encabezado de solicitud

Use esta acción para modificar los encabezados presentes en las solicitudes enviadas a su origen.

### <a name="required-fields"></a>Campos obligatorios

Acción | Nombre del encabezado HTTP | Value
-------|------------------|------
Append | Cuando esta opción está seleccionada y la regla coincide, el encabezado especificado en **Nombre del encabezado** se agrega a la solicitud con el valor especificado. Si el encabezado ya está presente, este valor se anexa al valor existente. | String
Sobrescribir | Cuando esta opción está seleccionada y la regla coincide, el encabezado especificado en **Nombre del encabezado** se agrega a la solicitud con el valor especificado. Si el encabezado ya está presente, el valor especificado sobrescribe el valor existente. | String
Eliminar | Cuando esta opción está seleccionada, la regla coincide y el encabezado especificado en la regla está presente, el encabezado se elimina de la solicitud. | String

## <a name="modify-response-header"></a>Modificación del encabezado de respuesta

Use esta acción para modificar los encabezados presentes en las respuestas devueltas a los clientes.

### <a name="required-fields"></a>Campos obligatorios

Acción | Nombre del encabezado HTTP | Value
-------|------------------|------
Append | Cuando esta opción está seleccionada y la regla coincide, el encabezado especificado en **Nombre del encabezado** se agrega a la respuesta con el **Valor** especificado. Si el encabezado ya está presente, el **Valor** se anexa al valor existente. | String
Sobrescribir | Cuando esta opción está seleccionada y la regla coincide, el encabezado especificado en **Nombre del encabezado** se agrega a la respuesta con el **Valor** especificado. Si el encabezado ya está presente, el **Valor** sobrescribe el valor existente. | String
Eliminar | Cuando esta opción está seleccionada, la regla coincide y el encabezado especificado en la regla está presente, el encabezado se elimina de la respuesta. | String

## <a name="route-configuration-overrides"></a>Invalidaciones de las configuraciones de enrutamiento 

### <a name="route-type-redirect"></a>Tipo de enrutamiento: Redirect

Use esta acción para redirigir a los clientes a una nueva dirección URL. 

#### <a name="required-fields"></a>Campos obligatorios

Campo | Descripción 
------|------------
Tipo de redireccionamiento | Seleccione el tipo de respuesta que se devuelva al solicitante: Encontrado (302), Movido (301), Redireccionamiento temporal (307) y Redirección permanente (308).
Protocolo de redireccionamiento | Confrontar solicitud, HTTP, HTTPS.
Host de destino | Seleccione el nombre de host al que desea que se redirija la solicitud. Déjelo en blanco para conservar el host entrante.
Ruta de acceso de destino | Defina la ruta de acceso que se va a usar en la redirección. Déjelo en blanco para conservar la ruta de acceso entrante.  
Cadena de consulta | Defina la cadena de consulta utilizada en la redirección. Déjelo en blanco para conservar la cadena de consulta entrante. 
Fragmento de destino | Defina el fragmento que se va a usar en la redirección. Déjelo en blanco para conservar el fragmento entrante. 


### <a name="route-type-forward"></a>Tipo de enrutamiento: Adelante

Use esta acción para reenviar clientes a una nueva dirección URL. Esta acción también contiene subacciones para la reescritura de direcciones URL y almacenamiento en caché. 

Campo | Descripción 
------|------------
Grupo back-end | Seleccione el grupo de back-end que se va a invalidar y desde el que se van a atender las solicitudes. Esto mostrará todos los grupos de back-end preconfigurados actualmente en el perfil de Front Door. 
Protocolo de reenvío | Confrontar solicitud, HTTP, HTTPS.
Reescritura de direcciones URL | Use esta acción para reescribir la ruta de acceso de una solicitud en camino hacia el origen. Si está habilitada, consulte a continuación los campos adicionales necesarios.
Almacenamiento en memoria caché | Enabled (Habilitada), Disabled (Deshabilitada). Consulte a continuación los campos adicionales necesarios si está habilitado. 

#### <a name="url-rewrite"></a>Reescritura de direcciones URL

Use esta configuración para configurar una **ruta de acceso de reenvío personalizada** opcional que se usará al construir la solicitud para reenviar al back-end.

Campo | Descripción 
------|------------
Ruta de acceso de reenvío personalizada | Defina la ruta de acceso a la que reenviar las solicitudes. 

#### <a name="caching"></a>Almacenamiento en memoria caché

Use esta configuración para controlar el modo en que los archivos se almacenan en caché para las solicitudes que contienen cadenas de consulta y si almacenar en caché el contenido en función de todos los parámetros o solo de los parámetros seleccionados. Puede usar la configuración adicional para sobrescribir el valor de período de vida (TTL) que controla cuánto tiempo permanecen en caché los contenidos grandes de las solicitudes que especifican las condiciones de coincidencia de las reglas. Para forzar el almacenamiento en caché como una acción, establezca el campo correspondiente en "Habilitado". Al hacerlo, aparecen las siguientes opciones: 

Comportamiento de la caché |  Descripción              
---------------|----------------
Pasar por alto las cadenas de consulta | Una vez almacenado en caché el recurso, todas las solicitudes posteriores ignorarán las cadenas de consulta hasta que expire el recurso almacenado en caché.
Almacenar en caché todas las URL únicas | Cada solicitud con una URL única, incluida la cadena de consulta, se trata como un recurso único con su propia memoria caché.
Omitir las cadenas de consulta especificadas | Las cadenas de consulta de la dirección URL de solicitud que aparecen en el valor "Parámetros de la consulta" se omiten para el almacenamiento en caché.
Incluir las cadenas de consulta especificadas | Las cadenas de consulta de la dirección URL de solicitud que aparecen en el valor "Parámetros de la consulta" se usan para el almacenamiento en caché.

Campos adicionales |  Descripción 
------------------|---------------
Compresión dinámica | Front Door puede comprimir dinámicamente el contenido en el borde, lo que genera una respuesta más pequeña y rápida.
Parámetros de consulta | Lista separada por comas de parámetros permitidos (o no permitidos) que se pueden usar como base para el almacenamiento en caché.
Duración de la caché | Duración de la expiración de caché en días, horas, minutos, segundos. Todos los valores deben ser enteros. 

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a configurar su primera [configuración del motor de reglas](front-door-tutorial-rules-engine.md). 
- Obtenga más información sobre las [Condiciones de coincidencia del motor de reglas](front-door-rules-engine-match-conditions.md).
- Obtenga más información acerca del [motor de reglas de Azure Front Door](front-door-rules-engine.md).
