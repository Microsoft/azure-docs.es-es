---
title: Presencia regional con Azure Cosmos DB
description: En este artículo se explica la presencia regional de Azure Cosmos DB y los distintos entornos en la nube.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 07/23/2019
ms.custom: seodec18
ms.openlocfilehash: e2242e8febad5d55813721266f1286250af47111
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87082892"
---
# <a name="regional-presence-with-azure-cosmos-db"></a>Presencia regional con Azure Cosmos DB

Azure Cosmos DB es un servicio fundamental en Azure y, de forma predeterminada, está disponible en [todas las regiones en las que Azure está disponible](https://azure.microsoft.com/global-infrastructure/services/?products=cosmos-db&regions=all).

:::image type="content" source="./media/regional-presence/regional-presence.png" alt-text="Regiones donde está disponible Azure Cosmos DB" lightbox="./media/regional-presence/regional-presence.png" border="false":::

Cosmos DB está disponible en los cinco entornos en la nube de Azure disponibles para los clientes:

* La nube **pública de Azure**, que está disponible globalmente.

* **Azure China 21Vianet** está disponible gracias a una exclusiva asociación entre Microsoft y 21Vianet, uno de los proveedores de acceso a Internet más importantes del país en China.

* **Azure Alemania** proporciona servicios bajo un modelo de administrador de datos que garantiza que los datos de los clientes permanecen en Alemania bajo el control de T-Systems International GmbH, una subsidiaria de Deutsche Telekom, que actúa como administrador de datos para Alemania.

* **Azure Government** está disponible en cuatro regiones en los Estados Unidos para agencias gubernamentales de EE. UU. y sus asociados. 

* **Azure Government para el Departamento de Defensa (DoD)** está disponible en dos regiones de Estados Unidos para el Departamento de Defensa de EE. UU.

## <a name="regional-presence-with-global-distribution"></a>Presencia regional con distribución global

En todas las regiones de Azure están disponibles de forma predeterminada todas las API expuestas por Azure Cosmos DB (incluidas SQL, MongoDB, Cassandra, Gremlin y Table). Por ejemplo, puede tener MongoDB y Cassandra API expuestas por Azure Cosmos DB no solo en todas las regiones de Azure globales, sino también en las nubes soberanas, como China, Alemania, Government y del Departamento de Defensa (DoD).

Azure Cosmos DB es un servicio de base de datos [distribuido globalmente](distribute-data-globally.md). Puede asociar cualquier número de regiones de Azure con su cuenta de Azure Cosmos y los datos se replican de forma automática y transparente. Puede agregar o quitar una región a la cuenta de Azure Cosmos en cualquier momento. Con la funcionalidad de distribución global de llave en mano y el protocolo de replicación con arquitectura multimaestro, Azure Cosmos DB ofrece latencias de lectura y escritura de menos de 10 ms en el percentil 99, disponibilidad de lectura y escritura del 99,999 % y la capacidad de escalar de forma flexible el rendimiento aprovisionado para lectura y escritura en todas las regiones asociadas con su cuenta de Azure Cosmos. Azure Cosmos DB también ofrece cinco modelos de coherencia bien definidos y se puede aplicar un modelo específico a sus datos. Por último, Azure Cosmos DB es el único servicio de base de datos en el sector que proporciona un [Acuerdo de Nivel de Servicio (SLA)](https://azure.microsoft.com/support/legal/sla/cosmos-db/v1_2/) completo que abarca rendimiento aprovisionado, latencia en el percentil 99, alta disponibilidad y coherencia. Las funcionalidades anteriores están disponibles en todas las nubes de Azure.

La compatibilidad con la replicación global se limita a un entorno de nube de Azure. Por ejemplo, las cuentas de Azure Cosmos DB en *Azure público* se pueden replicar en cualquier región de Azure en *Azure público*, pero no se pueden replicar en regiones de Azure en *Azure China 21Vianet*.

## <a name="next-steps"></a>Pasos siguientes

Ahora puede obtener información acerca de los conceptos básicos de Azure Cosmos DB con los siguientes artículos:

* [Aplicaciones distribuidas globalmente](distribute-data-globally.md)
* [Administración de cuentas de base de datos en Azure Cosmos DB](manage-account.md)
* [Aprovisionamiento del rendimiento de contenedores y bases de datos de Cosmos DB](set-throughput.md)
* [Acuerdo de Nivel de Servicio de Azure Cosmos DB](https://azure.microsoft.com/support/legal/sla/cosmos-db/v1_2/)
