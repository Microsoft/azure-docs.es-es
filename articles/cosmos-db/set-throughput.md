---
title: Aprovisionamiento del rendimiento en contenedores y bases de datos de Azure Cosmos
description: Aprenda cómo establecer el rendimiento aprovisionado para las bases de datos y los contenedores de Azure Cosmos.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 08/19/2020
ms.openlocfilehash: 00ed8f6ff9839c227f3d8a929a071834c5559226
ms.sourcegitcommit: d661149f8db075800242bef070ea30f82448981e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88605739"
---
# <a name="introduction-to-provisioned-throughput-in-azure-cosmos-db"></a>Introducción al rendimiento aprovisionado en Azure Cosmos DB

Azure Cosmos DB le permite establecer el rendimiento aprovisionado en las bases de datos y contenedores. Hay dos tipos de rendimiento aprovisionado, estándar (manual) o de escalabilidad automática. En este artículo se proporciona información general sobre cómo funciona el rendimiento aprovisionado. 

Una base de datos de Azure Cosmos es una unidad de administración de un conjunto de contenedores. Una base de datos consta de un conjunto de contenedores independiente del esquema. Un contenedor de Azure Cosmos es la unidad de escalabilidad de rendimiento y almacenamiento. Un contenedor se divide horizontalmente en un conjunto de máquinas dentro de una región de Azure y se distribuye en todas las regiones de Azure asociadas a su cuenta de Azure Cosmos.

En Azure Cosmos DB, puede aprovisionar el rendimiento en dos granularidades:
 
- Contenedores de Azure Cosmos
- Bases de datos de Azure Cosmos

## <a name="set-throughput-on-a-container"></a>Establecimiento del rendimiento en un contenedor  

El rendimiento aprovisionado en un contenedor de Azure Cosmos se reserva exclusivamente para ese contenedor. El contenedor recibe el rendimiento aprovisionado todo el tiempo. El rendimiento aprovisionado en un contenedor está respaldado financieramente por los contratos de nivel de servicio. Para obtener información sobre cómo configurar el rendimiento estándar (manual) en un contenedor, consulte [Aprovisionamiento del rendimiento en un contenedor de Azure Cosmos](how-to-provision-container-throughput.md). Para obtener información sobre cómo configurar el rendimiento de escalabilidad automática en un contenedor, consulte [Aprovisionamiento del rendimiento de escalabilidad automática](how-to-provision-autoscale-throughput.md).

La configuración del rendimiento aprovisionado en un contenedor es la opción más utilizada. Puede escalar elásticamente el rendimiento de un contenedor con el aprovisionamiento de cualquier cantidad de rendimiento mediante el uso de [Unidades de solicitud (RU)](request-units.md). 

El rendimiento aprovisionado de un contenedor se distribuye uniformemente entre las particiones físicas y, suponiendo una buena clave de partición que distribuye las particiones lógicas uniformemente entre las particiones físicas, el rendimiento también se distribuye uniformemente entre todas las particiones lógicas del contenedor. No puede especificar de forma selectiva el rendimiento de las particiones lógicas. Puesto que una o varias particiones lógicas de un contenedor se hospedan en una partición física, las particiones físicas pertenecen exclusivamente al contenedor y admiten el rendimiento aprovisionado en dicho contenedor. 

Si la carga de trabajo que se ejecuta en una partición lógica consume más que el rendimiento que se asignó a la partición física subyacente, es posible que las operaciones tengan una velocidad limitada. Lo que se conoce como _partición activa_ ocurre cuando una partición lógica tiene de manera desproporcionada más solicitudes que otros valores de clave de partición.

Cuando se produce una limitación de velocidad, puede aumentar el rendimiento aprovisionado de todo el contenedor o volver a intentar las operaciones. También debe asegurarse de elegir una clave de partición que distribuya de manera uniforme el volumen de la solicitud y el almacenamiento. Para más información sobre la creación de particiones, consulte [Creación de particiones y escalado horizontal en Azure Cosmos DB](partition-data.md).

Se recomienda configurar el rendimiento en la granularidad del contenedor cuando se quiere un rendimiento garantizado para dicho contenedor.

En la imagen siguiente se muestra cómo una partición física hospeda una o varias particiones lógicas de un contenedor:

