---
title: Directivas de Azure API Management | Microsoft Docs
description: Aprenda sobre las directivas de transformación disponibles para su uso en Azure API Management. Las directivas permiten que el publicador cambie el comportamiento de la API mediante la configuración.
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/19/2017
ms.author: apimpm
ms.openlocfilehash: 4cdd9b7ace107d09ec14b23e02c56d2b13f264e6
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87905676"
---
# <a name="api-management-policies"></a>Directivas de administración de API
En esta sección se proporciona una referencia para las siguientes directivas de API Management. Para obtener más información sobre cómo agregar y configurar directivas, consulte [Directivas en Administración de API](api-management-howto-policies.md).  
  
 Las directivas constituyen una eficaz funcionalidad del sistema que permite al publicador cambiar el comportamiento de la API a través de la configuración. Las directivas son una colección de declaraciones que se ejecutan secuencialmente en la solicitud o respuesta de una API. Entre las declaraciones más usadas se encuentran la conversión de formato de XML a JSON y la limitación de tasa de llamadas para restringir la cantidad de llamadas entrantes de un desarrollador. Hay muchas más directivas disponibles y listas para usar.  
  
 Las expresiones de directiva pueden utilizarse como valores de atributos o valores de texto en cualquiera de las directivas de API Management, a menos que la directiva especifique lo contrario. Algunas directivas como [Flujo de control](api-management-advanced-policies.md#choose) y [Establecer variable](api-management-advanced-policies.md#set-variable) se basan en expresiones de directiva. Para más información, consulte [Directivas avanzadas](api-management-advanced-policies.md#AdvancedPolicies) y [Expresiones de directiva](api-management-policy-expressions.md).  
  
##  <a name="policies"></a><a name="ProxyPolicies"></a> Directivas  
  
-   [Directivas de restricción de acceso](api-management-access-restriction-policies.md#AccessRestrictionPolicies)  
    -   [Activar encabezado HTTP](api-management-access-restriction-policies.md#CheckHTTPHeader) : aplica la existencia o el valor de un encabezado HTTP.  
    -   [Limitar la frecuencia de llamadas por suscripción](api-management-access-restriction-policies.md#LimitCallRate) : evita los picos de uso de la API limitando la frecuencia de llamadas, por suscripción.  
    -   [Limitar la frecuencia de llamadas por clave](api-management-access-restriction-policies.md#LimitCallRateByKey) : evita los picos de uso de la API limitando la frecuencia de llamadas, por clave.  
    -   [Restringir IP de autor de llamada](api-management-access-restriction-policies.md#RestrictCallerIPs) : filtra (permite/deniega) las llamadas de direcciones IP específicas o de intervalos de direcciones.  
    -   [Establecer cuota de uso por suscripción](api-management-access-restriction-policies.md#SetUsageQuota) : le permite aplicar un volumen de llamadas renovables o permanentes o una cuota de ancho de banda por suscripción.  
    -   [Establecer cuota de uso por clave](api-management-access-restriction-policies.md#SetUsageQuotaByKey) : le permite aplicar un volumen de llamadas renovables o permanentes o una cuota de ancho de banda por clave.  
    -   [Validar JWT](api-management-access-restriction-policies.md#ValidateJWT) : aplica la existencia y la validez de un JWT extraído de un encabezado HTTP especificado o un parámetro de consulta especificado.  
-   [Directivas avanzadas](api-management-advanced-policies.md#AdvancedPolicies)  
    -   [Flujo de control](api-management-advanced-policies.md#choose): aplica condicionalmente instrucciones de directiva basadas en la evaluación de expresiones booleanas.  
    -   [Reenviar solicitud](api-management-advanced-policies.md#ForwardRequest) : reenvía la solicitud al servicio back-end.
    -   [Limitar la simultaneidad](api-management-advanced-policies.md#LimitConcurrency): evita que las directivas delimitadas las ejecute simultáneamente un número de solicitudes mayor que el especificado.
    -   [Registro en el centro de eventos](api-management-advanced-policies.md#log-to-eventhub): envía mensajes en el formato especificado a un destino de mensaje definido por una entidad del registrador.
    -   [Mock response](api-management-advanced-policies.md#mock-response) (Simular respuesta): anula la ejecución de la canalización y devuelve la respuesta ficticia directamente al llamador.
    -   [Reintentar](api-management-advanced-policies.md#Retry) : reintenta ejecutar las instrucciones de directiva adjuntas, si y hasta que se cumple la condición. La ejecución se repite en los intervalos de tiempo especificados y hasta el número de reintentos indicado.  
    -   [Devolver respuesta](api-management-advanced-policies.md#ReturnResponse) : anula la ejecución de la canalización y devuelve la respuesta especificada directamente al llamador.  
    -   [Enviar solicitud unidireccional](api-management-advanced-policies.md#SendOneWayRequest) : envía una solicitud a la dirección URL especificada sin esperar una respuesta.  
    -   [Enviar solicitud](api-management-advanced-policies.md#SendRequest) : envía una solicitud a la dirección URL especificada.
    -   [Establecer el proxy HTTP](api-management-advanced-policies.md#SetHttpProxy): permite enrutar las solicitudes reenviadas a través de un proxy HTTP.
    -   [Establecer variable](api-management-advanced-policies.md#set-variable): conserva un valor en una variable de contexto con nombre para el acceso posterior.  
    -   [Establecer método de solicitud](api-management-advanced-policies.md#SetRequestMethod) : le permite cambiar el método HTTP de una solicitud.  
    -   [Establecimiento de código de estado](api-management-advanced-policies.md#SetStatus): cambia el código de estado HTTP al valor especificado.  
    -   [Trace](api-management-advanced-policies.md#Trace): agrega seguimientos personalizados a la salida de la [inspección de la API](./api-management-howto-api-inspector.md), a la telemetría de Application Insights y a los registros de recursos.  
    -   [Wait](api-management-advanced-policies.md#Wait) (Esperar): espera a que se completen las directivas adjuntas [Send request](api-management-advanced-policies.md#SendRequest) (Enviar solicitud), [Get value from cache](api-management-caching-policies.md#GetFromCacheByKey) (Obtener el valor de caché) o [Control flow](api-management-advanced-policies.md#choose) (Flujo de control) antes de continuar.  
-   [Directivas de autenticación](api-management-authentication-policies.md#AuthenticationPolicies)  
    -   [Autenticar con opción básica](api-management-authentication-policies.md#Basic) : autenticar con un servicio de back-end mediante la autenticación básica.  
    -   [Autenticar con certificado de cliente](api-management-authentication-policies.md#ClientCertificate) : autenticar con un servicio de back-end mediante certificados de cliente.  
    -   [Autenticar con identidad administrada](api-management-authentication-policies.md#ManagedIdentity): autenticar con un servicio de back-end mediante una [identidad administrada](../active-directory/managed-identities-azure-resources/overview.md).  
-   [Directivas de almacenamiento en caché](api-management-caching-policies.md#CachingPolicies)  
    -   [Obtener de caché](api-management-caching-policies.md#GetFromCache) : realiza una búsqueda en caché y devuelve una respuesta en caché válida cuando esté disponible.  
    -   [Almacenar en caché](api-management-caching-policies.md#StoreToCache) : almacena en caché la respuesta de acuerdo con la configuración de control de caché especificada.  
    -   [Obtener valor de caché](api-management-caching-policies.md#GetFromCacheByKey) : recupere un elemento almacenado en caché por clave.  
    -   [Almacenar valor en caché](api-management-caching-policies.md#StoreToCacheByKey) : almacene un elemento en la memoria caché por clave.  
    -   [Quitar valor de caché](api-management-caching-policies.md#RemoveCacheByKey) ; quita un elemento de la memoria caché por clave.  
-   [Directivas entre dominios](api-management-cross-domain-policies.md#CrossDomainPolicies)  
    -   [Permitir llamadas entre dominios](api-management-cross-domain-policies.md#AllowCrossDomainCalls) : permite que la API sea accesible desde los clientes basados en explorador de Adobe Flash y Microsoft Silverlight.  
    -   [CORS](api-management-cross-domain-policies.md#CORS) : agrega soporte de uso compartido de recursos entre orígenes (CORS) a una operación o a una API para permitir llamadas entre dominios desde clientes basados en explorador.  
    -   [JSONP](api-management-cross-domain-policies.md#JSONP) : agrega JSON con soporte de relleno (JSONP) a una operación o a una API para permitir llamadas entre dominios desde clientes basados en explorador de JavaScript.  
-   [Directivas de transformación](api-management-transformation-policies.md#TransformationPolicies)  
    -   [Convertir JSON a XML](api-management-transformation-policies.md#ConvertJSONtoXML) : convierte el cuerpo de solicitud o respuesta de JSON a XML.  
    -   [Convertir XML a JSON](api-management-transformation-policies.md#ConvertXMLtoJSON) : convierte el cuerpo de solicitud o respuesta de XML a JSON.  
    -   [Buscar y reemplazar la cadena en el cuerpo](api-management-transformation-policies.md#Findandreplacestringinbody) : encuentra una subcadena de solicitud o de respuesta y la reemplaza por una subcadena diferente.  
    -   [Enmascarar URL en el contenido](api-management-transformation-policies.md#MaskURLSContent) : reescribe (enmascara) vínculos en el cuerpo de respuesta para que apunten al vínculo equivalente a través de la puerta de enlace.  
    -   [Establecer el servicio back-end](api-management-transformation-policies.md#SetBackendService) : cambia el servicio back-end para una solicitud entrante.  
    -   [Establecer cuerpo](api-management-transformation-policies.md#SetBody) -establece el cuerpo del mensaje para las solicitudes entrantes y salientes.  
    -   [Establecer encabezado HTTP](api-management-transformation-policies.md#SetHTTPheader) : asigna un valor a un encabezado de respuesta o de solicitud existente o agrega un nuevo encabezado de este tipo.  
    -   [Establecer el parámetro de cadena de consulta](api-management-transformation-policies.md#SetQueryStringParameter) : agrega, reemplaza el valor o elimina el parámetro de la cadena de consulta de la solicitud.  
    -   [URL de reescritura](api-management-transformation-policies.md#RewriteURL) : convierte una URL de solicitud de su forma pública a la forma esperada por el servicio web.  
    -   [Transformar XML mediante una XSLT](api-management-transformation-policies.md#XSLTransform): aplica una transformación de XSL al XML del cuerpo de la solicitud o respuesta.  



## <a name="next-steps"></a>Pasos siguientes
Para obtener más información sobre cómo trabajar con directivas, consulte:

+ [Directivas de Azure API Management](api-management-howto-policies.md)
+ [API de transformación](transform-api.md)
+ [Ejemplos de directivas](policy-samples.md)   
