---
title: Códigos de error de Azure Media Services | Microsoft Docs
description: Es posible que reciba códigos de error HTTP del servicio debido a problemas como la expiración de los tokens de autenticación o acciones que no se admiten en Media Services. En este artículo se proporciona información general de los códigos de error de API de Azure Media Services v2.
author: Juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: d3a62a64-7608-4b17-8667-479b26ba0d6c
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.openlocfilehash: c30808a46b5cecfaf2e761b3f7a611020cd2cdfb
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89263684"
---
# <a name="azure-media-services-error-codes"></a>Códigos de error de Azure Media Services

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Al utilizar Microsoft Azure Media Services, puede recibir los códigos de error HTTP desde el servicio dependiendo de los problemas como los tokens de autenticación que expiran en acciones que no son compatibles en Media Services. La siguiente es una lista de **códigos de error HTTP** que pueden devolver Media Services y las posibles causas de ellos.  

## <a name="400-bad-request"></a>400 - Solicitud incorrecta
La solicitud contiene información no válida y se rechazará debido a uno de los siguientes motivos:

* Se especifica una versión de API no compatible. Para obtener la versión más actualizada, consulte [Configuración del desarrollo de la API de REST de Media Services](media-services-rest-how-to-use.md).
* No se especifica la versión de la API de Media Services. Para más información sobre cómo especificar la versión de la API, vea [Media Services Operations REST API Reference (Referencia de la API de REST de operaciones de Media Services)](/rest/api/media/operations/azure-media-services-rest-api-reference).
  
  > [!NOTE]
  > Si usa SDK de Java o .NET para conectarse a Media Services, la versión de la API se especificará automáticamente cada vez que intente realizar alguna acción con Media Services.
  > 
  > 
* Se ha especificado una propiedad no definida. El nombre de la propiedad se encuentra en el mensaje de error. Se pueden especificar solo las propiedades miembro de una entidad determinada. Consulte [Referencia de la API de REST de Azure Media Services](/rest/api/media/operations/azure-media-services-rest-api-reference) para obtener una lista de entidades y sus propiedades.
* Se ha especificado un valor de propiedad no válido. El nombre de la propiedad se encuentra en el mensaje de error. Vea el vínculo anterior para tipos de propiedad válidos y sus valores.
* Falta un valor de propiedad y es obligatorio.
* Parte de la dirección URL especificada contiene un valor incorrecto.
* Se intentó actualizar una propiedad WriteOnce.
* Se ha intentado crear un trabajo que tiene un recurso de entrada con un AssetFile principal que no se especificó o no se pudo determinar.
* Se realizó un intento de actualizar un localizador SAS. Solo se pueden crear o eliminar localizadores SAS. Los localizadores en streaming pueden actualizarse. Para obtener más información, consulte [Localizadores](/rest/api/media/operations/locator).
* Se ha enviado una operación no compatible o una consulta.

## <a name="401-unauthorized"></a>401 No autorizado
La solicitud no se pudo autenticar (antes de que se pudiera autorizar) debido a una de las siguientes razones:

* Falta el encabezado de autenticación.
* Valor del encabezado de autenticación incorrecto.
  * El token ha expirado. 
  * El token contiene una firma no válida.

## <a name="403-forbidden"></a>403 Prohibido
No se aceptó la solicitud debido a una de las siguientes razones:

* La cuenta de Media Services no se encuentra o se ha eliminado.
* La cuenta de Media Services está deshabilitada y el tipo de solicitud no es HTTP GET. Las operaciones de servicio devolverán también la respuesta 403.
* El token de autenticación no contiene información de credenciales del usuario: AccountName o SubscriptionId. Puede encontrar esta información en la extensión de la interfaz de usuario de Media Services de su cuenta de Media Services en el Portal de administración de Azure.
* No se pueden tener acceso al recurso.
  
  * Se intentó usar un MediaProcessor que no está disponible para su cuenta de Media Services.
  * Se intentó actualizar una JobTemplate definida por Media Services.
  * Se ha intentado sobrescribir el localizador de la cuenta de Media Services.
  * Se ha intentado sobrescribir algún otro ContentKey de la cuenta de Media Services.
