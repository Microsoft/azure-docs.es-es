---
title: 'Cuotas de Live Video Analytics en IoT Edge: Azure'
description: En este artículo se describen las cuotas y limitaciones de Live Video Analytics en IoT Edge.
ms.topic: conceptual
ms.date: 05/22/2020
ms.openlocfilehash: 90141fa850c9ab3e3abbea15001249da0736ac45
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87091817"
---
# <a name="quotas-and-limitations"></a>Cuotas y limitaciones

En este artículo se enumeran las cuotas y limitaciones del módulo Live Video Analytics en IoT Edge.

## <a name="maximum-period-of-disconnected-use"></a>Período máximo de uso sin conexión

El módulo perimetral puede admitir la pérdida temporal de conectividad de red. Si el módulo permanece desconectado durante más de 36 horas, desactivará las instancias de grafos que estaban en ejecución y se bloquearán otras llamadas de método directas.

Para reanudar el estado operativo del módulo perimetral, tendrá que restaurar la conectividad de red y el módulo debe ser capaz de comunicarse correctamente con la cuenta de Azure Media Services.

## <a name="maximum-number-of-graph-instances"></a>Número máximo de instancias de grafos

Puede tener como máximo 1000 instancias de grafo por módulo (creadas a través de GraphInstanceSet).

## <a name="maximum-number-of-graph-topologies"></a>Número máximo de topologías de grafo

Puede tener como máximo 50 topologías de grafo por módulo (creadas a través de GraphTopologySet).

## <a name="limitations-on-graph-topologies-at-preview"></a>Limitaciones de las topologías de grafo en versión preliminar

Con la versión preliminar, hay limitaciones sobre los diferentes nodos que se pueden conectar entre sí en una topología de grafo multimedia.

* Origen RTSP
   * Solo se permite un origen RTSP por topología de grafo.
* Procesador de filtros de velocidad de fotogramas
   * Debe estar inmediatamente después del origen RTSP o el procesador de detección de movimiento.
   * No se puede usar antes que un procesador de extensión HTTP.
   * No puede estar antes que un procesador de detección de movimiento.
* Procesador de extensión HTTP
   * Como máximo puede haber un procesador por cada topología de grafo.
* Procesador de detección de movimiento
   * Debe estar inmediatamente después que el origen RTSP.
   * Como máximo puede haber un procesador por cada topología de grafo.
   * No se puede usar antes que un procesador de extensión HTTP.
* Procesador de la puerta de señales
   * Debe estar inmediatamente después que el origen RTSP.
* Receptor de recursos 
   * Debe estar inmediatamente después del origen RTSP o el procesador de la puerta de señales.
* Receptor de archivos
   * Debe estar inmediatamente después del procesador de la puerta de señales.
   * No puede estar inmediatamente después del procesador de extensión HTTP o del procesador de detección de movimiento
* Receptor de IoT Hub
   * No puede estar inmediatamente después de un origen de IoT Hub.

Si se usan nodos de procesador de detección de movimiento y tasa de filtrado, deben estar en la misma cadena que conduce al nodo de origen RTSP.

## <a name="limitations-on-media-service-operations-at-preview"></a>Limitaciones de las operaciones de Media Services en versión preliminar

En la versión preliminar, Live Video Analytics en IoT Edge no admite lo siguiente:

* La capacidad de migrar la cuenta de Media Services de una suscripción a otra sin interrupción.
* La capacidad de usar más de una cuenta de almacenamiento con la cuenta de Media Services.
* La capacidad de cambiar de forma dinámica la información de la entidad de servicio en las propiedades deseadas del módulo, sin necesidad de reiniciar.

## <a name="next-steps"></a>Pasos siguientes

[Información general](overview.md)
