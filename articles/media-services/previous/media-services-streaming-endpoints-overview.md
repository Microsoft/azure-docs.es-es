---
title: Información general de punto de conexión de streaming de Azure Media Services | Microsoft Docs
description: En este artículo se proporciona información general sobre los puntos de conexión de streaming de Azure Media Services.
services: media-services
documentationcenter: ''
author: Juliako
writer: juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.openlocfilehash: a541e1b068ec3667120bbb31e65ca7bc35febadb
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89265922"
---
# <a name="streaming-endpoints-overview"></a>Información general de los puntos de conexión de streaming  

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> No hay características o funcionalidades nuevas para agregar a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).

En Microsoft Azure Media Services (AMS), un **punto de conexión de streaming** representa un servicio de streaming que puede entregar contenido directamente a una aplicación de reproducción de cliente o a una red Content Delivery Network (CDN) para la distribución posterior. Media Services también proporciona integración sin problemas de Azure CDN. La secuencia de salida de un servicio de punto de conexión de streaming puede ser una secuencia en vivo, un vídeo bajo demanda o una descarga progresiva de su recurso en su cuenta de Media Services. Cada cuenta de Azure Media Services incluye un punto de conexión de streaming predeterminado. Es posible crear puntos de conexión de streaming adicionales en la cuenta. Existen dos versiones de puntos de conexión de streaming: 1.0 y 2.0. A partir del 10 de enero de 2017, las cuentas recién creadas de AMS incluirán de manera **predeterminada** la versión 2.0 del punto de conexión de streaming. Los puntos de conexión de streaming adicionales que agregue a esta cuenta también se generarán en la versión 2.0. Este cambio no afectará a las cuentas existentes; los puntos de conexión de streaming actuales mantendrán la versión 1.0, aunque es posible actualizarlos a la versión 2.0. Este cambio implicará modificaciones en cuanto al comportamiento, la facturación y las características (para obtener más información, consulte la sección **Tipos y versiones de streaming** a continuación).

En Azure Media Services se agregaron las siguientes propiedades a la entidad punto de conexión de streaming: **CdnProvider**, **CdnProfile** y **StreamingEndpointVersion**. Para obtener información general detallada de estas propiedades, consulte [aquí](/rest/api/media/operations/streamingendpoint). 

Cuando se crea una cuenta de Azure Media Services, se genera automáticamente un punto de conexión de streaming estándar predeterminado en el estado **Stopped** (Detenido). No se puede eliminar el punto de conexión de streaming predeterminado. Dependiendo de la disponibilidad de Azure CDN en la región de destino, el punto de conexión de streaming recién creado incluye también de manera predeterminada la integración con el proveedor de red CDN "StandardVerizon". 
                
> [!NOTE]
> Es posible deshabilitar la integración de Azure CDN antes de iniciar el punto de conexión de streaming. `hostname` y la dirección URL de streaming permanecen igual habilite o no la red CDN.

Este tema proporciona información general de las principales funcionalidades proporcionadas por los puntos de conexión de streaming.

## <a name="naming-conventions"></a>Convenciones de nomenclatura

Para el punto de conexión predeterminado: `{AccountName}.streaming.mediaservices.windows.net`

Para los puntos de conexión adicionales: `{EndpointName}-{AccountName}.streaming.mediaservices.windows.net`

## <a name="streaming-types-and-versions"></a>Tipos y versiones de streaming

### <a name="standardpremium-types-version-20"></a>Tipos estándar o premium (versión 2.0)

A partir de la versión de enero de 2017 de Media Services, existen dos tipos de streaming: **Estándar** (versión preliminar) y **Premium**. Estos tipos forman parte de la versión "2.0" de los puntos de conexión de streaming.


|Tipo|Descripción|
|--------|--------|  
|**Estándar**|El punto de conexión de streaming predeterminado es de tipo **Estándar** y se puede cambiar al tipo Premium mediante el ajuste de unidades de streaming.|
|**Premium** |Esta opción es la preferible para escenarios profesionales en los que se requiere mayor escala o control. El cambio a un tipo **Premium** se realiza ajustando las unidades de streaming.<br/>Los puntos de conexión de streaming dedicados residen en un entorno aislado y no compiten por los recursos.|

Para los clientes que quieren entregar contenido a grandes audiencias de Internet, se recomienda habilitar CDN en el punto de conexión de streaming.

