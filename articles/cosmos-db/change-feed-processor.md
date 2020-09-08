---
title: Procesadores de fuente de cambios de Azure Cosmos DB
description: Aprenda a usar el procesador de fuente de cambios de Azure Cosmos DB para leer la fuente de cambios, y los componentes del procesador de fuente de cambios.
author: timsander1
ms.author: tisande
ms.service: cosmos-db
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 05/13/2020
ms.reviewer: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: 3a802cc3d6178302445e0c31c52785d00207d0bd
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88998550"
---
# <a name="change-feed-processor-in-azure-cosmos-db"></a>Procesadores de fuente de cambios de Azure Cosmos DB

El procesador de fuente de cambios es parte del [SDK V3 de Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v3). Simplifica el proceso de lectura de la fuente de cambios y distribuye el procesamiento de eventos entre varios consumidores de manera eficaz.

La principal ventaja de la biblioteca de procesador de fuente de cambios es su comportamiento tolerante a errores que garantiza una entrega "al menos una vez" de todos los eventos de la fuente de cambios.

## <a name="components-of-the-change-feed-processor"></a>Componentes del procesador de fuente de cambios

Hay cuatro componentes principales en la implementación del procesador de fuente de cambios:

1. **El contenedor supervisado:** el contenedor supervisado tiene los datos a partir de los cuales se genera la fuente de cambios. Todas las inserciones y actualizaciones realizadas en el contenedor supervisado se reflejan en la fuente de cambios del contenedor.

1. **El contenedor de concesión**: El contenedor de concesión actúa como un almacenamiento de estado y coordina el procesamiento de la fuente de cambios entre varios trabajadores. El contenedor de concesión se puede almacenar en la misma cuenta que el contenedor supervisado o en una cuenta independiente.

1. **El host:** Un host es una instancia de aplicación que usa el procesador de fuente de cambios para escuchar los cambios. Se pueden ejecutar varias instancias con la misma configuración de concesión en paralelo, pero cada instancia debe tener **un nombre de instancia** diferente.

1. **El delegado:** El delegado es el código que define lo que usted, el desarrollador, desea hacer con cada lote de cambios que el procesador de la fuente de cambios lea. 

Para comprender mejor cómo funcionan estos cuatro elementos del procesador de fuente de cambios juntos, echemos un vistazo a un ejemplo en el diagrama siguiente. El contenedor supervisado almacena documentos y usa "City" como clave de partición. Vemos que los valores de la clave de partición se distribuyen en intervalos que contienen elementos. Hay dos instancias de host y el procesador de fuente de cambios está asignando distintos intervalos de valores de clave de partición a cada instancia para maximizar la distribución de proceso. Cada intervalo se lee en paralelo y su progreso se mantiene por separado de otros intervalos en el contenedor de concesión.

:::image type="content" source="./media/change-feed-processor/changefeedprocessor.png" alt-text="Ejemplo de procesador de fuente de cambios" border="false":::

## <a name="implementing-the-change-feed-processor"></a>Implementación del procesador de fuente de cambios

El punto de entrada es siempre el contenedor supervisado, desde una instancia de `Container` se llama a `GetChangeFeedProcessorBuilder`:

[!code-csharp[Main](~/samples-cosmosdb-dotnet-change-feed-processor/src/Program.cs?name=DefineProcessor)]

El primer parámetro es un nombre distintivo que describe el objetivo de este procesador y el segundo nombre es la implementación de delegado que controlará los cambios. 

Un ejemplo de un delegado sería:


[!code-csharp[Main](~/samples-cosmosdb-dotnet-change-feed-processor/src/Program.cs?name=Delegate)]

Por último, se define un nombre para esta instancia del procesador con `WithInstanceName`, que es el contenedor con el que se mantiene el estado de concesión con `WithLeaseContainer`.

La llamada a `Build` le proporcionará la instancia del procesador que puede iniciar mediante una llamada a `StartAsync`.

## <a name="processing-life-cycle"></a>Ciclo de vida de procesamiento

El ciclo de vida normal de una instancia de host es:

1. Leer la fuente de cambios.
1. Si no hay ningún cambio, mantenerse suspendida durante un periodo de tiempo predefinido (personalizable con `WithPollInterval` en el generador) e ir a #1.
1. Si hay cambios, enviarlos al **delegado.**
1. Cuando el delegado termina de procesar los cambios **correctamente**, actualizar el almacén de concesión con el último punto en el tiempo e ir al nº 1.

## <a name="error-handling"></a>Control de errores

El procesador de fuente de cambios es resistente a los errores de código de usuario. Esto significa que si la implementación del delegado tiene una excepción no controlada (paso nº 4), el subproceso que está procesando ese lote específico de cambios se detendrá y se creará un nuevo subproceso. El nuevo subproceso comprobará cuál fue el último punto en el tiempo que el almacén de concesiones tiene para ese intervalo de valores de clave de partición, y se reiniciará desde allí, enviando de forma eficaz el mismo lote de cambios al delegado. Este comportamiento continúa hasta que el delegado procesa los cambios correctamente, y es el motivo por el que el procesador de fuente de cambios tiene una garantía de "al menos una vez", ya que, si se produce una excepción en el código del delegado, ese lote se vuelve a intentar.

