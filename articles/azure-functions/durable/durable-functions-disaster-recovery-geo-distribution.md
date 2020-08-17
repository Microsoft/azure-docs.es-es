---
title: Recuperación ante desastres y distribución geográfica en Azure Durable Functions
description: Información acerca de la recuperación ante desastres y la distribución geográfica en Durable Functions.
author: MS-Santi
ms.topic: conceptual
ms.date: 04/25/2018
ms.author: azfuncdf
ms.openlocfilehash: 1bddb6af777f36d04bae4a8b68c476789048be1b
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87081787"
---
# <a name="disaster-recovery-and-geo-distribution-in-azure-durable-functions"></a>Recuperación ante desastres y distribución geográfica en Azure Durable Functions

En Durable Functions, todos los estados se conservan en Azure Storage. Una [central de tareas](durable-functions-task-hubs.md) es un contenedor lógico para los recursos de Azure Storage que se usan para las orquestaciones. Las funciones de orquestador y actividad solo pueden interactuar entre sí si pertenecen a la misma central de tareas.
Los escenarios descritos proponen opciones de implementación para aumentar la disponibilidad y minimizar el tiempo de inactividad durante las actividades de recuperación ante desastres.

Es importante tener en cuenta que estos escenarios se basan en configuraciones activas/pasivas, ya que se guían por Azure Storage. Este patrón consiste en implementar una aplicación de función de copia de seguridad (pasiva) en otra región. Traffic Manager supervisará la disponibilidad de la aplicación de función principal (activa). Si se produce un error en la principal, conmutará por error a la aplicación de función de copia de seguridad. Para más información, consulte [Método de enrutamiento de tráfico Prioridad](../../traffic-manager/traffic-manager-routing-methods.md#priority-traffic-routing-method) de [Traffic Manager](https://azure.microsoft.com/services/traffic-manager/).

>[!NOTE]
>
> - La configuración activa/pasiva propuesta garantiza que el cliente siempre puede desencadenar nuevas orquestaciones a través de HTTP. Sin embargo, como consecuencia de tener dos funciones de aplicación que comparten almacenamiento, el procesamiento en segundo plano se distribuye entre ambos, por lo que compiten por los mensajes de las mismas colas. Esta configuración implica costos de salida agregados a la aplicación de función secundaria.
> - La cuenta de almacenamiento subyacente y la central de tareas se crean en la región primaria, y las comparten ambas aplicaciones de función.
> - Todas las aplicaciones de función que se implementan de forma redundante deben compartir las claves de acceso de la función en caso de que se activen a través de HTTP. Functions Runtime expone una [API de administración](https://github.com/Azure/azure-functions-host/wiki/Key-management-API) que permite a los consumidores agregar, eliminar y actualizar las teclas de función mediante programación.

## <a name="scenario-1---load-balanced-compute-with-shared-storage"></a>Escenario 1: proceso con almacenamiento compartido y equilibrio de carga

Si se produce un error en la infraestructura de proceso de Azure, la aplicación de función podría no estar disponible. Para minimizar este riesgo de tiempo de inactividad, en este escenario se utilizan dos aplicaciones de función implementadas en regiones diferentes.
Traffic Manager está configurado para detectar problemas en la aplicación de función principal y redirigir el tráfico automáticamente a la aplicación de función de la región secundaria. Esta aplicación de función comparte la cuenta de Azure Storage y la central de tareas. Por lo tanto, el estado de las aplicaciones de función no se pierde y el trabajo se reinicia con normalidad. Una vez restaurado el estado en la región primaria, Azure Traffic Manager empezará automáticamente a enrutar las solicitudes a esa aplicación de función.

![Diagrama que muestra el escenario 1.](./media/durable-functions-disaster-recovery-geo-distribution/durable-functions-geo-scenario01.png)

Este escenario de implementación supone varias ventajas:

- Si se produce un error en la infraestructura de proceso, el trabajo se puede reanudar en la región donde se conmutó por error sin pérdida de estado.
- Traffic Manager se encarga de la conmutación por error a la aplicación de función correcta automáticamente.
- Traffic Manager restablece automáticamente el tráfico a la aplicación de función principal una vez corregida la interrupción.

Sin embargo, con este escenario, valore:

- Si la aplicación de función se implementó mediante un plan de App Service específico, la replicación de la infraestructura de proceso en el centro de datos de conmutación por error aumenta los costos.
- En este escenario se tratan las interrupciones en la infraestructura de proceso, pero la cuenta de Storage sigue siendo el único punto de error de la función de aplicación. Si se produce una interrupción en Storage, la aplicación sufre tiempo de inactividad.
- Si la aplicación de función se conmuta por error, la latencia aumentará, ya que se accederá a las cuentas de Storage de las distintas regiones.
- Acceder al servicio de almacenamiento desde una región distinta a donde se ubica implica mayores costos debido al tráfico de salida de la red.
- Este escenario depende de Traffic Manager. Teniendo en cuenta [cómo funciona Traffic Manager](../../traffic-manager/traffic-manager-how-it-works.md), una aplicación cliente que consuma una instancia de Durable Functions puede tardar un tiempo en necesitar consultar de nuevo la dirección de la aplicación de función de Traffic Manager.

## <a name="scenario-2---load-balanced-compute-with-regional-storage"></a>Escenario 2: proceso con almacenamiento regional y equilibrio de carga

En el escenario anterior se cubre solo el caso de error en la infraestructura de proceso. Si se produce un error en el servicio de almacenamiento, se producirá una interrupción en la aplicación de función.
Para garantizar el funcionamiento continuo de Durable Functions, para este escenario se utiliza una cuenta de almacenamiento local en cada región donde se implementan las aplicaciones de función.

![Diagrama que muestra el escenario 2.](./media/durable-functions-disaster-recovery-geo-distribution/durable-functions-geo-scenario02.png)

Este método agrega mejoras al escenario anterior:

- Si se produce un error en la aplicación de función, Traffic Manager se encarga de la conmutación por error a la región secundaria. Sin embargo, dado que la aplicación de función se basa en su propia cuenta de almacenamiento, Durable Functions continuará funcionando.
- Durante la conmutación por error no hay latencia adicional en la región donde se realiza, ya que la aplicación de función y la cuenta de almacenamiento se ubican en el mismo sitio.
- Un error de la capa de almacenamiento provocará errores en Durable Functions que, a su vez, desencadenarían el redireccionamiento a la región de conmutación por error. Una vez más, como la aplicación de función y el almacenamiento están aislados por región, Durable Functions continúa funcionando.

Consideraciones importantes sobre este escenario:

- Si la aplicación de función se implementó mediante un plan de App Service específico, la replicación de la infraestructura de proceso en el centro de datos de conmutación por error aumenta los costos.
- El estado actual no se conmuta por error, lo cual implica que las funciones en ejecución y en el punto de control producirán un error. La aplicación cliente decide reiniciar el trabajo o volver a intentarlo.

## <a name="scenario-3---load-balanced-compute-with-grs-shared-storage"></a>Escenario 3: proceso con almacenamiento con redundancia geográfica compartido y equilibrio de carga

Este escenario es una modificación del primer escenario en el que se implementa una cuenta de almacenamiento compartido. La diferencia principal es que la cuenta de almacenamiento se crea con la replicación geográfica habilitada.
Funcionalmente, este escenario proporciona las mismas ventajas que el número 1, pero incorpora ventajas de recuperación de datos adicionales:

- El almacenamiento con redundancia geográfica (GRS) y el almacenamiento con redundancia geográfica y acceso de lectura (RA-GRS) maximizan la disponibilidad de la cuenta de almacenamiento.
- Si se produce una interrupción en la región del servicio de almacenamiento, una de las posibilidades es que las operaciones del centro de datos determinen qué almacenamiento debe conmutar por error a la región secundaria. En este caso, el acceso a la cuenta de almacenamiento se redirigirá de forma transparente a la copia de replicación geográfica de la cuenta de almacenamiento, sin intervención del usuario.
- En este caso, el estado de Durable Functions se conservará hasta la última replicación de la cuenta de almacenamiento, que se produce cada pocos minutos.

Al igual que con los demás escenarios, hay consideraciones importantes:

- La conmutación por error a la réplica se realiza mediante operadores del centro de datos y puede tardar un tiempo. Hasta ese momento, la aplicación de función sufrirá una interrupción del servicio.
- El uso de cuentas de almacenamiento con replicación geográfica no aumenta el costo.
- El almacenamiento con redundancia geográfica se produce de forma asincrónica. Algunas de las últimas transacciones pueden perderse a causa de la latencia del proceso de replicación.

![Diagrama que muestra el escenario 3.](./media/durable-functions-disaster-recovery-geo-distribution/durable-functions-geo-scenario03.png)

## <a name="next-steps"></a>Pasos siguientes

Más información sobre el [Diseño de aplicaciones de alta disponibilidad mediante RA-GRS](../../storage/common/geo-redundant-design.md)
