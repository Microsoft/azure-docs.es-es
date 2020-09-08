---
title: Escalado de Azure Cosmos DB según una programación mediante el temporizador de Azure Functions
description: Aprenda a escalar los cambios de rendimiento en Azure Cosmos DB con PowerShell y Azure Functions.
author: markjbrown
ms.service: cosmos-db
ms.topic: how-to
ms.date: 01/13/2020
ms.author: mjbrown
ms.openlocfilehash: ec5c98d90facf9458769f235880f17d14708e425
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87923662"
---
# <a name="scale-azure-cosmos-db-throughput-by-using-azure-functions-timer-trigger"></a>Escalado del rendimiento de Azure Cosmos DB mediante el desencadenador de temporizador de Azure Functions

El rendimiento de una cuenta de Azure Cosmos se basa en la cantidad de rendimiento aprovisionado expresado en las unidades de solicitud por segundo (RU/s). El aprovisionamiento se realiza a una granularidad de un segundo y se factura en función de los mayores RU/s por hora. Este modelo de capacidad aprovisionada permite que el servicio proporcione un rendimiento predecible y coherente, baja latencia garantizada y alta disponibilidad. La mayor parte de la producción realiza cargas de trabajo de estas características. Sin embargo, en entornos de desarrollo y pruebas en los que Azure Cosmos DB solo se usa durante las horas de trabajo, se puede escalar verticalmente el rendimiento de la mañana y reducirse verticalmente por la tarde después del horario de trabajo.

Puede establecer el rendimiento a través de las [plantillas de Azure Resource Manager](resource-manager-samples.md), [CLI de Azure](cli-samples.md) y [PowerShell](powershell-samples.md) para las cuentas de API principales (SQL), o mediante los SDK de Azure Cosmos DB específicos del lenguaje. La ventaja de usar plantillas de Resource Manager, CLI de Azure o PowerShell es que admiten todas las API de modelo de Azure Cosmos DB.

## <a name="throughput-scheduler-sample-project"></a>Proyecto de ejemplo del programador de rendimiento

Para simplificar el proceso de escalado de Azure Cosmos DB en una programación, hemos creado un proyecto de ejemplo denominado [Programador de rendimiento de Azure Cosmos](https://github.com/Azure-Samples/azure-cosmos-throughput-scheduler). Este proyecto es una aplicación de Azure Functions con dos desencadenadores de temporizador: "ScaleUpTrigger" y "ScaleDownTrigger". Los desencadenadores ejecutan un script de PowerShell que establece el rendimiento de cada recurso tal y como se define en el archivo `resources.json` de cada desencadenador. ScaleUpTrigger está configurado para ejecutarse a las 8:00 a.m. UTC y ScaleDownTrigger está configurado para ejecutarse a las 6:00 p.m. UTC. Estas horas se pueden actualizar fácilmente dentro del archivo `function.json` para cada desencadenador.

Puede clonar este proyecto localmente, modificarlo para especificar los recursos de Azure Cosmos DB para escalar y reducir verticalmente y la programación que se va a ejecutar. Posteriormente, puede implementarlo en una suscripción de Azure y protegerlo mediante la identidad de servicio administrada con permisos de [control de acceso basado en rol de Azure (Azure RBAC)](role-based-access-control.md) con el rol "operador de Azure Cosmos DB" para establecer el rendimiento en las cuentas de Azure Cosmos.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información y descargue el ejemplo del [Programador de rendimiento de Azure Cosmos DB](https://github.com/Azure-Samples/azure-cosmos-throughput-scheduler).
