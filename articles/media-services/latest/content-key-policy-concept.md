---
title: 'Directivas de clave de contenido de Media Services: Azure | Microsoft Docs'
description: En este artículo se explica qué son las directivas de clave de contenido y cómo las usa Azure Media Services.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: seodec18
ms.openlocfilehash: dd19d0730b25685b008af16d1c1eac6537750612
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89297276"
---
# <a name="content-key-policies"></a>Directivas de clave de contenido

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Con Media Services puede entregar el contenido cifrado de forma dinámica en vivo y a petición con el Estándar de cifrado avanzado (AES-128) o cualquiera de los tres sistemas de administración de derechos digitales (DRM) principales: Microsoft PlayReady, Google Widevine y Apple FairPlay. Media Services también proporciona un servicio para entregar claves AES y licencias de DMR (PlayReady, Widevine y FairPlay) a los clientes autorizados. 

Para especificar opciones de cifrado en la secuencia, debe crear la [directiva de streaming](streaming-policy-concept.md) y asociarla con su [localizador de streaming](streaming-locators-concept.md). La [directiva de clave de contenido](/rest/api/media/contentkeypolicies) se crea para configurar el modo en que la clave de contenido (que proporciona acceso seguro a sus [recursos](assets-concept.md)) se entrega a los clientes finales. Debe establecer los requisitos (restricciones) en la directiva de clave de contenido que se deben cumplir para que las claves con la configuración especificada se entreguen a los clientes. La directiva de clave de contenido no es necesaria para el streaming ni la descarga sin cifrar. 

Por lo general, se asocia la directiva de clave de contenido con el [localizador de streaming](streaming-locators-concept.md). Como alternativa, puede especificar la directiva de clave de contenido dentro de una [directiva de streaming](streaming-policy-concept.md) (al crear una directiva de streaming personalizada para escenarios avanzados). 

## <a name="best-practices-and-considerations"></a>Procedimientos recomendados y consideraciones

> [!IMPORTANT]
> Revise las siguientes recomendaciones.

* Debería diseñar un conjunto limitado de directivas para su cuenta de Media Service y reutilizarlas con los localizadores de streaming siempre que se necesiten las mismas opciones. Para obtener más información, consulte [Cuotas y límites](limits-quotas-constraints.md).
* Las directivas de clave de contenido son actualizables. Puede tardar hasta 15 minutos para que las memorias caché de entrega de claves se actualicen y recojan la directiva actualizada. 

   Al actualizar la directiva, está sobrescribiendo la memoria caché existente de CDN, lo que podría producir un problema de reproducción para los clientes que usan contenido almacenado en caché.  
* Se recomienda no crear una nueva directiva de clave de contenido para cada recurso. Las principales ventajas de compartir la misma directiva de clave de contenido entre los recursos que necesitan las mismas opciones de directiva son:
   
   * Es más fácil administrar un pequeño número de directivas.
   * Si tiene que hacer actualizaciones en la directiva de clave de contenido, los cambios entran en vigor en todas las solicitudes de licencia nuevas casi de inmediato.
* Si tiene que crear una nueva directiva, tiene que crear un nuevo localizador de streaming para el recurso.
* Se recomienda permitir que Media Services genere automáticamente la clave de contenido. 

   Normalmente, usaría una clave de larga duración y comprobaría la existencia de la directiva de clave de contenido con [Get](/rest/api/media/contentkeypolicies/get). Para obtener la clave, debe llamar a un método de acción independiente para conseguir los secretos o credenciales. Vea el ejemplo siguiente.

## <a name="example"></a>Ejemplo

Para obtener la clave, use `GetPolicyPropertiesWithSecretsAsync`, tal y como se muestra en el ejemplo [Obtención de una clave de firma de la directiva existente](get-content-key-policy-dotnet-howto.md#get-contentkeypolicy-with-secrets).

## <a name="filtering-ordering-paging"></a>Filtrado, ordenación, paginación

Consulte [Filtrado, ordenación y paginación de entidades de Media Services](entities-overview.md).

## <a name="additional-notes"></a>Notas adicionales

* Las propiedades de directivas de clave de contenido que son de tipo `Datetime` siempre están en formato UTC.
* Widevine es un servicio que ofrece Google Inc. y que está sujeto a los términos del servicio y la directiva de privacidad de Google, Inc.

## <a name="next-steps"></a>Pasos siguientes

* [Uso del cifrado dinámico AES-128 y el servicio de entrega de claves](protect-with-aes128.md)
* [Uso del cifrado dinámico de DRM y el servicio de entrega de licencias](protect-with-drm.md)
* [EncodeHTTPAndPublishAESEncrypted](https://github.com/Azure-Samples/media-services-v3-dotnet-core-tutorials/tree/master/NETCore/EncodeHTTPAndPublishAESEncrypted)
