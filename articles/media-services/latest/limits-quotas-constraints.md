---
title: Cuotas y limitaciones en Azure Media Services v3 | Microsoft Docs
description: En este tema se describen las cuotas y limitaciones en Azure Media Services v3
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 05/16/2019
ms.author: juliako
ms.openlocfilehash: 1aa15a42893d867ae18c267e163e8df94af50723
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65824451"
---
# <a name="quotas-and-limitations-in-azure-media-services-v3"></a>Cuotas y limitaciones en Azure Media Services v3

En este artículo se describen las cuotas y limitaciones de Azure Media Services v3.

| Recurso | Límite predeterminado | 
| --- | --- | 
| Recursos por cuenta de Azure Media Services | 1 000 000|
| Filtros de manifiesto dinámico|100|
| JobInputs por trabajo | 50 (cantidad fija)|
| JobOutputs por trabajo | 20 (cantidad fija) |
| TransformOutputs en una transformación | 20 (cantidad fija) |
| Archivos por JobInput|10 (cantidad fija)|
| Tamaño de archivo| En algunos casos, existe un límite máximo de tamaño de archivo admitido para el procesamiento en Media Services. <sup>(1)</sup> |
| Trabajos por cuenta de Media Services | 500 000 <sup>(2)</sup> (cantidad fija)|
| Listado de transformaciones|Pagine la respuesta, con 1000 transformaciones por página|
| Listado de trabajos|Pagine la respuesta, con 500 trabajos por página|
| Eventos en directo por cuenta de Media Services |5|
| Cuentas de Media Services en una suscripción única | 25 (fijo) |
| Live salidas por evento en directo |3 <sup>(3)</sup> |
| Duración de la salida de Live Max | 25 horas |
| Cuentas de almacenamiento | 100<sup>(4)</sup> (cantidad fija) |
| Puntos de conexión de streaming (detenidos o en ejecución) por cuenta de Media Services|2 (fijo)|
| Directivas de streaming | 100 <sup>(5)</sup> |
| Transformaciones por cuenta de Media Services | 100 (cantidad fija)|
| Localizadores de streaming únicos asociados con un recurso al mismo tiempo | 100<sup>(6)</sup> (fijo) |
| Opciones de directiva de clave de contenido |30 | 

<sup>1</sup> El tamaño máximo admitido para un único blob es actualmente de 5 TB en Azure Blob Storage. Se aplican límites adicionales en Media Services en función de los tamaños de máquina virtual que se usan por el servicio. El límite de tamaño se aplica a los archivos que se cargan y también los archivos que se generan como resultado de procesar (codificar o analizar) de Media Services. Si el archivo de origen es mayor de 260 GB, es muy probable que el trabajo presente un error. 

En la tabla siguiente se muestra los límites en el medio reservado unidades de S1, S2 y S3. Si el archivo de origen es mayor que los límites definidos en la tabla, se produce un error en el trabajo de codificación. Si utiliza la codificación orígenes de resolución de 4K de larga duración, es necesario utilizar unidades reservada de multimedia S3 para lograr el rendimiento necesario. Si tiene contenido de 4K mayor que el límite de 260 GB en las unidades reservadas de multimedia S3, póngase en contacto con nosotros en amshelp@microsoft.com para posibles mitigaciones admitir su escenario.

|Tipo de unidad reservada de medios   |Tamaño máximo de entrada (GB)|
|---|---|
|S1 |   26|
|S2 | 60|
|S3 |260|

<sup>2</sup> Este número incluye los trabajos en cola, terminados, activos y cancelados. No incluye los trabajos eliminados. 

Se eliminarán automáticamente los registros de trabajo de más de 90 días de su cuenta, aunque el número total de registros no llegue a la cuota máxima. 

<sup>3</sup> live salidas iniciar durante la creación y detener cuando se elimina.

<sup>4</sup> Las cuentas de almacenamiento deben proceder de la misma suscripción de Azure.

<sup>5</sup> al utilizar una personalizada [directiva Streaming](https://docs.microsoft.com/rest/api/media/streamingpolicies), debe diseñar un conjunto limitado de dichas directivas para su cuenta de Media Services y volver a usarlos para sus objetos Streaminglocator siempre que sea el mismo cifrado opciones y protocolos se necesitan. No debe crear una nueva directiva de streaming para cada localizador de streaming.

<sup>6</sup> localizadores de streaming no están diseñados para administrar el control de acceso por usuario. Para conceder derechos de acceso diferentes a usuarios individuales, use las soluciones de administración de derechos digitales (DRM).

## <a name="support-ticket"></a>Incidencia de soporte técnico

En el caso de recursos que no son fijos, puede solicitar que se generen las cuotas abriendo una [incidencia de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest). Incluya información detallada en la solicitud en los cambios de cuota que se desean, en los escenarios de casos de uso y las regiones que se requieren. <br/>**No** cree cuentas adicionales de Azure Media Services para obtener límites mayores.

## <a name="next-steps"></a>Pasos siguientes

[Información general](media-services-overview.md)
