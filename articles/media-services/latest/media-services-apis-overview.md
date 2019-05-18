---
title: Desarrollar con las API de v3 - Azure | Microsoft Docs
description: En este artículo se describe las reglas que se aplican a las entidades y las API cuando se desarrolla con Media Services v3.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 05/02/2019
ms.author: juliako
ms.custom: seodec18
ms.openlocfilehash: 3ce20b56fc2cbebbed4b525eeccc2c12d14cccc3
ms.sourcegitcommit: 9e8dfa1169a55c3c8af93a6c5f4e0dace4de48b2
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65556246"
---
# <a name="developing-with-media-services-v3-apis"></a>Desarrollo con Media Services API v3

Como desarrollador, puede usar la [API de REST](https://aka.ms/ams-v3-rest-ref) de Media Services o bibliotecas de cliente que le permitan interactuar con la API de REST para crear, administrar y mantener fácilmente flujos de trabajo multimedia personalizados. La API de [Media Services v3](https://aka.ms/ams-v3-rest-sdk) se basa en la especificación OpenAPI (anteriormente conocida como Swagger).

En este artículo se describe las reglas que se aplican a las entidades y las API cuando se desarrolla con Media Services v3.

## <a name="accessing-the-azure-media-services-api"></a>Obtener acceso a la API de Azure Media Services

Para ser autorizado a acceder a recursos de Media Services y a Media Services API, se debe autenticar primero. Media Services admite [Azure Active Directory (Azure AD)-según](../../active-directory/fundamentals/active-directory-whatis.md) autenticación. Dos opciones de autenticación comunes son:
 
* **Autenticación de entidad de servicio** : se usa para autenticar un servicio (por ejemplo: web apps, aplicaciones de función, logic apps, API y microservicios). Las aplicaciones que normalmente utilizan este método de autenticación son las que ejecutan servicios de demonio, servicios de nivel intermedio o trabajos programados. Por ejemplo, para Web allí las aplicaciones siempre deben ser un nivel intermedio que se conecta a Media Services con una entidad de servicio.
* **Autenticación de usuario** : se usa para autenticar a alguien que está usando la aplicación para interactuar con recursos de Media Services. La aplicación interactiva, en primer lugar, debe solicitar al usuario las credenciales. Un ejemplo es una aplicación de consola de administración que usan los usuarios autorizados para supervisar trabajos de codificación o streaming en vivo.

La API de Media Services requiere que el usuario o aplicación que realiza la API de REST solicita tiene acceso al recurso de la cuenta de Media Services y usar un **colaborador** o **propietario** rol. Se puede tener acceso a la API con la **lector** , pero solo para el rol **obtener** o **lista**   operaciones estará disponibles. Para obtener más información, consulte [control de acceso basado en roles para cuentas de Media Services](rbac-overview.md).

En lugar de crear a una entidad de servicio, considere el uso de identidades administradas para los recursos de Azure para tener acceso a la API de Media Services mediante Azure Resource Manager. Para obtener más información acerca de las identidades administradas para los recursos de Azure, consulte [What ' s identidades administradas para los recursos de Azure](../../active-directory/managed-identities-azure-resources/overview.md).

### <a name="azure-ad-service-principal"></a>Entidad de servicio de Azure AD 

Si va a crear una aplicación de Azure AD y el servicio principal, la aplicación debe estar en su propio inquilino. Después de crear la aplicación, asigne a la aplicación **colaborador** o **propietario** rol el acceso a la cuenta de Media Services. 

Si no estás seguro de que tenga los permisos para crear una aplicación de Azure AD, consulte [permisos necesarios](../../active-directory/develop/howto-create-service-principal-portal.md#required-permissions).

En la siguiente ilustración, los números representan el flujo de las solicitudes en orden cronológico:

![Aplicaciones de nivel intermedio](./media/use-aad-auth-to-access-ams-api/media-services-principal-service-aad-app1.png)

1. Una aplicación de nivel intermedio solicita un token de acceso de Azure AD que tiene los siguientes parámetros:  

   * Punto de conexión de inquilino de Azure AD.
   * URI del recurso de Media Services.
   * URI del recurso de Media Services de REST.
   * Valores de aplicación de Azure AD: el Id. de cliente y el secreto de cliente.
   
   Para obtener todos los valores necesarios, consulte [Access Azure Media Services API con la CLI de Azure](access-api-cli-how-to.md)

2. El token de acceso de Azure AD se envía al nivel intermedio.
4. El nivel intermedio envía una solicitud a la API de REST de Azure Media Services con el token de Azure AD.
5. El nivel intermedio recibe los datos de Media Services.

### <a name="samples"></a>Ejemplos

Consulte los siguientes ejemplos que muestran cómo conectarse con la entidad de servicio de Azure AD:

* [Conectar con REST](media-rest-apis-with-postman.md)  
* [Conexión con Java](configure-connect-java-howto.md)
* [Conexión con .NET](configure-connect-dotnet-howto.md)
* [Conexión con Node.js](configure-connect-nodejs-howto.md)
* [Conexión con Python](configure-connect-python-howto.md)

## <a name="naming-conventions"></a>Convenciones de nomenclatura

Los nombres de recursos de Azure Media Services v3 (por ejemplo, recursos, trabajos y transformaciones) están sujetos a las restricciones de nomenclatura de Azure Resource Manager. De acuerdo con Azure Resource Manager, los nombres de los recursos siempre son únicos. Por lo tanto, puede usar cualquier cadena de identificador único (por ejemplo, GUID) para los nombres de recursos. 

Los nombres de recursos de Media Services no pueden incluir: '<', '>', '%', '&', ':', '&#92;', '?', '/', '*', '+', '.', el carácter de comilla simple o cualquier carácter de control. Todos los demás caracteres se permiten. La longitud máxima de un nombre de recurso es de 260 caracteres. 

Para más información sobre la nomenclatura de Azure Resource Manager, consulte: [Requisitos de nomenclatura](https://github.com/Azure/azure-resource-manager-rpc/blob/master/v1.0/resource-api-reference.md#arguments-for-crud-on-resource) y [Convenciones de nomenclatura](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions).

## <a name="long-running-operations"></a>Operaciones de larga ejecución

Las operaciones marcan con `x-ms-long-running-operation` de Azure Media Services [swagger archivos](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/mediaservices/resource-manager/Microsoft.Media/stable/2018-07-01/streamingservice.json) son largos las operaciones de ejecución. 

Para obtener más información acerca de cómo realizar un seguimiento de las operaciones asincrónicas de Azure, consulte [operaciones asincrónicas](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-async-operations#monitor-status-of-operation).

Media Services tiene las siguientes operaciones de larga ejecución:

* Crear LiveEvent
* Actualizar LiveEvent
* Eliminar LiveEvent
* Iniciar LiveEvent
* Detener LiveEvent
* Restablecer LiveEvent
* Crear LiveOutput
* Eliminar LiveOutput
* Crear StreamingEndpoint
* Actualizar StreamingEndpoint
* Eliminar StreamingEndpoint
* Iniciar StreamingEndpoint
* Detener StreamingEndpoint
* Escala StreamingEndpoint


## <a name="sdks"></a>SDK

> [!NOTE]
> No se garantiza que los SDK de Azure Media Services v3 sean seguros para subprocesos. Al desarrollar una aplicación multiproceso, debe agregar su propia lógica de sincronización de subprocesos para proteger el cliente o usar un nuevo objeto AzureMediaServicesClient por subproceso. También debe tener cuidado con los problemas de subprocesos múltiples introducidos por objetos opcionales que proporciona el código al cliente (por ejemplo, una instancia de HttpClient en .NET).

|SDK|Referencia|
|---|---|
|[.NET SDK](https://aka.ms/ams-v3-dotnet-sdk)|[Referencia de .NET](https://aka.ms/ams-v3-dotnet-ref)|
|[SDK de Java](https://aka.ms/ams-v3-java-sdk)|[Referencia de Java](https://aka.ms/ams-v3-java-ref)|
|[SDK de Python](https://aka.ms/ams-v3-python-sdk)|[Referencia de Python](https://aka.ms/ams-v3-python-ref)|
|[SDK de Node.js](https://aka.ms/ams-v3-nodejs-sdk) |[Referencia de Node.js](https://aka.ms/ams-v3-nodejs-ref)| 
|[Go SDK](https://aka.ms/ams-v3-go-sdk) |[Referencia de Go](https://aka.ms/ams-v3-go-ref)|
|[SDK de Ruby](https://aka.ms/ams-v3-ruby-sdk)||

### <a name="see-also"></a>Vea también

- [EventGrid .NET SDK that includes Media Service events](https://www.nuget.org/packages/Microsoft.Azure.EventGrid/) (SDK de .NET en EventGrid que incluye eventos de Media Services)
- [Definitions of Media Services events](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/eventgrid/data-plane/Microsoft.Media/stable/2018-01-01/MediaServices.json) (Definición de eventos de Media Services)

## <a name="azure-media-services-explorer"></a>Explorador de Azure Media Services

[Azure Media Services Explorer](https://github.com/Azure/Azure-Media-Services-Explorer) (AMSE) es una herramienta disponible para los clientes de Windows que desean obtener información acerca de Media Services. AMSE es una aplicación de Winforms o C# que permite cargar, descargar, codificar, transmitir vídeo bajo demanda y contenido en directo con Media Services. Esta herramienta es para aquellos clientes que deseen probar Media Services sin escribir ningún código. El código AMSE se proporciona como un recurso para los clientes que desean desarrollar con Media Services.

AMSE es un proyecto de código abierto en el que el soporte técnico lo facilita la comunidad (se pueden notificar los problemas a https://github.com/Azure/Azure-Media-Services-Explorer/issues)). Este proyecto ha adoptado el [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/) (Código de conducta de código abierto de Microsoft). Para más información, consulte las [preguntas más frecuentes del código de conducta](https://opensource.microsoft.com/codeofconduct/faq/) o póngase en contacto con opencode@microsoft.com si tiene cualquier otra pregunta o comentario.

## <a name="filtering-ordering-paging-of-media-services-entities"></a>Filtrado, ordenación y paginación de entidades de Media Services

Consulte [filtrado, ordenación, paginación de entidades de Azure Media Services](entities-overview.md)

## <a name="ask-questions-give-feedback-get-updates"></a>Formule preguntas, realice comentarios y obtenga actualizaciones

Consulte el artículo [Comunidad de Azure Media Services](media-services-community.md) para ver diferentes formas de formular preguntas, enviar comentarios y obtener actualizaciones de Media Services.

## <a name="next-steps"></a>Pasos siguientes

* [Conectarse a Media Services con Java](configure-connect-java-howto.md)
* [Conectarse a Media Services con .NET](configure-connect-dotnet-howto.md)
* [Conectarse a Media Services con Node.js](configure-connect-nodejs-howto.md)
* [Conectarse a Media Services con Python](configure-connect-python-howto.md)