* No se pudo crear el recurso debido a que se alcanzó una cuenta de servicio para la cuenta de Media Services. Para más información sobre las cuotas de servicio, consulte [Cuotas y limitaciones](media-services-quotas-and-limitations.md).

## <a name="404-not-found"></a>404 No encontrado
No se permitió la solicitud en un recurso debido a una de las siguientes razones:

* Se realizó un intento de actualizar una entidad que no existe.
* Se realizó un intento de eliminar una entidad que no existe.
* Se realizó un intento de crear una entidad que se vincula a una entidad que no existe.
* Se realizó un intento de obtener una entidad que no existe.
* Se intentó especificar una cuenta de almacenamiento que no está asociada a la cuenta de Media Services.  

## <a name="409-conflict"></a>409 Conflicto
No se aceptó la solicitud debido a una de las siguientes razones:

* Más de un AssetFile tiene un nombre especificado dentro del recurso.
* Se realizó un intento de crear un segundo AssetFile principal dentro del recurso.
* Se realizó un intento de crear una ContentKey con el identificador especificado que ya está en uso.
* Se realizó un intento de crear un localizador con el identificador especificado que ya está en uso.
* Más de un IngestManifestFile tiene el nombre especificado dentro de IngestManifest.
* Se realizó el intento de vincular un segundo ContentKey de cifrado de almacenamiento para el recurso cifrado de almacenamiento.
* Se intentó vincular el mismo ContentKey al recurso.
* Se ha intentado crear un localizador para un recurso cuyo contenedor de almacenamiento falta o ya no está asociado con el recurso.
* Se ha intentado crear un localizador para un recurso que ya tiene 5 localizadores en uso. (Azure Storage impone un límite de cinco directivas de acceso compartido en un contenedor de almacenamiento).
* La vinculación de la cuenta de almacenamiento de un recurso en un IngestManifestAsset no es la misma que la cuenta de almacenamiento usada por IngestManifest principal.  

## <a name="500-internal-server-error"></a>Error de servidor interno 500
Durante el procesamiento de la solicitud, Media Services encuentra algún error que impide que el procesamiento continúe. Esto puede deberse a uno de los siguientes motivos:

* Se produce un error al crear un recurso o trabajo porque la información de la cuota de servicio de la cuenta de Media Services no está disponible temporalmente.
* Se produce un error al crear recurso o contenedor de Blob Storage de IngestManifest porque la información de la cuenta de almacenamiento de la cuenta no está disponible temporalmente.
* Otro error inesperado.

## <a name="503-service-unavailable"></a>Servicio no disponible 503
El servidor actualmente no puede recibir solicitudes. Este error puede deberse a solicitudes excesivas en el servicio. El mecanismo de limitación de Media Services restringe el uso de recursos en las aplicaciones que realizan un número excesivo de solicitudes al servicio.

> [!NOTE]
> Compruebe el mensaje de error y la cadena de error para obtener más información detallada sobre la razón por la que tiene el error 503. Este error no siempre implica limitación.
> 
> 

Los estados posibles son:

* "El servidor está ocupado. Las ejecuciones anteriores de este tipo de solicitud tardaron más de {0} segundos".
* "El servidor está ocupado. Se pueden limitar más de {0} solicitudes por segundo".
* "El servidor está ocupado. Se pueden limitar más de {0} solicitudes en {1} segundos".

Para controlar este error, se recomienda usar la lógica de reintento de retroceso exponencial. Eso significa usar progresivamente esperas más largas entre reintentos para respuestas de error consecutivas.  Para más información, consulte [Bloque de aplicación de control de errores transitorios](/previous-versions/msp-n-p/hh680905(v=pandp.50)).

> [!NOTE]
> Si está utilizando [Azure Media Services SDK para .Net](https://github.com/Azure/azure-sdk-for-media-services/tree/master), la lógica de reintento para el error 503 se ha implementado por el SDK.  
> 
> 

## <a name="see-also"></a>Consulte también
[Códigos de error de administración de Media Services](/rest/api/media/)

## <a name="next-steps"></a>Pasos siguientes
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
