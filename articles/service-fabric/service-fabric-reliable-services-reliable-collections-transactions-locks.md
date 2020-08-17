---
title: Transacciones y modos de bloqueo en las colecciones de confianza
description: Transacciones y registros de Reliable State Manager y Reliable Collections de Azure Service Fabric.
ms.topic: conceptual
ms.date: 5/1/2017
ms.custom: sfrev
ms.openlocfilehash: d1094462ebabcea1fbead3d5b30fdfb8dda6463a
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87500289"
---
# <a name="transactions-and-lock-modes-in-azure-service-fabric-reliable-collections"></a>Transacciones y modos de bloqueo en Reliable Collections de Azure Service Fabric

## <a name="transaction"></a>Transacción

Una transacción es una secuencia de operaciones realizadas como una sola unidad lógica de trabajo. Exhibe las propiedades comunes de [ACID](https://en.wikipedia.org/wiki/ACID) (*atomicidad*, *consistencia*, *aislamiento* y *durabilidad*) de las transacciones de base de datos:

* **Atomicidad**: una transacción debe ser una unidad atómica de trabajo. Es decir, o se modifican todos sus datos o ninguno de ellos.
* **Coherencia**: Cuando finaliza, una transacción debe dejar todos los datos en un estado coherente. Todas las estructuras de datos internos deben ser correctas al final de la transacción.
* **Aislamiento**: Las modificaciones realizadas por transacciones simultáneas se deben aislar de las modificaciones llevadas a cabo por otras transacciones simultáneas. El nivel de aislamiento usado para una operación en una transacción [ITransaction](/dotnet/api/microsoft.servicefabric.data.itransaction?view=azure-dotnet) lo determina el [elemento IReliableState](/dotnet/api/microsoft.servicefabric.data.ireliablestate?view=azure-dotnet) que realiza la operación.
* **Durabilidad**: una vez concluida una transacción, sus efectos son permanentes en el sistema. Las modificaciones persisten aún en el caso de producirse un error del sistema.

### <a name="isolation-levels"></a>Niveles de aislamiento

Un nivel de aislamiento define el grado en el que debe aislarse la transacción de las modificaciones que realicen otras transacciones.
Hay dos niveles de aislamiento que se admiten en Colecciones confiables:

* **Lectura repetible**: especifica que las instrucciones no puedan leer datos que se hayan modificado, pero que otras transacciones aún no han confirmado y que ninguna otra puede modificar los datos leídos por la actual hasta que esta finalice.
* **Instantánea**: especifica que los datos que ha leído cualquier instrucción de una transacción sean la versión coherente, desde el punto de vista transaccional, de los datos existentes al comienzo de la transacción.
  La transacción solo puede reconocer las modificaciones de datos que se confirmaron antes del inicio de la transacción.
  Las instrucciones que se ejecuten en la transacción actual no verán las modificaciones de datos efectuadas por otras transacciones después del inicio de la transacción actual.
  El efecto es como si las instrucciones de una transacción obtienen una instantánea de los datos confirmados tal como se encontraban al inicio de la transacción.
  Las instantáneas son coherentes entre colecciones confiables.

Colecciones confiables elige automáticamente el nivel de aislamiento que se usará para una operación de lectura determinada dependiendo del funcionamiento y el rol de la réplica.
A continuación, se encuentra la tabla que muestra los valores predeterminados de nivel de aislamiento para las operaciones Diccionario confiable y Cola confiable.

| Operación\rol | Principal | Secundario |
| --- |:--- |:--- |
| Lectura de una sola entidad |Repeatable Read |Instantánea |
| Enumeración, recuento |Instantánea |Instantánea |

> [!NOTE]
> Algunos ejemplos comunes de las operaciones de entidad única son `IReliableDictionary.TryGetValueAsync` y `IReliableQueue.TryPeekAsync`.
> 

Tanto el diccionario de confianza como la cola de confianza admiten la función de *lectura de lo que se escribe*.
En otras palabras, cualquier escritura dentro de una transacción será visible en una lectura posterior que pertenezca a la misma transacción.

## <a name="locks"></a>Bloqueos

En Reliable Collections, todas las transacciones implementan un riguroso bloqueo en dos fases: una transacción no libera los bloqueos que ha adquirido hasta que la transacción finaliza con una anulación o confirmación.

El diccionario de confianza usa el bloqueo de nivel de fila para todas las operaciones de las entidades individuales.
Cola confiable equilibra la simultaneidad para la propiedad FIFO transaccional estricta.
La cola de confianza utiliza bloqueos de nivel de operación que permiten una transacción con `TryPeekAsync` o `TryDequeueAsync`, y una transacción con `EnqueueAsync` a la vez.
Tenga en cuenta que para conservar FIFO, si `TryPeekAsync` o `TryDequeueAsync` nunca observa que la Cola confiable está vacía, bloqueará también `EnqueueAsync`.

Las operaciones de escritura siempre tiene bloqueos exclusivos.
En el caso de las operaciones de lectura, el bloqueo depende de dos factores:

- Las operaciones de lectura realizadas mediante el aislamiento de instantáneas no tienen bloqueos.
- Cualquier operación Repeatable Read usa bloqueos compartidos de manera predeterminada.
- Sin embargo, para cualquier operación de lectura que admita Repeatable Read, el usuario puede solicitar un bloqueo de actualización en lugar del bloqueo compartido.
El bloqueo de actualización es un bloqueo asimétrico que se isa para evitar una forma común de interbloqueo que se produce cuando varias transacciones bloquean recursos para una posible actualización en un momento posterior.

La matriz de compatibilidad de bloqueo se puede encontrar en la tabla siguiente:

| Solicitar \ concedido | None | Compartido | Actualizar | Exclusivo |
| --- |:--- |:--- |:--- |:--- |
| Compartido |Ningún conflicto |Ningún conflicto |Conflicto |Conflicto |
| Actualizar |Ningún conflicto |Ningún conflicto |Conflicto |Conflicto |
| Exclusivo |Ningún conflicto |Conflicto |Conflicto |Conflicto |

El argumento de tiempo de expiración de las API de Reliable Collections se usa como una detección de interbloqueo.
Por ejemplo, dos transacciones (T1 y T2) están intentando leer y actualizar K1.
Es posible que se interbloqueen, porque ambas acaban disponiendo de bloqueo compartido.
En este caso, una o ambas operaciones superará el tiempo de espera. En este escenario, un bloqueo de actualización podría impedir dicho interbloqueo.

## <a name="next-steps"></a>Pasos siguientes

* [Trabajo con Reliable Collections](service-fabric-work-with-reliable-collections.md)
* [Notificaciones de Reliable Services](service-fabric-reliable-services-notifications.md)
* [Copia de seguridad y restauración de Reliable Services (recuperación ante desastres)](service-fabric-reliable-services-backup-restore.md)
* [Configuración del administrador de estado confiable](service-fabric-reliable-services-configuration.md)
* [Referencia para desarrolladores de colecciones confiables](/dotnet/api/microsoft.servicefabric.data.collections?view=azure-dotnet#microsoft_servicefabric_data_collections)
