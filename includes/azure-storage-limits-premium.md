---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 03/23/2019
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 7b75e24e5d59206ee8330c3ca9eaf86eacbad13d
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2019
ms.locfileid: "65751514"
---
### <a name="premium-performance-block-blob-storage"></a>Almacenamiento de blobs de bloque de rendimiento Premium

Una cuenta de almacenamiento de blobs de premium rendimiento bloque está optimizada para las aplicaciones que usan más pequeñas, intervalo de kilobytes, objetos. Es ideal para aplicaciones que requieren altas de transacciones o almacenamiento de baja latencia coherente. Almacenamiento de blobs de bloque de rendimiento Premium está diseñado para escalar con sus aplicaciones. Si va a implementar las aplicaciones que requieren cientos de miles de solicitudes por segundo o petabytes de capacidad de almacenamiento, póngase en contacto con nosotros enviando una solicitud de soporte técnico en el [portal Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

### <a name="premium-performance-filestorage-preview"></a>Rendimiento de Premium FileStorage (versión preliminar)

Archivos Premium usan una cuenta de almacenamiento único llamada **FileStorage (versión preliminar)**. Este tipo de cuenta está diseñado para cargas de trabajo con una IOPS elevada, alto rendimiento, baja latencia coherente. Almacenamiento de archivos Premium se escala con el tamaño del recurso compartido aprovisionado.

|Área  |Destino  |
|---------|---------|
|Max aprovisionado tamaño     |100 TB     |
|Recursos compartidos   |Sin límite  |
|E/S     |100 000    |
|Entrada|4,136 MiB/s     |
|Salida|6,204 MiB/s |

 Archivo premium comparten objetivos de escalabilidad, consulte el [archivos Premium escalar destinos](../articles/storage/common/storage-scalability-targets.md#premium-files-scale-targets) sección.

### <a name="premium-performance-page-blob-storage"></a>Almacenamiento de blobs de página de rendimiento Premium

Rendimiento de Premium, uso general v1 o v2 cuentas de almacenamiento tienen los siguientes objetivos de escalabilidad:

| Capacidad total de la cuenta                            | Ancho de banda total de una cuenta de almacenamiento con redundancia local                     |
| ------------------------------------------------- | --------------------------------------------------------------------------- |
| Capacidad de disco: 35 TB <br>Capacidad de instantánea: 10 TB | Hasta 50 gigabits por segundo de entrada<sup>1</sup> y salida<sup>2</sup> |

<sup>1</sup> Todos los datos (solicitudes) que se envían a una cuenta de almacenamiento

<sup>1</sup> Todos los datos (respuestas) que se reciben desde una cuenta de almacenamiento

Si usas cuentas de rendimiento de premium storage para discos no administrados y la aplicación supera los objetivos de escalabilidad de una sola cuenta de almacenamiento, es posible que desee migrar a managed disks. Si no desea migrar a Managed Disks, compile la aplicación para que use varias cuentas de almacenamiento. A continuación, divida los datos en esas cuentas de almacenamiento. Por ejemplo, si desea asociar discos de 51 TB entre varias VM, distribúyalos entre dos cuentas de almacenamiento. 35 TB es el límite de una cuenta de Premium Storage única. Asegúrese de que una cuenta de almacenamiento premium solo rendimiento nunca tiene más de 35 TB de discos aprovisionados.