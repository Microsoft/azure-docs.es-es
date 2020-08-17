---
title: Directivas de Azure API Management | Microsoft Docs
description: Obtenga información acerca de cómo crear, editar y configurar directivas en Administración de API. Vea ejemplos de código y examine los recursos adicionales disponibles.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/29/2017
ms.author: apimpm
ms.openlocfilehash: a0786c1ce5d3a693dabe422b65af7177e16c20b9
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87905523"
---
# <a name="policies-in-azure-api-management"></a>Directivas de Azure API Management

En Azure API Management (APIM), las directivas constituyen una eficaz funcionalidad del sistema que permite al editor cambiar el comportamiento de la API mediante la configuración. Las directivas son una colección de declaraciones que se ejecutan secuencialmente en la solicitud o respuesta de una API. Entre las declaraciones más usadas se encuentran la conversión de formato de XML a JSON y la limitación de tasa de llamadas para restringir la cantidad de llamadas entrantes de un desarrollador. Hay muchas más directivas disponibles y listas para usar.

Las directivas se aplican en la puerta de enlace que se encuentra entre el consumidor de la API y la API administrada. La puerta de enlace recibe todas las solicitudes y normalmente las reenvía sin modificar a la API subyacente. Sin embargo, una directiva puede aplicar cambios a la solicitud de entrada y a la respuesta de salida.

Las expresiones de directiva pueden utilizarse como valores de atributos o valores de texto en cualquiera de las directivas de API Management, a menos que la directiva especifique lo contrario. Algunas directivas como [Flujo de control][Control flow] y [Establecer variable][Set variable] se basan en expresiones de directiva. Para más información, consulte [Directivas avanzadas][Advanced policies] y [Expresiones de directiva][Policy expressions].

## <a name="understanding-policy-configuration"></a><a name="sections"> </a>Descripción de la configuración de directivas

La definición de la directiva es un documento XML simple que describe una secuencia de declaraciones de entrada y de salida. El XML se puede editar directamente en la ventana de definición. Se ofrece una lista de instrucciones a la derecha y las instrucciones aplicables al ámbito actual están habilitadas y resaltadas.

Al hacer clic en una declaración habilitada se agregará el XML correspondiente en la ubicación del cursor en la vista de definición. 

> [!NOTE]
> Si la directiva que desea agregar no está habilitada, asegúrese de que se encuentra en el ámbito correcto para esa directiva. Cada instrucción de la directiva está diseñada para su uso en determinados ámbitos y secciones de la directiva. Para revisar las secciones y los ámbitos de una directiva, compruebe la sección de **uso** de esa directiva en la [referencia de directivas][Policy Reference].
> 
> 

La configuración se divide en `inbound`, `backend`, `outbound` y `on-error`. La serie de instrucciones de una directiva determinada se ejecuta en orden para una solicitud y una respuesta.

```xml
<policies>
  <inbound>
    <!-- statements to be applied to the request go here -->
  </inbound>
  <backend>
    <!-- statements to be applied before the request is forwarded to 
         the backend service go here -->
  </backend>
  <outbound>
    <!-- statements to be applied to the response go here -->
  </outbound>
  <on-error>
    <!-- statements to be applied if there is an error condition go here -->
  </on-error>
</policies> 
```

Si se produce un error durante el procesamiento de una solicitud, los pasos restantes de las secciones `inbound`, `backend` o `outbound` se omiten y la ejecución salta a las instrucciones de la sección `on-error`. Mediante la colocación de instrucciones de directiva en la sección `on-error` puede revisar el error con la propiedad `context.LastError`, inspeccionar y personalizar la respuesta de error con la directiva `set-body` y configurar lo que ocurre si se produce un error. Existen códigos de error para pasos integrados y errores que pueden producirse durante el procesamiento de las instrucciones de directiva. Para más información, consulte [Control de errores en las directivas de administración de API](./api-management-error-handling-policies.md).

## <a name="how-to-configure-policies"></a><a name="scopes"> </a>Configuración de directivas

Para información sobre cómo configurar directivas, consulte el artículo sobre [edición o establecimiento de directivas](set-edit-policies.md).

## <a name="policy-reference"></a>Referencia de las directivas

Consulte [Referencia de directivas](./api-management-policies.md) para ver una lista completa de declaraciones de directivas y su configuración.

## <a name="policy-samples"></a>Ejemplos de directivas

Vea [ejemplos de directivas](policy-samples.md) para obtener más código de ejemplo.

## <a name="examples"></a>Ejemplos

### <a name="apply-policies-specified-at-different-scopes"></a>Aplicación de las directivas especificadas en distintos ámbitos

Si tiene una directiva de nivel global y una directiva configurada para una API, cuando se use esa API en concreto, se aplicarán ambas directivas. Administración de API tiene en cuenta el orden determinista de declaraciones de directiva combinadas mediante el elemento base. 

```xml
<policies>
    <inbound>
        <cross-domain />
        <base />
        <find-and-replace from="xyz" to="abc" />
    </inbound>
</policies>
```

En la definición de directiva del ejemplo anterior, la instrucción `cross-domain` se ejecutaría antes de las directivas superiores que, a su vez, irían seguidas de la directiva `find-and-replace`. 

### <a name="restrict-incoming-requests"></a>Restricción de las solicitudes de entrada

Para agregar una nueva instrucción a fin de restringir las solicitudes de entrada a direcciones IP concretas, sitúe el cursor exactamente dentro del contenido del elemento XLM `inbound` y haga clic en la instrucción **Restrict caller IPs** (Restringir las IP del autor de llamada).

![Restriction policies][policies-restrict]

Así, se agregará un fragmento de código XML al elemento `inbound` que ofrece orientación sobre cómo configurar la instrucción.

```xml
<ip-filter action="allow | forbid">
    <address>address</address>
    <address-range from="address" to="address"/>
</ip-filter>
```

Para limitar las solicitudes de entrada y aceptar solo las procedentes de una dirección IP de 1.2.3.4, modifique el XML de la manera siguiente:

```xml
<ip-filter action="allow">
    <address>1.2.3.4</address>
</ip-filter>
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre cómo trabajar con directivas, consulte:

+ [API de transformación](transform-api.md)
+ En la [Referencia de directivas](./api-management-policies.md) se muestra una lista completa de declaraciones de directivas y su configuración
+ [Ejemplos de directivas](policy-samples.md)   

[Policy Reference]: ./api-management-policies.md
[Product]: api-management-howto-add-products.md
[API]: api-management-howto-add-products.md
[Operation]: ./mock-api-responses.md

[Advanced policies]: ./api-management-advanced-policies.md
[Control flow]: ./api-management-advanced-policies.md#choose
[Set variable]: ./api-management-advanced-policies.md#set-variable
[Policy expressions]: ./api-management-policy-expressions.md

[policies-restrict]: ./media/api-management-howto-policies/api-management-policies-restrict.png
