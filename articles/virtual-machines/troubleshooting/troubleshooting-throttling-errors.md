---
title: Solución de errores de limitación en Azure | Microsoft Docs
description: Errores de limitación, reintentos y retrocesos de Azure Compute.
services: virtual-machines
documentationcenter: ''
author: changov
manager: gwallace
editor: ''
tags: azure-resource-manager,azure-service-management
ms.service: virtual-machines
ms.topic: troubleshooting
ms.workload: infrastructure-services
ms.date: 09/18/2018
ms.author: changov
ms.reviewer: vashan, rajraj
ms.openlocfilehash: b1cc8a43423ecd33218948aaa001fc34877eac60
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87074285"
---
# <a name="troubleshooting-api-throttling-errors"></a>Solución de problemas de errores de limitación de API 

Las solicitudes de Azure Compute pueden limitarse en una suscripción y en una región para favorecer el rendimiento general del servicio. Garantizamos que todas las llamadas realizadas al proveedor de recursos de Azure Compute (CRP), que administra los recursos del espacio de nombres Microsoft.Compute, no superan la frecuencia de solicitudes de API máxima permitida. En este documento, se describe la limitación de las API, se explica cómo solucionar los problemas de limitación y se detallan procedimientos recomendados para evitar las limitaciones.  

## <a name="throttling-by-azure-resource-manager-vs-resource-providers"></a>Limitaciones efectuadas por Azure Resource Manager frente a limitaciones efectuadas por proveedores de recursos  

Al actuar como puerta de entrada de Azure, Azure Resource Manager se encarga de la autenticación, la validación de primer orden y la limitación de todas las solicitudes de API. Los límites de frecuencia de llamada de Azure Resource Manager y los encabezados HTTP de respuesta de diagnóstico relacionados se describen [aquí](../../azure-resource-manager/management/request-limits-and-throttling.md).
 
Cuando se produce un error de limitación en un cliente de API de Azure, el estado HTTP es "429 Demasiadas solicitudes". Para comprender si la limitación de solicitudes se está realizando mediante Azure Resource Manager o un proveedor de recursos subyacente, como CRP, inspeccione las solicitudes GET de `x-ms-ratelimit-remaining-subscription-reads` y las solicitudes que no son GET de `x-ms-ratelimit-remaining-subscription-writes`. Si el número de llamadas restante está próximo a 0, significa que se ha alcanzado el límite general de llamadas de la suscripción definido por Azure Resource Manager. Las actividades de todos los clientes de la suscripción se cuentan de forma conjunta. De lo contrario, la limitación procede del proveedor de recursos de destino (el especificado por el segmento `/providers/<RP>` de la dirección URL de la solicitud). 

## <a name="call-rate-informational-response-headers"></a>Encabezados de respuesta que informan sobre la frecuencia de llamadas 

| Encabezado                            | Formato del valor                           | Ejemplo                               | Descripción                                                                                                                                                                                               |
|-----------------------------------|----------------------------------------|---------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| x-ms-ratelimit-remaining-resource |```<source RP>/<policy or bucket>;<count>```| Microsoft.Compute/HighCostGet3Min;159 | Número restantes de llamadas API de la directiva de limitación que se aplica al cubo de recursos o al grupo de operaciones que contiene el destino de esta solicitud.                                                                   |
| x-ms-request-charge               | ```<count>```                             | 1                                     | Número de recuentos de llamadas "cargados" en esta solicitud HTTP en relación con el límite de la directiva aplicable. Suele ser 1. Las solicitudes por lotes, como las solicitudes para escalar un conjunto de escalado de máquinas virtuales, pueden "cargar" varios recuentos. |


Tenga en cuenta que las solicitudes de API pueden estar sujetas a varias directivas de limitación. Cada directiva tendrá un encabezado `x-ms-ratelimit-remaining-resource` diferente. 

A continuación, se muestra una respuesta de ejemplo para eliminar una solicitud de un conjunto de escalado de máquinas virtuales.

```
x-ms-ratelimit-remaining-resource: Microsoft.Compute/DeleteVMScaleSet3Min;107 
x-ms-ratelimit-remaining-resource: Microsoft.Compute/DeleteVMScaleSet30Min;587 
x-ms-ratelimit-remaining-resource: Microsoft.Compute/VMScaleSetBatchedVMRequests5Min;3704 
x-ms-ratelimit-remaining-resource: Microsoft.Compute/VmssQueuedVMOperations;4720 
```

## <a name="throttling-error-details"></a>Información sobre los errores de limitación

El código de estado HTTP 429 normalmente se utiliza para rechazar una solicitud porque se alcanza el límite de frecuencia de llamadas. A continuación, se muestra el aspecto que tendría una respuesta de error de limitación típica del proveedor de recursos de Azure Compute (solo se muestran los encabezados pertinentes):

