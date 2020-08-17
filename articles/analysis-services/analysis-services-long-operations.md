---
title: Prácticas recomendadas para operaciones de larga duración en Azure Analysis Services | Microsoft Docs
description: En este artículo se describen los procedimientos recomendados para las operaciones de larga duración.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 04/14/2020
ms.author: owend
ms.openlocfilehash: 5195d63180d45ad408256264e3cf10174e55cd96
ms.sourcegitcommit: 1b2d1755b2bf85f97b27e8fbec2ffc2fcd345120
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87551955"
---
# <a name="best-practices-for-long-running-operations"></a>Procedimientos recomendados para las operaciones de larga duración

En Azure Analysis Services, un *nodo* representa una máquina virtual de host en la que se ejecuta un recurso de servidor. Algunas operaciones como las consultas de larga duración, las operaciones de actualización y la sincronización de escalado horizontal de consultas pueden producir errores si un recurso de servidor se mueve a otro nodo. Entre los mensajes de error comunes en este escenario se incluyen los siguientes:

- "Se produjo un error al tratar de localizar una solicitud XMLA de larga duración. Es posible que una actualización del servicio o reinicio del servidor haya interrumpido la solicitud".
- "El trabajo con id. ‘<guid> del modelo '<database>’ se canceló debido a un error de servicio (inactividad) con el mensaje ‘Se cancelará la solicitud de actualización, ya que se bloqueó sin actualizaciones. Este es un problema de servicio interno. Vuelva a enviar el trabajo o envíe un vale de soporte técnico para obtener ayuda si este problema se presenta varias veces".

Hay muchas razones por las que se pueden interrumpir las operaciones de larga duración. Por ejemplo, las actualizaciones de Azure como las siguientes: 
- Revisiones del sistema operativo 
- Actualizaciones de seguridad
- Actualizaciones del servicio Azure Analysis Services
- Actualizaciones de Service Fabric. Este un componente de plataforma que usan varios servicios en la nube de Microsoft, incluido Azure Analysis Services.

Además de las actualizaciones que se producen en el servicio, los servicios se mueven de forma natural entre los nodos debido al equilibrio de carga. Los movimientos de nodo son una parte esperada de los servicios en la nube. Azure Analysis Services intenta minimizar el impacto de los movimientos de nodo, aunque no es posible eliminarlos por completo. 

Además de los movimientos de nodo, se pueden producir otros errores. Por ejemplo, un sistema de base de datos de origen de datos podría estar sin conexión o podría perderse la conectividad de red. Si durante la actualización una partición tiene 10 millones de filas y se produce un error en la fila número 9 millones, no hay ninguna manera de reiniciar la actualización en el momento del error. El servicio tiene que volver a empezar desde el principio. 

## <a name="refresh-rest-api"></a>Actualización de la API REST

Las interrupciones del servicio pueden ser un desafío para las operaciones de larga duración, como la actualización de datos. Azure Analysis Services incluye una API REST para ayudarle a mitigar los impactos negativos de las interrupciones en el servicio. Para obtener más información, consulte [Actualización asincrónica con la API REST](analysis-services-async-refresh.md).
 
Además de la API REST, existen otros métodos que puede usar para minimizar los posibles problemas durante las operaciones de actualización de larga duración. El objetivo principal es evitar que sea necesario reiniciar la operación de actualización desde el principio y, en su lugar, que se realicen las actualizaciones en lotes más pequeños que se pueden confirmar en fases. 
 
La API REST permite este tipo de reinicio, pero no admite la flexibilidad total de la creación y eliminación de particiones. Si un escenario requiere operaciones complejas de administración de datos, la solución debe incluir algún tipo de procesamiento por lotes en su lógica. Por ejemplo, al usar transacciones para procesar los datos en varios lotes distintos, permite que un error no requiera reiniciar el proceso desde el principio; en su lugar, se hace desde un punto de control intermedio. 
 
## <a name="scale-out-query-replicas"></a>Escalabilidad horizontal de las réplicas de consultas

Tanto si se usa REST o una lógica personalizada, las consultas de aplicación cliente pueden devolver resultados incoherentes o intermedios durante el procesamiento de los lotes. Si se requiere que las consultas de la aplicación cliente devuelvan datos coherentes durante el procesamiento y los datos del modelo están en un estado intermedio, puede usar la [escalabilidad horizontal](analysis-services-scale-out.md) con réplicas de consulta de solo lectura.

Mediante el uso de réplicas de consulta de solo lectura, aunque se estén realizando actualizaciones en los lotes, los usuarios de la aplicación cliente pueden seguir consultando la instantánea anterior de los datos desde las réplicas de solo lectura. Una vez finalizadas las actualizaciones, se puede realizar una operación de sincronización para actualizar las réplicas de solo lectura.


## <a name="next-steps"></a>Pasos siguientes

[Actualización asincrónica con la API de REST](analysis-services-async-refresh.md)  
[Escalabilidad horizontal de Azure Analysis Services](analysis-services-scale-out.md)  
[Alta disponibilidad de Analysis Services](analysis-services-bcdr.md)  
[Guía de reintentos para los servicios de Azure](https://docs.microsoft.com/azure/architecture/best-practices/retry-service-specific)   

