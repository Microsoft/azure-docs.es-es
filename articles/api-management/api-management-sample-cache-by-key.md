---
title: Almacenamiento en caché personalizado en Azure API Management
description: Aprenda a almacenar en caché elementos en función de la clave en Azure API Management. Puede modificar la clave mediante el uso de encabezados de solicitud.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 772bc8dd-5cda-41c4-95bf-b9f6f052bc85
ms.service: api-management
ms.devlang: dotnet
ms.custom: devx-track-csharp
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/15/2016
ms.author: apimpm
ms.openlocfilehash: cf9901b4e49460dd2fb91dceaf239571058c5284
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88213317"
---
# <a name="custom-caching-in-azure-api-management"></a>Almacenamiento en caché personalizado en Azure API Management
El servicio Azure API Management integra compatibilidad para [el almacenamiento en caché de respuestas HTTP](api-management-howto-cache.md) mediante el uso de la dirección URL como clave. La clave se puede modificar por los encabezados de solicitud con las propiedades `vary-by` . Esto es útil para almacenar en caché respuestas HTTP completas (también llamadas representaciones), pero a veces es útil para almacenar en caché solo una parte de una representación. Las nuevas directivas [cache-lookup-value](./api-management-caching-policies.md#GetFromCacheByKey) y [cache-store-value](./api-management-caching-policies.md#StoreToCacheByKey) permiten almacenar y recuperar fragmentos arbitrarios de datos desde las definiciones de directiva. Esta capacidad también agrega valor a la directiva [send-request](./api-management-advanced-policies.md#SendRequest) previamente introducida porque ahora se puede almacenar en caché las respuestas desde servicios externos.

## <a name="architecture"></a>Architecture
El servicio API Management usa una caché de datos compartida por inquilino que, cuando escale verticalmente a varias unidades, seguirá obteniendo acceso a los mismos datos en caché. Sin embargo, cuando se trabaja con una implementación de varias regiones son memorias caché independientes dentro de cada una de las regiones. Es importante no tratar la memoria caché como un almacén de datos, donde está el único origen de información. Sin embargo, si ya lo hizo y más adelante decide aprovechar las ventajas de la implementación en varias regiones, es posible que los clientes con los usuarios que viajan puedan perder el acceso a los datos almacenados en caché.

## <a name="fragment-caching"></a>Almacenamiento en caché de fragmentos
Hay algunos casos donde las respuestas que se devuelven contienen parte de datos que es costoso determinar y todavía siguen actualizadas durante un tiempo razonable. Como ejemplo, piense en un servicio creado por una línea aérea que ofrece información relacionada con las reservas de vuelos, estado del vuelo, etc. Si el usuario es miembro del programa de puntos de la compañía aérea, tendría además información relativa a su estado actual y a las millas acumuladas. Esta información relacionada con el usuario se puede almacenar en un sistema diferente, pero quizá sea deseable incluirlo en las respuestas devueltas sobre las reservas y el estado de los vuelos. Esto puede hacerse mediante un proceso denominado almacenamiento en caché de fragmentos. La representación principal puede devolverse desde el servidor de origen mediante algún tipo de token para indicar donde se insertará la información relacionada con el usuario. 

Considere la siguiente respuesta JSON de una API de back-end.

```json
{
  "airline" : "Air Canada",
  "flightno" : "871",
  "status" : "ontime",
  "gate" : "B40",
  "terminal" : "2A",
  "userprofile" : "$userprofile$"
}  
```

Y el recurso secundario en `/userprofile/{userid}` que es como sigue,

```json
{ "username" : "Bob Smith", "Status" : "Gold" }
```

Para determinar la información de usuario adecuada que se va a incluir, API Management tiene que identificar quién es el usuario final. Este mecanismo depende de la implementación. Por ejemplo, estamos usando la notificación `Subject` de un token `JWT`. 

```xml
<set-variable
  name="enduserid"
  value="@(context.Request.Headers.GetValueOrDefault("Authorization","").Split(' ')[1].AsJwt()?.Subject)" />
```

API Management almacena el valor `enduserid` en una variable de contexto para su uso posterior. El siguiente paso es determinar si una solicitud anterior ya ha recuperado la información de usuario y la ha almacenado en la memoria caché. Para ello, API Management utiliza la directiva `cache-lookup-value`.

```xml
<cache-lookup-value
key="@("userprofile-" + context.Variables["enduserid"])"
variable-name="userprofile" />
```

Si no hay ninguna entrada en la memoria caché que se corresponda con el valor de clave, no se creará la variable de contexto `userprofile`. API Management comprueba el éxito de la búsqueda mediante la directiva de flujo de control `choose`.

```xml
<choose>
    <when condition="@(!context.Variables.ContainsKey("userprofile"))">
        <!-- If the userprofile context variable doesn’t exist, make an HTTP request to retrieve it.  -->
    </when>
</choose>
```

Si la variable de contexto `userprofile` no existe, API Management tendrá que realizar una solicitud HTTP para recuperarla.

```xml
<send-request
  mode="new"
  response-variable-name="userprofileresponse"
  timeout="10"
  ignore-error="true">

  <!-- Build a URL that points to the profile for the current end-user -->
  <set-url>@(new Uri(new Uri("https://apimairlineapi.azurewebsites.net/UserProfile/"),
      (string)context.Variables["enduserid"]).AbsoluteUri)
  </set-url>
  <set-method>GET</set-method>
</send-request>
```

API Management usa `enduserid` para construir la dirección URL en el recurso del perfil de usuario. Una vez que API Management tenga la respuesta, extrae el texto del cuerpo de la respuesta y lo almacena en una variable de contexto.

```xml
<set-variable
    name="userprofile"
    value="@(((IResponse)context.Variables["userprofileresponse"]).Body.As<string>())" />
```

Para evitar que API Management realice esta solicitud HTTP de nuevo, cuando el mismo usuario realiza otra solicitud, puede especificar que se almacenar el perfil de usuario en la memoria caché.

```xml
<cache-store-value
    key="@("userprofile-" + context.Variables["enduserid"])"
    value="@((string)context.Variables["userprofile"])" duration="100000" />
```

API Management almacena el valor en la memoria caché con la misma clave con la que lo intentó recuperar originalmente. La duración que API Management elige para almacenar el valor debe basarse en la frecuencia de los cambios de la información y en lo tolerantes que son los usuarios con la información sin actualizar. 

Es importante tener en cuenta que la recuperación desde la caché sigue siendo una solicitud de red fuera del proceso y posiblemente pueda agregar decenas de milisegundos a la solicitud. Las ventajas se derivan de determinar que la información de perfil de usuario tarda más que eso, debido a que se necesitan realizar consultas en la base de datos o agregar información de varios back-end.

El último paso del proceso consiste en actualizar la respuesta devuelta con la información de perfil de usuario.

```xml
<!-- Update response body with user profile-->
<find-and-replace
    from='"$userprofile$"'
    to="@((string)context.Variables["userprofile"])" />
```

Puede elegir incluir las comillas como parte del token para que incluso cuando no se produzca el reemplazo, la respuesta siga siendo un JSON válido.  

Una vez que se combinan todos estos pasos, el resultado final es una directiva similar a la siguiente.

```xml
<policies>
    <inbound>
        <!-- How you determine user identity is application dependent -->
        <set-variable
          name="enduserid"
          value="@(context.Request.Headers.GetValueOrDefault("Authorization","").Split(' ')[1].AsJwt()?.Subject)" />

        <!--Look for userprofile for this user in the cache -->
        <cache-lookup-value
          key="@("userprofile-" + context.Variables["enduserid"])"
          variable-name="userprofile" />

        <!-- If API Management doesn’t find it in the cache, make a request for it and store it -->
        <choose>
            <when condition="@(!context.Variables.ContainsKey("userprofile"))">
                <!-- Make HTTP request to get user profile -->
                <send-request
                  mode="new"
                  response-variable-name="userprofileresponse"
                  timeout="10"
                  ignore-error="true">

                   <!-- Build a URL that points to the profile for the current end-user -->
                    <set-url>@(new Uri(new Uri("https://apimairlineapi.azurewebsites.net/UserProfile/"),(string)context.Variables["enduserid"]).AbsoluteUri)</set-url>
                    <set-method>GET</set-method>
                </send-request>

                <!-- Store response body in context variable -->
                <set-variable
                  name="userprofile"
                  value="@(((IResponse)context.Variables["userprofileresponse"]).Body.As<string>())" />

                <!-- Store result in cache -->
                <cache-store-value
                  key="@("userprofile-" + context.Variables["enduserid"])"
                  value="@((string)context.Variables["userprofile"])"
                  duration="100000" />
            </when>
        </choose>
        <base />
    </inbound>
    <outbound>
        <!-- Update response body with user profile-->
        <find-and-replace
              from='"$userprofile$"'
              to="@((string)context.Variables["userprofile"])" />
        <base />
    </outbound>
</policies>
```

Este enfoque de almacenamiento en caché se utiliza principalmente en los sitios web donde se compone HTML en el servidor para que se pueda representar como una sola página. También puede ser útil en las API donde los clientes no puedan realizar almacenamiento en caché de HTTP del lado cliente o sea deseable que la responsabilidad no recaiga en el cliente.

Este mismo tipo de almacenamiento en caché también puede realizarse en los servidores web de back-end con un servidor de Caché en Redis; sin embargo, el uso del servicio Administración de API para realizar este trabajo resulta útil cuando los fragmentos en caché proceden de servidores back-end distintos a los de las respuestas principales.

## <a name="transparent-versioning"></a>Control de versiones transparente
Es una práctica común que se admitan al mismo tiempo varias versiones de implementación distintas de una API. Por ejemplo, admitir diferentes entornos (desarrollo, prueba, producción, etc.) o versiones anteriores de la API para dar tiempo a los consumidores de la API a migrar a versiones más recientes. 

Un método para tratar eso, en lugar de requerir a los desarrolladores de cliente que cambien las direcciones URL de `/v1/customers` a `/v2/customers`, es almacenar en los datos de perfil del cliente la versión de la API que quieren usar y llamar a la dirección URL adecuada de back-end. Para determinar la dirección URL correcta de back-end a fin de llamar a un cliente determinado, es necesario consultar algunos datos de configuración. Al almacenar en caché estos datos de configuración, API Management puede minimizar la reducción del rendimiento al realizar esta búsqueda.

El primer paso consiste en determinar el identificador usado para configurar la versión deseada. En este ejemplo, decidí asociar la versión con la clave de suscripción del producto. 

```xml
<set-variable name="clientid" value="@(context.Subscription.Key)" />
```

Después, API Management hace una búsqueda en la caché para ver si ya recuperó la versión de cliente deseada.

```xml
<cache-lookup-value
key="@("clientversion-" + context.Variables["clientid"])"
variable-name="clientversion" />
```

A continuación, comprueba si no la encontró en la memoria caché.

```xml
<choose>
    <when condition="@(!context.Variables.ContainsKey("clientversion"))">
```

Si API Management no la encontró, la recupera.

```xml
<send-request
    mode="new"
    response-variable-name="clientconfiguresponse"
    timeout="10"
    ignore-error="true">
            <set-url>@(new Uri(new Uri(context.Api.ServiceUrl.ToString() + "api/ClientConfig/"),(string)context.Variables["clientid"]).AbsoluteUri)</set-url>
            <set-method>GET</set-method>
</send-request>
```

Extraemos el texto del cuerpo de respuesta de la respuesta.

```xml
<set-variable
      name="clientversion"
      value="@(((IResponse)context.Variables["clientconfiguresponse"]).Body.As<string>())" />
```

Lo almacenamos en la caché para un uso futuro.

```xml
<cache-store-value
      key="@("clientversion-" + context.Variables["clientid"])"
      value="@((string)context.Variables["clientversion"])"
      duration="100000" />
```

Y finalmente actualizamos la dirección URL de back-end para seleccionar la versión del servicio requerido por el cliente.

```xml
<set-backend-service
      base-url="@(context.Api.ServiceUrl.ToString() + "api/" + (string)context.Variables["clientversion"] + "/")" />
```

La directiva completa es la siguiente:

```xml
<inbound>
    <base />
    <set-variable name="clientid" value="@(context.Subscription.Key)" />
    <cache-lookup-value key="@("clientversion-" + context.Variables["clientid"])" variable-name="clientversion" />

    <!-- If API Management doesn’t find it in the cache, make a request for it and store it -->
    <choose>
        <when condition="@(!context.Variables.ContainsKey("clientversion"))">
            <send-request mode="new" response-variable-name="clientconfiguresponse" timeout="10" ignore-error="true">
                <set-url>@(new Uri(new Uri(context.Api.ServiceUrl.ToString() + "api/ClientConfig/"),(string)context.Variables["clientid"]).AbsoluteUri)</set-url>
                <set-method>GET</set-method>
            </send-request>
            <!-- Store response body in context variable -->
            <set-variable name="clientversion" value="@(((IResponse)context.Variables["clientconfiguresponse"]).Body.As<string>())" />
            <!-- Store result in cache -->
            <cache-store-value key="@("clientversion-" + context.Variables["clientid"])" value="@((string)context.Variables["clientversion"])" duration="100000" />
        </when>
    </choose>
    <set-backend-service base-url="@(context.Api.ServiceUrl.ToString() + "api/" + (string)context.Variables["clientversion"] + "/")" />
</inbound>
```

Permitir a los consumidores de la API controlar de forma transparente la versión de back-end a la que acceden los clientes sin tener que actualizar y volver a implementar los clientes es una solución elegante que soluciona muchos problemas de control de versiones de la API.

## <a name="tenant-isolation"></a>Aislamiento de inquilinos
En implementaciones grandes multiinquilino algunas compañías crean grupos independientes de inquilinos en distintas implementaciones del hardware de back-end. Esto minimiza el número de clientes que se ven afectados por un problema de hardware en el back-end. También permite que nuevas versiones de software se implementen en etapas. Lo ideal es que esta arquitectura de back-end sea transparente para los usuarios de las API. Esto puede lograrse de manera similar con el control de versiones transparente porque se basa en la misma técnica de manipular la dirección URL de back-end mediante el estado de configuración por clave de API.  

En lugar de devolver una versión preferida de la API para cada clave de la suscripción, debe devolver un identificador que relacione un inquilino con el grupo de hardware asignado. Este identificador se puede usar para construir la dirección URL apropiada de back-end.

## <a name="summary"></a>Resumen
La libertad de utilizar la caché de Administración de API de Azure para almacenar cualquier tipo de datos permite un acceso eficaz a los datos de configuración, lo que puede afectar a la manera de procesar una solicitud entrante. También puede usarse para almacenar fragmentos de datos que pueden aumentar las respuestas devueltas desde una API de back-end.