Para evitar que el procesador de fuente de cambios se "atasque" al reintentar continuamente el mismo lote de cambios, debe agregar lógica al código del delegado para escribir documentos, en caso de excepción, en una cola de mensajes fallidos. Este diseño garantiza que se pueda realizar un seguimiento de los cambios sin procesar a la vez que se siguen procesando cambios futuros. La cola de mensajes fallidos podría ser simplemente otro contenedor de Cosmos. El almacén de datos exacto no importa, simplemente que se conserven los cambios sin procesar.

Además, puede usar el [estimador de fuente de cambios](how-to-use-change-feed-estimator.md) para supervisar el progreso de las instancias del procesador de fuente de cambios a medida que leen la fuente de cambios. Además de supervisar si el procesador de fuente de cambios se "atasca" al reintentar continuamente el mismo lote de cambios, también puede entender si el procesador de fuente de cambios va retrasado debido a recursos disponibles como la CPU, la memoria y el ancho de banda de red.

## <a name="deployment-unit"></a>Unidad de implementación

Una sola unidad de implementación de procesador de fuente de cambios está formada por una o varias instancias con el mismo `processorName` y la misma configuración de contenedor de concesión. Puede tener muchas unidades de implementación donde cada una tiene un flujo de negocio diferente para los cambios y cada unidad de implementación que consta de una o varias instancias. 

Por ejemplo, podría tener una unidad de implementación que desencadene una API externa en cualquier momento en que se produzca un cambio en el contenedor. Otra unidad de implementación podría trasladar los datos en tiempo real, cada vez que se produzca un cambio. Cuando se produce un cambio en el contenedor supervisado, todas las unidades de implementación recibirán una notificación.

## <a name="dynamic-scaling"></a>Escalado dinámico

Como se mencionó antes, en una unidad de implementación puede tener una o más instancias. Para beneficiarse de la distribución de proceso dentro de la unidad de implementación, los únicos requisitos clave son los siguientes:

1. Todas las instancias deben tener la misma configuración de contenedor de concesión.
1. Todas las instancias deben tener el mismo `processorName`.
1. Cada instancia tiene que tener un nombre de instancia diferente (`WithInstanceName`).

Si estas tres condiciones son aplicables, el procesador de fuente de cambios, mediante el uso de un algoritmo de distribución equitativa, distribuye todas las concesiones en el contenedor de concesión en todas las instancias en ejecución de esa unidad de implementación y paraleliza el proceso. Una concesión solo puede ser propiedad de una instancia en un momento dado, por lo que el número máximo de instancias es igual al número de concesiones.

El número de instancias pueden aumentar y reducir, y el procesador de fuente de cambios ajustará dinámicamente la carga redistribuyéndola en consecuencia.

Además, el procesador de fuente de cambios puede ajustarse de forma dinámica a la escala de los contenedores debido a un aumento en el rendimiento o almacenamiento. Cuando el contenedor crece, el procesador de fuente de cambios controla de forma transparente estos escenarios al aumentar dinámicamente las concesiones y distribuir las nuevas concesiones entre las instancias existentes.

## <a name="change-feed-and-provisioned-throughput"></a>Fuente de cambios y rendimiento aprovisionado

Se le cobrarán las RU consumidas, puesto que la entrada y la salida de datos de contenedores de Cosmos siempre consumen RU. Se le cobrarán las RU consumidas por el contenedor de concesión.

## <a name="where-to-host-the-change-feed-processor"></a>Dónde hospedar el procesador de fuente de cambios

El procesador de fuente de cambios se puede hospedar en cualquier plataforma que admita procesos o tareas de larga duración:

* Un [WebJob de Azure](https://docs.microsoft.com/learn/modules/run-web-app-background-task-with-webjobs/) que se ejecuta continuamente.
* Un proceso de una [máquina virtual de Azure](https://docs.microsoft.com/azure/architecture/best-practices/background-jobs#azure-virtual-machines).
* Un trabajo en segundo plano de [Azure Kubernetes Service](https://docs.microsoft.com/azure/architecture/best-practices/background-jobs#azure-kubernetes-service).
* Un [servicio hospedado de ASP.NET](https://docs.microsoft.com/aspnet/core/fundamentals/host/hosted-services).

Aunque el procesador de fuente de cambios puede ejecutarse en entornos de corta duración, ya que el contenedor de concesión mantiene el estado, el ciclo de inicio y detención de estos entornos agregará retraso a la recepción de las notificaciones (debido a la sobrecarga que supone iniciar el procesador cada vez que se inicie el entorno).

## <a name="additional-resources"></a>Recursos adicionales

* [SDK de Azure Cosmos DB](sql-api-sdk-dotnet.md)
* [Aplicación de ejemplo completa en GitHub](https://github.com/Azure-Samples/cosmos-dotnet-change-feed-processor)
* [Ejemplos de uso adicionales en GitHub](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed)
* [Laboratorios de taller de Cosmos DB para el procesador de fuente de cambios](https://azurecosmosdb.github.io/labs/dotnet/labs/08-change_feed_with_azure_functions.html#consume-cosmos-db-change-feed-via-the-change-feed-processor)

## <a name="next-steps"></a>Pasos siguientes

Puede obtener más información sobre el procesador de la fuente de cambios en los siguientes artículos:

* [Introducción a la fuente de cambios](change-feed.md)
* [Modelo de extracción de fuente de cambios](change-feed-pull-model.md)
* [Migración desde la biblioteca de procesadores de fuente de cambios](how-to-migrate-from-change-feed-library.md)
* [Uso del calculador de la fuente de cambios](how-to-use-change-feed-estimator.md)
* [Hora de inicio del procesador de la fuente de cambios](how-to-configure-change-feed-start-time.md)