```
HTTP/1.1 429 Too Many Requests
x-ms-ratelimit-remaining-resource: Microsoft.Compute/HighCostGet3Min;46
x-ms-ratelimit-remaining-resource: Microsoft.Compute/HighCostGet30Min;0
Retry-After: 1200
Content-Type: application/json; charset=utf-8
{
  "code": "OperationNotAllowed",
  "message": "The server rejected the request because too many requests have been received for this subscription.",
  "details": [
    {
      "code": "TooManyRequests",
      "target": "HighCostGet30Min",
      "message": "{\"operationGroup\":\"HighCostGet30Min\",\"startTime\":\"2018-06-29T19:54:21.0914017+00:00\",\"endTime\":\"2018-06-29T20:14:21.0914017+00:00\",\"allowedRequestCount\":800,\"measuredRequestCount\":1238}"
    }
  ]
}

```

La directiva en la que el número de llamadas restante es 0 es la responsable de que se produzca el error de limitación. En este caso, es el error `HighCostGet30Min`. El formato global del cuerpo de la respuesta es el formato de error general de Azure Resource Manager API (conforme con OData). El código de error principal, `OperationNotAllowed`, es el que el proveedor de recursos de Azure Compute utiliza para notificar los errores de limitación (entre otros tipos de errores de cliente). La propiedad `message` de los errores internos contiene una estructura JSON serializada con los detalles de la violación de la limitación.

Tal y como se ilustra arriba, todos los errores de limitación contienen el encabezado `Retry-After`, donde se especifica el número mínimo de segundos que el cliente debe esperar antes de reintentar la solicitud. 

## <a name="api-call-rate-and-throttling-error-analyzer"></a>Frecuencia de llamadas API y analizador de errores de limitación
Hay una versión preliminar de una característica de solución de problemas disponible para la API del proveedor de recursos de Compute. Estos cmdlets de PowerShell proporcionan estadísticas sobre la frecuencia de solicitudes de la API por intervalo de tiempo en función de la operación, así como infracciones de limitación según el grupo de operación (directiva):
-   [Export-AzLogAnalyticRequestRateByInterval](/powershell/module/az.compute/export-azloganalyticrequestratebyinterval)
-   [Export-AzLogAnalyticThrottledRequest](/powershell/module/az.compute/export-azloganalyticthrottledrequest)

Las estadísticas de la llamada API pueden proporcionar conclusiones valiosas sobre el comportamiento de los clientes de la suscripción y permiten identificar fácilmente los patrones de llamada que provocan una limitación.

Por el momento, una limitación del analizador es que no cuenta las solicitudes para los tipos de recursos de discos e instantáneas (esto beneficia a los discos administrados). Puesto que recopila datos de telemetría de CRP, tampoco puede ayudar a identificar los errores de limitación de ARM. Sin embargo, estos se pueden identificar fácilmente en función de los encabezados de respuesta de ARM específicos, tal y como se explicó anteriormente.

Los cmdlets de PowerShell usan una API REST de servicio, a la que los clientes pueden llamar de forma fácil y directa (aunque aún no se admite formalmente). Para ver el formato de solicitud HTTP, ejecute los cmdlets con el conmutador -Debug o inspecciona su ejecución con Fiddler.


## <a name="best-practices"></a>Procedimientos recomendados 

- No reintente los errores de las API de servicio de Azure incondicionalmente o inmediatamente. Suele ocurrir que el código del cliente entra en un bucle de reintentos rápidos cuando encuentra un error que no se puede reintentar. Los reintentos terminarán por agotar el límite de llamadas del grupo de la operación de destino, lo que afectará a otros clientes de la suscripción. 
- En los casos de automatización de API de gran volumen, considere la posibilidad de implementar una autolimitación proactiva del lado cliente cuando el recuento de llamadas disponible del grupo de una operación de destino caiga por debajo de algún umbral inferior. 
- Cuando realice el seguimiento de las operaciones asincrónicas, respete las sugerencias de los encabezados Retry-After. 
- Si el código del cliente necesita información acerca de una máquina virtual determinada, consulte dicha máquina virtual directamente, en lugar de mostrar todas las máquinas virtuales del grupo de recursos correspondiente o de toda la suscripción y seleccionar la máquina virtual apropiada en el lado cliente. 
- Si el código del cliente necesita máquinas virtuales, discos e instantáneas de una ubicación concreta de Azure, utilice el formulario basado en ubicación de la consulta, en lugar de consultar todas las máquinas virtuales de la suscripción y filtrarlas después por su ubicación en el lado cliente: utilice `GET /subscriptions/<subId>/providers/Microsoft.Compute/locations/<location>/virtualMachines?api-version=2017-03-30` para consultar los puntos de conexión regionales del proveedor de recursos de Azure Compute. 
-   Cuando crea o actualiza recursos de API específicos, máquinas virtuales y conjuntos de escalado de máquinas virtuales, es mucho más eficaz hacer un seguimiento de la operación asincrónica que se devuelve hasta su finalización que sondear la propia dirección URL del recurso (que se basa en `provisioningState`).

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la guía de reintentos de otros servicios de Azure, consulte [Guía de reintentos para servicios específicos](/azure/architecture/best-practices/retry-service-specific).
