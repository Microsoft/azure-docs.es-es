---
title: Cuotas y límites en Azure Media Services
description: En este tema se describen las cuotas y límites de Microsoft Azure Media Services.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 04/01/2020
ms.author: juliako
ms.openlocfilehash: c2845cd0fad8375db67d1aa86333f144222dc0aa
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87446325"
---
<!-- If you update limits in this topic, make sure to also update https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits#media-services-limits -->
# <a name="azure-media-services-quotas-and-limits"></a>Cuotas y límites de Azure Media Services

Este documento enumera algunos de los límites más comunes de Microsoft Azure Media Services, que a veces se llaman cuotas.

> [!NOTE]
> En relación con los recursos que no están fijados, abra una incidencia de soporte técnico para solicitar un aumento en las cuotas. No cree cuentas adicionales de Azure Media Services para obtener límites mayores.

## <a name="account-limits"></a>Límites de cuentas

| Resource | Límite predeterminado |
| --- | --- |
| [Cuentas de Media Services](media-services-account-concept.md) en una suscripción única | 25 (fijo) |

## <a name="asset-limits"></a>Límites de recursos

| Resource | Límite predeterminado |
| --- | --- |
| [Recursos](assets-concept.md) por cuenta de Media Services | 1 000 000|

## <a name="storage-limits"></a>Límites de Storage

| Resource | Límite predeterminado | 
| --- | --- | 
| Tamaño de archivo| En algunos casos, existe un límite máximo de tamaño de archivo admitido para el procesamiento en Media Services. <sup>(1)</sup> |
| [Cuentas de almacenamiento](storage-account-concept.md) | 100<sup>(2)</sup> (corregido) |

<sup>1</sup> El tamaño máximo admitido para un único blob es actualmente de 5 TB en Azure Blob Storage. En Media Services, se aplican límites adicionales en función de los tamaños de máquina virtual utilizados por el servicio. El límite de tamaño se aplica a los archivos que se cargan y también a los que se generan como resultado del procesamiento (codificación o análisis) de Media Services. Si el archivo de origen es mayor de 260 GB, es muy probable que el trabajo presente un error. 

En la tabla siguiente se muestran los límites en cada una de las unidades reservadas de multimedia (S1, S2 y S3). Si el archivo de origen es mayor que los límites definidos en la tabla, se producirá un error en el trabajo de codificación. Si codifica orígenes de resolución en 4K de larga duración, debe usar unidades reservadas de multimedia S3 para lograr el rendimiento necesario. Si tiene contenido de 4K mayor que el límite de 260 GB en las unidades reservadas de multimedia S3, abra una incidencia de soporte técnico.

|Tipo de unidad reservada de medios|Tamaño máximo de entrada (GB)|
|---|---|
|S1 |    26|
|S2    | 60|
|S3    |260|

<sup>2</sup> Las cuentas de almacenamiento deben proceder de la misma suscripción de Azure.

## <a name="jobs-encoding--analyzing-limits"></a>Límites de trabajos (codificación y análisis)

| Resource | Límite predeterminado | 
| --- | --- | 
| [Trabajos](transforms-jobs-concept.md) por cuenta de Media Services | 500 000 <sup>(3)</sup> (corregido)|
| Entradas de trabajo por trabajo | 50 (cantidad fija)|
| Salidas de trabajo por trabajo | 20 (cantidad fija) |
| [Transformaciones](transforms-jobs-concept.md) por cuenta de Media Services | 100 (cantidad fija)|
| Salidas de transformación en una transformación | 20 (cantidad fija) |
| Archivos por entrada de trabajo|10 (cantidad fija)|

<sup>3</sup> Este número incluye los trabajos en cola, terminados, activos y cancelados. No incluye los trabajos eliminados. 

Se eliminarán automáticamente los registros de trabajo de más de 90 días de su cuenta, aunque el número total de registros no llegue a la cuota máxima. 

## <a name="live-streaming-limits"></a>Límites de streaming en vivo

| Resource | Límite predeterminado | 
| --- | --- | 
| [Eventos en directo](live-events-outputs-concept.md) <sup>(4)</sup> por cuenta de Media Services |5|
| Salidas en directo por evento en directo |3 <sup>(5)</sup> |
| Duración máxima de la salida en directo | [Tamaño de la ventana de DVR](live-event-cloud-dvr.md) |

<sup>4</sup> Para obtener información detallada sobre los límites de los eventos en directo, consulte [Comparación de tipos y límites de los eventos en directo](live-event-types-comparison.md).

<sup>5</sup> Los objetos LiveOutput comienzan cuando se crean y se detienen cuando se eliminan.

## <a name="packaging--delivery-limits"></a>Límites de empaquetado y entrega

| Resource | Límite predeterminado |
| --- | --- |
| [Puntos de conexión de streaming](streaming-endpoint-concept.md) (detenidos o en ejecución) por cuenta de Media Services|2 |
| Unidades de streaming premium | 10 |
| [Filtros de manifiesto dinámico](filters-dynamic-manifest-overview.md)|100|
| [Directivas de streaming](streaming-policy-concept.md) | 100 <sup>(6)</sup> |
| [Localizadores de streaming](streaming-locators-concept.md) únicos asociados con un recurso al mismo tiempo | 100<sup>(7)</sup> (corregido) |

<sup>6</sup> Al usar una [directiva de streaming](/rest/api/media/streamingpolicies) personalizada, debe diseñar un conjunto limitado de dichas directivas para su cuenta de Media Service y reutilizarlas para sus localizadores de streaming siempre que se necesiten las mismas opciones y protocolos de cifrado. No debe crear una nueva directiva de streaming para cada localizador de streaming.

<sup>7</sup> Los localizadores de streaming no están diseñados para administrar el control de acceso por usuario. Para conceder derechos de acceso diferentes a usuarios individuales, use las soluciones de administración de derechos digitales (DRM).

## <a name="protection-limits"></a>Límites de protección

| Resource | Límite predeterminado | 
| --- | --- | 
| Opciones por [directiva de clave de contenido](content-key-policy-concept.md) |30 | 
| Licencias por mes para cada uno de los tipos DRM en el servicio de entrega de claves de Media Services por cuenta|1 000 000|

## <a name="support-ticket"></a>Incidencia de soporte técnico

En el caso de recursos que no son fijos, puede solicitar que se generen las cuotas abriendo una [incidencia de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest). Incluya información detallada en la solicitud en los cambios de cuota que se desean, en los escenarios de casos de uso y las regiones que se requieren. <br/>**No** cree cuentas adicionales de Azure Media Services para obtener límites mayores.

## <a name="next-steps"></a>Pasos siguientes

[Información general](media-services-overview.md)