:::image type="content" source="./media/set-throughput/resource-partition.png" alt-text="Partición física" border="false":::

## <a name="set-throughput-on-a-database"></a>Establecimiento del rendimiento en una base de datos

> [!NOTE]
> Actualmente no es posible aprovisionar el rendimiento en una base de datos de Azure Cosmos en cuentas en las que se habilitan las [claves administradas por el cliente](how-to-setup-cmk.md).

Al aprovisionar el rendimiento en una base de datos de Azure Cosmos, dicho rendimiento se comparte entre todos los contenedores (denominados contenedores de base de datos compartidos) de la base de datos. Una excepción es si ha especificado un rendimiento aprovisionado en determinados contenedores de la base de datos. Compartir el rendimiento aprovisionado de nivel de base de datos entre sus contenedores equivale a hospedar una base de datos en un clúster de máquinas. Dado que todos los contenedores de una base de datos comparten los recursos disponibles en una máquina, no obtendrá un rendimiento predecible en ningún contenedor específico. Para obtener información sobre cómo configurar el rendimiento aprovisionado en una base de datos, consulte [Aprovisionamiento del rendimiento en una base de datos de Azure Cosmos](how-to-provision-database-throughput.md). Para obtener información sobre cómo configurar el rendimiento de escalabilidad automática en una base de datos, consulte [Aprovisionamiento del rendimiento de escalabilidad automática](how-to-provision-autoscale-throughput.md).

La configuración del rendimiento en una base de datos de Azure Cosmos garantiza la recepción del rendimiento aprovisionado para esa base de datos todo el tiempo. Dado que todos los contenedores de la base de datos comparten el rendimiento aprovisionado, Azure Cosmos DB no proporciona ninguna garantía de rendimiento predecible para un contenedor determinado de dicha base de datos. La parte del rendimiento que puede recibir un contenedor específico depende de los siguientes factores:

* El número de contenedores.
* La elección de las claves de partición de varios contenedores.
* La distribución de la carga de trabajo en varias particiones lógicas de los contenedores. 

Se recomienda configurar el rendimiento en una base de datos si quiere compartir el rendimiento entre varios contenedores, pero no dedicarlo a ningún contenedor específico. 

En los siguientes ejemplos se muestra dónde es preferible aprovisionar el rendimiento en el nivel de base de datos:

* El uso compartido del rendimiento aprovisionado de una base de datos a través de un conjunto de contenedores es útil para una aplicación de varios inquilinos. Cada usuario puede representarse mediante un contenedor de Azure Cosmos diferente.

* El uso compartido del rendimiento aprovisionado de una base de datos a través de un conjunto de contenedores es útil al migrar una base de datos de NoSQL, como MongoDB o Cassandra, hospedada en un clúster de VM o desde servidores físicos locales a Azure Cosmos DB. Puede considerar el rendimiento aprovisionado configurado en la base de datos de Azure Cosmos como un equivalente lógico, pero más rentable y flexible, de la capacidad de proceso del clúster de MongoDB o Cassandra.  

Todos los contenedores creados en una base de datos con rendimiento aprovisionado deben crearse con una [clave de partición](partition-data.md). En cualquier momento, el rendimiento asignado a un contenedor de una base de datos se distribuye entre todas las particiones lógicas de dicho contenedor. Cuando hay contenedores que comparten el rendimiento aprovisionado configurado en una base de datos, el rendimiento no se puede aplicar de forma selectiva a un contenedor específico o a una partición lógica. 

Si la carga de trabajo en una partición lógica consume más que el rendimiento que se asignó a una partición lógica específica, las operaciones tendrán una velocidad limitada. Cuando se produce una limitación de velocidad, puede aumentar el rendimiento de toda la base de datos o volver a intentar las operaciones. Para más información sobre las particiones, consulte [Particiones lógicas](partition-data.md).

Los contenedores de una base de datos de rendimiento compartido comparten el rendimiento (RU/s) asignado a dicha base de datos. Puede tener hasta cuatro contenedores con un mínimo de 400 RU/s en la base de datos. Con el rendimiento aprovisionado estándar (manual), cada nuevo contenedor después de los cuatro primeros requerirá un mínimo de 100 RU/s adicionales. Por ejemplo, si tiene una base de datos de rendimiento compartido con ocho contenedores, el mínimo de RU/s en la base de datos será de 800 RU/s. Con el rendimiento aprovisionado de escalabilidad automática, puede tener hasta 25 contenedores en una base de datos con 4000 RU/s como máximo de escalabilidad automática (escalas entre 400 y 4000 RU/s).