Para obtener más información, consulte a continuación la sección [Comparar tipos de streaming](#comparing-streaming-types).

### <a name="classic-type-version-10"></a>Tipo clásico (versión 1.0)

Los usuarios que disponen de cuentas de AMS anteriores a la versión del 10 de enero de 2017 tienen un punto de conexión de streaming de tipo **clásico**. Este tipo forma parte de la versión "1.0" de los puntos de conexión de streaming.

Si el punto de conexión de streaming de la **versión "1.0"** tiene al menos 1 unidad de streaming premium, será un punto de conexión de streaming premium y ofrecerá todas las características de AMS (al igual que el tipo **Estándar o Premium**) sin ningún paso de configuración adicional.

>[!NOTE]
>Los puntos de conexión de streaming **clásicos** (versión "1.0" y 0 unidades de streaming) ofrecen características limitadas y no incluyen un Acuerdo de Nivel de Servicio. Se recomienda migrar al tipo **Estándar** para disfrutar de una mejor experiencia y usar características como el empaquetado dinámico, el cifrado y otras que se ofrecen con **dicho** tipo. Para migrar al tipo **Estándar**, vaya a [Azure Portal](https://portal.azure.com/) y seleccione **Opt-in to Standard** (Participar en Estándar). Para obtener más información acerca de la migración, consulte la sección [Migración](#migration-between-types).
>
>Tenga en cuenta que esta operación no se puede revertir y afectará al precio.
>
 
## <a name="comparing-streaming-types"></a>Comparación de tipos de streaming

### <a name="versions"></a>Versiones

|Tipo|Versión de punto de conexión de streaming|Unidades de escalado|CDN|Facturación|
|--------------|----------|-----------------|-----------------|-----------------|
|Clásico|1.0|0|N/D|Gratuito|
|Punto de conexión de streaming estándar (versión preliminar)|2.0|0|Sí|De pago|
|Unidades de streaming premium|1.0|>0|Sí|De pago|
|Unidades de streaming premium|2.0|>0|Sí|De pago|

### <a name="features"></a>Características

Característica|Estándar|Premium
---|---|---
Throughput |Hasta 600 Mbps y puede proporcionar un rendimiento eficaz mucho mayor cuando se usa una red CDN.|200 Mbps por unidad de streaming. Puede proporcionar un rendimiento eficaz mucho mayor cuando se usa una red CDN.
CDN|Azure CDN, red de entrega de contenido de terceros o ninguna red de entrega de contenido.|Azure CDN, red de entrega de contenido de terceros o ninguna red de entrega de contenido.
La facturación se prorratea| Diario|Diario
Cifrado dinámico|Sí|Sí
Empaquetado dinámico|Sí|Sí
Escala|Se amplía automáticamente hasta el rendimiento objetivo.|Unidades de streaming adicionales.
Filtrado de direcciones IP/G20/host personalizado <sup>1</sup>|Sí|Sí
Descarga progresiva|Sí|Sí
Uso recomendado |Se recomienda para la gran mayoría de escenarios de streaming.|Uso profesional. 

<sup>1</sup> Solo se usa directamente en el punto de conexión de streaming cuando la red CDN no está habilitada en el punto de conexión.<br/>

Para obtener información del contrato de nivel de servicio, consulte [Precios y SLA](https://azure.microsoft.com/pricing/details/media-services/).

## <a name="migration-between-types"></a>Migración entre tipos

De | A | Acción
---|---|---
Clásico|Estándar|Necesidad de solicitar la participación
Clásico|Premium| Escala (unidades de streaming adicionales)
Estándar/Premium|Clásico|No está disponible (si la versión de punto de conexión de streaming es 1.0. Se permite cambiar al modelo clásico con la configuración de unidades de escalado en "0")
Estándar (con o sin red CDN)|Premium con la misma configuración|Permitido en el estado **started** (iniciado). (mediante Azure Portal)
Premium (con o sin red CDN)|Estándar, con la misma configuración|Permitido en el estado **started** (iniciado) (a través de Azure Portal)
Estándar (con o sin red CDN)|Premium con diferente configuración|Permitido en el estado **stopped** (detenido) (a través de Azure Portal). No se permite en el estado en ejecución.
Premium (con o sin red CDN)|Estándar con diferente configuración|Permitido en el estado **stopped** (detenido) (a través de Azure Portal). No se permite en el estado en ejecución.
Versión 1.0 con al menos una unidad de streaming en red CDN|Estándar o Premium sin ninguna red CDN|Permitido en el estado **stopped** (detenido). No se permite en el estado **started** (iniciado).
Versión 1.0 con al menos una unidad de streaming en red CDN|Estándar con o sin red CDN|Permitido en el estado **stopped** (detenido). No se permite en el estado **started** (iniciado). La red CDN de la versión 1.0 se eliminará y se creará e iniciará una nueva.
Versión 1.0 con al menos una unidad de streaming en red CDN|Premium con o sin red CDN|Permitido en el estado **stopped** (detenido). No se permite en el estado **started** (iniciado). La red CDN clásica se eliminará y se creará e iniciará una nueva.

## <a name="next-steps"></a>Pasos siguientes
Consulte las rutas de aprendizaje de Media Services.

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
