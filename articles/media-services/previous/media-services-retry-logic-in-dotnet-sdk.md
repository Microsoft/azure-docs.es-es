---
title: Lógica de reintento en el SDK de Media Services para .NET | Microsoft Docs
description: El tema proporciona una descripción de la lógica de reintento en el SDK de Media Services para .NET.
author: Juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: 527b61a6-c862-4bd8-bcbc-b9aea1ffdee3
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.openlocfilehash: 120b7e044452dc47126923449a3e1a6e55cfd6a8
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87000030"
---
# <a name="retry-logic-in-the-media-services-sdk-for-net"></a>Lógica de reintento en el SDK de Media Services para .NET  

Cuando se trabaja con servicios de Microsoft Azure, pueden producirse errores transitorios. Si se produce un error transitorio, en la mayoría de los casos, el funcionamiento se produce después de algunos reintentos. El SDK de Media Services para .NET implementa la lógica de reintento para administrar los errores transitorios asociados a las excepciones y los errores causados por las solicitudes web, que ejecutan consultas, guardan cambios y operaciones de almacenamiento.  De forma predeterminada, el SDK de Media Services para .NET ejecuta cuatro reintentos antes de volver a producir la excepción en la aplicación. El código de la aplicación debe controlar esta excepción correctamente.  

 La siguiente es una breve directriz de las directivas de solicitud de servicio web, almacenamiento, consulta y SaveChanges:  

* La directiva de almacenamiento se usa para las operaciones de Blob Storage (cargas o descarga de archivos de recursos).  
* La directiva de solicitud web se utiliza para las solicitudes web genéricas (por ejemplo, para obtener un token de autenticación y resolver el punto de conexión del clúster de los usuarios).  
* La directiva de consulta se usa para consultar entidades de REST (por ejemplo, mediaContext.Assets.Where(...)).  
* La directiva de SaveChanges se utiliza para realizar cualquier acción que cambia los datos en el servicio (por ejemplo, la creación de una entidad que actualiza una entidad, o una llamada a una función de servicio para una operación).  
  
  En este tema se enumeran los tipos de excepción y los códigos de error que administra el SDK de Media Services para la lógica de reintento de .NET.  

## <a name="exception-types"></a>Tipos de excepciones
En la tabla siguiente se describen las excepciones que administra el SDK de Media Services para .NET o no administra algunas operaciones que pueden provocar errores transitorios.  

| Excepción | Solicitud web | Storage | Consultar | SaveChanges |
| --- | --- | --- | --- | --- |
| WebException<br/>Para más información, vea la sección [Códigos de estado de WebException](media-services-retry-logic-in-dotnet-sdk.md#WebExceptionStatus). |Sí |Sí |Sí |Sí |
| DataServiceClientException<br/> Para obtener más información, consulte [Códigos de estado de error HTTP](media-services-retry-logic-in-dotnet-sdk.md#HTTPStatusCode). |No |Sí |Sí |Sí |
| DataServiceQueryException<br/> Para obtener más información, consulte [Códigos de estado de error HTTP](media-services-retry-logic-in-dotnet-sdk.md#HTTPStatusCode). |No |Sí |Sí |Sí |
| DataServiceRequestException<br/> Para obtener más información, consulte [Códigos de estado de error HTTP](media-services-retry-logic-in-dotnet-sdk.md#HTTPStatusCode). |No |Sí |Sí |Sí |
| DataServiceTransportException |No |No |Sí |Sí |
| TimeoutException |Sí |Sí |Sí |No |
| SocketException |Sí |Sí |Sí |Sí |
| StorageException |No |Sí |No |No |
| IOException |No |Sí |No |No |

### <a name="webexception-status-codes"></a><a name="WebExceptionStatus"></a> Códigos de estado WebException
La tabla siguiente se muestra para qué códigos de error de WebException se implementa la lógica de reintento. La enumeración [WebExceptionStatus](/dotnet/api/system.net.webexceptionstatus?view=netcore-3.1) define los códigos de estado.  

| Status | Solicitud web | Storage | Consultar | SaveChanges |
| --- | --- | --- | --- | --- |
| ConnectFailure |Sí |Sí |Sí |Sí |
| NameResolutionFailure |Sí |Sí |Sí |Sí |
| ProxyNameResolutionFailure |Sí |Sí |Sí |Sí |
| SendFailure |Sí |Sí |Sí |Sí |
| PipelineFailure |Sí |Sí |Sí |No |
| ConnectionClosed |Sí |Sí |Sí |No |
| KeepAliveFailure |Sí |Sí |Sí |No |
| UnknownError |Sí |Sí |Sí |No |
| ReceiveFailure |Sí |Sí |Sí |No |
| RequestCanceled |Sí |Sí |Sí |No |
| Tiempo de espera |Sí |Sí |Sí |No |
| ProtocolError <br/>El reintento de ProtocolError se controla mediante la administración del código de estado HTTP. Para obtener más información, consulte [Códigos de estado de error HTTP](media-services-retry-logic-in-dotnet-sdk.md#HTTPStatusCode). |Sí |Sí |Sí |Sí |

### <a name="http-error-status-codes"></a><a name="HTTPStatusCode"></a> Códigos de estado de error HTTP
Cuando las operaciones Query y SaveChanges producen DataServiceClientException, DataServiceQueryException o DataServiceQueryException, se devuelve el código de estado de error HTTP en la propiedad StatusCode.  La tabla siguiente se muestra para qué códigos de error se implementa la lógica de reintento.  

| Status | Solicitud web | Storage | Consultar | SaveChanges |
| --- | --- | --- | --- | --- |
| 401 |No |Sí |No |No |
| 403 |No |Sí<br/>Administración de reintentos con esperas más prolongadas. |No |No |
| 408 |Sí |Sí |Sí |Sí |
| 429 |Sí |Sí |Sí |Sí |
| 500 |Sí |Sí |Sí |No |
| 502 |Sí |Sí |Sí |No |
| 503 |Sí |Sí |Sí |Sí |
| 504 |Sí |Sí |Sí |No |

Si desea echar un vistazo a la implementación real del SDK de Media Services para la lógica de reintento. NET, consulte [azure-sdk-for-media-services](https://github.com/Azure/azure-sdk-for-media-services/tree/dev/src/net/Client/TransientFaultHandling).

## <a name="next-steps"></a>Pasos siguientes
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