> [!NOTE]
> En febrero de 2020, se presentó un cambio que permite tener un máximo de 25 contenedores en una base de datos de rendimiento compartida, lo que permite el uso compartido del rendimiento entre los contenedores. Después de los primeros 25 contenedores, solo puede agregar más a la base de datos si están [aprovisionados con un rendimiento dedicado](#set-throughput-on-a-database-and-a-container), que es independiente del rendimiento compartido de la base de datos.<br>
Si la cuenta de Azure Cosmos DB ya contiene una base de datos de rendimiento compartida con >= 25 contenedores, la cuenta y todas las demás cuentas de la misma suscripción de Azure están exentas de este cambio. [Póngase en contacto con el servicio de soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) si tiene comentarios o preguntas. 

Si las cargas de trabajo implican eliminar y volver a crear todas las colecciones de una base de datos, se recomienda quitar la base de datos vacía y volver a crear una nueva base de datos antes de la creación de la colección. En la siguiente imagen se muestra cómo una partición física puede hospedar una o varias particiones lógicas que pertenecen a distintos contenedores dentro de una base de datos:

:::image type="content" source="./media/set-throughput/resource-partition2.png" alt-text="Partición física" border="false":::

## <a name="set-throughput-on-a-database-and-a-container"></a>Establecimiento del rendimiento en un contenedor y una base de datos

Puede combinar los dos modelos. Se permite el aprovisionamiento del rendimiento tanto en la base de datos como en el contenedor. En el ejemplo siguiente se muestra cómo aprovisionar el rendimiento aprovisionado estándar (manual) en un contenedor y una base de datos de Azure Cosmos:

* Puede crear una base de datos de Azure Cosmos llamada *Z* con rendimiento aprovisionado estándar (manual) de *"K"*  RU. 
* Después, cree cinco contenedores llamados *A*, *B*, *C*, *D* y *E* en la base de datos. Al crear el contenedor B, asegúrese de habilitar la opción **Provision dedicated throughput for this container** (Aprovisionar el rendimiento dedicado para este contenedor) y configurar de forma explícita las Unidades de solicitud *"P"* del rendimiento aprovisionado en este contenedor. Tenga en cuenta que únicamente puede configurar el rendimiento compartido y dedicado al crear la base de datos y el contenedor. 

   :::image type="content" source="./media/set-throughput/coll-level-throughput.png" alt-text="Configuración del rendimiento en el nivel de contenedor":::

* El rendimiento de las Unidades de solicitud *"K"* se comparte entre los cuatro contenedores *A*, *C*, *D* y *E*. La cantidad exacta de rendimiento disponible en *A*, *C*, *D* o *E* varía. No hay ningún acuerdo de nivel de servicio para el rendimiento de cada contenedor individual.
* Se garantiza que el contenedor llamado *B* obtendrá el rendimiento de las Unidades de solicitud de *"P"* todo el tiempo. Estará respaldado por los Acuerdos de Nivel de Servicio.

> [!NOTE]
> No se puede convertir un contenedor con rendimiento aprovisionado en un contenedor de base de datos compartido. A la inversa, un contenedor de base de datos compartido no se puede convertir para tener un rendimiento dedicado.

## <a name="update-throughput-on-a-database-or-a-container"></a>Actualización del rendimiento en un contenedor o una base de datos

Después de crear un contenedor o una base de datos de Cosmos Azure, se puede actualizar el rendimiento aprovisionado. No existen límites para el rendimiento aprovisionado máximo que se puede configurar en la base de datos o el contenedor. 

Para estimar el [rendimiento aprovisionado mínimo](concepts-limits.md#storage-and-database-operations) de una base de datos o contenedor, encuentre el valor máximo de:

* 400 RU/s 
* Almacenamiento actual en GB * 10 RU/s
* El mayor valor de RU/s aprovisionado en la base de datos o el contenedor / 100
* Número de contenedores * 100 RU/s (solo base de datos de rendimiento compartido)

El valor mínimo real de RU/s puede variar en función de la configuración de la cuenta. Puede usar [métricas de Azure Monitor](monitor-cosmos-db.md#view-operation-level-metrics-for-azure-cosmos-db) para ver el historial de rendimiento aprovisionado (RU/s) y el almacenamiento en un recurso.

Puede recuperar el rendimiento mínimo de un contenedor o una base de datos mediante programación con los SDK o ver el valor en Azure Portal. Al usar el SDK de .NET, el método [DocumentClient.ReplaceOfferAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.client.documentclient.replaceofferasync?view=azure-dotnet) permite escalar el valor de rendimiento aprovisionado. Al usar el SDK de Java, el método [RequestOptions.setOfferThroughput](sql-api-java-sdk-samples.md) permite escalar el valor de rendimiento aprovisionado. 

Al usar el SDK de .NET, el método [DocumentClient.ReadOfferAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.client.documentclient.readofferasync?view=azure-dotnet) permite recuperar el rendimiento mínimo de un contenedor o una base de datos. 

Puede escalar el rendimiento aprovisionado de un contenedor o una base de datos en cualquier momento. Cuando se realiza una operación de escala para aumentar el rendimiento, puede tardar más tiempo debido a las tareas del sistema destinadas a aprovisionar los recursos necesarios. Puede comprobar el estado de la operación de escalado en Azure Portal o mediante programación con el SDK. Al usar el SDK de .NET, puede obtener el estado de la operación de escalado mediante el método `DocumentClient.ReadOfferAsync`.

## <a name="comparison-of-models"></a>Comparación de modelos
En esta tabla se muestra una comparación entre el aprovisionamiento del rendimiento estándar (manual) en una base de datos frente a un contenedor. 

|**Parámetro**  |**Rendimiento estándar (manual) en una base de datos**  |**Rendimiento estándar (manual) en un contenedor**|**Rendimiento de escalabilidad automática en una base de datos** | **Rendimiento de escalabilidad automática en un contenedor**|
|---------|---------|---------|---------|---------|
|Punto de entrada (mínimo de RU/s) |400 RU/s. Después de los cuatro primeros contenedores, cada contenedor adicional requiere un mínimo de 100 RU/s.</li> |400| Escalabilidad automática entre 400 y 4000 RU/s. Puede tener hasta 25 contenedores sin un mínimo de RU/s por contenedor</li> | Escalabilidad automática entre 400 y 4000 RU/s.|
|Mínimo de RU/s por contenedor|100|400|--|Escalabilidad automática entre 400 y 4000 RU/s|
|Número máximo de RU|Ilimitado, en la base de datos.|Ilimitado, en el contenedor.|Ilimitado, en la base de datos.|Ilimitado, en el contenedor.
|RU asignadas o disponibles para un contenedor específico|Sin garantías. Las RU asignadas a un contenedor determinado dependen de las propiedades. Las propiedades pueden ser la elección de las claves de partición de contenedores que comparten el rendimiento, la distribución de la carga de trabajo y el número de contenedores. |Todas las RU configuradas en el contenedor se reservan exclusivamente para el contenedor.|Sin garantías. Las RU asignadas a un contenedor determinado dependen de las propiedades. Las propiedades pueden ser la elección de las claves de partición de contenedores que comparten el rendimiento, la distribución de la carga de trabajo y el número de contenedores. |Todas las RU configuradas en el contenedor se reservan exclusivamente para el contenedor.|
|Almacenamiento máximo de un contenedor|Sin límite.|Sin límite|Sin límite|Sin límite|
|Rendimiento máximo por partición lógica de un contenedor|10 000 RU/s|10 000 RU/s|10 000 RU/s|10 000 RU/s|
|Almacenamiento máximo (datos + índice) por partición lógica de un contenedor|20 GB|20 GB|20 GB|20 GB|

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre las [particiones lógicas](partition-data.md).
* Obtenga más información sobre el [aprovisionamiento del rendimiento estándar (manual) de un contenedor de Azure Cosmos](how-to-provision-container-throughput.md).
* Obtenga más información sobre el [aprovisionamiento del rendimiento estándar (manual) en una base de datos de Azure Cosmos](how-to-provision-database-throughput.md).
* Obtenga más información sobre el [aprovisionamiento del rendimiento de escalabilidad automática en una base de datos o contenedor de Azure Cosmos](how-to-provision-autoscale-throughput.md).
